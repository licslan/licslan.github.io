---
layout:     post
title:      sample to use flink & spark
subtitle:   big data ml flink spark 
date:       2018-11-28
author:     LICSLAN
header-img: img/zjm.jpg
catalog: true
tags:
     - tec
     - big data
---


flink  

Apache Flink is a framework and distributed processing engine for stateful computations over unbounded and bounded data streams. Flink has been designed to run in all common cluster environments, perform computations at in-memory speed and at any scale.

Here, we explain important aspects of Flink’s architecture.

Process Unbounded and Bounded Data
Any kind of data is produced as a stream of events. Credit card transactions, sensor measurements, machine logs, or user interactions on a website or mobile application, all of these data are generated as a stream.

Data can be processed as unbounded or bounded streams.

Unbounded streams have a start but no defined end. They do not terminate and provide data as it is generated. Unbounded streams must be continuously processed, i.e., events must be promptly handled after they have been ingested. It is not possible to wait for all input data to arrive because the input is unbounded and will not be complete at any point in time. Processing unbounded data often requires that events are ingested in a specific order, such as the order in which events occurred, to be able to reason about result completeness.

Bounded streams have a defined start and end. Bounded streams can be processed by ingesting all data before performing any computations. Ordered ingestion is not required to process bounded streams because a bounded data set can always be sorted. Processing of bounded streams is also known as batch processing.







                                                     flink  leaning  on  the  way 


实时计算  （大数据框架）


三大模块  [flink standalone 搭建      卡夫卡 集群搭建     zookeeper集群搭建]








1.flink  demo  

Requirements
Software Requirements
Flink runs on all UNIX-like environments, e.g. Linux, Mac OS X, and Cygwin (for Windows) and expects the cluster to consist of one master node and one or more worker nodes. Before you start to setup the system, make sure you have the following software installed on each node:

Java 1.8.x or higher,
ssh (sshd must be running to use the Flink scripts that manage remote components)
If your cluster does not fulfill these software requirements you will need to install/upgrade it.

Having passwordless SSH and the same directory structure on all your cluster nodes will allow you to use our scripts to control everything.


JAVA_HOME Configuration
Flink requires the JAVA_HOME environment variable to be set on the master and all worker nodes and point to the directory of your Java installation.

You can set this variable in conf/flink-conf.yaml via the env.java.home key.


Flink Setup
Go to the downloads page and get the ready-to-run package. Make sure to pick the Flink package matching your Hadoop version. If you don’t plan to use Hadoop, pick any version.

After downloading the latest release, copy the archive to your master node and extract it:

tar xzf flink-*.tgz
cd flink-*
Configuring Flink
After having extracted the system files, you need to configure Flink for the cluster by editing conf/flink-conf.yaml.

Set the jobmanager.rpc.address key to point to your master node. You should also define the maximum amount of main memory the JVM is allowed to allocate on each node by setting the jobmanager.heap.mb and taskmanager.heap.mb keys.

These values are given in MB. If some worker nodes have more main memory which you want to allocate to the Flink system you can overwrite the default value by setting the environment variable FLINK_TM_HEAP on those specific nodes.

Finally, you must provide a list of all nodes in your cluster which shall be used as worker nodes. Therefore, similar to the HDFS configuration, edit the file conf/slaves and enter the IP/host name of each worker node. Each worker node will later run a TaskManager.

The following example illustrates the setup with three nodes (with IP addresses from 10.0.0.1 to 10.0.0.3 and hostnames master, worker1, worker2) and shows the contents of the configuration files (which need to be accessible at the same path on all machines):


/path/to/flink/conf/
flink-conf.yaml

jobmanager.rpc.address: 192.168.108.128
/path/to/flink/
conf/slaves

192.168.108.130
192.168.108.131


Starting Flink
The following script starts a JobManager on the local node and connects via SSH to all worker nodes listed in the slaves file to start the TaskManager on each node. Now your Flink system is up and running. The JobManager running on the local node will now accept jobs at the configured RPC port.

Assuming that you are on the master node and inside the Flink directory:

bin/start-cluster.sh
To stop Flink, there is also a stop-cluster.sh script.


