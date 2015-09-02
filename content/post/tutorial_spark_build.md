+++
author = "cirocavani"
comments = true
date = "2015-09-02T10:41:08-03:00"
draft = true
image = ""
menu = ""
share = true
slug = "compilacao-do-spark"
tags = ["Tutorial", "Spark"]
title = "Compilação do Spark"

+++

Esse tutorial é sobre a construção do pacote do Spark 1.5.0.

## Compilação

**Java**

(Linux)

{{< source sh >}}
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/7u80-b15/jdk-7u80-linux-x64.tar.gz

tar zxf jdk-7u80-linux-x64.tar.gz

export JAVA_HOME=`pwd`/jdk1.7.0_80
export PATH=$JAVA_HOME/bin:$PATH

java -version

> java version "1.7.0_80"
> Java(TM) SE Runtime Environment (build 1.7.0_80-b15)
> Java HotSpot(TM) 64-Bit Server VM (build 24.80-b11, mixed mode)

{{< /source >}}

(OSX)

Necessário instalar o dmg do Java7.

{{< source sh >}}
export JAVA_HOME="$(/usr/libexec/java_home -v 1.7)"
{{< /source >}}

**Maven**

{{< source sh >}}
wget http://archive.apache.org/dist/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz

tar zxf apache-maven-3.3.3-bin.tar.gz

export PATH=`pwd`/apache-maven-3.3.3/bin

mvn -version

> Apache Maven 3.3.3 (7994120775791599e205a5524ec3e0dfe41d4a06; 2015-04-22T08:57:37-03:00)
> Maven home: /home/cavani/Software/apache-maven-3.3.3
> Java version: 1.7.0_80, vendor: Oracle Corporation
> Java home: /home/cavani/Software/jdk1.7.0_80/jre
> Default locale: en_US, platform encoding: UTF-8
> OS name: "linux", version: "4.0.4-2-arch", arch: "amd64", family: "unix"

{{< /source >}}

**Spark**

{{< source sh >}}
git clone https://github.com/apache/spark.git --branch v1.5.0-rc3 --depth 1 spark-1.5

cd spark-1.5

./make-distribution.sh --tgz --skip-java-test -Phadoop-2.6 -Pyarn -Phive -Phive-thriftserver -Dhadoop.version=2.7.1

> (...)
> [INFO] Reactor Summary:
> [INFO]
> [INFO] Spark Project Parent POM ........................... SUCCESS [ 24.122 s]
> [INFO] Spark Project Launcher ............................. SUCCESS [ 18.627 s]
> [INFO] Spark Project Networking ........................... SUCCESS [ 13.282 s]
> [INFO] Spark Project Shuffle Streaming Service ............ SUCCESS [  7.895 s]
> [INFO] Spark Project Unsafe ............................... SUCCESS [  8.561 s]
> [INFO] Spark Project Core ................................. SUCCESS [03:04 min]
> [INFO] Spark Project Bagel ................................ SUCCESS [  7.892 s]
> [INFO] Spark Project GraphX ............................... SUCCESS [ 20.191 s]
> [INFO] Spark Project Streaming ............................ SUCCESS [ 50.711 s]
> [INFO] Spark Project Catalyst ............................. SUCCESS [01:11 min]
> [INFO] Spark Project SQL .................................. SUCCESS [01:23 min]
> [INFO] Spark Project ML Library ........................... SUCCESS [01:37 min]
> [INFO] Spark Project Tools ................................ SUCCESS [  3.307 s]
> [INFO] Spark Project Hive ................................. SUCCESS [01:27 min]
> [INFO] Spark Project REPL ................................. SUCCESS [ 11.907 s]
> [INFO] Spark Project YARN ................................. SUCCESS [ 12.878 s]
> [INFO] Spark Project Hive Thrift Server ................... SUCCESS [ 12.658 s]
> [INFO] Spark Project Assembly ............................. SUCCESS [02:29 min]
> [INFO] Spark Project External Twitter ..................... SUCCESS [  9.703 s]
> [INFO] Spark Project External Flume Sink .................. SUCCESS [  8.613 s]
> [INFO] Spark Project External Flume ....................... SUCCESS [ 12.102 s]
> [INFO] Spark Project External Flume Assembly .............. SUCCESS [  3.986 s]
> [INFO] Spark Project External MQTT ........................ SUCCESS [ 28.259 s]
> [INFO] Spark Project External MQTT Assembly ............... SUCCESS [  8.307 s]
> [INFO] Spark Project External ZeroMQ ...................... SUCCESS [  9.224 s]
> [INFO] Spark Project External Kafka ....................... SUCCESS [ 14.552 s]
> [INFO] Spark Project Examples ............................. SUCCESS [02:09 min]
> [INFO] Spark Project External Kafka Assembly .............. SUCCESS [  9.130 s]
> [INFO] Spark Project YARN Shuffle Service ................. SUCCESS [  8.115 s]
> [INFO] ------------------------------------------------------------------------
> [INFO] BUILD SUCCESS
> [INFO] ------------------------------------------------------------------------
> [INFO] Total time: 18:28 min
> [INFO] Finished at: 2015-09-01T11:28:14-03:00
> [INFO] Final Memory: 415M/1675M
> [INFO] ------------------------------------------------------------------------
> (...)

