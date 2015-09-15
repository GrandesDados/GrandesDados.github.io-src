+++
author = "cirocavani"
comments = true
date = "2015-09-12T21:49:07-03:00"
draft = true
image = ""
menu = ""
share = true
slug = "configuracao-do-hadoop-hbase-e-kafka-na-maquina-local-com-docker"
tags = ["Tutorial", "Hadoop", "HBase", "Kafka", "Docker"]
title = "Configuração do Hadoop, HBase e Kafka na Máquina Local com Docker"

+++

Esse tutorial é sobre a criação de uma imagem do Docker com a configuração local do Hadoop, HBase e Kafka. Nesse procedimento, o Hadoop é configurado no modo pseudo-distribuído com cada serviço rodando em uma instância própria da JVM, mas todas na mesma máquina. O HBase e o Kafka também rodam em modo 'distribuído' compartilhando uma instância separada do ZooKeeper. Esse procedimento é muito útil para testar funcionalidades desses serviços e aprendizado, mas não é uma solução completa para uso em produção.

## Pré-requisito

Nesse procedimento, é necessário que o Docker esteja instalado e funcionando; também é necessário acesso à Internet.

Originalmente, esse procedimento foi testado no ArchLinux atualizado até final de Agosto/2015.

https://wiki.archlinux.org/index.php/Docker

{{< source sh >}}
sudo docker version

> Client:
>  Version:      1.8.1
>  API version:  1.20
>  Go version:   go1.4.2
>  Git commit:   d12ea79
>  Built:        Sat Aug 15 17:29:10 UTC 2015
>  OS/Arch:      linux/amd64
>
> Server:
>  Version:      1.8.1
>  API version:  1.20
>  Go version:   go1.4.2
>  Git commit:   d12ea79
>  Built:        Sat Aug 15 17:29:10 UTC 2015
>  OS/Arch:      linux/amd64
{{< /source >}}


## Configuração

Hadoop, ZooKeeper, HBase e Kafka.

### Container

Começamos com a criação de um conainer do Docker com a imagem do CentOS6.

Ao executar o comando `run`, o Docker automaticamente fará o download da imagem e a shell será inicializada dentro de um novo container.

(o identificador do container nesse exemplo é `da0c7171989b`)

{{< source sh >}}
sudo docker run -i -t centos:6 /bin/bash

> Unable to find image 'centos:6' locally
> 6: Pulling from library/centos
>
> f1b10cd84249: Pull complete
> fb9cc58bde0c: Pull complete
> a005304e4e74: Already exists
> library/centos:6: The image you are pulling has been verified. Important: image verification is a tech preview feature and should not be relied on to provide security.
>
> Digest: sha256:25d94c55b37cb7a33ad706d5f440e36376fec20f59e57d16fe02c64698b531c1
> Status: Downloaded newer image for centos:6
> [root@da0c7171989b /]#
{{< /source >}}

Já dentro do container criamos um usuário e local que serão usados para a instalação e execução dos processos.

{{< source sh >}}
adduser -m -d /hadoop hadoop
cd hadoop
{{< /source >}}

A versão usada nesse procedimento é o Java 8, atual versão estável da Oracle.

{{< source sh >}}
curl -k -L -H "Cookie: oraclelicense=accept-securebackup-cookie" -O http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.rpm
rpm -i jdk-8u60-linux-x64.rpm

java -version

> java version "1.8.0_60"
> Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
> Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
{{< /source >}}

Para completar o ambiente de execução, instalamos os serviços e bibliotecas necessárias.

{{< source sh >}}
yum install -y tar openssh-clients openssh-server rsync gzip zlib openssl fuse bzip2 snappy

> (...)
{{< /source >}}

(configuração do SSH para acesso sem senha)

{{< source sh >}}
service sshd start
chkconfig sshd on

su - hadoop

ssh-keygen -C hadoop -P '' -f ~/.ssh/id_rsa
cp ~/.ssh/{id_rsa.pub,authorized_keys}

ssh-keyscan localhost >> ~/.ssh/known_hosts
ssh-keyscan 127.0.0.1 >> ~/.ssh/known_hosts

