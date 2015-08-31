+++
author = "cirocavani"
comments = true
date = "2015-08-27T20:41:23-03:00"
draft = true
image = ""
menu = ""
share = true
slug = "tutorial-hadoop-local"
tags = ["Tutorial", "Hadoop", "Docker", "CentOS6"]
title = "Compilação e Configuração do Hadoop na Máquina Local"

+++

Esse tutorial é sobre ...

Para esse procedimento, é assumido que o Docker esteja instalado e funcionando; também é assumido acesso à Internet.

(originalmente, esse procedimento foi testado no ArchLinux atualizado até final de Agosto)

## Compilação

Documento com instruções de build do Hadoop [aqui](https://github.com/apache/hadoop/blob/trunk/BUILDING.txt).

O resultado desse procedimento é um pacote do Hadoop com as bibliotecas nativas compiladas para o CentOS6.

...

Instalação do container Docker na Máquina local.
<br/>(abre a shell dentro do container)

{{< source sh >}}
sudo docker run -i -t centos:6 /bin/
{{< /source >}}

Criação do usuário e local a serem usados na compilação e e geração do pacote.

{{< source sh >}}
adduser -m -d /hadoop hadoop
cd hadoop
{{< /source >}}

Instalação das dependências para compilar as bibliotecas nativas.
<br/>(específicas para o host, CentOS6)

{{< source sh >}}
yum install -y tar gzip gcc-c++ cmake zlib zlib-devel openssl openssl-devel fuse fuse-devel bzip2 bzip2-devel snappy snappy-devel
{{< /source >}}

Instalação do Protobuf usado no Hadoop.
<br/>(não tem no CentOS6)

{{< source sh >}}
curl -L -O https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz
tar zxf protobuf-2.5.0.tar.gz
cd protobuf-2.5.0
./configure --prefix=/usr --libdir=/usr/lib64
make
make check
make install

cd ..
{{< /source >}}

Instalação do Jansson usado no WebHDFS.
<br/>(não tem no CentOS6)

{{< source sh >}}
curl -O http://www.digip.org/jansson/releases/jansson-2.7.tar.gz
tar zxf jansson-2.7.tar.gz
cd jansson-2.7
./configure --prefix=/usr --libdir=/usr/lib64
make
make install

cd ..
{{< /source >}}

Instalação do JDK.

{{< source sh >}}
curl -k -L -H "Cookie: oraclelicense=accept-securebackup-cookie" -O http://download.oracle.com/otn-pub/java/jdk/8u60-b27/jdk-8u60-linux-x64.rpm
rpm -i jdk-8u60-linux-x64.rpm
{{< /source >}}

Instalação do Maven.

{{< source sh >}}
curl -O http://archive.apache.org/dist/maven/maven-3/3.3.3/binaries/apache-maven-3.3.3-bin.tar.gz
tar zxf apache-maven-3.3.3-bin.tar.gz
{{< /source >}}

Compilação do Hadoop.

{{< source sh >}}
su - hadoop

export PATH=$PATH:/hadoop/apache-maven-3.3.3/bin

curl -O http://archive.apache.org/dist/hadoop/common/hadoop-2.7.1/hadoop-2.7.1-src.tar.gz
tar zxf hadoop-2.7.1-src.tar.gz
cd hadoop-2.7.1-src

mvn clean package -Pdist,native -DskipTests -Drequire.snappy -Drequire.openssl -Dtar
{{< /source >}}

Bateria de Testes.

{{< source sh >}}
mkdir hadoop-common-project/hadoop-common/target/test-classes/webapps/test

mvn test -Pnative -Drequire.snappy -Drequire.openssl -Dmaven.test.failure.ignore=true -Dsurefire.rerunFailingTestsCount=3
{{< /source >}}

(alguns testes com falha intermitente)

* org.apache.hadoop.ipc.TestDecayRpcScheduler#testAccumulate
* org.apache.hadoop.ipc.TestDecayRpcScheduler#testPriority
* org.apache.hadoop.hdfs.server.datanode.TestDataNodeMetrics#testDataNodeTimeSpend
* org.apache.hadoop.hdfs.shortcircuit.TestShortCircuitCache#testDataXceiverHandlesRequestShortCircuitShmFailure

## Configuração

Procedimento para configuração local do Hadoop em modo pseudo-distribuído com uma JVM por serviço.

Serviços:

* HDFS: NameNode, SecondaryNameNode, DataNode
* YARN: ResouceManager, NodeManager, JobHistory

Requisitos:

* Java8
* SSH
* rsync
* zlib
* openssl
* fuse
* bzip2
* snappy

'ssh localhost' tem que abrir um shell sem pedir senha (usando authorized_keys)

Diretório:

* /data/hadoop

...

**Instalação do Hadoop**

(versão atual é a 2.7.1)

{{< source sh >}}
tar zxf hadoop-2.7.1.tar.gz -C /opt
{{< /source >}}

Configurar `JAVA_HOME` no arquivo `/opt/hadoop-2.7.1/etc/hadoop/hadoop-env.sh`:

{{< source sh >}}
export JAVA_HOME="..."
{{< /source >}}

Editar `/home/hadoop/.bash_profile`:
<br/>(scripts do Hadoop no PATH)

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

**Start/Stop**

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


**Teste MR**

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