{{< /source >}}

Resultado:

`spark-1.5.0-bin-2.7.1.tgz`

(Artefatos do Maven)

{{< source sh >}}
rm -rf ~/.m2/repository/org/apache/spark

mvn install -Phadoop-2.6 -Pyarn -Phive -Phive-thriftserver -Dhadoop.version=2.7.1 -DskipTests

> (...)
> [INFO] Reactor Summary:
> [INFO]
> [INFO] Spark Project Parent POM ........................... SUCCESS [ 12.246 s]
> [INFO] Spark Project Launcher ............................. SUCCESS [ 17.973 s]
> [INFO] Spark Project Networking ........................... SUCCESS [ 14.932 s]
> [INFO] Spark Project Shuffle Streaming Service ............ SUCCESS [  7.900 s]
> [INFO] Spark Project Unsafe ............................... SUCCESS [  9.596 s]
> [INFO] Spark Project Core ................................. SUCCESS [02:06 min]
> [INFO] Spark Project Bagel ................................ SUCCESS [  9.492 s]
> [INFO] Spark Project GraphX ............................... SUCCESS [ 24.895 s]
> [INFO] Spark Project Streaming ............................ SUCCESS [ 44.429 s]
> [INFO] Spark Project Catalyst ............................. SUCCESS [01:11 min]
> [INFO] Spark Project SQL .................................. SUCCESS [01:06 min]
> [INFO] Spark Project ML Library ........................... SUCCESS [01:17 min]
> [INFO] Spark Project Tools ................................ SUCCESS [ 10.111 s]
> [INFO] Spark Project Hive ................................. SUCCESS [ 53.255 s]
> [INFO] Spark Project REPL ................................. SUCCESS [ 20.327 s]
> [INFO] Spark Project YARN ................................. SUCCESS [ 18.751 s]
> [INFO] Spark Project Hive Thrift Server ................... SUCCESS [ 14.980 s]
> [INFO] Spark Project Assembly ............................. SUCCESS [02:22 min]
> [INFO] Spark Project External Twitter ..................... SUCCESS [ 10.691 s]
> [INFO] Spark Project External Flume Sink .................. SUCCESS [ 11.497 s]
> [INFO] Spark Project External Flume ....................... SUCCESS [ 13.606 s]
> [INFO] Spark Project External Flume Assembly .............. SUCCESS [  3.793 s]
> [INFO] Spark Project External MQTT ........................ SUCCESS [ 23.596 s]
> [INFO] Spark Project External MQTT Assembly ............... SUCCESS [ 10.091 s]
> [INFO] Spark Project External ZeroMQ ...................... SUCCESS [ 10.015 s]
> [INFO] Spark Project External Kafka ....................... SUCCESS [ 16.020 s]
> [INFO] Spark Project Examples ............................. SUCCESS [02:12 min]
> [INFO] Spark Project External Kafka Assembly .............. SUCCESS [  9.298 s]
> [INFO] Spark Project YARN Shuffle Service ................. SUCCESS [ 10.658 s]
> [INFO] ------------------------------------------------------------------------
> [INFO] BUILD SUCCESS
> [INFO] ------------------------------------------------------------------------
> [INFO] Total time: 16:37 min
> [INFO] Finished at: 2015-09-01T12:13:25-03:00
> [INFO] Final Memory: 117M/2053M
> [INFO] ------------------------------------------------------------------------

cd ~/.m2/repository/
tar cf spark-1.5.0-m2.tar org/apache/spark

{{< /source >}}

Resultado:

`spark-1.5.0-m2.tar`


## Conclusão

...

Você trabalha com Spark ou tem experiência com as tecnologias envolvidas? Venha trabalhar conosco.

http://talentos.globo.com/
