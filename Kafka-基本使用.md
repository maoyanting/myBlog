---
title: Kafka-基本使用
date: 2018-01-08 11:27:46
categories: 
- 分布式
tags:
- Kafka
---

## 摘要

Kafka最简单的使用

<!--more-->

#### Download the code

下载解压缩并打开

`cd kafka_2.11-1.0.0`

#### Start the server

安装使用zookeeper，可使用自带的或者自己编辑的

`bin/zookeeper-server-start.sh config/zookeeper.properties`

然后启动Kafka

`bin/kafka-server-start.sh config/server.properties`

#### Create a topic

新建一个Topic，名字叫"test",只有一个分区和一个备份。

```
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```

查看存在的Topic列表

`bin/kafka-topics.sh --list --zookeeper localhost:2181`

除了手动创建topics，你还可以配置brokers，在一个不存在的topic发表的时候自动创建对应的topic。

####  Send some messages

Kafka提供了一个命令行的工具，可以从输入文件或者命令行中读取消息并发送给Kafka集群。每一行是一条消息。

```
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
This is a message
This is another message
```

#### Start a consumer

Kafka也提供了一个消费消息的命令行工具。

```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
This is a message
This is another message
```

如果你上面的命令运行在两个不同的终端，那么你现在应该能够进入消息生产者终端，并看到他们出现在消费者终端。

这些命令行工具有很多的选项，你可以查看他们的文档来了解更多的功能。

#### Setting up a multi-broker cluster

目前我们一直运行在一个broker上，不好玩。

对于Kafka来说，一个 broker只是表示cluster的大小为1，多几个broker并不会有什么变化。

让我们扩大集群为三个节点(仍然运行在我们的本地机器上)。

首先为每个broker创建一个配置文件。

```
cp config/server.properties config/server-1.properties
cp config/server.properties config/server-2.properties
```

现在编辑这些新的文件，设置以下属性:

```
config/server-1.properties:
    broker.id=1
    port=9093
    log.dir=/tmp/kafka-logs-1
 
config/server-2.properties:
    broker.id=2
    port=9094
    log.dir=/tmp/kafka-logs-2
```

broker.id属性别重样，这个也是集群中每个节点的唯一不可修改的名字。

为了在一台机器上启动两个broker，改了一下它们的port的。
Zookeeper还在，上面用的broker还活着。 来启动这两个broker.

```
> bin/kafka-server-start.sh config/server-1.properties &
...
> bin/kafka-server-start.sh config/server-2.properties &
...
```

创建一个新的topic，把备份设置为3:

```
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic my-replicated-topic
```

成功了，但是我们怎么知道这个broker能做什么呢？运行 "describe topics" 命令看看：

```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic	PartitionCount:1	ReplicationFactor:3	Configs:
	Topic: my-replicated-topic	Partition: 0	Leader: 1	Replicas: 1,2,0	Isr: 1,2,0
```

总结一下：第一行给出了partitions的汇总信息。剩下每一行显示一个partition的信息。因为我们只有一个partition，这个 topic只有一行。

来看看一开始创建的节点:

- "leader"node负责所有的读和写的partitions。每个node将负责一个随机分配的partition。
- "replicas" 是node的列表。复制了partition的日志，不管node是死是活，是否是leader，包含了所有的node。
- "isr" 是工作中的复制节点的集合，也就是活的节点的集合。

在我们的例子中，node1是我们topic唯一partition的leader。

我们可以在之前的topic运行相同的命令：

```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic test
Topic:test	PartitionCount:1	ReplicationFactor:1	Configs:
	Topic: test	Partition: 0	Leader: 0	Replicas: 0	Isr: 0
```

很明显的是，我们之前的topic没有replicas，运行在我们创建它时生成的唯一一个服务器上。

我们用新的topic发布一点消息

```
> bin/kafka-console-producer.sh --broker-list localhost:9092 --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

然后消费它：

```
> bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

测试一下容错.。杀掉leader，也就是Broker 1:

