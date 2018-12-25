---
layout:     post
title:      tec kafka zookeeper
subtitle:   Simply build a kafka_zookeeper cluster
date:       2018-12-25
author:     LICSLAN
header-img: img/zjm.jpg
catalog: true
tags:
     - tec 
     - build a kafka_zookeeper
---

下面主要介绍使用官网来搭建kafka zookeeper 集群 后面也会整合jstorm讲一个实时计算的demo 资料来源于官网比较好 所以我们也要坚持学习英语<br>

我对英语还是蛮有兴趣的  加上工作中也会接触吧  多少都会去官网了解和学习  首先写个demo  后面再在此基础上不断扩展和性能优化  后面我会不断讲自己<br>

学习的东西记录下来 好记性不如烂笔头  关注于大数据  机器学习  容器技术  devops  k8s  docker  后面也会不断写点内容<br>

主机IP 192.168.108.128(master)  192.168.108.130   192.168.108.131

废话不多说，让我们来访问kafka.apache.org  Apache的顶级项目大家发现了吧  比如要毕业的dubbo.apache.org  ....<br>

plz  see  the  http://kafka.apache.org/...  <br>

Apache Kafka® is a distributed streaming platform. What exactly does that mean?<br>
A streaming platform has three key capabilities:<br>

Publish and subscribe to streams of records, similar to a message queue or enterprise messaging system.<br>
Store streams of records in a fault-tolerant durable way.<br>
Process streams of records as they occur.<br>
Kafka is generally used for two broad classes of applications:<br>

Building real-time streaming data pipelines that reliably get data between systems or applications<br>
Building real-time streaming applications that transform or react to the streams of data<br>
To understand how Kafka does these things, let's dive in and explore Kafka's capabilities from the bottom up.<br>

First a few concepts:<br>

Kafka is run as a cluster on one or more servers that can span multiple datacenters.<br>
The Kafka cluster stores streams of records in categories called topics.<br>
Each record consists of a key, a value, and a timestamp.<br>

Step 1: Download the code<br>
> tar -xzf kafka_2.11-2.1.0.tgz
> cd kafka_2.11-2.1.0

Step 2: Start the server<br>
Kafka uses ZooKeeper so you need to first start a ZooKeeper server if you don't already have one. You can use the convenience script packaged with kafka to get a quick-and-dirty single-node ZooKeeper instance.

> bin/zookeeper-server-start.sh config/zookeeper.properties<br>

Now start the Kafka server:<br>

> bin/kafka-server-start.sh config/server.properties<br>

Step 3: Create a topic<br>
Let's create a topic named "test" with a single partition and only one replica:

> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
We can now see that topic if we run the list topic command:

> bin/kafka-topics.sh --list --zookeeper localhost:2181
test

Alternatively, instead of manually creating topics you can also configure your brokers to auto-create topics when a non-existent topic is published to.

Step 4: Send some messages<br>
Kafka comes with a command line client that will take input from a file or from standard input and send it out as messages to the Kafka cluster. By default, each line will be sent as a separate message.

Run the producer and then type a few messages into the console to send to the server.


> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message

Step 5: Start a consumer<br>
Kafka also has a command line consumer that will dump out messages to standard output.


> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
If you have each of the above commands running in a different terminal then you should now be able to type messages into the producer terminal and see them appear in the consumer terminal.

All of the command line tools have additional options; running the command with no arguments will display usage information documenting them in more detail.

Step 6: Setting up a multi-broker cluster<br>
So far we have been running against a single broker, but that's no fun. For Kafka, a single broker is just a cluster of size one, so nothing much changes other than starting a few more broker instances. But just to get feel for it, let's expand our cluster to three nodes (still all on our local machine).

First we make a config file for each of the brokers (on Windows use the copy command instead):


> cp config/server.properties config/server-1.properties
> cp config/server.properties config/server-2.properties
Now edit these new files and set the following properties:


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


> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
Now create a new topic with a replication factor of three:

> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
Okay but now that we have a cluster how can we know which broker is doing what? To see that run the "describe topics" command:


> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 1   Replicas: 1,2,0 Isr: 1,2,0
Here is an explanation of output. The first line gives a summary of all the partitions, each additional line gives information about one partition. Since we have only one partition for this topic there is only one line.

"leader" is the node responsible for all reads and writes for the given partition. Each node will be the leader for a randomly selected portion of the partitions.
"replicas" is the list of nodes that replicate the log for this partition regardless of whether they are the leader or even if they are currently alive.
"isr" is the set of "in-sync" replicas. This is the subset of the replicas list that is currently alive and caught-up to the leader.
Note that in my example node 1 is the leader for the only partition of the topic.

We can run the same command on the original topic we created to see where it is:


> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test  PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: test Partition: 0    Leader: 0   Replicas: 0 Isr: 0
So there is no surprise there—the original topic has no replicas and is on server 0, the only server in our cluster when we created it.

Let's publish a few messages to our new topic:


> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
Now let's consume these messages:


> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
Now let's test out fault-tolerance. Broker 1 was acting as the leader so let's kill it:


> ps aux | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.8/Home/bin/java...
> kill -9 7564
On Windows use:

> wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
ProcessId
6016
> taskkill /pid 6016 /f
Leadership has switched to one of the slaves and node 1 is no longer in the in-sync replica set:

> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic   PartitionCount:1    ReplicationFactor:3 Configs:
    Topic: my-replicated-topic  Partition: 0    Leader: 2   Replicas: 1,2,0 Isr: 2,0
But the messages are still available for consumption even though the leader that took the writes originally is down:


> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
Step 7: Use Kafka Connect to import/export data<br>
Writing data from the console and writing it back to the console is a convenient place to start, but you'll probably want to use data from other sources or export data from Kafka to other systems. For many systems, instead of writing custom integration code you can use Kafka Connect to import or export data.

Kafka Connect is a tool included with Kafka that imports and exports data to Kafka. It is an extensible tool that runs connectors, which implement the custom logic for interacting with an external system. In this quickstart we'll see how to run Kafka Connect with simple connectors that import data from a file to a Kafka topic and export data from a Kafka topic to a file.

First, we'll start by creating some seed data to test with:

> echo -e "foo\nbar" > test.txt
Or on Windows:
> echo foo> test.txt
> echo bar>> test.txt
Next, we'll start two connectors running in standalone mode, which means they run in a single, local, dedicated process. We provide three configuration files as parameters. The first is always the configuration for the Kafka Connect process, containing common configuration such as the Kafka brokers to connect to and the serialization format for data. The remaining configuration files each specify a connector to create. These files include a unique connector name, the connector class to instantiate, and any other configuration required by the connector.

> bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
These sample configuration files, included with Kafka, use the default local cluster configuration you started earlier and create two connectors: the first is a source connector that reads lines from an input file and produces each to a Kafka topic and the second is a sink connector that reads messages from a Kafka topic and produces each as a line in an output file.

During startup you'll see a number of log messages, including some indicating that the connectors are being instantiated. Once the Kafka Connect process has started, the source connector should start reading lines from test.txt and producing them to the topic connect-test, and the sink connector should start reading messages from the topic connect-test and write them to the file test.sink.txt. We can verify the data has been delivered through the entire pipeline by examining the contents of the output file:

> more test.sink.txt
foo
bar
Note that the data is being stored in the Kafka topic connect-test, so we can also run a console consumer to see the data in the topic (or use custom consumer code to process it):

> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
...
The connectors continue to process data, so we can add data to the file and see it move through the pipeline:

> echo Another line>> test.txt
You should see the line appear in the console consumer output and in the sink file.

Step 8: Use Kafka Streams to process data<br>
Kafka Streams is a client library for building mission-critical real-time applications and microservices, where the input and/or output data is stored in Kafka clusters. Kafka Streams combines the simplicity of writing and deploying standard Java and Scala applications on the client side with the benefits of Kafka's server-side cluster technology to make these applications highly scalable, elastic, fault-tolerant, distributed, and much more. This quickstart example will demonstrate how to run a streaming application coded in this library.<br>

ok 上面的呢  其实我是官网贴的啊  不过呢   本人也是尝试了哈  自己也做个demo测试也去做过的 kafka 解压的文件里面也自带了zookeeper配置 下面我们来看zookeeper官网 来搭建自己的zookeeper集群 不是kafka自带的 <br>

plz see the http://zookeeper.apache.org/...  <br>
Apache ZooKeeper is an effort to develop and maintain an open-source server which enables highly reliable distributed coordination.

What is ZooKeeper?<br>
ZooKeeper is a centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. All of these kinds of services are used in some form or another by distributed applications. Each time they are implemented there is a lot of work that goes into fixing the bugs and race conditions that are inevitable. Because of the difficulty of implementing these kinds of services, applications initially usually skimp on them, which make them brittle in the presence of change and difficult to manage. Even when done correctly, different implementations of these services lead to management complexity when the applications are deployed.

Learn more about ZooKeeper on the ZooKeeper Wiki.

Getting Started<br>
Start by installing ZooKeeper on a single machine or a very small cluster.

Learn about ZooKeeper by reading the documentation.<br>
Download ZooKeeper from the release page.
Getting Involved
Apache ZooKeeper is an open source volunteer project under the Apache Software Foundation. We encourage you to learn about the project and contribute your expertise. Here are some starter links:

See our How to Contribute to ZooKeeper page.<br>
Give us feedback: What can we do better?
Join the mailing list: Meet the community.







