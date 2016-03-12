+++
author = "ricardopaiva"
comments = true
date = "2016-03-12T1:30:34-03:00"
description = ""
draft = false
image = "images/posts/testes-spark-junit/header.jpg"
menu = ""
share = true
slug = "testes-spark-junit"
tags = ["spark", "test", "junit", "gradle"]
title = "Testes de jobs Spark com JUnit"

+++

A boa prática de desenvolvimento de software diz que devemos criar sempre testes para nossos códigos, e no universo de Big Data não deveria ser diferente. Neste artigo apresento como testar um código Spark com JUnit para jobs que rodam em batch (não-streaming).

<!--more-->

Os códigos deste artigo estão disponíveis em https://github.com/werneckpaiva/spark-junit

## Build e gerenciamento de dependências

O script abaixo define a configuração do build usando a ferramenta Gradle. Com ele baixamos as dependências do Spark para a nossa máquina e configuramos os testes

{{< source javascript >}}
apply plugin: 'scala'
sourceCompatibility = 1.7

repositories {
    mavenCentral()
}

dependencies {
    compile 'org.slf4j:slf4j-api:1.7.12'
    compile 'org.slf4j:slf4j-log4j12:1.7.12'
    compile 'org.scala-lang:scala-library:2.10.6'
    compile('org.apache.hadoop:hadoop-client:2.7.1') {
        exclude group: 'javax.servlet'
        exclude group: 'javax.servlet.jsp'
        exclude group: 'org.mortbay.jetty'
    }
    compile('org.apache.spark:spark-core_2.10:1.6.1') {
        exclude group: 'org.apache.hadoop'
    }
    compile 'org.apache.spark:spark-sql_2.10:1.6.1'
    compile 'org.apache.spark:spark-streaming_2.10:1.6.1'
    testCompile 'junit:junit:4.11'
}

test {
    testLogging {
        events "passed", "skipped", "failed"
        exceptionFormat "full"
    }
}
{{< /source >}}

## Criando a classe de testes

Ao criar um teste de um job Spark você precisa iniciar o SparkContext local antes de todos os testes e encerrá-lo no final. O ponto chave desta configuração é definir o master do Spark para `local`, desta forma o Spark irá rodar localmente, sem depender de nenhum gerenciador de recursos (como o Yarn).


Primeiro crie um object que inicia e termina o SparkContext no início dos testes:

{{< source python >}}
object TestMyCode {
  var sc: SparkContext = _

  @BeforeClass
  def before(): Unit = {
    val sparkConf = new SparkConf()
      .setAppName("Test Spark Batch")
      .setMaster("local")
    sc = new SparkContext(sparkConf)
  }

  @AfterClass
  def after(): Unit = {
    sc.stop()
  }

}
{{< /source >}}

Crie a classe com os seus testes. Inicie a classe permitindo que o SparkContext esteja disponível, e crie um contexto SQLContext baseado no SparkContext para poder fazer uso de recursos de DataFrame:

{{< source python >}}
class TestMyCode {
  import TestMyCode._
  val sql = new SQLContext(sc)
  import sql.implicits._

  // ...
}
{{< /source >}}

O código de seus testes devem começar criando um RDD utilizando o método `parallellize()` do SparkContext. Se o seu código é baseado em DataFrames, você pode usar o método `toDF`, disponível quando você importa o implict do SqlContext.

{{< source python >}}
@Test
def testCountPairNumbers(): Unit = {
  val data = List(1, 2, 3, 4, 5, 6)
  val df =  sc.parallelize(data).toDF

  # Código que está querendo testar
  val count = MyOperations.countPairs(df)

  assertEquals(3, count)
}
{{< /source >}}

## Rodando os testes

Para rodar os teste, você pode usar seu IDE (no Eclipse, clique com botão direito no arquivo do teste, selecione Run as > Scala JUnit Test). Podemos usar o Gradle para rodar todos os testes pela linha de comando:

{{< source sh >}}
gradle clean test
{{< /source >}}

Pronto! Você já pode criar e rodar testes para seu código Spark. Espero que tenha ajudado. Em breve escreverei um post ensinando como testar um código de Spark Streaming.