```
> ps | grep server-1.properties
7564 ttys002    0:15.91 /System/Library/Frameworks/JavaVM.framework/Versions/1.6/Home/bin/java...
> kill -9 7564
```

Leader被切换到一个follower节点，node 1 不会被列在isr中了，因为它死了:

```
> bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic my-replicated-topic
Topic:my-replicated-topic	PartitionCount:1	ReplicationFactor:3	Configs:
	Topic: my-replicated-topic	Partition: 0	Leader: 2	Replicas: 1,2,0	Isr: 2,0
```

但是消息还是可以消费的：

(But the messages are still available for consumption even though the leader that took the writes originally is down:)

```
> bin/kafka-console-consumer.sh --zookeeper localhost:2181 --from-beginning --topic my-replicated-topic
...
my test message 1
my test message 2
^C
```

#### Use Kafka Connect to import/export data

从控制台写入数据和写回控制台是一个简单的操作，但是你可能想要使用从别的地方来的数据或从 Kafka导出数据到其他系统。对于许多系统，相对于编写自定义集成代码而言，你还可以使用 Kafka Connect导入或导出数据。

Kafka Connect是一种包含在Kafka内，用于导入和导出数据的工具。它是一个可扩展的工具，运行连接器，实现与外部系统交互的自定义逻辑。接下来我们将看到如何运行Kafka Connect，导入的数据文件到Kafka的topic和导出数据到一个文件。

首先,我们将首先创建一些数据用于测试:

```
> echo -e "foo\nbar" > test.txt
```

接下来，我们将两个connector以独立模式运行，这意味着它们的运行过程是单一的，本地的，专用的。

我们提供三个配置文件作为参数。

第一个总是Kafka Connect过程的配置文件，包含常见的配置如Kafka brokers连接和数据的序列化格式。

剩下的每个配置文件指定一个connector创建。这些文件包括一个独特的connector名称，connector类实例化和任何connector要求的其他配置。

```
> bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

这些Kafka的示例配置文件，使用默认的本地集群配置（你之前使用过），创建两个connector：

第一种是source connector，从输入文件中依行读取，并存入Kafka topic中。

第二个是sink connector，从Kafka的topic读取消息，并将每一行输出到文件。

在启动期间，你将看到大量的日志消息，包括一些消息显示connectors被实例化。一旦Kafka Connect过程已经开始，source connector应该开始读取test.txt并放入的名为connect-test的topic，sink connector应该从名为connect-test的topic开始阅读消息，并写入test.sink.txt。

我们可以验证数据已经通过整个pipeline用以检查输出文件的内容：

```
> more test.sink.txt
foo
bar
```

数据被存储在 Kafka的connect-test topic，所以我们也可以运行一个消费者控制台，看到topic中的数据(或消费者使用自定义代码来处理):

```
> bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic connect-test --from-beginning
{"schema":{"type":"string","optional":false},"payload":"foo"}
{"schema":{"type":"string","optional":false},"payload":"bar"}
...
```

connectors继续处理数据，所以我们可以将数据添加到文件，并看到它穿过pipeline：

```
> echo Another line>> test.txt
```

您应该看到线出现在控制台消费者输出和文件中。

#### Use Kafka Streams to process data

Kafka Streams is a client library for building mission-critical real-time applications and microservices, where the input and/or output data is stored in Kafka clusters. Kafka Streams combines the simplicity of writing and deploying standard Java and Scala applications on the client side with the benefits of Kafka's server-side cluster technology to make these applications highly scalable, elastic, fault-tolerant, distributed, and much more. This quickstart will demonstrate how to run a streaming application coded in this library.

Kafka Streams 是一个客户端库，用于构建实时任务关键型应用程序，microservices输入和/或输出的数据存储在Kafka clusters。Kafka Streams结合简单的编写和部署标准Java和Scala应用程序在客户端与卡夫卡的服务器集群技术，使这些应用程序高度可伸缩的，弹性、容错、分布式、等等。本[快速入门](https://kafka.apache.org/10/documentation/streams/quickstart)将演示如何运行一个流媒体应用程序编码在这个library。