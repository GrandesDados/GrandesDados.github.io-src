+++
author = "Ricardo Paiva"
comments = true
date = "2015-07-20T16:11:34-03:00"
draft = false
image = ""
menu = ""
share = true
slug = "como-o-spark-usa-memoria-para-obter-performance-superior-ao-mapreduce"
tags = ["spark", "apache", "mapreduce", "hadoop"]
title = "Como Spark usa a memória para obter performance superior ao MapReduce"

+++

Muitas Aplicações de Big Data executam múltiplas operações paralelas sobre o mesmo conjunto de dados. No tradicional modelo MapReduce, estes algoritmos exigem o encadeamento múltiplas operações de map e reduce o que torna o processo lento e dispendioso.

> O Spark é um framework de processamento paralelo que que visa atender
> aplicações que se beneficiam do reuso de um conjunto de dados,
> mantendo a escalabilidade e tolerança a falhas encontradas no modelo
> MapReduce.

RDD (Resilient Distributed Dataset)
-----------------------------------

O ponto central do Spark, e razão de sua eficiência, é sua estrutura de dados distribuída, o RDD (Resilient Distributed Dataset). Esta é uma abstração de dados tolerante a falha e paralela. O princípio básido do RDD é que cada estrutura contém informação suficiente para computar todas as transformações dos dados a partir do passo anterior. Ou seja, se um nó falha, a computação referente à partição RDD naquele nó pode ser refeita desde o ponto onde os dados estão acessíveis, seja uma partição de um arquivo no cluster, ou dados em memória de outro nó.

RDDs são criados a partir de operações determinísticas sobre alguma unidade persistente de armazenamento ou outros RDDs. Os elementos de um RDD não precisam ser materializados em memória a cada transformação. As partições vão sendo transformadas sob demanda e descartadas depois de usadas, a menos que o usuário explicitamente solicite que o dado seja persistido. Por esta razão, os RDDs são considerados lazy e transitórios. Veja o exemplo:

    val file = spark.textFile("hdfs://...")
    val errs = file.filter(_.contains("ERROR"))
    val ones = errs.map(_ => 1)
    val count = ones.reduce(_+_)

Linha 1: um RDD é criado para ler um arquivo em disco;
Linha 2: outro RDD é criado e filtra linhas que contenham a palavra "ERROR";
Linha 3: para cada linha com a palavra "ERROR", uma operação de map associa ao valor 1 em outro RDD;
Linha 4: Uma operação de reduce, soma todos os valores e retorna para o programa driver.

![Encadeamento de dependências dos RDDs](/images/posts/como_funciona_o_spark/spark_rdd.svg)

RDD versus Memória Compartilhada
--------------------------------

Os RDDs podem ser comparados com sistemas de memória compartilhada. A principal diferença entre os dois modelos é que RDDs só podem ser criados a partir de um encadeamento de transformações nos dados, enquanto o modelo de memória compartilhada permite ler e escrever em qualquer localização do espaço de endereçamento global.

Muitos sistemas de memória compartilhada ofercem tolerância a falha a partir de checkpoints. Este tipo de abordagem exige que todas as partições sejam reconstruídas a partir de um ponto de recuperação e toda a computação seja refeita desde aquele ponto. Também adicionam um custo extra para o armazenamento dos estados. O RDD, apesar de tornar o modelo de programação mais restrito, permite reconstrução dos dados de maneira eficiente, em caso de falha de um nó.

A natureza imutável do RDD também permite que múltiplas cópias de um mesmo processamento sejam executadas em paralelo (tarefas backup), evitando que nós lentos engargalem todo o sistema, da mesma maneira que acontece no modelo MapReduce. Tarefas backup são difíceis de serem implementadas no modelo de memória compartilhada, já que duas cópias de uma mesma tarefa podem escrever na mesma área da memória global e interferir uma na outra.

Quando não há espaço suficiente para armazenar os dados em memória, os RDDs têm performance similar aos sistemas paralelos atuais, como o MapReduce, já que irão armazenar os dados em disco.

Processamento Paralelo
----------------------

O Spark foi construído a partir do sistema Mesos, uma espécie de sistema operacional de cluster, que cria uma abstração para que aplicações Spark possam usufruir de um cluster Hadoop. O Spark é escrito em Scala e os usuários podem escrever seu código em Scala, Java ou Python.

Uma aplicação Spark é composta de apenas um código principal, chamado de driver, que se conecta aos workers do cluster. O driver define um ou mais RDDs e realiza operações sobre eles.
![Arquitetura de tarefas Spark](/images/posts/como_funciona_o_spark/spark_workers.png)

