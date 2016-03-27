+++
author = "ricardopaiva"
comments = true
date = "2016-03-26T22:33:20-03:00"
description = ""
draft = false
image = "images/posts/testes-spark-junit/header.jpg"
menu = ""
share = true
slug = "testes-spark-streaming-junit"
tags = ["spark streaming", "test", "junit"]
title = "Testes de Spark Streaming com JUnit"

+++

Dando continuidade ao artigo sobre <a href="http://grandesdados.com/post/testes-spark-junit/">testes no Spark</a>, apresento agora como fazer testes em jobs streaming. O maior desafio ao criar estes testes é fazer com que eles não dependam do tempo real para executar, pois alguns jobs podem aguardar muitos minutos para gerar um processamento.

<!--more-->

Os jobs Spark do tipo streaming obtém dados de alguma fonte com fluxo contínuo de dados, como um barramento mensagens como o Kafka, logs etc. Os jobs são configurados com um intervalo de tempo e processam todas as mensagens que chegaram dentro deste período.

A estratégia que adotaremos será criar um relógio virtual, onde poderemos avançar mais rapidamente e simular o relógio real. Desta forma, nossos testes não precisarão esperar o tempo total do streaming para terminarem. Também criaremos uma fila para simular a sequência dos dados. Veja as classes que precisam ser criadas para que tudo isso funcione.

### Relógio virtual

A primeira classe a ser criada define um relógio virtual e sobrescreve a classe original do Spark:

**org.apache.spark.ClockWrapper**
{{< source python >}}
package org.apache.spark

import org.apache.spark.streaming.{StreamingContext, StreamingContextWrapper}

class ClockWrapper(ssc: StreamingContext) {

  private val manualClock = new StreamingContextWrapper(ssc).manualClock
  def getTimeMillis: Long = manualClock.getTimeMillis()
  def setTime(timeToSet: Long) = manualClock.setTime(timeToSet)
  def advance(timeToAdd: Long) = manualClock.advance(timeToAdd)
  def waitTillTime(targetTime: Long): Long = manualClock.waitTillTime(targetTime)

}
{{< /source >}}

A segunda classe também sobrescreve o comportamento natural do Spark para usar um relógio manual:

**org.apache.spark.streaming.StreamingContextWrapper**
{{< source python >}}
package org.apache.spark.streaming

import org.apache.spark.util.ManualClock

class StreamingContextWrapper(ssc: StreamingContext) {
  val manualClock = ssc.scheduler.clock.asInstanceOf[ManualClock]
}
{{< /source >}}

Não se esqueça de criar os pacotes corretos!

### Test DStream

A próxima classe cria um DStream para os testes baseado em uma fila em memória. Você pode criar esta classe no pacote que for melhor para você.

**werneckpaiva.spark.test.util.TestInputDStream**

{{< source python >}}
package werneckpaiva.spark.test.util

import scala.collection.mutable.ArrayBuffer
import scala.collection.mutable.Queue
import scala.reflect.ClassTag

import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.StreamingContext
import org.apache.spark.streaming.Time
import org.apache.spark.streaming.dstream.InputDStream


class TestInputDStream[T:ClassTag](
    ssc: StreamingContext, queue: Queue[RDD[T]], defaultRDD: RDD[T])
      extends InputDStream[T](ssc) {

  def start() {}
  def stop() {}
  def compute(validTime: Time): Option[RDD[T]] = {
    val buffer = new ArrayBuffer[RDD[T]]()
    if (!queue.isEmpty) {
      Some(queue.dequeue())
    } else {
      Some(defaultRDD)
    }
  }
}
{{< /source >}}

### Classe Teste Base

A última classe a ser criada é uma classe utilitária para simplificar o código de nossos testes. Ela inicia o contexto Spark Streaming e declara um método que sincroniza nosso código até que os dados estejam prontos para que o streaming seja processado. Neste caso estamos sempre iniciando o streaming de teste com duração de 5 segundos. Você pode definir o período que achar melhor e colocar no pacote que achar apropriado.