Adding JobManager/TaskManager Instances to a Cluster
You can add both JobManager and TaskManager instances to your running cluster with the bin/jobmanager.sh and bin/taskmanager.sh scripts.

Adding a JobManager
bin/jobmanager.sh ((start|start-foreground) [host] [webui-port])|stop|stop-all
Adding a TaskManager
bin/taskmanager.sh start|start-foreground|stop|stop-all
Make sure to call these scripts on the hosts on which you want to start/stop the respective instance.


1.下载flink  文件到Linux系统   centos 7+    也可以使用docker环境   搭建Linux  docker  比较简单这里就不做介绍了   。。。

2.解压到指定的目录  tar zxvf  flink.tar.gz  -C  /path 

3.编辑  /path/to/flink/conf/flink-conf.yaml    jobmanager.rpc.address: 192.168.108.128   
  编辑  /path/to/flink/conf/conf/slaves    192.168.108.130  192.168.108.131

4.启动集群  bin/start-cluster.sh

5.添加 
Adding a JobManager
bin/jobmanager.sh ((start|start-foreground) [host] [webui-port])|stop|stop-all
Adding a TaskManager
bin/taskmanager.sh start|start-foreground|stop|stop-all  


demo   wiki  Monitoring the Wikipedia Edit Stream

In this guide we will start from scratch and go from setting up a Flink project to running a streaming analysis program on a Flink cluster.

Wikipedia provides an IRC channel where all edits to the wiki are logged. We are going to read this channel in Flink and count the number of bytes that each user edits within a given window of time. This is easy enough to implement in a few minutes using Flink, but it will give you a good foundation from which to start building more complex analysis programs on your own.

We are going to use a Flink Maven Archetype for creating our project structure. Please see Java API Quickstart for more details about this. For our purposes, the command to run is this:

$ mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-quickstart-java \
    -DarchetypeVersion=1.6.1 \
    -DgroupId=wiki-edits \
    -DartifactId=wiki-edits \
    -Dversion=0.1 \
    -Dpackage=wikiedits \
    -DinteractiveMode=false



There is our pom.xml file that already has the Flink dependencies added in the root directory and several example Flink programs in src/main/java. We can delete the example programs, since we are going to start from scratch:

$ rm wiki-edits/src/main/java/wikiedits/*.java
As a last step we need to add the Flink Wikipedia connector as a dependency so that we can use it in our program. Edit the dependencies section of the pom.xml so that it looks like this:

<dependencies>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-java</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-clients_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-wikiedits_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>
</dependencies>
Notice the flink-connector-wikiedits_2.11 dependency that was added. (This example and the Wikipedia connector were inspired by the Hello Samza example of Apache Samza.)



The complete code so far is this:

package wikiedits;

import org.apache.flink.api.common.functions.FoldFunction;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.streaming.connectors.wikiedits.WikipediaEditEvent;
import org.apache.flink.streaming.connectors.wikiedits.WikipediaEditsSource;

public class WikipediaAnalysis {

  public static void main(String[] args) throws Exception {

    StreamExecutionEnvironment see = StreamExecutionEnvironment.getExecutionEnvironment();

    DataStream<WikipediaEditEvent> edits = see.addSource(new WikipediaEditsSource());

    KeyedStream<WikipediaEditEvent, String> keyedEdits = edits
      .keyBy(new KeySelector<WikipediaEditEvent, String>() {
        @Override
        public String getKey(WikipediaEditEvent event) {
          return event.getUser();
        }
      });

    DataStream<Tuple2<String, Long>> result = keyedEdits
      .timeWindow(Time.seconds(5))
      .fold(new Tuple2<>("", 0L), new FoldFunction<WikipediaEditEvent, Tuple2<String, Long>>() {
        @Override
        public Tuple2<String, Long> fold(Tuple2<String, Long> acc, WikipediaEditEvent event) {
          acc.f0 = event.getUser();
          acc.f1 += event.getByteDiff();
          return acc;
        }
      });

    result.print();

    see.execute();
  }
}
You can run this example in your IDE or on the command line, using Maven:

$ mvn clean package
$ mvn exec:java -Dexec.mainClass=wikiedits.WikipediaAnalysis
The first command builds our project and the second executes our main class. The output should be similar to this:

1> (Fenix down,114)
6> (AnomieBOT,155)
8> (BD2412bot,-3690)
7> (IgnorantArmies,49)
3> (Ckh3111,69)
5> (Slade360,0)
7> (Narutolovehinata5,2195)
6> (Vuyisa2001,79)
4> (Ms Sarah Welch,269)
4> (KasparBot,-245)




和卡夫卡/zookeeper一起用

Bonus Exercise: Running on a Cluster and Writing to Kafka
Please follow our setup quickstart for setting up a Flink distribution on your machine and refer to the Kafka quickstart for setting up a Kafka installation before we proceed.

As a first step, we have to add the Flink Kafka connector as a dependency so that we can use the Kafka sink. Add this to the pom.xml file in the dependencies section:

<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.8_2.11</artifactId>
    <version>${flink.version}</version>
</dependency>
Next, we need to modify our program. We’ll remove the print() sink and instead use a Kafka sink. The new code looks like this:

result
    .map(new MapFunction<Tuple2<String,Long>, String>() {
        @Override
        public String map(Tuple2<String, Long> tuple) {
            return tuple.toString();
        }
    })
    .addSink(new FlinkKafkaProducer08<>("localhost:9092", "wiki-result", new SimpleStringSchema()));
The related classes also need to be imported:

import org.apache.flink.streaming.connectors.kafka.FlinkKafkaProducer08;
import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.api.common.functions.MapFunction;
Note how we first transform the Stream of Tuple2<String, Long> to a Stream of String using a MapFunction. We are doing this because it is easier to write plain strings to Kafka. Then, we create a Kafka sink. You might have to adapt the hostname and port to your setup. "wiki-result" is the name of the Kafka stream that we are going to create next, before running our program. Build the project using Maven because we need the jar file for running on the cluster:

$ mvn clean package
The resulting jar file will be in the target subfolder: target/wiki-edits-0.1.jar. We’ll use this later.

Now we are ready to launch a Flink cluster and run the program that writes to Kafka on it. Go to the location where you installed Flink and start a local cluster:

$ cd my/flink/directory
$ bin/start-cluster.sh
We also have to create the Kafka Topic, so that our program can write to it:

$ cd my/kafka/directory
$ bin/kafka-topics.sh --create --zookeeper localhost:2181 --topic wiki-results
Now we are ready to run our jar file on the local Flink cluster:

$ cd my/flink/directory
$ bin/flink run -c wikiedits.WikipediaAnalysis path/to/wikiedits-0.1.jar
The output of that command should look similar to this, if everything went according to plan:

03/08/2016 15:09:27 Job execution switched to status RUNNING.
03/08/2016 15:09:27 Source: Custom Source(1/1) switched to SCHEDULED
03/08/2016 15:09:27 Source: Custom Source(1/1) switched to DEPLOYING
03/08/2016 15:09:27 TriggerWindow(TumblingProcessingTimeWindows(5000), FoldingStateDescriptor{name=window-contents, defaultValue=(,0), serializer=null}, ProcessingTimeTrigger(), WindowedStream.fold(WindowedStream.java:207)) -> Map -> Sink: Unnamed(1/1) switched to SCHEDULED
03/08/2016 15:09:27 TriggerWindow(TumblingProcessingTimeWindows(5000), FoldingStateDescriptor{name=window-contents, defaultValue=(,0), serializer=null}, ProcessingTimeTrigger(), WindowedStream.fold(WindowedStream.java:207)) -> Map -> Sink: Unnamed(1/1) switched to DEPLOYING
03/08/2016 15:09:27 TriggerWindow(TumblingProcessingTimeWindows(5000), FoldingStateDescriptor{name=window-contents, defaultValue=(,0), serializer=null}, ProcessingTimeTrigger(), WindowedStream.fold(WindowedStream.java:207)) -> Map -> Sink: Unnamed(1/1) switched to RUNNING
03/08/2016 15:09:27 Source: Custom Source(1/1) switched to RUNNING
You can see how the individual operators start running. There are only two, because the operations after the window get folded into one operation for performance reasons. In Flink we call this chaining.

You can observe the output of the program by inspecting the Kafka topic using the Kafka console consumer:

bin/kafka-console-consumer.sh  --zookeeper localhost:2181 --topic wiki-result
You can also check out the Flink dashboard which should be running at http://localhost:8081. You get an overview of your cluster resources and running jobs:




所有呢  就要搭建卡夫卡集群和zookeeper集群的啊 

plz see doc   http://kafka.apache.org/documentation/#quickstart

下面有8个简单步骤可以搭建卡夫卡集群



Step 1: Download the code
Download the 2.1.0 release and un-tar it.
1
2
> tar -xzf kafka_2.11-2.1.0.tgz
> cd kafka_2.11-2.1.0
Step 2: Start the server
Kafka uses ZooKeeper so you need to first start a ZooKeeper server if you don't already have one. You can use the convenience script packaged with kafka to get a quick-and-dirty single-node ZooKeeper instance.

1
2
3
> bin/zookeeper-server-start.sh config/zookeeper.properties
[2013-04-22 15:01:37,495] INFO Reading configuration from: config/zookeeper.properties (org.apache.zookeeper.server.quorum.QuorumPeerConfig)
...
Now start the Kafka server:

1
2
3
4
> bin/kafka-server-start.sh config/server.properties
[2013-04-22 15:01:47,028] INFO Verifying properties (kafka.utils.VerifiableProperties)
[2013-04-22 15:01:47,051] INFO Property socket.send.buffer.bytes is overridden to 1048576 (kafka.utils.VerifiableProperties)
...
Step 3: Create a topic
Let's create a topic named "test" with a single partition and only one replica:

1
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
We can now see that topic if we run the list topic command:

1
2
> bin/kafka-topics.sh --list --zookeeper localhost:2181
test
Alternatively, instead of manually creating topics you can also configure your brokers to auto-create topics when a non-existent topic is published to.

Step 4: Send some messages
Kafka comes with a command line client that will take input from a file or from standard input and send it out as messages to the Kafka cluster. By default, each line will be sent as a separate message.

Run the producer and then type a few messages into the console to send to the server.

1
2
3
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
Step 5: Start a consumer
Kafka also has a command line consumer that will dump out messages to standard output.

1
2
3
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
If you have each of the above commands running in a different terminal then you should now be able to type messages into the producer terminal and see them appear in the consumer terminal.

All of the command line tools have additional options; running the command with no arguments will display usage information documenting them in more detail.

Step 6: Setting up a multi-broker cluster
So far we have been running against a single broker, but that's no fun. For Kafka, a single broker is just a cluster of size one, so nothing much changes other than starting a few more broker instances. But just to get feel for it, let's expand our cluster to three nodes (still all on our local machine).

First we make a config file for each of the brokers (on Windows use the copy command instead):

1
2
> cp config/server.properties config/server-1.properties
> cp config/server.properties config/server-2.properties
Now edit these new files and set the following properties:

1
2
3
4
5
6
7
8
9
config/server-1.properties:
    broker.id=1
    listeners=PLAINTEXT://:9093
    log.dirs=/tmp/kafka-logs-1
 
config/server-2.properties:
    broker.id=2
    listeners=PLAINTEXT://:9094
    log.dirs=/tmp/kafka-logs-2
The broker.id property is the unique and permanent name of each node in the cluster. We have to override the port and log directory only because we are running these all on the same machine and we want to keep the brokers from all trying to register on the same port or overwrite each other's data.

We already have Zookeeper and our single node started, so we just need to start the two new nodes:

1
2
3
4
> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
Now create a new topic with a replication factor of three:

1
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
Okay but now that we have a cluster how can we know which broker is doing what? To see that run the "describe topics" command:

1
2
3
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
Here is an explanation of output. The first line gives a summary of all the partitions, each additional line gives information about one partition. Since we have only one partition for this topic there is only one line.

"leader" is the node responsible for all reads and writes for the given partition. Each node will be the leader for a randomly selected portion of the partitions.
"replicas" is the list of nodes that replicate the log for this partition regardless of whether they are the leader or even if they are currently alive.
"isr" is the set of "in-sync" replicas. This is the subset of the replicas list that is currently alive and caught-up to the leader.
Note that in my example node 1 is the leader for the only partition of the topic.

We can run the same command on the original topic we created to see where it is:

1
2
3
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0
So there is no surprise there—the original topic has no replicas and is on server 0, the only server in our cluster when we created it.

Let's publish a few messages to our new topic:

1
2
3
4
5
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
Now let's consume these messages:

1
2
3
4
5
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
Now let's test out fault-tolerance. Broker 1 was acting as the leader so let's kill it:

1
2
3
> ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
> kill -9 7564
On Windows use:
1
2
3
4
> wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
ProcessId
6016
> taskkill /pid 6016 /f
Leadership has switched to one of the slaves and node 1 is no longer in the in-sync replica set:

1
2
3
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
But the messages are still available for consumption even though the leader that took the writes originally is down:

1
2
3
4
5
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
Step 7: Use Kafka Connect to import/export data
Writing data from the console and writing it back to the console is a convenient place to start, but you'll probably want to use data from other sources or export data from Kafka to other systems. For many systems, instead of writing custom integration code you can use Kafka Connect to import or export data.

Kafka Connect is a tool included with Kafka that imports and exports data to Kafka. It is an extensible tool that runs connectors, which implement the custom logic for interacting with an external system. In this quickstart we'll see how to run Kafka Connect with simple connectors that import data from a file to a Kafka topic and export data from a Kafka topic to a file.

First, we'll start by creating some seed data to test with:

1
> echo -e "foo\nbar" > test.txt
Or on Windows:
1
2
> echo foo> test.txt
> echo bar>> test.txt
Next, we'll start two connectors running in standalone mode, which means they run in a single, local, dedicated process. We provide three configuration files as parameters. The first is always the configuration for the Kafka Connect process, containing common configuration such as the Kafka brokers to connect to and the serialization format for data. The remaining configuration files each specify a connector to create. These files include a unique connector name, the connector class to instantiate, and any other configuration required by the connector.

1
> bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
These sample configuration files, included with Kafka, use the default local cluster configuration you started earlier and create two connectors: the first is a source connector that reads lines from an input file and produces each to a Kafka topic and the second is a sink connector that reads messages from a Kafka topic and produces each as a line in an output file.

During startup you'll see a number of log messages, including some indicating that the connectors are being instantiated. Once the Kafka Connect process has started, the source connector should start reading lines from test.txt and producing them to the topic connect-test, and the sink connector should start reading messages from the topic connect-test and write them to the file test.sink.txt. We can verify the data has been delivered through the entire pipeline by examining the contents of the output file:

1
2
3
> more test.sink.txt
foo
bar
Note that the data is being stored in the Kafka topic connect-test, so we can also run a console consumer to see the data in the topic (or use custom consumer code to process it):

1
2
3
4
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
...
The connectors continue to process data, so we can add data to the file and see it move through the pipeline:

1
> echo Another line>> test.txt
You should see the line appear in the console consumer output and in the sink file.

Step 8: Use Kafka Streams to process data
Kafka Streams is a client library for building mission-critical real-time applications and microservices, where the input and/or output data is stored in Kafka clusters. Kafka Streams combines the simplicity of writing and deploying standard Java and Scala applications on the client side with the benefits of Kafka's server-side cluster technology to make these applications highly scalable, elastic, fault-tolerant, distributed, and much more. This quickstart example will demonstrate how to run a streaming application coded in this library.



搭建的zookeeper集群呢  so plz see http://zookeeper.apache.org/doc/current/zookeeperStarted.html

ZooKeeper Getting Started Guide
Getting Started: Coordinating Distributed Applications with ZooKeeper
Pre-requisites
Download
Standalone Operation
Managing ZooKeeper Storage
Connecting to ZooKeeper
Programming to ZooKeeper
Running Replicated ZooKeeper
Other Optimizations
Getting Started: Coordinating Distributed Applications with ZooKeeper
This document contains information to get you started quickly with ZooKeeper. It is aimed primarily at developers hoping to try it out, and contains simple installation instructions for a single ZooKeeper server, a few commands to verify that it is running, and a simple programming example. Finally, as a convenience, there are a few sections regarding more complicated installations, for example running replicated deployments, and optimizing the transaction log. However for the complete instructions for commercial deployments, please refer to the ZooKeeper Administrator's Guide.

Pre-requisites
See System Requirements in the Admin guide.

Download
To get a ZooKeeper distribution, download a recent stable release from one of the Apache Download Mirrors.

Standalone Operation
Setting up a ZooKeeper server in standalone mode is straightforward. The server is contained in a single JAR file, so installation consists of creating a configuration.

Once you've downloaded a stable ZooKeeper release unpack it and cd to the root

To start ZooKeeper you need a configuration file. Here is a sample, create it in conf/zoo.cfg:

tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
This file can be called anything, but for the sake of this discussion call it conf/zoo.cfg. Change the value of dataDir to specify an existing (empty to start with) directory. Here are the meanings for each of the fields:

tickTime
the basic time unit in milliseconds used by ZooKeeper. It is used to do heartbeats and the minimum session timeout will be twice the tickTime.

dataDir
the location to store the in-memory database snapshots and, unless specified otherwise, the transaction log of updates to the database.

clientPort
the port to listen for client connections

Now that you created the configuration file, you can start ZooKeeper:

bin/zkServer.sh start
ZooKeeper logs messages using log4j -- more detail available in the Logging section of the Programmer's Guide. You will see log messages coming to the console (default) and/or a log file depending on the log4j configuration.

The steps outlined here run ZooKeeper in standalone mode. There is no replication, so if ZooKeeper process fails, the service will go down. This is fine for most development situations, but to run ZooKeeper in replicated mode, please see Running Replicated ZooKeeper.

Managing ZooKeeper Storage
For long running production systems ZooKeeper storage must be managed externally (dataDir and logs). See the section on maintenance for more details.

Connecting to ZooKeeper
$ bin/zkCli.sh -server 127.0.0.1:2181
This lets you perform simple, file-like operations.

Once you have connected, you should see something like:


Connecting to localhost:2181
log4j:WARN No appenders could be found for logger (org.apache.zookeeper.ZooKeeper).
log4j:WARN Please initialize the log4j system properly.
Welcome to ZooKeeper!
JLine support is enabled
[zkshell: 0]
        
From the shell, type help to get a listing of commands that can be executed from the client, as in:


[zkshell: 0] help
ZooKeeper host:port cmd args
        get path [watch]
        ls path [watch]
        set path data [version]
        delquota [-n|-b] path
        quit
        printwatches on|off
        createpath data acl
        stat path [watch]
        listquota path
        history
        setAcl path acl
        getAcl path
        sync path
        redo cmdno
        addauth scheme auth
        delete path [version]
        setquota -n|-b val path

        
From here, you can try a few simple commands to get a feel for this simple command line interface. First, start by issuing the list command, as in ls, yielding:


[zkshell: 8] ls /
[zookeeper]
        
Next, create a new znode by running create /zk_test my_data. This creates a new znode and associates the string "my_data" with the node. You should see:


[zkshell: 9] create /zk_test my_data
Created /zk_test
      
Issue another ls / command to see what the directory looks like:


[zkshell: 11] ls /
[zookeeper, zk_test]

        
Notice that the zk_test directory has now been created.

Next, verify that the data was associated with the znode by running the get command, as in:


[zkshell: 12] get /zk_test
my_data
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 5
mtime = Fri Jun 05 13:57:06 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0
dataLength = 7
numChildren = 0
        
We can change the data associated with zk_test by issuing the set command, as in:


[zkshell: 14] set /zk_test junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
[zkshell: 15] get /zk_test
junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
      
(Notice we did a get after setting the data and it did, indeed, change.

Finally, let's delete the node by issuing:


[zkshell: 16] delete /zk_test
[zkshell: 17] ls /
[zookeeper]
[zkshell: 18]
That's it for now. To explore more, continue with the rest of this document and see the Programmer's Guide.

Programming to ZooKeeper
ZooKeeper has a Java bindings and C bindings. They are functionally equivalent. The C bindings exist in two variants: single threaded and multi-threaded. These differ only in how the messaging loop is done. For more information, see the Programming Examples in the ZooKeeper Programmer's Guide for sample code using of the different APIs.

Running Replicated ZooKeeper
Running ZooKeeper in standalone mode is convenient for evaluation, some development, and testing. But in production, you should run ZooKeeper in replicated mode. A replicated group of servers in the same application is called a quorum, and in replicated mode, all servers in the quorum have copies of the same configuration file.

Note
For replicated mode, a minimum of three servers are required, and it is strongly recommended that you have an odd number of servers. If you only have two servers, then you are in a situation where if one of them fails, there are not enough machines to form a majority quorum. Two servers is inherently less stable than a single server, because there are two single points of failure.

The required conf/zoo.cfg file for replicated mode is similar to the one used in standalone mode, but with a few differences. Here is an example:

tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
initLimit=5
syncLimit=2
server.1=zoo1:2888:3888
server.2=zoo2:2888:3888
server.3=zoo3:2888:3888
The new entry, initLimit is timeouts ZooKeeper uses to limit the length of time the ZooKeeper servers in quorum have to connect to a leader. The entry syncLimit limits how far out of date a server can be from a leader.

With both of these timeouts, you specify the unit of time using tickTime. In this example, the timeout for initLimit is 5 ticks at 2000 milleseconds a tick, or 10 seconds.

The entries of the form server.X list the servers that make up the ZooKeeper service. When the server starts up, it knows which server it is by looking for the file myid in the data directory. That file has the contains the server number, in ASCII.

Finally, note the two port numbers after each server name: " 2888" and "3888". Peers use the former port to connect to other peers. Such a connection is necessary so that peers can communicate, for example, to agree upon the order of updates. More specifically, a ZooKeeper server uses this port to connect followers to the leader. When a new leader arises, a follower opens a TCP connection to the leader using this port. Because the default leader election also uses TCP, we currently require another port for leader election. This is the second port in the server entry.

Note
If you want to test multiple servers on a single machine, specify the servername as localhost with unique quorum & leader election ports (i.e. 2888:3888, 2889:3889, 2890:3890 in the example above) for each server.X in that server's config file. Of course separate dataDirs and distinct clientPorts are also necessary (in the above replicated example, running on a single localhost, you would still have three config files).

Please be aware that setting up multiple servers on a single machine will not create any redundancy. If something were to happen which caused the machine to die, all of the zookeeper servers would be offline. Full redundancy requires that each server have its own machine. It must be a completely separate physical server. Multiple virtual machines on the same physical host are still vulnerable to the complete failure of that host.

Other Optimizations
There are a couple of other configuration parameters that can greatly increase performance:

To get low latencies on updates it is important to have a dedicated transaction log directory. By default transaction logs are put in the same directory as the data snapshots and myid file. The dataLogDir parameters indicates a different directory to use for the transaction logs.

[tbd: what is the other config param?]




记得要设置  ssh  免密登陆

设置ssh免登录

生成ssh密码的命令，-t 参数表示生成算法，有rsa和dsa两种；-P表示使用的密码，这里使用“”空字符串表示无密码。直接回车。

  [root@vm1 ~]# ssh-keygen -t rsa -P ""
将生成的密钥写入authorized_keys文件。

  [root@vm1 ~]# cat .ssh/id_rsa.pub >>.ssh/authorized_keys
将.ssh目录拷贝到其它主机相同目录下。

  [root@vm1 ~]# scp -r ~/.ssh root@192.168.108.130:~/
  [root@vm1 ~]# scp -r ~/.ssh root@192.168.108.131:~/












spark

Apache Spark is a fast and general-purpose cluster computing system. It provides high-level APIs in Java, Scala, Python and R, and an optimized engine that supports general execution graphs. It also supports a rich set of higher-level tools including Spark SQL for SQL and structured data processing, MLlib for machine learning, GraphX for graph processing, and Spark Streaming.