As operações sobre os RDDs podem ser de dois tipos:

 - Transformações: cria um novo conjunto de dados a partir de uma
   operação em um dataset já existente;
 - Ações: retorna um valor para o programa driver depois de
   computá-lo a partir de um conjunto de dados.

Todas as transformações no Spark são lazy, ou seja, eles não são computados imediatamente. O processamento só irá acontecer quando uma ação for executada e o resultado precisar ser retornado para o programa driver.

Algumas operações de transformação:

**map**: retorna um novo RDD formado pelo resultado da função parâmetro para cada valor do conjunto.<br/>
**filter**: filtra cada elemento do dataset a partir de uma função parâmetro.<br/>
**sample**: retorna uma amostra a partir do dataset.<br/>
**union**: faz a união de 2 RDDs.<br/>
**intersection**: retorna os elementos que pertencem a interseção entre 2 RDDs.<br/>
**distinct**: elimina elementos duplicados do conjunto.<br/>
**reduceByKey**: executa uma operação de reduce para cada valor que compartilha a mesma chave.<br/>
**cartesian**: retorna o produto cartesiano entre 2 datasets.<br/>

Algumas operações de ação:
**reduce**: agrega os elementos do dataset usando uma função parâmetro que retornar um valor único.<br/>
**collect**: retorna todos os elementos do RDD como um array.
**count**: retorna o números de elementos do dataset.<br/>
**take**: retorna os n primeiro elementos do conjunto como um array.<br/>
**saveAsTextFile**: salva o dataset em disco, como um arquivo em disco.<br/>
**foreach**: executa uma função para cada elemento do dataset, em geral usado com variáveis acumuladoras<br/>

Internamente, cada objeto RDD implementa uma interface simples, que consistem em 3 operações:

`getPartitions()` - retorna uma lista de IDs de partições.<br/>
`getIterator(parition)` - retorna um iterator de partições.<br/>
`getPreferredLocations(partition)` - usado para o escalonador identificar a localização dos dados.

Quando uma operação paralela é chamada sobre um dataset, o Spark cria tarefas para processar cada partição do dataset e envia essas tarefas para os nós workers. A localização de cada tarefa vai ser influenciada pelo resultado do método getPreferredLocations() do RDD. Uma vez iniciada no worker, cada tarefa irá chamar o método getIterator para começar a ler os dados da partição.

RDDs podem ser criados a partir de 4 fontes:

 - Arquivos em disco em um sistema HDFS;
 - Paralelizando uma coleção, por exemplo um array
 - Transformando um RDD existente, usando uma operação de flatMap (filter, map)
 - Mudando a persistência de um RDD existente.
  - É possível manter o RDD em memória, depois da primeira vez que ele foi computado; Se não houver memória suficiente, o Spark irá salvar os dados em disco.
  - Salvando o RDD no sistema de arquivos, como o
   HDFS.

Variáveis compartilhadas
------------------------

As operações são informadas por meio de closures. A linguagem Scala representa cada closure como um objeto Java, que pode ser serializado e enviado pela rede. Para manter as mesmas variáveis acessíveis nas closures, o Scala empacota as variáveis no objeto java que representa a closure. Existem alguns tipos especiais de variáveis para lidar com dados compartilhados entre múltiplos workers:

**Variáveis de broadcast**

Se um dado que não será atualizado (somente leitura) é compartilhado por múltiplas operações paralelas, é preferível distribuí-lo pelos workers, ao invés de empacotá-lo a cada vez que uma closure irá rodar. As variáveis de broadcast, portanto, são criadas uma única vez e propagadas para consulta.

**Variáveis acumuladoras**

As variáveis acumuladoras permitem aos workers “adicionar” dados usando uma função associativa que apenas o programa driver será capaz de ler. Este tipo de variável é bastante útil na criação de contadores ou somas globais.

* * *

O Spark tem se mostrado uma ótima alternativa para processamento de Big Data sobretudo em algoritmos que iteram múltiplas vezes sobre o mesmo conjunto de dados. Sua performance superior ao MapReduce tem sido comprovada por diversos benchmarks e experimentos.

O Spark também tem chamado atenção pela sua forma elegante e simplificada de desenvolvimento. Disponível em 3 linguagens (Scala, Java e Python), o uso de closures (disponível no Java 8) enxuga bastante o código. Spark provê uma rica biblioteca de operadores, que executam tarefas em paralelo, sem as necessidades especiais que existiam no modelo MapReduce.

O projeto Spark tem um desenvolvimento bastante intenso e um grande número de empresas investindo em seu crescimento. Vale a pena experimentar!