**werneckpaiva.spark.test.BaseTestSparkStreaming**

{{< source python >}}
package werneckpaiva.spark.test

import org.apache.spark.streaming.StreamingContext
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.junit.After
import org.junit.Before
import org.apache.spark.ClockWrapper
import org.apache.spark.streaming.Seconds
import java.nio.file.Files
import scala.reflect.ClassTag
import werneckpaiva.spark.test.util.TestInputDStream
import scala.collection.mutable.ListBuffer
import scala.collection.mutable.Queue
import org.apache.spark.streaming.dstream.DStream
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.Duration

class BaseTestSparkStreaming {
  var ssc:StreamingContext = _
  var sc:SparkContext = _
  var clock:ClockWrapper = _

  @Before
  def before():Unit = {
    val sparkConf = new SparkConf()
      .setAppName("Test Spark Streaming")
      .setMaster("local[*]")
      .set("spark.streaming.clock", "org.apache.spark.streaming.util.ManualClock")

    val checkpointDir = Files.createTempDirectory("test").toString
    ssc = new StreamingContext(sparkConf, Seconds(5))
    ssc.checkpoint(checkpointDir)
    sc = ssc.sparkContext
    clock = new ClockWrapper(ssc)
  }

  @After
  def after():Unit = {
    ssc.stop()
    sc.stop()
  }

  def makeStream[T:ClassTag]():(Queue[RDD[T]], TestInputDStream[T]) = {
    val lines = new Queue[RDD[T]]()
    val stream = new TestInputDStream[T](ssc, lines, sc.makeRDD(Seq[T](), 1))
    (lines, stream)
  }

  def waitForResultReady[T](stream:DStream[T], time:Duration):ListBuffer[Array[T]] = {
    val results = ListBuffer.empty[Array[T]]
    stream.foreachRDD((rdd, time) => {
      results.append(rdd.collect())
    })
    ssc.start()
    clock.advance(time.milliseconds)
    for(i <- 1 to 100){
      if(results.length >= 1) return results
      Thread.sleep(100)
    }
    throw new Exception("Can't load stream")
  }
}
{{< /source >}}

## Criando seus testes

Depois de criar essas classes, já temos todas as ferramentas para construir nossos testes. Cada teste inicializa o DStream de teste e uma fila em memória. Executa o código a ser testado, e popula a fila com os dados de teste. Para podermos executar os asserts, precisamos antes executar o método que avança o relógio virtual.

{{< source python >}}
class TestSparkStreaming extends BaseTestSparkStreaming{

  @Test
  def testSimpleSum():Unit = {
    val (lines, stream) = makeStream[(String, Long)]()

    // Código que você quer testar
    val reducedStream = StreamOperations.streamSum(stream)

    lines += sc.makeRDD(Seq(("a", 1L), ("a", 1L), ("b", 1L)))
    lines += sc.makeRDD(Seq(("a", 1L), ("a", 1L)))
    lines += sc.makeRDD(Seq(("a", 1L), ("a", 2L), ("b", 3L), ("b", 2L)))

    val results = waitForResultReady(reducedStream, Seconds(20))

    assertEquals(("a", 2), results(0)(0))
    assertEquals(("b", 1), results(0)(1))
  }
}
{{< /source >}}

No exemplo acima criamos um stream com 3 períodos representados por 3 sequência de dados. Nosso assert só está testando o primeiro período. O código testado não faz nada mais do que agrupar os valores por chave e somar os valores.

Eu penei um pouco para chegar até essas classes e iniciar meus testes, mas agora é bem simples criá-los e isso aumentou bem a qualidade do código que produzo.

O código desses exemplo está neste projeto do Github <a href="https://github.com/werneckpaiva/spark-junit">https://github.com/werneckpaiva/spark-junit</a>

Espero ter ajudado.

Post original em http://blog.werneckpaiva.com.br/2016/03/testes-de-spark-streaming-com-junit/