ssh localhost

> (nova shell, sem login nem confirmação)

# (sair do shell do ssh)
exit
# (sair do shell do su)
exit

whoami

> root
{{< /source >}}


### Hadoop

Procedimento para configuração local do Hadoop em modo pseudo-distribuído com uma JVM por serviço.

Esse procedimento é baseado na [documentação do Hadoop](http://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/SingleCluster.html).

Serviços:

* HDFS: NameNode, SecondaryNameNode, DataNode
* YARN: ResouceManager, NodeManager, Timeline

Diretório:

* /data/hadoop

...

**Instalação**

(versão atual é a 2.7.1)

{{< source sh >}}
tar zxf hadoop-2.7.1.tar.gz -C /opt
{{< /source >}}

Configurar `JAVA_HOME` no arquivo `/opt/hadoop-2.7.1/etc/hadoop/hadoop-env.sh`:

{{< source sh >}}
export JAVA_HOME="..."
{{< /source >}}

Editar `/hadoop/.bash_profile` (adicionar):

{{< source sh >}}
export PATH=$PATH:/opt/hadoop-2.7.1/bin:/opt/hadoop-2.7.1/sbin
{{< /source >}}

(Verificação)

{{< source sh >}}
hadoop version

> Hadoop 2.7.1
> Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r cc72e9b000545b86b75a61f4835eb86d57bfafc0
> Compiled by jenkins on 2014-11-14T23:45Z
> Compiled with protoc 2.5.0
> From source with checksum df7537a4faa4658983d397abf4514320
> This command was run using /opt/hadoop-2.7.1/share/hadoop/common/hadoop-common-2.7.1.jar
{{< /source >}}

Editar `/opt/hadoop-2.7.1/etc/hadoop/core-site.xml`:

{{< source xml >}}
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost</value>
    </property>
</configuration>
{{< /source >}}

Editar `/opt/hadoop-2.7.1/etc/hadoop/hdfs-site.xml`:

{{< source xml >}}
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.blocksize</name>
        <value>8M</value>
    </property>
</configuration>
{{< /source >}}

Editar `/opt/hadoop-2.7.1/etc/hadoop/yarn-site.xml`:

{{< source xml >}}
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
{{< /source >}}

Editar `/opt/hadoop-2.7.1/etc/hadoop/mapred-site.xml`:

{{< source xml >}}
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
    <property>
        <name>mapreduce.jobtracker.staging.root.dir</name>
        <value>/user</value>
    </property>
</configuration>
{{< /source >}}

**Setup Inicial**

(antes da primeira inicialização)

{{< source sh >}}
hdfs namenode -format
{{< /source >}}

**Start / Stop**

{{< source sh >}}
start-dfs.sh
start-yarn.sh
mr-jobhistory-daemon.sh start historyserver
{{< /source >}}

{{< source sh >}}
stop-yarn.sh
stop-dfs.sh
mr-jobhistory-daemon.sh stop historyserver
{{< /source >}}

**Console Web**

HDFS

http://localhost:50070/

YARN

http://localhost:8088/

Job History

http://localhost:19888/


**Teste**

(Teste 1) Cálculo do Pi

{{< source sh >}}
yarn jar /opt/hadoop-2.5.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.2.jar pi 16 100000
{{< /source >}}

(Teste 2) Grep

{{< source sh >}}
hdfs dfs -put /opt/hadoop-2.5.2/etc/hadoop /hadoop_test

yarn jar /opt/hadoop-2.5.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.2.jar grep /hadoop_test /hadoop_output 'dfs[a-z.]+'

hdfs dfs -cat /hadoop_output/*

> 6       dfs.audit.logger4       dfs.class
> 3       dfs.server.namenode.
> 2       dfs.period
> 2       dfs.audit.log.maxfilesize
> 2       dfs.audit.log.maxbackupindex
> 1       dfsmetrics.log
> 1       dfsadmin
> 1       dfs.servers
> 1       dfs.replication
> 1       dfs.file

hdfs dfs -rm -r /hadoop_test /hadoop_output
{{< /source >}}


### ZooKeeper

Esse procedimento é baseado na [documentação do ZooKeeper](https://zookeeper.apache.org/doc/r3.4.6/zookeeperStarted.html).

{{< source sh >}}
curl -L -O http://archive.apache.org/dist/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
tar zxf zookeeper-3.4.6.tar.gz -C /opt
{{< /source >}}

Editar `/hadoop/.bash_profile` (adicionar):

{{< source sh >}}
export PATH=$PATH:/opt/zookeeper-3.4.6/bin
{{< /source >}}

Editar `/opt/zookeeper-3.4.6/conf/zoo.cfg`:

{{< source ini >}}
tickTime=6000
dataDir=/data/zookeeper
clientPort=2181
{{< /source >}}


**Start / Stop**

{{< source sh >}}
zkServer.sh start
{{< /source >}}

{{< source sh >}}
zkServer.sh stop
{{< /source >}}

**Teste**

...


### HBase

Esse procedimento é baseado na [documentação do HBase](http://hbase.apache.org/book.html#quickstart).

{{< source sh >}}
curl -L -O http://archive.apache.org/dist/hbase/hbase-1.1.2/hbase-1.1.2-bin.tar.gz
tar zxf hbase-1.1.2-bin.tar.gz -C /opt
{{< /source >}}

Editar `/hadoop/.bash_profile` (adicionar):

{{< source sh >}}
export PATH=$PATH:/opt/hbase-1.1.2/bin
{{< /source >}}

Editar `/opt/hbase-1.1.2/conf/hbase-site.xml`:

{{< source xml >}}
<configuration>
  <property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
  </property>
  <property>
    <name>hbase.rootdir</name>
    <value>file:///data/hbase/root</value>
  </property>
  <property>
    <name>hbase.tmp.dir</name>
    <value>/data/hbase/tmp</value>
  </property>
  <property>
    <name>hbase.zookeeper.quorum</name>
    <value>localhost</value>
  </property>
  <property>
    <name>zookeeper.znode.parent</name>
    <value>/grandesdados-hbase</value>
  </property>
</configuration>
{{< /source >}}

Editar `/opt/hbase-1.1.2/conf/hbase-env.sh`:

{{< source sh >}}
export HBASE_OPTS="-XX:+UseConcMarkSweepGC -Djava.net.preferIPv4Stack=true"
export HBASE_MANAGES_ZK=false
{{< /source >}}

**Start / Stop**

{{< source sh >}}
start-hbase.sh
{{< /source >}}

{{< source sh >}}
stop-hbase.sh
{{< /source >}}

**Teste**

...


### Kafka

Esse procedimento é baseado na [documentação do Kafka](http://kafka.apache.org/documentation.html#quickstart).

{{< source sh >}}
curl -L -O http://archive.apache.org/dist/kafka/0.8.2.1/kafka_2.10-0.8.2.1.tgz
tar zxf kafka_2.10-0.8.2.1.tgz -C /opt
{{< /source >}}

Editar `/hadoop/.bash_profile` (adicionar):

{{< source sh >}}
export PATH=$PATH:/opt/kafka_2.10-0.8.2.1/bin
{{< /source >}}

Editar `/opt/kafka_2.10-0.8.2.1/config/server.properties`:
<br/>(manter conteúdo original, só alterar os valores abaixo)

{{< source ini >}}
log.dirs=/data/kafka
zookeeper.connect=localhost:2181/grandesdados-kafka
{{< /source >}}

**Start / Stop**

{{< source sh >}}
kafka-server-start.sh /opt/kafka_2.10-0.8.2.1/config/server.properties
{{< /source >}}

{{< source sh >}}
kafka-server-stop.sh /opt/kafka_2.10-0.8.2.1/config/server.properties
{{< /source >}}

**Teste**

...


## Conclusão

...

Você trabalha com Hadoop, HBase, Kafka ou tem experiência com as tecnologias envolvidas? Venha trabalhar conosco.

http://talentos.globo.com/

