---
title: 翻译-Spring for Apache Kafka
date: 2018-01-08 12:27:46
categories: 
- 翻译
tags:
- Kafka
- Spring
---

翻译——Spring for Apache Kafka官方文档

<!--more-->

## 作者

Gary Russell Artem Bilan Biju Kunjummen

2.1.0.RELEASE

版权所有 © 2016-2017 Pivotal Software Inc.

 本文档的副本可以为您自己使用并分发给其他人，前提是您不收取这些副本的任何费用，并进一步规定每份副本均包含此版权声明，无论是以印刷版还是电子版分发。 

------

**Table of Contents**

[TOC]


# 1.前言

Spring for Apache Kafka将核心Spring概念应用到基于Kafka的消息传递解决方案的开发中。 我们提供一个“template”作为发送消息的高级抽象。 我们还提供对消息驱动的POJO的支持。

# 2. 更新了什么?

## 2.1 从 2.0到2.1的更新

### 2.1.1 Kafka 客户端版本

本版本需要 `kafka-clients` 1.0.0以上

### 2.1.2 JSON 改进

The `StringJsonMessageConverter` and `JsonSerializer` now add type information in `Headers`, allowing the converter and `JsonDeserializer` to create specific types on reception, based on the message itself rather than a fixed configured type.See [Section 4.1.4, “Serialization/Deserialization and Message Conversion”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#serdes) for more information.

### 2.1.3 Container Stopping Error Handlers

Container Error handlers are now provided for both record and batch listeners that treat any exceptions thrown by the listener as fatal; they stop the container.See [Section 4.1.7, “Handling Exceptions”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#annotation-error-handling) for more information.

# 3. 介绍

参考文档的第一部分是对Spring for Apache Kafka的高度概述以及底层概念和一些代码片段，旨在让你快速开始使用Kafka。

## 3.1 快速开始（如果你是个很不耐烦的人的话）

### 3.1.1 介绍

我们首先花费5分钟的时间来学习开始使用Spring Kafka.

事先准备：安装并运行Apache Kafka，然后导入spring-kafka JAR 和所有依赖

最简单的方法是添加一个依赖进你的构建工具，比如Maven：

```xml
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
  <version>2.1.0.RELEASE</version>
</dependency>
```

比如Gradle:

```
compile 'org.springframework.kafka:spring-kafka:2.1.0.RELEASE'
```

#### 兼容性

- Apache Kafka Clients 1.0.0
- Spring Framework 5.0.x
- Minimum Java version: 8

#### 使用纯Java配置

使用纯Java来发送和接收消息:

```java
@Test
public void testAutoCommit() throws Exception {
    logger.info("Start auto");
    ContainerProperties containerProps = new ContainerProperties("topic1", "topic2");
    final CountDownLatch latch = new CountDownLatch(4);
    containerProps.setMessageListener(new MessageListener<Integer, String>() {

        @Override
        public void onMessage(ConsumerRecord<Integer, String> message) {
            logger.info("received: " + message);
            latch.countDown();
        }

    });
    KafkaMessageListenerContainer<Integer, String> container = createContainer(containerProps);
    container.setBeanName("testAuto");
    container.start();
    Thread.sleep(1000); // wait a bit for the container to start
    KafkaTemplate<Integer, String> template = createTemplate();
    template.setDefaultTopic(topic1);
    template.sendDefault(0, "foo");
    template.sendDefault(2, "bar");
    template.sendDefault(0, "baz");
    template.sendDefault(2, "qux");
    template.flush();
    assertTrue(latch.await(60, TimeUnit.SECONDS));
    container.stop();
    logger.info("Stop auto");

}
```

```java
private KafkaMessageListenerContainer<Integer, String> createContainer(
                        ContainerProperties containerProps) {
    Map<String, Object> props = consumerProps();
    DefaultKafkaConsumerFactory<Integer, String> cf =
                            new DefaultKafkaConsumerFactory<Integer, String>(props);
    KafkaMessageListenerContainer<Integer, String> container =
                            new KafkaMessageListenerContainer<>(cf, containerProps);
    return container;
}

private KafkaTemplate<Integer, String> createTemplate() {
    Map<String, Object> senderProps = senderProps();
    ProducerFactory<Integer, String> pf =
              new DefaultKafkaProducerFactory<Integer, String>(senderProps);
    KafkaTemplate<Integer, String> template = new KafkaTemplate<>(pf);
    return template;
}

private Map<String, Object> consumerProps() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(ConsumerConfig.GROUP_ID_CONFIG, group);
    props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
    props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "100");
    props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, "15000");
    props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class);
    props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
    return props;
}

private Map<String, Object> senderProps() {
    Map<String, Object> props = new HashMap<>();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(ProducerConfig.RETRIES_CONFIG, 0);
    props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);
    props.put(ProducerConfig.LINGER_MS_CONFIG, 1);
    props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, IntegerSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    return props;
}
```

#### 使用Spring配置

使用Spring配置相同的项目

```java
@Autowired
private Listener listener;

@Autowired
private KafkaTemplate<Integer, String> template;

@Test
public void testSimple() throws Exception {
    template.send("annotated1", 0, "foo");
    template.flush();
    assertTrue(this.listener.latch1.await(10, TimeUnit.SECONDS));
}

@Configuration
@EnableKafka
public class Config {

    @Bean
    ConcurrentKafkaListenerContainerFactory<Integer, String>
                        kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<Integer, String> factory =
                                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }

    @Bean
    public ConsumerFactory<Integer, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, embeddedKafka.getBrokersAsString());
        ...
        return props;
    }

    @Bean
    public Listener listener() {
        return new Listener();
    }

    @Bean
    public ProducerFactory<Integer, String> producerFactory() {
        return new DefaultKafkaProducerFactory<>(producerConfigs());
    }

    @Bean
    public Map<String, Object> producerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, embeddedKafka.getBrokersAsString());
        ...
        return props;
    }

    @Bean
    public KafkaTemplate<Integer, String> kafkaTemplate() {
        return new KafkaTemplate<Integer, String>(producerFactory());
    }

}
```

```java
public class Listener {

    private final CountDownLatch latch1 = new CountDownLatch(1);

    @KafkaListener(id = "foo", topics = "annotated1")
    public void listen1(String foo) {
        this.latch1.countDown();
    }

}
```

#### 使用Spring Boot，我们甚至可以更快

下面的Spring Boot项目发送了3个消息给Topic，然后接收它们，再停止运行。

**Application. **

```java
@SpringBootApplication
public class Application implements CommandLineRunner {

    public static Logger logger = LoggerFactory.getLogger(Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args).close();
    }

    @Autowired
    private KafkaTemplate<String, String> template;

    private final CountDownLatch latch = new CountDownLatch(3);

    @Override
    public void run(String... args) throws Exception {
        this.template.send("myTopic", "foo1");
        this.template.send("myTopic", "foo2");
        this.template.send("myTopic", "foo3");
        latch.await(60, TimeUnit.SECONDS);
        logger.info("All received");
    }

    @KafkaListener(topics = "myTopic")
    public void listen(ConsumerRecord<?, ?> cr) throws Exception {
        logger.info(cr.toString());
        latch.countDown();
    }

}
```

Spring Boot负责了绝大部分的配置；当我们需要使用一个本地的broker的时候，我们只需要配置：

**application.properties. **

```
spring.kafka.consumer.group-id=foo
spring.kafka.consumer.auto-offset-reset=earliest
```

第一行是因为我们正在使用group management去分配topic partitions给consumers，所以我们需要一个group。

第二行是为了保证新的 consumer group能接收到我们刚刚发送的messages，因为container可能在我们完成发送messages的任务后开启。

# 4. 参考

参考文档的这一部分详细介绍了构成Spring for Apache Kafka的各种组件。 [主要章节](https://docs.spring.io/spring-kafka/reference/htmlsingle/#kafka) 涵盖了开发Spring for Apache Kafka应用程序的核心类。

## 4.1 使用 Spring for Apache Kafka

### 4.1.1 配置 Topics

如果你在你的`application context`中定义了 `KafkaAdmin` ，它会自动把topics添加到broker。

你只需要简单地在`application context`给每一个topic添加 `NewTopic` `@Bean` 。

```
@Bean
public KafkaAdmin admin() {
    Map<String, Object> configs = new HashMap<>();
    configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG,
            StringUtils.arrayToCommaDelimitedString(kafkaEmbedded().getBrokerAddresses()));
    return new KafkaAdmin(configs);
}

@Bean
public NewTopic topic1() {
    return new NewTopic("foo", 10, (short) 2);
}

@Bean
public NewTopic topic2() {
    return new NewTopic("bar", 10, (short) 2);
}
```

默认情况下,如果broker 不可用，message将被记录,但上下文将继续加载（but the context will continue to load）。您可以调用admin的 `initialize()`方法重试。如果你希望这种情况被认为是致命的,设置管理员的`fatalIfBrokerNotAvailable`属性为true，上下文将无法初始化。

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| 如果broker支持（1.0.0 或者更高的版本），在发现现有的topic比`NewTopic.numPartitions`的partitions少的时候，admin会增加partitions的数量。 |

对于更高级的功能,如分配分区的副本（assigning partitions to replicas），您可以直接使用`AdminClient`：

```
@Autowired
private KafkaAdmin admin;

...

    AdminClient client = AdminClient.create(admin.getConfig());
    ...
    client.close();
```

### 4.1.2 发送 Messages

#### KafkaTemplate

##### 概述

`KafkaTemplate` 封装了一个 producer，并且提供了一个简便方法，用于将数据发送给kafka的topics。

```java
ListenableFuture<SendResult<K, V>> sendDefault(V data);

ListenableFuture<SendResult<K, V>> sendDefault(K key, V data);

ListenableFuture<SendResult<K, V>> sendDefault(Integer partition, K key, V data);

ListenableFuture<SendResult<K, V>> sendDefault(Integer partition, Long timestamp, K key, V data);

ListenableFuture<SendResult<K, V>> send(String topic, V data);

ListenableFuture<SendResult<K, V>> send(String topic, K key, V data);

ListenableFuture<SendResult<K, V>> send(String topic, Integer partition, K key, V data);

ListenableFuture<SendResult<K, V>> send(String topic, Integer partition, Long timestamp, K key, V data);

ListenableFuture<SendResult<K, V>> send(ProducerRecord<K, V> record);

ListenableFuture<SendResult<K, V>> send(Message<?> message);

Map<MetricName, ? extends Metric> metrics();

List<PartitionInfo> partitionsFor(String topic);

<T> T execute(ProducerCallback<K, V, T> callback);

// Flush the producer.

void flush();

interface ProducerCallback<K, V, T> {

    T doInKafka(Producer<K, V> producer);

}
```

 `sendDefault` 方法需要向template提供默认topic。

API将时间作为参数并储存在记录中。用户提供的时间戳是否会被储存，取决于在Kfaka topic的时间戳类型配置。

如果主题是配置为使用CREATE_TIME，那么用户指定的时间戳如果未指定，也会被记录和生成。（the user specified timestamp will be recorded or generated if not specified）

如果主题是配置为使用LOG_APPEND_TIME，那么用户指定的时间戳将被忽略，broker将添加broker的时间。



`metrics` 和 `partitionsFor` 方法只是代表相同的底层`Producer`上的方法 

 `execute` 方法提供了访问底层 `Producer`的路径。

//这里看不懂啊，啥意思？

（The `metrics` and `partitionsFor` methods simply delegate to the same methods on the underlying [`Producer`](https://kafka.apache.org/0101/javadoc/org/apache/kafka/clients/producer/Producer.html).The `execute` method provides direct access to the underlying [`Producer`](https://kafka.apache.org/0101/javadoc/org/apache/kafka/clients/producer/Producer.html).）

为了使用 template，首先配置一个生产工厂，并在 template的构造函数里面使用：

```java
@Bean
public ProducerFactory<Integer, String> producerFactory() {
    return new DefaultKafkaProducerFactory<>(producerConfigs());
}

@Bean
public Map<String, Object> producerConfigs() {
    Map<String, Object> props = new HashMap<>();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
    // See https://kafka.apache.org/documentation/#producerconfigs for more properties
    return props;
}

@Bean
public KafkaTemplate<Integer, String> kafkaTemplate() {
    return new KafkaTemplate<Integer, String>(producerFactory());
}
```

template也能通过使用 `<bean/>` 定义。

使用template只需要调用它其中一个方法

当我们使用含有 `Message<?>`参数的方法时，messageheader会提供topic, partition和 key的相关信息：

- `KafkaHeaders.TOPIC`
- `KafkaHeaders.PARTITION_ID`
- `KafkaHeaders.MESSAGE_KEY`
- `KafkaHeaders.TIMESTAMP`

message payload 就是数据

 或者，您可以使用`ProducerListener`配置`KafkaTemplate` ，以获得发送结果（成功或失败）的异步回调，而不是等待`Future`完成。 

```
public interface ProducerListener<K, V> {

    void onSuccess(String topic, Integer partition, K key, V value, RecordMetadata recordMetadata);

    void onError(String topic, Integer partition, K key, V value, Exception exception);

    boolean isInterestedInSuccess();

}
```

默认情况下，template配置了一个 `LoggingProducerListener` ，用于记录错误，当发送成功的时候不做任何事情。

`onSuccess` 只有在 `isInterestedInSuccess` 返回 `true`的时候调用。

为了方便起见，如果你只想实现其中的一个方法，我们还提供了抽象的 `ProducerListenerAdapter` 。 `isInterestedInSuccess`此时返回为 `false` 。

send方法会返回一个 `ListenableFuture<SendResult>`，你可以向侦听器注册回调，异步接收发送的结果。

```
ListenableFuture<SendResult<Integer, String>> future = template.send("foo");
future.addCallback(new ListenableFutureCallback<SendResult<Integer, String>>() {

    @Override
    public void onSuccess(SendResult<Integer, String> result) {
        ...
    }

    @Override
    public void onFailure(Throwable ex) {
        ...
    }

});
```

 `SendResult` 有两个属性，`ProducerRecord`和` RecordMetadata`；请参考Kafka API文档查询关于这些对象的信息。

如果你想阻塞发送的线程，等待结果，您可以调用`future`的`get()`方法。

您可能希望在等待之前调用`flush()`，或者说为了方便，template有一个构造函数，参数为`autoFlush`，将导致template每次send的时候都调用 `flush()`方法 。但是需要注意的是， flushing可能会显著降低性能。

（If you wish to block the sending thread, to await the result, you can invoke the future’s `get()` method.

You may wish to invoke `flush()` before waiting or, for convenience, the template has a constructor with an `autoFlush`parameter which will cause the template to `flush()` on each send.

Note, however that flushing will likely significantly reduce performance.）

##### 例子

**非阻塞 (Async异步). **

```java
public void sendToKafka(final MyOutputData data) {
    final ProducerRecord<String, String> record = createRecord(data);

    ListenableFuture<SendResult<Integer, String>> future = template.send(record);
    future.addCallback(new ListenableFutureCallback<SendResult<Integer, String>>() {

        @Override
        public void onSuccess(SendResult<Integer, String> result) {
            handleSuccess(data);
        }

        @Override
        public void onFailure(Throwable ex) {
            handleFailure(data, record, ex);
        }

    });
}
```

**阻塞 (同步). **

```java
public void sendToKafka(final MyOutputData data) {
    final ProducerRecord<String, String> record = createRecord(data);

    try {
        template.send(record).get(10, TimeUnit.SECONDS);
        handleSuccess(data);
    }
    catch (ExecutionException e) {
        handleFailure(data, record, e.getCause());
    }
    catch (TimeoutException | InterruptedException e) {
        handleFailure(data, record, e);
    }
}
```

#### 事务

0.11.0.0客户端库添加事务的支持。Spring for Apache Kafka增加了在几个方面的支持。

- `KafkaTransactionManager` - used with normal Spring transaction support (`@Transactional`, `TransactionTemplate` etc).
- Transactional `KafkaMessageListenerContainer`
- Local transactions with `KafkaTemplate`

Transactions are enabled by providing the `DefaultKafkaProducerFactory` with a `transactionIdPrefix`.In that case, instead of managing a single shared `Producer`, the factory maintains a cache of transactional producers.When the user `close()` s a producer, it is returned to the cache for reuse instead of actually being closed.The `transactional.id` property of each producer is `transactionIdPrefix` + `n`, where `n` starts with `0` and is incremented for each new producer.

##### KafkaTransactionManager

The `KafkaTransactionManager` is an implementation of Spring Framework’s `PlatformTransactionManager`; it is provided with a reference to the producer factory in its constructor.If you provide a custom producer factory, it must support transactions - see `ProducerFactory.transactionCapable()`.

You can use the `KafkaTransactionManager` with normal Spring transaction support (`@Transactional`, `TransactionTemplate` etc).If a transaction is active, any `KafkaTemplate` operations performed within the scope of the transaction will use the transaction’s `Producer`.The manager will commit or rollback the transaction depending on success or failure.The `KafkaTemplate` must be configured to use the same `ProducerFactory` as the transaction manager.

##### Transactional Listener Container

You can provide a listener container with a `KafkaTransactionManager` instance; when so configured, the container will start a transaction before invoking the listener.If the listener successfully processes the record (or records when using a `BatchMessageListener`), the container will send the offset(s) to the transaction using `producer.sendOffsetsToTransaction()`), before the transaction manager commits the transaction.If the listener throws an exception, the transaction is rolled back and the consumer is repositioned so that the rolled-back records will be retrieved on the next poll.

##### Transaction Synchronization

If you need to synchronize a Kafka transaction with some other transaction; simply configure the listener container with the appropriate transaction manager (one that supports synchronization, such as the `DataSourceTransactionManager`).Any operations performed on a **transactional** `KafkaTemplate` from the listener will participate in a single transaction.The Kafka transaction will be committed (or rolled back) immediately after the controlling transaction.Before exiting the listener, you should invoke one of the template’s `sendOffsetsToTransaction` methods.For convenience, the listener container binds its consumer group id to the thread so, generally, you can use the first method:

```
void sendOffsetsToTransaction(Map<TopicPartition, OffsetAndMetadata> offsets);

void sendOffsetsToTransaction(Map<TopicPartition, OffsetAndMetadata> offsets, String consumerGroupId);
```

For example:

```
@Bean
KafkaMessageListenerContainer container(ConsumerFactory<String, String> cf,
            final KafkaTemplate template) {
    ContainerProperties props = new ContainerProperties("foo");
    props.setGroupId("group");
    props.setTransactionManager(new SomeOtherTransactionManager());
    ...
    props.setMessageListener((MessageListener<String, String>) m -> {
        template.send("foo", "bar");
        template.send("baz", "qux");
        template.sendOffsetsToTransaction(
            Collections.singletonMap(new TopicPartition(m.topic(), m.partition()),
                new OffsetAndMetadata(m.offset() + 1)));
    });
    return new KafkaMessageListenerContainer<>(cf, props);
}
```

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| The offset to be committed is one greater than the offset of the record(s) processed by the listener. |

| ![[Important]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/important.png) | Important |
| ---------------------------------------- | --------- |
| This should only be called when using transaction synchronization.When a listener container is configured to use a `KafkaTransactionManager`, it will take care of sending the offsets to the transaction. |           |

##### KafkaTemplate Local Transactions

You can use the `KafkaTemplate` to execute a series of operations within a local transaction.

```
boolean result = template.executeInTransaction(t -> {
    t.sendDefault("foo", "bar");
    t.sendDefault("baz", "qux");
    return true;
});
```

The argument in the callback is the template itself (`this`).If the callback exits normally, the transaction is committed; if an exception is thrown, the transaction is rolled-back.

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| If there is a `KafkaTransactionManager` (or synchronized) transaction in process, it will not be used; a new "nested" transaction is used. |

### 4.1.3 接收 Messages

通过配置 `MessageListenerContainer` 并提供一个Message Listener，或者使用`@KafkaListener` 注释。

#### Message Listeners

使用 [Message Listener Container](https://docs.spring.io/spring-kafka/reference/htmlsingle/#message-listener-container) 的时候你必须提供一个 listener用于接收数据。目前有八个支持消息监听的的接口：

```java
public interface MessageListener<K, V> { 
    void onMessage(ConsumerRecord<K, V> data);
}

public interface AcknowledgingMessageListener<K, V> { 
    void onMessage(ConsumerRecord<K, V> data, Acknowledgment acknowledgment);
}

public interface ConsumerAwareMessageListener<K, V> extends MessageListener<K, V> { 
    void onMessage(ConsumerRecord<K, V> data, Consumer<?, ?> consumer);
}

public interface AcknowledgingConsumerAwareMessageListener<K, V> extends MessageListener<K, V> { 
    void onMessage(ConsumerRecord<K, V> data, Acknowledgment acknowledgment, Consumer<?, ?> consumer);
}

public interface BatchMessageListener<K, V> { 
    void onMessage(List<ConsumerRecord<K, V>> data);
}

public interface BatchAcknowledgingMessageListener<K, V> { 
    void onMessage(List<ConsumerRecord<K, V>> data, Acknowledgment acknowledgment);
}

public interface BatchConsumerAwareMessageListener<K, V> extends BatchMessageListener<K, V> { 
    void onMessage(List<ConsumerRecord<K, V>> data, Consumer<?, ?> consumer);
}

public interface BatchAcknowledgingConsumerAwareMessageListener<K, V> extends BatchMessageListener<K, V> { 
    void onMessage(List<ConsumerRecord<K, V>> data, Acknowledgment acknowledgment, Consumer<?, ?> consumer);
}
```

1 、使用自动提交，或者容器管理的[提交方法](https://docs.spring.io/spring-kafka/reference/htmlsingle/#committing-offsets)，处理从kafka consumer`poll()`操作中收到的个人`ConsumerRecord`。

2、使用手动 [提交的方法](https://docs.spring.io/spring-kafka/reference/htmlsingle/#committing-offsets)，处理从kafka consumer`poll()`操作中收到的个人`ConsumerRecord`。

3、使用自动提交，或者容器管理的 [提交方法](https://docs.spring.io/spring-kafka/reference/htmlsingle/#committing-offsets)，处理从kafka consumer`poll()`操作中收到的个人`ConsumerRecord`。

需要提供对 `Consumer` 对象的访问。

4、使用一种手动 [提交方法](https://docs.spring.io/spring-kafka/reference/htmlsingle/#committing-offsets)，处理从kafka consumer`poll()`操作中收到的个人`ConsumerRecord`。

需要提供对 `Consumer` 对象的访问。

5、使用自动提交，或者容器管理的 [提交方法](https://docs.spring.io/spring-kafka/reference/htmlsingle/#committing-offsets)，处理从kafka consumer`poll()`操作中收到的所有个人`ConsumerRecord`。

在使用此接口时，不支持`AckMode.RECORD` ，因为侦听器被提供了完整的批处理。

6、使用一种手动 [提交方法](https://docs.spring.io/spring-kafka/reference/htmlsingle/#committing-offsets)，处理从kafka consumer`poll()`操作中收到的所有个人`ConsumerRecord`。

7、使用自动提交，或者容器管理的 [提交方法](https://docs.spring.io/spring-kafka/reference/htmlsingle/#committing-offsets)，处理从kafka consumer`poll()`操作中收到的所有个人`ConsumerRecord`。

在使用此接口时，不支持`AckMode.RECORD` ，因为侦听器被提供了完整的批处理。

需要提供对 `Consumer` 对象的访问。

8、使用一种手动 [提交方法](https://docs.spring.io/spring-kafka/reference/htmlsingle/#committing-offsets)，处理从kafka consumer`poll()`操作中收到的所有个人`ConsumerRecord`。

需要提供对 `Consumer` 对象的访问。



​    ![[Important]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/important.png)**Important**

`Consumer` 对象不是线程安全的；你必须用侦听器的线程调用它的方法。

#### Message Listener Containers

这里我们提供了两个 `MessageListenerContainer` 的实现：

- `KafkaMessageListenerContainer`
- `ConcurrentMessageListenerContainer`

 `KafkaMessageListenerContainer` 从单个线程上的所有topics/partitions接收所有消息。

 `ConcurrentMessageListenerContainer` 委托一个或者更多`KafkaMessageListenerContainer` 以提供多线程消费。

##### KafkaMessageListenerContainer

下述的构造函数是可用的。

```
public KafkaMessageListenerContainer(ConsumerFactory<K, V> consumerFactory,
                    ContainerProperties containerProperties)

public KafkaMessageListenerContainer(ConsumerFactory<K, V> consumerFactory,
                    ContainerProperties containerProperties,
                    TopicPartitionInitialOffset... topicPartitions)
```

每个构造函数都需要`ConsumerFactory`，topics和 partitions的信息，以及`ContainerPropertiesobject`的其他配置。

`ConcurrentMessageListenerContainer`(见下文)使用第二个构造函数，用于消费者实例分发`TopicPartitionInitialOffset` 。

（The second constructor is used by the `ConcurrentMessageListenerContainer` (see below) to distribute `TopicPartitionInitialOffset` across the consumer instances）

`ContainerProperties` 构造函数如下:

```
public ContainerProperties(TopicPartitionInitialOffset... topicPartitions)

public ContainerProperties(String... topics)

public ContainerProperties(Pattern topicPattern)
```

The first takes an array of `TopicPartitionInitialOffset` arguments to explicitly instruct the container which partitions to use(using the consumer `assign()` method), and with an optional initial offset: a positive value is an absolute offset by default; a negative value is relative to the current last offset within a partition by default.A constructor for `TopicPartitionInitialOffset` is provided that takes an additional `boolean` argument.If this is `true`, the initial offsets (positive or negative) are relative to the current position for this consumer.The offsets are applied when the container is started.The second takes an array of topics and Kafka allocates the partitions based on the `group.id` property - distributingpartitions across the group.The third uses a regex `Pattern` to select the topics.

To assign a `MessageListener` to a container, use the `ContainerProps.setMessageListener` method when creating the Container:

```
ContainerProperties containerProps = new ContainerProperties("topic1", "topic2");
containerProps.setMessageListener(new MessageListener<Integer, String>() {
    ...
});
DefaultKafkaConsumerFactory<Integer, String> cf =
                        new DefaultKafkaConsumerFactory<Integer, String>(consumerProps());
KafkaMessageListenerContainer<Integer, String> container =
                        new KafkaMessageListenerContainer<>(cf, containerProps);
return container;
```

Refer to the JavaDocs for `ContainerProperties` for more information about the various properties that can be set.

##### ConcurrentMessageListenerContainer

The single constructor is similar to the first `KafkaListenerContainer` constructor:

```
public ConcurrentMessageListenerContainer(ConsumerFactory<K, V> consumerFactory,
                            ContainerProperties containerProperties)
```

It also has a property `concurrency`, e.g. `container.setConcurrency(3)` will create 3 `KafkaMessageListenerContainer` s.

For the first constructor, kafka will distribute the partitions across the consumers.For the second constructor, the `ConcurrentMessageListenerContainer` distributes the `TopicPartition` s across thedelegate `KafkaMessageListenerContainer` s.

If, say, 6 `TopicPartition` s are provided and the `concurrency` is 3; each container will get 2 partitions.For 5 `TopicPartition` s, 2 containers will get 2 partitions and the third will get 1.If the `concurrency` is greater than the number of `TopicPartitions`, the `concurrency` will be adjusted down such thateach container will get one partition.

![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png)The `client.id` property (if set) will be appended with `-n` where `n` is the consumer instance according to the concurrency.This is required to provide unique names for MBeans when JMX is enabled.

Starting with *version 1.3*, the `MessageListenerContainer` provides an access to the metrics of the underlying `KafkaConsumer`.In case of `ConcurrentMessageListenerContainer` the `metrics()` method returns the metrics for all the target `KafkaMessageListenerContainer` instances.The metrics are grouped into the `Map<MetricName, ? extends Metric>` by the `client-id` provided for the underlying `KafkaConsumer`.

##### 提交 Offsets

我们还为提交offsets提供了几个选项。如果consumer的 `enable.auto.commit` 属性为true， kafka将会根据其配置自动提交offsets。如果是false，容器将支持以下 `AckMode` 。

consumer的 `poll()` 方法将会返回一个或多个 `ConsumerRecords`；每次记录都会调用`MessageListener` ；以下描述了容器为每个 `AckMode`所采取的操作：

- RECORD - 在处理记录后，当侦听器返回时提交 offset。(//每处理一条commit一次)
- BATCH - 当 `poll()` 返回的所有记录都已处理后，提交offset。(//每次poll的时候批量提交一次，频率取决于每次poll的调用频率)
- TIME - 当`poll()`返回的所有记录已经被处理，只要自上一次提交以来的`ackTime`已经被处理，就提交offset。 
- COUNT - 当`poll()`返回的所有记录已经被处理，只要自上一次提交以来已经收到`ackCount`记录就提交offset。 
- COUNT_TIME - 类似于TIME和COUNT，但是如果任一condition为true，则执行提交。
- MANUAL - 消息监听器负责 `Acknowledgment`的 `acknowledge()` 确认；之后，应用与BATCH相同的语义
- MANUAL_IMMEDIATE - 当侦听器调用`Acknowledgement.acknowledge()`方法时立即提交offset。



![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png)`MANUAL`,和 `MANUAL_IMMEDIATE` 要求监听器是`AcknowledgingMessageListener` 或者`BatchAcknowledgingMessageListener`；查看 [Message Listeners](https://docs.spring.io/spring-kafka/reference/htmlsingle/#message-listeners).



使用消费者的`commitSync()`或`commitAsync()`方法，具体取决于`syncCommits`容器属性。

 `Acknowledgment` 有以下方法：

```
public interface Acknowledgment {

    void acknowledge();

}
```

这使侦听器能够控制何时提交offsets。

##### Listener Container 自动启动

 Listener containers实现 `SmartLifecycle` 接口，其中， `autoStartup` 默认为true；容器在后期开始 (`Integer.MAX-VALUE - 100`)。其他实现`SmartLifecycle`接口的和处理来自监听器的数据的组件，应该在早期阶段启动。 `- 100` 留出空间用于以后的阶段，以便组件在容器之后自动启动。

#### @KafkaListener 注释

`@KafkaListener`注释提供了一个简单的POJO监听器机制:

```
public class Listener {

    @KafkaListener(id = "foo", topics = "myTopic")
    public void listen(String data) {
        ...
    }

}
```

This mechanism requires an `@EnableKafka` annotation on one of your `@Configuration` classes and a listener container factory, which is used to configure the underlying`ConcurrentMessageListenerContainer`: by default, a bean with name `kafkaListenerContainerFactory` is expected.

```
@Configuration
@EnableKafka
public class KafkaConfig {

    @Bean
    KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<Integer, String>>
                        kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<Integer, String> factory =
                                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3);
        factory.getContainerProperties().setPollTimeout(3000);
        return factory;
    }

    @Bean
    public ConsumerFactory<Integer, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
    }

    @Bean
    public Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, embeddedKafka.getBrokersAsString());
        ...
        return props;
    }
}
```

Notice that to set container properties, you must use the `getContainerProperties()` method on the factory.It is used as a template for the actual properties injected into the container.

You can also configure POJO listeners with explicit topics and partitions (and, optionally, their initial offsets):

```
@KafkaListener(id = "bar", topicPartitions =
        { @TopicPartition(topic = "topic1", partitions = { "0", "1" }),
          @TopicPartition(topic = "topic2", partitions = "0",
             partitionOffsets = @PartitionOffset(partition = "1", initialOffset = "100"))
        })
public void listen(ConsumerRecord<?, ?> record) {
    ...
}
```

Each partition can be specified in the `partitions` or `partitionOffsets` attribute, but not both.

When using manual `AckMode`, the listener can also be provided with the `Acknowledgment`; this example also showshow to use a different container factory.

```
@KafkaListener(id = "baz", topics = "myTopic",
          containerFactory = "kafkaManualAckListenerContainerFactory")
public void listen(String data, Acknowledgment ack) {
    ...
    ack.acknowledge();
}
```

Finally, metadata about the message is available from message headers, the following header names can be used for retrieving the headers of the message:

- `KafkaHeaders.RECEIVED_MESSAGE_KEY`
- `KafkaHeaders.RECEIVED_TOPIC`
- `KafkaHeaders.RECEIVED_PARTITION_ID`
- `KafkaHeaders.RECEIVED_TIMESTAMP`
- `KafkaHeaders.TIMESTAMP_TYPE`

```
@KafkaListener(id = "qux", topicPattern = "myTopic1")
public void listen(@Payload String foo,
        @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) Integer key,
        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition,
        @Header(KafkaHeaders.RECEIVED_TOPIC) String topic,
        @Header(KafkaHeaders.RECEIVED_TIMESTAMP) long ts
        ) {
    ...
}
```

Starting with *version 1.1*, `@KafkaListener` methods can be configured to receive the entire batch of consumer records received from the consumer poll.To configure the listener container factory to create batch listeners, set the `batchListener` property:

```
@Bean
public KafkaListenerContainerFactory<?> batchFactory() {
    ConcurrentKafkaListenerContainerFactory<Integer, String> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    factory.setBatchListener(true);  // <<<<<<<<<<<<<<<<<<<<<<<<<
    return factory;
}
```

To receive a simple list of payloads:

```
@KafkaListener(id = "list", topics = "myTopic", containerFactory = "batchFactory")
public void listen(List<String> list) {
    ...
}
```

The topic, partition, offset etc are available in headers which parallel the payloads:

```
@KafkaListener(id = "list", topics = "myTopic", containerFactory = "batchFactory")
public void listen(List<String> list,
        @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) List<Integer> keys,
        @Header(KafkaHeaders.RECEIVED_PARTITION_ID) List<Integer> partitions,
        @Header(KafkaHeaders.RECEIVED_TOPIC) List<String> topics,
        @Header(KafkaHeaders.OFFSET) List<Long> offsets) {
    ...
}
```

Alternatively you can receive a List of `Message<?>` objects with each offset, etc in each message, but it must be the only parameter (aside from an optional `Acknowledgment` when using manual commits) defined on the method:

```
@KafkaListener(id = "listMsg", topics = "myTopic", containerFactory = "batchFactory")
public void listen14(List<Message<?>> list) {
    ...
}

@KafkaListener(id = "listMsgAck", topics = "myTopic", containerFactory = "batchFactory")
public void listen15(List<Message<?>> list, Acknowledgment ack) {
    ...
}
```

You can also receive a list of `ConsumerRecord<?, ?>` objects but it must be the only parameter (aside from an optional `Acknowledgment` when using manual commits) defined on the method:

```
@KafkaListener(id = "listCRs", topics = "myTopic", containerFactory = "batchFactory")
public void listen(List<ConsumerRecord<Integer, String>> list) {
    ...
}

@KafkaListener(id = "listCRsAck", topics = "myTopic", containerFactory = "batchFactory")
public void listen(List<ConsumerRecord<Integer, String>> list, Acknowledgment ack) {
    ...
}
```

Starting with *version 2.0*, the `id` attribute (if present) is used as the Kafka `group.id` property, overriding the configured property in the consumer factory, if present.You can also set `groupId` explicitly, or set `idIsGroup` to false, to restore the previous behavior of using the consumer factory `group.id`.

#### Container Thread Naming

Listener containers currently use two task executors, one to invoke the consumer and another which will be used to invoke the listener, when the kafka consumer property `enable.auto.commit` is `false`.You can provide custom executors by setting the `consumerExecutor` and `listenerExecutor` properties of the container’s `ContainerProperties`.When using pooled executors, be sure that enough threads are available to handle the concurrency across all the containers in which they are used.When using the `ConcurrentMessageListenerContainer`, a thread from each is used for each consumer (`concurrency`).

If you don’t provide a consumer executor, a `SimpleAsyncTaskExecutor` is used; this executor creates threads with names `<beanName>-C-1` (consumer thread).For the `ConcurrentMessageListenerContainer`, the `<beanName>` part of the thread name becomes `<beanName>-m`, where `m` represents the consumer instance.`n` increments each time the container is started.So, with a bean name of `container`, threads in this container will be named `container-0-C-1`, `container-1-C-1` etc., after the container is started the first time; `container-0-C-2`, `container-1-C-2` etc., after a stop/start.

#### @KafkaListener on a class

当在类级别使用`@KafkaListener`，还需要在方法级别指定`@KafkaHandler`。当消息传递后，根据转换后的message payload type，确定要调用哪个方法。

```
@KafkaListener(id = "multi", topics = "myTopic")
static class MultiListenerBean {

    @KafkaHandler
    public void listen(String foo) {
        ...
    }

    @KafkaHandler
    public void listen(Integer bar) {
        ...
    }

}
```

#### @KafkaListener 生命周期管理

用`@KafkaListener`注释创建的侦听器容器不是应用程序上下文中的bean。而是使用`KafkaListenerEndpointRegistry`类型的基础结构bean进行注册。这个bean管理容器的生命周期；它会自动启动任何`autoStartup`设置为true的容器。所有容器工厂创建的所有容器必须处于同一阶段

请参阅“[侦听器容器自动启动](https://docs.spring.io/spring-kafka/reference/htmlsingle/#container-auto-startup) ”一节了解更多信息。

您可以使用注册表以编程方式管理生命周期；启动/停止注册表将启动/停止所有注册的容器。或者，您可以使用其id属性获取对单个容器的引用；您可以在注释中设置`autoStartup`，这将覆盖配置到容器工厂的默认设置。

```
@Autowired
private KafkaListenerEndpointRegistry registry;

...

@KafkaListener(id = "myContainer", topics = "myTopic", autoStartup = "false")
public void listen(...) { ... }

...

    registry.getListenerContainer("myContainer").start();
```

#### Rebalance Listeners

`ContainerProperties`有一个属性`consumerRebalanceListener`，它是Kafka客户端的`ConsumerRebalanceListener`接口的实现。如果未提供此属性，则容器将配置一个简单的日志记录侦听器，用于在`INFO`级别下记录 rebalance events。该框架还添加了一个子接口`ConsumerAwareRebalanceListener`：

```
public interface ConsumerAwareRebalanceListener extends ConsumerRebalanceListener {

    void onPartitionsRevokedBeforeCommit(Consumer<?, ?> consumer, Collection<TopicPartition> partitions);

    void onPartitionsRevokedAfterCommit(Consumer<?, ?> consumer, Collection<TopicPartition> partitions);

    void onPartitionsAssigned(Consumer<?, ?> consumer, Collection<TopicPartition> partitions);

}
```

Notice that there are two callbacks when partitions are revoked: the first is called immediately; the second is called after any pending offsets are committed.This is useful if you wish to maintain offsets in some external repository; for example:

```
containerProperties.setConsumerRebalanceListener(new ConsumerAwareRebalanceListener() {

    @Override
    public void onPartitionsRevokedBeforeCommit(Consumer<?, ?> consumer, Collection<TopicPartition> partitions) {
        // acknowledge any pending Acknowledgments (if using manual acks)
    }

    @Override
    public void onPartitionsRevokedAfterCommit(Consumer<?, ?> consumer, Collection<TopicPartition> partitions) {
        // ...
            store(consumer.position(partition));
        // ...
    }

    @Override
    public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
        // ...
            consumer.seek(partition, offsetTracker.getOffset() + 1);
        // ...
    }
});
```

#### Forwarding Listener Results using @SendTo

从2.0版本开始，如果你使用@KafkaListener和 @SendTo，方法调用返回一个结果，结果将是@SendTo指定的topic。

The `@SendTo` value can have several forms:

- `@SendTo("someTopic")` routes to the literal topic
- `@SendTo("#{someExpression}")` routes to the topic determined by evaluating the expression once during application context initialization.
- `@SendTo("!{someExpression}")` routes to the topic determined by evaluating the expression at runtime.The `#root` object for the evaluation has 3 properties:
- request - the inbound `ConsumerRecord` (or `ConsumerRecords` object for a batch listener))
- source - the `org.springframework.messaging.Message<?>` converted from the `request`.
- result - the method return result.

表达式求值的结果，必须是一个 `String` 代表主题名称。

```java
@KafkaListener(topics = "annotated21")
@SendTo("!{request.value()}") // runtime SpEL
public String replyingListener(String in) {
    ...
}

@KafkaListener(topics = "annotated22")
@SendTo("#{myBean.replyTopic}") // config time SpEL
public Collection<String> replyingBatchListener(List<String> in) {
    ...
}

@KafkaListener(topics = "annotated23", errorHandler = "replyErrorHandler")
@SendTo("annotated23reply") // static reply topic definition
public String replyingListenerWithErrorHandler(String in) {
    ...
}
...
@KafkaListener(topics = "annotated25")
@SendTo("annotated25reply1")
public class MultiListenerSendTo {

    @KafkaHandler
    public String foo(String in) {
        ...
    }

    @KafkaHandler
    @SendTo("!{'annotated25reply2'}")
    public String bar(@Payload(required = false) KafkaNull nul,
            @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) int key) {
        ...
    }

}
```

When using `@SendTo`, the `ConcurrentKafkaListenerContainerFactory` must be configured with a `KafkaTemplate` in its `replyTemplate` property, to perform the send.Note: only the simple `send(topic, value)` method is used, so you may wish to create a subclass to generate the partition and/or key…

```
@Bean
public KafkaTemplate<String, String> myReplyingTemplate() {
    return new KafkaTemplate<Integer, String>(producerFactory()) {

        @Override
        public ListenableFuture<SendResult<String, String>> send(String topic, String data) {
            return super.send(topic, partitionForData(data), keyForData(data), data);
        }

        ...

    };
}
```

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| You can annotate a `@KafkaListener` method with `@SendTo` even if no result is returned.This is to allow the configuration of an `errorHandler` that can forward information about a failed message delivery to some topic. |

```
@KafkaListener(id = "voidListenerWithReplyingErrorHandler", topics = "someTopic",
        errorHandler = "voidSendToErrorHandler")
@SendTo("failures")
public void voidListenerWithReplyingErrorHandler(String in) {
    throw new RuntimeException("fail");
}

@Bean
public KafkaListenerErrorHandler voidSendToErrorHandler() {
    return (m, e) -> {
        return ... // some information about the failure and input data
    };
}
```

See [Section 4.1.7, “Handling Exceptions”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#annotation-error-handling) for more information.

#### Filtering Messages

In certain scenarios, such as rebalancing, a message may be redelivered that has already been processed.The framework cannot know whether such a message has been processed or not, that is an application-levelfunction.This is known as the [IdempotentReceiver](http://www.enterpriseintegrationpatterns.com/patterns/messaging/IdempotentReceiver.html) pattern and Spring Integration provides an[implementation thereof](https://docs.spring.io/spring-integration/reference/html/messaging-endpoints-chapter.html#idempotent-receiver).

The Spring for Apache Kafka project also provides some assistance by means of the `FilteringMessageListenerAdapter`class, which can wrap your `MessageListener`.This class takes an implementation of `RecordFilterStrategy` where you implement the `filter` method to signalthat a message is a duplicate and should be discarded.

A `FilteringAcknowledgingMessageListenerAdapter` is also provided for wrapping an `AcknowledgingMessageListener`.This has an additional property `ackDiscarded` which indicates whether the adapter should acknowledge the discarded record; it is `true` by default.

When using `@KafkaListener`, set the `RecordFilterStrategy` (and optionally `ackDiscarded`) on the container factory and the listener will be wrapped in the appropriate filtering adapter.

In addition, a `FilteringBatchMessageListenerAdapter` is provided, for when using a batch [message listener](https://docs.spring.io/spring-kafka/reference/htmlsingle/#message-listeners).

#### Retrying Deliveries

If your listener throws an exception, the default behavior is to invoke the `ErrorHandler`, if configured, or logged otherwise.

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| Two error handler interfaces are provided `ErrorHandler` and `BatchErrorHandler`; the appropriate type must be configured to match the [Message Listener](https://docs.spring.io/spring-kafka/reference/htmlsingle/#message-listeners). |

To retry deliveries, convenient listener adapters - `RetryingMessageListenerAdapter` and `RetryingAcknowledgingMessageListenerAdapter` are provided, depending on whether you are using a `MessageListener` or an `AcknowledgingMessageListener`.

These can be configured with a `RetryTemplate` and `RecoveryCallback<Void>` - see the [spring-retry](https://github.com/spring-projects/spring-retry)project for information about these components.If a recovery callback is not provided, the exception is thrown to the container after retries are exhausted.In that case, the `ErrorHandler` will be invoked, if configured, or logged otherwise.

When using `@KafkaListener`, set the `RetryTemplate` (and optionally `recoveryCallback`) on the container factory and the listener will be wrapped in the appropriate retrying adapter.

The contents of the `RetryContext` passed into the `RecoveryCallback` will depend on the type of listener.The context will always have an attribute `record` which is the record for which the failure occurred.If your listener is acknowledging and/or consumer aware, additional attributes `acknowledgment` and/or `consumer` will be available.For convenience, the `RetryingAcknowledgingMessageListenerAdapter` provides static constants for these keys.See its javadocs for more information.

A retry adapter is not provided for any of the batch [message listeners](https://docs.spring.io/spring-kafka/reference/htmlsingle/#message-listeners) because the framework has no knowledge of where, in a batch, the failure occurred.Users wishing retry capabilities, when using a batch listener, are advised to use a `RetryTemplate` within the listener itself.

#### Detecting Idle and Non-Responsive Consumers

While efficient, one problem with asynchronous consumers is detecting when they are idle - users might want to takesome action if no messages arrive for some period of time.

You can configure the listener container to publish a `ListenerContainerIdleEvent` when some time passes with no message delivery.While the container is idle, an event will be published every `idleEventInterval` milliseconds.

To configure this feature, set the `idleEventInterval` on the container:

```
@Bean
public KafKaMessageListenerContainer(ConnectionFactory connectionFactory) {
    ContainerProperties containerProps = new ContainerProperties("topic1", "topic2");
    ...
    containerProps.setIdleEventInterval(60000L);
    ...
    KafKaMessageListenerContainer<String, String> container = new KafKaMessageListenerContainer<>(...);
    return container;
}
```

Or, for a `@KafkaListener`…

```
@Bean
public ConcurrentKafkaListenerContainerFactory kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
    ...
    factory.getContainerProperties().setIdleEventInterval(60000L);
    ...
    return factory;
}
```

In each of these cases, an event will be published once per minute while the container is idle.

In addition, if the broker is unreachable (at the time of writing), the consumer `poll()` method does not exit, so no messages are received, and idle events can’t be generated.To solve this issue, the container will publish a `NonResponsiveConsumerEvent` if a poll does not return within 3x the `pollInterval` property.By default, this check is performed once every 30 seconds in each container.You can modify the behavior by setting the `monitorInterval` and `noPollThreshold` properties in the `ContainerProperties` when configuring the listener container.Receiveing such an event will allow you to stop the container(s), thus waking the consumer so it can terminate.

##### Event Consumption

You can capture these events by implementing `ApplicationListener` - either a general listener, or one narrowed to only receive this specific event.You can also use `@EventListener`, introduced in Spring Framework 4.2.

The following example combines the `@KafkaListener` and `@EventListener` into a single class.It’s important to understand that the application listener will get events for all containers so you may need tocheck the listener id if you want to take specific action based on which container is idle.You can also use the `@EventListener` `condition` for this purpose.

The events have 5 properties:

- `source` - the listener container instance
- `id` - the listener id (or container bean name)
- `idleTime` - the time the container had been idle when the event was published
- `topicPartitions` - the topics/partitions that the container was assigned at the time the event was generated
- `consumer` - a reference to the kafka `Consumer` object; for example, if the consumer was previously `pause()` d, it can be `resume()` d when the event is received.

The event is published on the consumer thread, so it is safe to interact with the `Consumer` object.

```
public class Listener {

    @KafkaListener(id = "qux", topics = "annotated")
    public void listen4(@Payload String foo, Acknowledgment ack) {
        ...
    }

    @EventListener(condition = "event.listenerId.startsWith('qux-')")
    public void eventHandler(ListenerContainerIdleEvent event) {
        ...
    }

}
```

| ![[Important]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/important.png) | Important |
| ---------------------------------------- | --------- |
| Event listeners will see events for all containers; so, in the example above, we narrow the events received based on the listener ID.Since containers created for the `@KafkaListener` support concurrency, the actual containers are named `id-n` where the `n` is a unique value for each instance to support the concurrency.Hence we use `startsWith` in the condition. |           |

| ![[Caution]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/caution.png) | Caution |
| ---------------------------------------- | ------- |
| If you wish to use the idle event to stop the lister container, you should not call `container.stop()` on the thread that calls the listener - it will cause delays and unnecessary log messages.Instead, you should hand off the event to a different thread that can then stop the container.Also, you should not `stop()` the container instance in the event if it is a child container, you should stop the concurrent container instead. |         |

##### Current Positions when Idle

Note that you can obtain the current positions when idle is detected by implementing `ConsumerSeekAware` in your listener; see `onIdleContainer()` in `[the section called “Seeking to a Specific Offset”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#seek).

#### Topic/Partition Initial Offset

There are several ways to set the initial offset for a partition.

When manually assigning partitions, simply set the initial offset (if desired) in the configured `TopicPartitionInitialOffset` arguments (see [the section called “Message Listener Containers”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#message-listener-container)).You can also seek to a specific offset at any time.

When using group management where the broker assigns partitions:

- For a new `group.id`, the initial offset is determined by the `auto.offset.reset` consumer property (`earliest` or `latest`).
- For an existing group id, the initial offset is the current offset for that group id.You can, however, seek to a specific offset during initialization (or at any time thereafter).

#### Seeking to a Specific Offset

In order to seek, your listener must implement `ConsumerSeekAware` which has the following methods:

```
void registerSeekCallback(ConsumerSeekCallback callback);

void onPartitionsAssigned(Map<TopicPartition, Long> assignments, ConsumerSeekCallback callback);

void onIdleContainer(Map<TopicPartition, Long> assignments, ConsumerSeekCallback callback);
```

The first is called when the container is started; this callback should be used when seeking at some arbitrary time after initialization.You should save a reference to the callback; if you are using the same listener in multiple containers (or in a `ConcurrentMessageListenerContainer`) you should store the callback in a `ThreadLocal` or some other structure keyed by the listener `Thread`.

When using group management, the second method is called when assignments change.You can use this method, for example, for setting initial offsets for the partitions, by calling the callback; you must use the callback argument, not the one passed into `registerSeekCallback`.This method will never be called if you explicitly assign partitions yourself; use the `TopicPartitionInitialOffset` in that case.

The callback has these methods:

```
void seek(String topic, int partition, long offset);

void seekToBeginning(String topic, int partition);

void seekToEnd(String topic, int partition);
```

You can also perform seek operations from `onIdleContainer()` when an idle container is detected; see [the section called “Detecting Idle and Non-Responsive Consumers”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#idle-containers) for how to enable idle container detection.

To arbitrarily seek at runtime, use the callback reference from the `registerSeekCallback` for the appropriate thread.

### 4.1.4 Serialization/Deserialization and Message Conversion

Apache Kafka provides a high-level API for serializing/deserializing record values as well as their keys.It is present with the `org.apache.kafka.common.serialization.Serializer<T>` and`org.apache.kafka.common.serialization.Deserializer<T>` abstractions with some built-in implementations.Meanwhile we can specify simple (de)serializer classes using Producer and/or Consumer configuration properties, e.g.:

```
props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, IntegerDeserializer.class);
props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
...
props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, IntegerSerializer.class);
props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
```

for more complex or particular cases, the `KafkaConsumer`, and therefore `KafkaProducer`, provides overloadedconstructors to accept `(De)Serializer` instances for `keys` and/or `values`, respectively.

To meet this API, the `DefaultKafkaProducerFactory` and `DefaultKafkaConsumerFactory` also provide properties to allowto inject a custom `(De)Serializer` to target `Producer`/`Consumer`.

For this purpose, Spring for Apache Kafka also provides `JsonSerializer`/`JsonDeserializer` implementations based on theJackson JSON object mapper.The `JsonSerializer` is quite simple and just allows writing any Java object as a JSON `byte[]`, the `JsonDeserializer`requires an additional `Class<?> targetType` argument to allow the deserialization of a consumed `byte[]` to the proper targetobject.

```
JsonDeserializer<Bar> barDeserializer = new JsonDeserializer<>(Bar.class);
```

Both `JsonSerializer` and `JsonDeserializer` can be customized with an `ObjectMapper`.You can also extend them to implement some particular configuration logic in the`configure(Map<String, ?> configs, boolean isKey)` method.

Starting with *version 2.1*, type information can be conveyed in record `Headers`, allowing the handling of multiple types.In addition, the serializer/deserializer can be configured using Kafka properties.

- `JsonSerializer.ADD_TYPE_INFO_HEADERS` (default `true`); set to `false` to disable this feature.
- `JsonDeserializer.DEFAULT_KEY_TYPE`; fallback type for deserialization of keys if no header information is present.
- `JsonDeserializer.DEFAULT_VALUE_TYPE`; fallback type for deserialization of values if no header information is present.
- `JsonDeserializer.TRUSTED_PACKAGES` (default `java.util`, `java.lang`); comma-delimited list of package patterns allowed for deserialization; `*` means deserialize all.

Although the `Serializer`/`Deserializer` API is quite simple and flexible from the low-level Kafka `Consumer` and`Producer` perspective, you might need more flexibility at the Spring Messaging level, either when using `@KafkaListener` or [Spring Integration](https://docs.spring.io/spring-kafka/reference/htmlsingle/#si-kafka).To easily convert to/from `org.springframework.messaging.Message`, Spring for Apache Kafka provides a `MessageConverter`abstraction with the `MessagingMessageConverter` implementation and its `StringJsonMessageConverter` customization.The `MessageConverter` can be injected into `KafkaTemplate` instance directly and via`AbstractKafkaListenerContainerFactory` bean definition for the `@KafkaListener.containerFactory()` property:

```
@Bean
public KafkaListenerContainerFactory<?> kafkaJsonListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<Integer, String> factory =
        new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    factory.setMessageConverter(new StringJsonMessageConverter());
    return factory;
}
...
@KafkaListener(topics = "jsonData",
                containerFactory = "kafkaJsonListenerContainerFactory")
public void jsonListener(Foo foo) {
...
}
```

When using a `@KafkaListener`, the parameter type is provided to the message converter to assist with the conversion.

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| This type inference can only be achieved when the `@KafkaListener` annotation is declared at the method level.With a class-level `@KafkaListener`, the payload type is used to select which `@KafkaHandler` method to invoke so it must already have been converted before the method can be chosen. |

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| 使用 `StringJsonMessageConverter`的时候，你需要在consumer配置中使用 `StringDeserializer` ，在producer配置中使用`StringSerializer` ，在使用 Spring Integration或者 `KafkaTemplate.send(Message<?> message)` 方法的时候。 |

Starting with *version 1.3.2* you can also use a `StringJsonMessageConverter` within a `BatchMessagingMessageConverter` for converting batch messages, when using a batch listener container factory.

By default, the type for the conversion is inferred from the listener argument.If you configure the `StringJsonMessageConverter` with a `DefaultJackson2TypeMapper` that has its `TypePrecedence` set to `TYPE_ID` (instead of the default `INFERRED`), then the converter will use type information in headers (if present) instead.This allows, for example, listener methods to be declared with interfaces instead of concrete classes.Also, the type converter supports mapping so the deserialization can be to a different type than the source (as long as the data is compatible).

```
@Bean
public KafkaListenerContainerFactory<?> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<Integer, String> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
    factory.setConsumerFactory(consumerFactory());
    factory.setBatchListener(true);
    factory.setMessageConverter(new BatchMessagingMessageConverter(converter()));
    return factory;
}

@Bean
public StringJsonMessageConverter converter() {
    return new StringJsonMessageConverter();
}
```

Note that for this to work, the method signature for the conversion target must be a container object with a single generic parameter type, such as:

```
@KafkaListener(topics = "blc1")
public void listen(List<Foo> foos, @Header(KafkaHeaders.OFFSET) List<Long> offsets) {
    ...
}
```

Notice that you can still access the batch headers too.

### 4.1.5 Message Headers

The 0.11.0.0 client introduced support for headers in messages.Spring for Apache Kafka *version 2.0* now supports mapping these headers to/from `spring-messaging` `MessageHeaders`.

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| Previous versions mapped `ConsumerRecord` and `ProducerRecord` to spring-messaging `Message<?>` where the value property is mapped to/from the `payload` and other properties (`topic`, `partition`, etc) were mapped to headers.This is still the case but additional, arbitrary, headers can now be mapped. |

Apache Kafka headers have a simple API:

```
public interface Header {

    String key();

    byte[] value();

}
```

The `KafkaHeaderMapper` strategy is provided to map header entries between Kafka `Headers` and `MessageHeaders`:

```
public interface KafkaHeaderMapper {

    void fromHeaders(MessageHeaders headers, Headers target);

    void toHeaders(Headers source, Map<String, Object> target);

}
```

The `DefaultKafkaHeaderMapper` maps the key to the `MessageHeaders` header name and, in order to support rich header types, for outbound messages, JSON conversion is performed.A "special" header, with key, `spring_json_header_types` contains a JSON map of `<key>:<type>`.This header is used on the inbound side to provide appropriate conversion of each header value to the original type.

On the inbound side, all Kafka `Header` s are mapped to `MessageHeaders`.On the outbound side, by default, all `MessageHeaders` are mapped except `id`, `timestamp`, and the headers that map to `ConsumerRecord` properties.

You can specify which headers are to be mapped for outbound messages, by providing patterns to the mapper.

```
public DefaultKafkaHeaderMapper() {
    ...
}

public DefaultKafkaHeaderMapper(ObjectMapper objectMapper) {
    ...
}

public DefaultKafkaHeaderMapper(String... patterns) {
    ...
}

public DefaultKafkaHeaderMapper(ObjectMapper objectMapper, String... patterns) {
    ...
}
```

The first constructor will use a default Jackson `ObjectMapper` and map most headers, as discussed above.The second constructor will use the provided Jackson `ObjectMapper` and map most headers, as discussed above.The third constructor will use a default Jackson `ObjectMapper` and map headers according to the provided patterns.The third constructor will use the provided Jackson `ObjectMapper` and map headers according to the provided patterns.

Patterns are rather simple and can contain either a leading or trailing wildcard `*`, or both, e.g. `*.foo.*`.Patterns can be negated with a leading `!`.The first pattern that matches a header name wins (positive or negative).

When providing your own patterns, it is recommended to include `!id` and `!timestamp` since these headers are read-only on the inbound side.

| ![[Important]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/important.png) | Important |
| ---------------------------------------- | --------- |
| By default, the mapper will only deserialize classes in `java.lang` and `java.util`.You can trust other (or all) packages by adding trusted packages using the `addTrustedPackages` method.If you are receiving messages from untrusted sources, you may wish to add just those packages that you trust.To trust all packages use `mapper.addTrustedPackages("*")`. |           |

The `DefaultKafkaHeaderMapper` is used in the `MessagingMessageConverter` and `BatchMessagingMessageConverter` by default, as long as Jackson is on the class path.

With the batch converter, the converted headers are available in the `KafkaHeaders.BATCH_CONVERTED_HEADERS` as a `List<Map<String, Object>>` where the map in a position of the list corresponds to the data position in the payload.

If the converter has no converter (either because Jackson is not present, or it is explicitly set to `null`), the headers from the consumer record are provided unconverted in the `KafkaHeaders.NATIVE_HEADERS` header (a `Headers` object, or a `List<Headers>` in the case of the batch converter, where the position in the list corresponds to the data position in the payload).

| ![[Important]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/important.png) | Important |
| ---------------------------------------- | --------- |
| The Jackson `ObjectMapper` (even if provided) will be enhanced to support deserializing `org.springframework.util.MimeType` objects, often used in the `spring-messaging` `contentType` header.If you don’t wish your mapper to be enhanced in this way, for some reason, you should subclass the `DefaultKafkaHeaderMapper` and override `getObjectMapper()` to return your mapper. |           |

### 4.1.6 Log Compaction

When using [Log Compaction](https://cwiki.apache.org/confluence/display/KAFKA/Log+Compaction), it is possible to send and receive messages with `null` payloads which identifies the deletion of a key.

Starting with *version 1.0.3*, this is now fully supported.

To send a `null` payload using the `KafkaTemplate` simply pass null into the value argument of the `send()` methods.One exception to this is the `send(Message<?> message)` variant.Since `spring-messaging` `Message<?>` cannot have a `null` payload, a special payload type `KafkaNull` is used and the framework will send `null`.For convenience, the static `KafkaNull.INSTANCE` is provided.

When using a message listener container, the received `ConsumerRecord` will have a `null` `value()`.

To configure the `@KafkaListener` to handle `null` payloads, you must use the `@Payload` annotation with `required = false`; you will usually also need the key so your application knows which key was "deleted":

```
@KafkaListener(id = "deletableListener", topics = "myTopic")
public void listen(@Payload(required = false) String value, @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) String key) {
    // value == null represents key deletion
}
```

When using a class-level `@KafkaListener`, some additional configuration is needed - a `@KafkaHandler` method with a `KafkaNull` payload:

```
@KafkaListener(id = "multi", topics = "myTopic")
static class MultiListenerBean {

    @KafkaHandler
    public void listen(String foo) {
        ...
    }

    @KafkaHandler
    public void listen(Integer bar) {
        ...
    }

    @KafkaHandler
    public void delete(@Payload(required = false) KafkaNull nul, @Header(KafkaHeaders.RECEIVED_MESSAGE_KEY) int key) {
        ...
    }

}
```

### 4.1.7 Handling Exceptions

#### Listener Error Handlers

Starting with *version 2.0*, the `@KafkaListener` annotation has a new attribute: `errorHandler`.

This attribute is not configured by default.

Use the `errorHandler` to provide the bean name of a `KafkaListenerErrorHandler` implementation.This functional interface has one method:

```
@FunctionalInterface
public interface KafkaListenerErrorHandler {

    Object handleError(Message<?> message, ListenerExecutionFailedException exception) throws Exception;

}
```

As you can see, you have access to the spring-messaging `Message<?>` object produced by the message converter and the exception that was thrown by the listener, wrapped in a `ListenerExecutionFailedException`.The error handler can throw the original or a new exception which will be thrown to the container. Anything returned by the error handler is ignored.

It has a sub-interface `ConsumerAwareListenerErrorHandler` that has access to the consumer object, via the method:

```
Object handleError(Message<?> message, ListenerExecutionFailedException exception, Consumer<?, ?> consumer);
```

If your error handler implements this interface you can, for example, adjust the offsets accordingly.For example, to reset the offset to replay the failed message, you could do something like the following; note however, these are simplistic implementations and you would probably want more checking in the error handler.

```
@Bean
public ConsumerAwareListenerErrorHandler listen3ErrorHandler() {
    return (m, e, c) -> {
        this.listen3Exception = e;
        MessageHeaders headers = m.getHeaders();
        c.seek(new org.apache.kafka.common.TopicPartition(
                headers.get(KafkaHeaders.RECEIVED_TOPIC, String.class),
                headers.get(KafkaHeaders.RECEIVED_PARTITION_ID, Integer.class)),
                headers.get(KafkaHeaders.OFFSET, Long.class));
        return null;
    };
}
```

And for a batch listener:

```
@Bean
public ConsumerAwareListenerErrorHandler listen10ErrorHandler() {
    return (m, e, c) -> {
        this.listen10Exception = e;
        MessageHeaders headers = m.getHeaders();
        List<String> topics = headers.get(KafkaHeaders.RECEIVED_TOPIC, List.class);
        List<Integer> partitions = headers.get(KafkaHeaders.RECEIVED_PARTITION_ID, List.class);
        List<Long> offsets = headers.get(KafkaHeaders.OFFSET, List.class);
        Map<TopicPartition, Long> offsetsToReset = new HashMap<>();
        for (int i = 0; i < topics.size(); i++) {
            int index = i;
            offsetsToReset.compute(new TopicPartition(topics.get(i), partitions.get(i)),
                    (k, v) -> v == null ? offsets.get(index) : Math.min(v, offsets.get(index)));
        }
        offsetsToReset.forEach((k, v) -> c.seek(k, v));
        return null;
    };
}
```

This resets each topic/partition in the batch to the lowest offset in the batch.

#### Container Error Handlers

You can specify a global error handler used for all listeners in the container factory.

```
@Bean
public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<Integer, String>>
        kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<Integer, String> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
    ...
    factory.getContainerProperties().setErrorHandler(myErrorHandler);
    ...
    return factory;
}
```

or

```
@Bean
public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<Integer, String>>
        kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<Integer, String> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
    ...
    factory.getContainerProperties().setBatchErrorHandler(myBatchErrorHandler);
    ...
    return factory;
}
```

By default, if an annotated listener method throws an exception, it is thrown to the container, and the message will be handled according to the container configuration.

#### Consumer-Aware Container Error Handlers

The container-level error handlers (`ErrorHandler` and `BatchErrorHandler`) have sub-interfaces `ConsumerAwareErrorHandler` and `ConsumerAwareBatchErrorHandler` with method signatures:

```
void handle(Exception thrownException, ConsumerRecord<?, ?> data, Consumer<?, ?> consumer);

void handle(Exception thrownException, ConsumerRecords<?, ?> data, Consumer<?, ?> consumer);
```

respectively.

Similar to the `@KafkaListener` error handlers, you can reset the offsets as needed based on the data that failed.

| ![[Note]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/note.png) |
| ---------------------------------------- |
| Unlike the listener-level error handlers, however, you should set the container property `ackOnError` to false when making adjustments; otherwise any pending acks will be applied after your repositioning. |

#### Seek To Current Container Error Handlers

If an `ErrorHandler` implements `RemainingRecordsErrorHandler`, the error handler is provided with the failed record and any unprocessed records retrieved by the previous `poll()`.Those records will not be passed to the listener after the handler exits.

```
@FunctionalInterface
public interface RemainingRecordsErrorHandler extends ConsumerAwareErrorHandler {

    void handle(Exception thrownException, List<ConsumerRecord<?, ?>> records, Consumer<?, ?> consumer);

}
```

This allows implementations to seek all unprocessed topic/partitions so the current record (and the others remaining) will be retrieved by the next poll.The `SeekToCurrentErrorHandler` does exactly this.

The container will commit any pending offset commits before calling the error handler.

To configure the listener container with this handler, add it to the `ContainerProperties`.

For example, with the `@KafkaListener` container factory:

```
@Bean
public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
    ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory();
    factory.setConsumerFactory(consumerFactory());
    factory.getContainerProperties().setAckOnError(false);
    factory.getContainerProperties().setErrorHandler(new SeekToCurrentErrorHandler());
    factory.getContainerProperties().setAckMode(AckMode.RECORD);
    return factory;
}
```

As an example; if the `poll` returns 6 records (2 from each partition 0, 1, 2) and the listener throws an exception on the fourth record, the container will have acknowledged the first 3 by committing their offsets.The `SeekToCurrentErrorHandler` will seek to offset 1 for partition 1 and offset 0 for partition 2.The next `poll()` will return the 3 unprocessed records.

If the `AckMode` was `BATCH`, the container commits the offsets for the first 2 partitions before calling the error handler.

The `SeekToCurrentBatchErrorHandler` seeks each partition to the first record in each partition in the batch so the whole batch is replayed.

After seeking, an exception wrapping the `ListenerExecutionFailedException` is thrown.This is to cause the transaction to roll back (if transactions are enabled).

#### Container Stopping Error Handlers

The `ContainerStoppingErrorHandler` (used with record listeners) will stop the container if the listener throws an exception.When the `AckMode` is `RECORD`, offsets for already processed records will be committed.When the `AckMode` is any manual, offsets for already acknowledged records will be committed.When the `AckMode` is `BATCH`, the entire batch will be replayed when the container is restarted, unless transactions are enabled in which case only the unprocessed records will be re-fetched.

The `ContainerStoppingBatchErrorHandler` (used with batch listeners) will stop the container and the entire batch will be replayed when the container is restarted.

After the container stops, an exception wrapping the `ListenerExecutionFailedException` is thrown.This is to cause the transaction to roll back (if transactions are enabled).

### 4.1.8 Kerberos

Starting with version 2.0 a `KafkaJaasLoginModuleInitializer` class has been added to assist with Kerberos configuration.Simply add this bean, with the desired configuration, to your application context.

```
@Bean
public KafkaJaasLoginModuleInitializer jaasConfig() throws IOException {
    KafkaJaasLoginModuleInitializer jaasConfig = new KafkaJaasLoginModuleInitializer();
    jaasConfig.setControlFlag("REQUIRED");
    Map<String, String> options = new HashMap<>();
    options.put("useKeyTab", "true");
    options.put("storeKey", "true");
    options.put("keyTab", "/etc/security/keytabs/kafka_client.keytab");
    options.put("principal", "kafka-client-1@EXAMPLE.COM");
    jaasConfig.setOptions(options);
    return jaasConfig;
}
```

## 4.2 Kafka Streams Support

### 4.2.1 Introduction

Starting with *version 1.1.4*, Spring for Apache Kafka provides first class support for [Kafka Streams](https://kafka.apache.org/documentation/streams).For using it from a Spring application, the `kafka-streams` jar must be present on classpath.It is an optional dependency of the `spring-kafka` project and isn’t downloaded transitively.

### 4.2.2 Basics

The reference Apache Kafka Streams documentation suggests this way of using the API:

```
// Use the builders to define the actual processing topology, e.g. to specify
// from which input topics to read, which stream operations (filter, map, etc.)
// should be called, and so on.

StreamsBuilder builder = ...;  // when using the Kafka Streams DSL

// Use the configuration to tell your application where the Kafka cluster is,
// which serializers/deserializers to use by default, to specify security settings,
// and so on.
StreamsConfig config = ...;

KafkaStreams streams = new KafkaStreams(builder, config);

// Start the Kafka Streams instance
streams.start();

// Stop the Kafka Streams instance
streams.close();
```

So, we have two main components: `StreamsBuilder` with an API to build `KStream` (or `KTable`) instances and `KafkaStreams` to manage their lifecycle.Note: all `KStream` instances exposed to a `KafkaStreams` instance by a single `StreamsBuilder` will be started and stopped at the same time, even if they have a fully different logic.In other words all our streams defined by a `StreamsBuilder` are tied with a single lifecycle control.Once a `KafkaStreams` instance has been closed via `streams.close()` it cannot be restarted, and a new `KafkaStreams` instance to restart stream processing must be created instead.

### 4.2.3 Spring 管理

为了简化从Spring应用程序上下文角度使用Kafka Streams的步骤，并利用容器进行生命周期管理，Spring for Apache Kafka引入了`StreamsBuilderFactoryBean`。这是一个`AbstractFactoryBean`实现，将`StreamsBuilder`单例实例显示为一个bean：

```java
@Bean
public FactoryBean<StreamsBuilderFactoryBean> myKStreamBuilder(StreamsConfig streamsConfig) {
    return new StreamsBuilderFactoryBean(streamsConfig);
}
```

`StreamsBuilderFactoryBean`还实现了`SmartLifecycle`来管理内部`KafkaStreams`实例的生命周期。与Kafka Streams API类似，必须在启动`KafkaStreams`之前定义`KStream`实例，这也适用于Kafka Streams的Spring API。

因此，当我们在`StreamsBuilderFactoryBean`上使用默认设置`autoStartup = true`时，我们必须在刷新应用程序上下文之前在`StreamsBuilder`上声明`KStream`。例如，`KStream`可以像普通的bean定义一样，同时使用Kafka Streams API没有任何影响

```java
@Bean
public KStream<?, ?> kStream(StreamsBuilder kStreamBuilder) {
    KStream<Integer, String> stream = kStreamBuilder.stream(STREAMING_TOPIC1);
    // Fluent KStream API
    return stream;
}
```

如果您想手动控制生命周期（例如，通过某些条件停止和启动），则可以使用工厂bean（＆）前缀直接引用`StreamsBuilderFactoryBean`。由于`StreamsBuilderFactoryBean`使用其内部的`KafkaStreams`实例，因此可以安全地停止并重新启动它 ——每次`start()`，创建一个新的`KafkaStreams`。如果您想单独控制`KStream`实例的生命周期，请考虑使用不同的`StreamsBuilderFactoryBean`。

你可以在`StreamsBuilderFactoryBean`上指定`KafkaStreams.StateListener`和`Thread.UncaughtExceptionHandler`选项，这些选项被委托给内部的`KafkaStreams`实例。如果您需要直接执行一些`KafkaStream`操作，那么可以通过`StreamsBuilderFactoryBean.getKafkaStreams()`来访问内部的`KafkaStreams`实例。您可以按类型自动装配`StreamsBuilderFactoryBean`
 bean，但应确保在bean定义中使用完整类型，例如：

```java
@Bean
public StreamsBuilderFactoryBean myKStreamBuilder(StreamsConfig streamsConfig) {
    return new StreamsBuilderFactoryBean(streamsConfig);
}
...
@Autowired
private StreamsBuilderFactoryBean myKStreamBuilderFactoryBean;
```

或者，如果使用接口bean定义，则添加用于注入的`@Qualifier`：

```java
@Bean
public FactoryBean<StreamsBuilder> myKStreamBuilder(StreamsConfig streamsConfig) {
    return new StreamsBuilderFactoryBean(streamsConfig);
}
...
@Autowired
@Qualifier("&myKStreamBuilder")
private StreamsBuilderFactoryBean myKStreamBuilderFactoryBean;
```

### 4.2.4 JSON Serdes

For serializing and deserializing data when reading or writing to topics or state stores in JSON format, Spring Kafka provides a `JsonSerde` implementation using JSON, delegating to the `JsonSerializer` and `JsonDeserializer` described in [the serialization/deserialization section](https://docs.spring.io/spring-kafka/reference/htmlsingle/#serdes).The `JsonSerde` provides the same configuration options via its constructor (target type and/or `ObjectMapper`).In the following example we use the `JsonSerde` to serialize and deserialize the `Foo` payload of a Kafka stream - the `JsonSerde` can be used in a similar fashion wherever an instance is required.

```
stream.through(Serdes.Integer(), new JsonSerde<>(Foo.class), "foos");
```

### 4.2.5 Configuration

To configure the Kafka Streams environment, the `StreamsBuilderFactoryBean` requires a `Map` of particular properties or a `StreamsConfig` instance.See Apache Kafka [documentation](https://kafka.apache.org/0102/documentation/#streamsconfigs) for all possible options.

To avoid boilerplate code for most cases, especially when you develop micro services, Spring for Apache Kafka provides the `@EnableKafkaStreams` annotation, which should be placed alongside with `@Configuration`.Only you need is to declare `StreamsConfig` bean with the `defaultKafkaStreamsConfig` name.A `StreamsBuilder` bean with the `defaultKafkaStreamsBuilder` name will be declare in the application context automatically.Any additional `StreamsBuilderFactoryBean` beans can be declared and used as well.

### 4.2.6 Kafka Streams Example

Putting it all together:

```
@Configuration
@EnableKafka
@EnableKafkaStreams
public static class KafkaStreamsConfiguration {

    @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
    public StreamsConfig kStreamsConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "testStreams");
        props.put(StreamsConfig.KEY_SERDE_CLASS_CONFIG, Serdes.Integer().getClass().getName());
        props.put(StreamsConfig.VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        props.put(StreamsConfig.TIMESTAMP_EXTRACTOR_CLASS_CONFIG, WallclockTimestampExtractor.class.getName());
        return new StreamsConfig(props);
    }

    @Bean
    public KStream<Integer, String> kStream(StreamsBuilder kStreamBuilder) {
        KStream<Integer, String> stream = kStreamBuilder.stream("streamingTopic1");
        stream
                .mapValues(String::toUpperCase)
                .groupByKey()
                .reduce((String value1, String value2) -> value1 + value2,
                		TimeWindows.of(1000),
                		"windowStore")
                .toStream()
                .map((windowedId, value) -> new KeyValue<>(windowedId.key(), value))
                .filter((i, s) -> s.length() > 40)
                .to("streamingTopic2");

        stream.print();

        return stream;
    }

}
```

## 4.3 Testing Applications

### 4.3.1 Introduction

The `spring-kafka-test` jar contains some useful utilities to assist with testing your applications.

### 4.3.2 JUnit

`o.s.kafka.test.utils.KafkaTestUtils` provides some static methods to set up producer and consumer properties:

```
/**
 * Set up test properties for an {@code <Integer, String>} consumer.
 * @param group the group id.
 * @param autoCommit the auto commit.
 * @param embeddedKafka a {@link KafkaEmbedded} instance.
 * @return the properties.
 */
public static Map<String, Object> consumerProps(String group, String autoCommit,
                                       KafkaEmbedded embeddedKafka) { ... }

/**
 * Set up test properties for an {@code <Integer, String>} producer.
 * @param embeddedKafka a {@link KafkaEmbedded} instance.
 * @return the properties.
 */
public static Map<String, Object> senderProps(KafkaEmbedded embeddedKafka) { ... }
```

A JUnit `@Rule` is provided that creates an embedded Kafka and an embedded Zookeeper server.

```
/**
 * Create embedded Kafka brokers.
 * @param count the number of brokers.
 * @param controlledShutdown passed into TestUtils.createBrokerConfig.
 * @param topics the topics to create (2 partitions per).
 */
public KafkaEmbedded(int count, boolean controlledShutdown, String... topics) { ... }

/**
 *
 * Create embedded Kafka brokers.
 * @param count the number of brokers.
 * @param controlledShutdown passed into TestUtils.createBrokerConfig.
 * @param partitions partitions per topic.
 * @param topics the topics to create.
 */
public KafkaEmbedded(int count, boolean controlledShutdown, int partitions, String... topics) { ... }
```

The embedded kafka class has a utility method allowing you to consume for all the topics it created:

```
Map<String, Object> consumerProps = KafkaTestUtils.consumerProps("testT", "false", embeddedKafka);
DefaultKafkaConsumerFactory<Integer, String> cf = new DefaultKafkaConsumerFactory<Integer, String>(
        consumerProps);
Consumer<Integer, String> consumer = cf.createConsumer();
embeddedKafka.consumeFromAllEmbeddedTopics(consumer);
```

The `KafkaTestUtils` has some utility methods to fetch results from the consumer:

```
/**
 * Poll the consumer, expecting a single record for the specified topic.
 * @param consumer the consumer.
 * @param topic the topic.
 * @return the record.
 * @throws org.junit.ComparisonFailure if exactly one record is not received.
 */
public static <K, V> ConsumerRecord<K, V> getSingleRecord(Consumer<K, V> consumer, String topic) { ... }

/**
 * Poll the consumer for records.
 * @param consumer the consumer.
 * @return the records.
 */
public static <K, V> ConsumerRecords<K, V> getRecords(Consumer<K, V> consumer) { ... }
```

Usage:

```
...
template.sendDefault(0, 2, "bar");
ConsumerRecord<Integer, String> received = KafkaTestUtils.getSingleRecord(consumer, "topic");
...
```

When the embedded Kafka and embedded Zookeeper server are started by JUnit, a system property `spring.embedded.kafka.brokers` is set to the address of the Kafka broker(s) and a system property `spring.embedded.zookeeper.connect` is set to the address of Zookeeper.Convenient constants `KafkaEmbedded.SPRING_EMBEDDED_KAFKA_BROKERS` and `KafkaEmbedded.SPRING_EMBEDDED_ZOOKEEPER_CONNECT` are provided for this property.

With the `KafkaEmbedded.brokerProperties(Map<String, String>)` you can provide additional properties for the Kafka server(s).See [Kafka Config](https://kafka.apache.org/documentation/#brokerconfigs) for more information about possible broker properties.

### 4.3.3 @EmbeddedKafka Annotation

It is generally recommended to use the rule as a `@ClassRule` to avoid starting/stopping the broker between tests (and use a different topic for each test).Starting with *version 2.0*, if you are using Spring’s test application context caching, you can also declare a `KafkaEmbedded` bean, so a single broker can be used across multiple test classes.The JUnit `ExternalResource` `before()/after()` lifecycle is wrapped to the `afterPropertiesSet()` and `destroy()` Spring infrastructure hooks.For convenience a test class level `@EmbeddedKafka` annotation is provided with the purpose to register `KafkaEmbedded` bean:

```
@RunWith(SpringRunner.class)
@DirtiesContext
@EmbeddedKafka(partitions = 1,
         topics = {
                 KafkaStreamsTests.STREAMING_TOPIC1,
                 KafkaStreamsTests.STREAMING_TOPIC2 })
public class KafkaStreamsTests {

    @Autowired
    private KafkaEmbedded kafkaEmbedded;

    @Test
    public void someTest() {
        Map<String, Object> consumerProps = KafkaTestUtils.consumerProps("testGroup", "true", this.kafkaEmbedded);
        consumerProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        ConsumerFactory<Integer, String> cf = new DefaultKafkaConsumerFactory<>(consumerProps);
        Consumer<Integer, String> consumer = cf.createConsumer();
        this.embeddedKafka.consumeFromAnEmbeddedTopic(consumer, KafkaStreamsTests.STREAMING_TOPIC2);
        ConsumerRecords<Integer, String> replies = KafkaTestUtils.getRecords(consumer);
        assertThat(replies.count()).isGreaterThanOrEqualTo(1);
    }

    @Configuration
    @EnableKafkaStreams
    public static class KafkaStreamsConfiguration {

        @Value("${" + KafkaEmbedded.SPRING_EMBEDDED_KAFKA_BROKERS + "}")
        private String brokerAddresses;

        @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
        public StreamsConfig kStreamsConfigs() {
            Map<String, Object> props = new HashMap<>();
            props.put(StreamsConfig.APPLICATION_ID_CONFIG, "testStreams");
            props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, this.brokerAddresses);
            return new StreamsConfig(props);
        }

    }

}
```

The `brokerProperties` and `brokerPropertiesLocation` attributes of `@EmbeddedKafka` support property placeholder resolutions:

```
@TestPropertySource(locations = "classpath:/test.properties")
@EmbeddedKafka(topics = "any-topic",
        brokerProperties = { "log.dir=${kafka.broker.logs-dir}",
                            "listeners=PLAINTEXT://localhost:${kafka.broker.port}",
                            "auto.create.topics.enable=${kafka.broker.topics-enable:true}" }
        brokerPropertiesLocation = "classpath:/broker.properties")
```

In th example above, the property placeholders `${kafka.broker.logs-dir}` and `${kafka.broker.port}` are resolved from the Spring `Environment`.In addition the broker properties are loaded from the `broker.properties` classpath resource specified by the `brokerPropertiesLocation`.Property placeholders are resolved for the `brokerPropertiesLocation` URL and for any property placeholders found in the resource.Properties defined by `brokerProperties` override properties found in `brokerPropertiesLocation`.

### 4.3.4 Hamcrest Matchers

The `o.s.kafka.test.hamcrest.KafkaMatchers` provides the following matchers:

```
/**
 * @param key the key
 * @param <K> the type.
 * @return a Matcher that matches the key in a consumer record.
 */
public static <K> Matcher<ConsumerRecord<K, ?>> hasKey(K key) { ... }

/**
 * @param value the value.
 * @param <V> the type.
 * @return a Matcher that matches the value in a consumer record.
 */
public static <V> Matcher<ConsumerRecord<?, V>> hasValue(V value) { ... }

/**
 * @param partition the partition.
 * @return a Matcher that matches the partition in a consumer record.
 */
public static Matcher<ConsumerRecord<?, ?>> hasPartition(int partition) { ... }

/**
 * Matcher testing the timestamp of a {@link ConsumerRecord} asssuming the topic has been set with
 * {@link org.apache.kafka.common.record.TimestampType#CREATE_TIME CreateTime}.
 *
 * @param ts timestamp of the consumer record.
 * @return a Matcher that matches the timestamp in a consumer record.
 */
public static Matcher<ConsumerRecord<?, ?>> hasTimestamp(long ts) {
  return hasTimestamp(TimestampType.CREATE_TIME, ts);
}

/**
 * Matcher testing the timestamp of a {@link ConsumerRecord}
 * @param type timestamp type of the record
 * @param ts timestamp of the consumer record.
 * @return a Matcher that matches the timestamp in a consumer record.
 */
public static Matcher<ConsumerRecord<?, ?>> hasTimestamp(TimestampType type, long ts) {
  return new ConsumerRecordTimestampMatcher(type, ts);
}
```

### 4.3.5 AssertJ Conditions

```
/**
 * @param key the key
 * @param <K> the type.
 * @return a Condition that matches the key in a consumer record.
 */
public static <K> Condition<ConsumerRecord<K, ?>> key(K key) { ... }

/**
 * @param value the value.
 * @param <V> the type.
 * @return a Condition that matches the value in a consumer record.
 */
public static <V> Condition<ConsumerRecord<?, V>> value(V value) { ... }

/**
 * @param partition the partition.
 * @return a Condition that matches the partition in a consumer record.
 */
public static Condition<ConsumerRecord<?, ?>> partition(int partition) { ... }

/**
 * @param value the timestamp.
 * @return a Condition that matches the timestamp value in a consumer record.
 */
public static Condition<ConsumerRecord<?, ?>> timestamp(long value) {
  return new ConsumerRecordTimestampCondition(TimestampType.CREATE_TIME, value);
}

/**
 * @param type the type of timestamp
 * @param value the timestamp.
 * @return a Condition that matches the timestamp value in a consumer record.
 */
public static Condition<ConsumerRecord<?, ?>> timestamp(TimestampType type, long value) {
  return new ConsumerRecordTimestampCondition(type, value);
}
```

### 4.3.6 Example

Putting it all together:

```
public class KafkaTemplateTests {

    private static final String TEMPLATE_TOPIC = "templateTopic";

    @ClassRule
    public static KafkaEmbedded embeddedKafka = new KafkaEmbedded(1, true, TEMPLATE_TOPIC);

    @Test
    public void testTemplate() throws Exception {
        Map<String, Object> consumerProps = KafkaTestUtils.consumerProps("testT", "false",
            embeddedKafka);
        DefaultKafkaConsumerFactory<Integer, String> cf =
                            new DefaultKafkaConsumerFactory<Integer, String>(consumerProps);
        ContainerProperties containerProperties = new ContainerProperties(TEMPLATE_TOPIC);
        KafkaMessageListenerContainer<Integer, String> container =
                            new KafkaMessageListenerContainer<>(cf, containerProperties);
        final BlockingQueue<ConsumerRecord<Integer, String>> records = new LinkedBlockingQueue<>();
        container.setupMessageListener(new MessageListener<Integer, String>() {

        	@Override
        	public void onMessage(ConsumerRecord<Integer, String> record) {
                System.out.println(record);
                records.add(record);
            }

        });
        container.setBeanName("templateTests");
        container.start();
        ContainerTestUtils.waitForAssignment(container, embeddedKafka.getPartitionsPerTopic());
        Map<String, Object> senderProps =
                            KafkaTestUtils.senderProps(embeddedKafka.getBrokersAsString());
        ProducerFactory<Integer, String> pf =
                            new DefaultKafkaProducerFactory<Integer, String>(senderProps);
        KafkaTemplate<Integer, String> template = new KafkaTemplate<>(pf);
        template.setDefaultTopic(TEMPLATE_TOPIC);
        template.sendDefault("foo");
        assertThat(records.poll(10, TimeUnit.SECONDS), hasValue("foo"));
        template.sendDefault(0, 2, "bar");
        ConsumerRecord<Integer, String> received = records.poll(10, TimeUnit.SECONDS);
        assertThat(received, hasKey(2));
        assertThat(received, hasPartition(0));
        assertThat(received, hasValue("bar"));
        template.send(TEMPLATE_TOPIC, 0, 2, "baz");
        received = records.poll(10, TimeUnit.SECONDS);
        assertThat(received, hasKey(2));
        assertThat(received, hasPartition(0));
        assertThat(received, hasValue("baz"));
    }

}
```

The above uses the hamcrest matchers; with `AssertJ`, the final part looks like this…

```
assertThat(records.poll(10, TimeUnit.SECONDS)).has(value("foo"));
template.sendDefault(0, 2, "bar");
ConsumerRecord<Integer, String> received = records.poll(10, TimeUnit.SECONDS);
assertThat(received).has(key(2));
assertThat(received).has(partition(0));
assertThat(received).has(value("bar"));
template.send(TEMPLATE_TOPIC, 0, 2, "baz");
received = records.poll(10, TimeUnit.SECONDS);
assertThat(received).has(key(2));
assertThat(received).has(partition(0));
assertThat(received).has(value("baz"));
```

# 5. Spring Integration

引用的这部分展示了如何使用Spring Integration的`spring-integration-kafka`模块。

## 5.1 Spring Integration for Apache Kafka

### 5.1.1 介绍

这个文档属于2.0.0版本及以上;文档之前的版本,请参阅[1.3.x README](https://github.com/spring-projects/spring-integration-kafka/blob/1.3.x/README.md).

Spring Integration Kafka 基于 [Spring for Apache Kafka project](https://projects.spring.io/spring-kafka/)。它提供了以下组件:

- Outbound Channel Adapter
- Message-Driven Channel Adapter

这些将在以下部分中讨论。

### 5.1.2 Outbound Channel Adapter

Outbound channel adapter用于把messages从 Spring Integration channel 推到Kafka topics.中。

application context中定义了channel，连接到发送消息到Kafka的application中

The channel is defined in the application context and then wired into the application that sends messages to Kafka.Sender applications can publish to Kafka via Spring Integration messages, which are internally convertedto Kafka messages by the outbound channel adapter, as follows: the payload of the Spring Integration message will beused to populate the payload of the Kafka message, and (by default) the `kafka_messageKey` header of the SpringIntegration message will be used to populate the key of the Kafka message.

The target topic and partition for publishing the message can be customized through the `kafka_topic`and `kafka_partitionId` headers, respectively.

In addition, the `<int-kafka:outbound-channel-adapter>` provides the ability to extract the key, target topic, andtarget partition by applying SpEL expressions on the outbound message. To that end, it supports the mutually exclusivepairs of attributes `topic`/`topic-expression`, `message-key`/`message-key-expression`, and`partition-id`/`partition-id-expression`, to allow the specification of `topic`,`message-key` and `partition-id`respectively as static values on the adapter, or to dynamically evaluate their values at runtime againstthe request message.

| ![[Important]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/important.png) | Important |
| ---------------------------------------- | --------- |
| The `KafkaHeaders` interface (provided by `spring-kafka`) contains constants used for interacting withheaders.The `messageKey` and `topic` default headers now require a `kafka_` prefix.When migrating from an earlier version that used the old headers, you need to specify`message-key-expression="headers['messageKey']"` and `topic-expression="headers['topic']"` on the`<int-kafka:outbound-channel-adapter>`, or simply change the headers upstream tothe new headers from `KafkaHeaders` using a `<header-enricher>` or `MessageBuilder`.Or, of course, configure them on the adapter using `topic` and `message-key` if you are using constant values. |           |

注意 ：如果adapter配置了一个topic 或者 message key（一个常数或表达式），那这些配置都会被采纳，相应的header将被忽略。如果你希望header覆盖相应的配置，你需要配置一个表达式，如:

`topic-expression="headers['topic'] != null ? headers['topic'] : 'myTopic'"`.

 adapter 需要一个 `KafkaTemplate`.

下面给出了用XML配置Kafka outbound channel adapter的例子：

```java
<int-kafka:outbound-channel-adapter id="kafkaOutboundChannelAdapter"
                                    kafka-template="template"
                                    auto-startup="false"
                                    channel="inputToKafka"
                                    topic="foo"
                                    sync="false"
                                    message-key-expression="'bar'"
                                    send-failure-channel="failures"
                                    send-success-channel="successes"
                                    error-message-strategy="ems"
                                    partition-id-expression="2">
</int-kafka:outbound-channel-adapter>

<bean id="template" class="org.springframework.kafka.core.KafkaTemplate">
    <constructor-arg>
        <bean class="org.springframework.kafka.core.DefaultKafkaProducerFactory">
            <constructor-arg>
                <map>
                    <entry key="bootstrap.servers" value="localhost:9092" />
                    ... <!-- more producer properties -->
                </map>
            </constructor-arg>
        </bean>
    </constructor-arg>
</bean>
```

就像你看到的一样，adapter需要一个 `KafkaTemplate` ，相应的，也需要一个配置好的`KafkaProducerFactory`

使用Spring配置：

```java
@Bean
@ServiceActivator(inputChannel = "toKafka")
public MessageHandler handler() throws Exception {
    KafkaProducerMessageHandler<String, String> handler =
            new KafkaProducerMessageHandler<>(kafkaTemplate());
    handler.setTopicExpression(new LiteralExpression("someTopic"));
    handler.setMessageKeyExpression(new LiteralExpression("someKey"));
    handler.setFailureChannel(failures());
    return handler;
}

@Bean
public KafkaTemplate<String, String> kafkaTemplate() {
    return new KafkaTemplate<>(producerFactory());
}

@Bean
public ProducerFactory<String, String> producerFactory() {
    Map<String, Object> props = new HashMap<>();
    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, this.brokerAddress);
    // set more properties
    return new DefaultKafkaProducerFactory<>(props);
}
```

When using Spring Integration Java DSL:

```java
@Bean
public ProducerFactory<Integer, String> producerFactory() {
    return new DefaultKafkaProducerFactory<>(KafkaTestUtils.producerProps(embeddedKafka));
}

@Bean
public IntegrationFlow sendToKafkaFlow() {
    return f -> f
            .<String>split(p -> Stream.generate(() -> p).limit(101).iterator(), null)
            .publishSubscribeChannel(c -> c
                    .subscribe(sf -> sf.handle(
                            kafkaMessageHandler(producerFactory(), TEST_TOPIC1)
                                    .timestampExpression("T(Long).valueOf('1487694048633')"),
                            e -> e.id("kafkaProducer1")))
                    .subscribe(sf -> sf.handle(
                            kafkaMessageHandler(producerFactory(), TEST_TOPIC2)
                                   .timestamp(m -> 1487694048644L),
                            e -> e.id("kafkaProducer2")))
            );
}

@Bean
public DefaultKafkaHeaderMapper mapper() {
    return new DefaultKafkaHeaderMapper();
}

private KafkaProducerMessageHandlerSpec<Integer, String, ?> kafkaMessageHandler(
        ProducerFactory<Integer, String> producerFactory, String topic) {
    return Kafka
            .outboundChannelAdapter(producerFactory)
            .messageKey(m -> m
                    .getHeaders()
                    .get(IntegrationMessageHeaderAccessor.SEQUENCE_NUMBER))
            .headerMapper(mapper())
            .partitionId(m -> 10)
            .topicExpression("headers[kafka_topic] ?: '" + topic + "'")
            .configureKafkaTemplate(t -> t.id("kafkaTemplate:" + topic));
}
```

If a `send-failure-channel` is provided, if a send failure is received (sync or async), an `ErrorMessage` is sent to the channel.The payload is a `KafkaSendFailureException` with properties `failedMessage`, `record` (the `ProducerRecord`) and `cause`.The `DefaultErrorMessageStrategy` can be overridden via the `error-message-strategy` property.

If a `send-success-channel` is provided, a message with a payload of type `org.apache.kafka.clients.producer.RecordMetadata` will be sent after a successful send.When using Java configuration, use `setOutputChannel` for this purpose.

### 5.1.3 Message Driven Channel Adapter

The `KafkaMessageDrivenChannelAdapter` (`<int-kafka:message-driven-channel-adapter>`) uses a `spring-kafka` `KafkaMessageListenerContainer` or `ConcurrentListenerContainer`.

从spring-integration-kafka version 2.1开始，模式属性可设置了（`record`或`batch`，默认为`record` )。

 `record` 模式下，每个message payload都是从一个 `ConsumerRecord`转换过来的

 `batch` 模式下，

payload 是一个列表的对象，转换从所有ConsumerRecord年代返回的消费者调查。与批处理@KafkaListener KafkaHeaders。RECEIVED_MESSAGE_KEY KafkaHeaders。RECEIVED_PARTITION_ID KafkaHeaders。RECEIVED_TOPIC KafkaHeaders。抵消头也列出,载荷与位置相对应的位置。

Starting with *spring-integration-kafka version 2.1*, the `mode` attribute is available (`record` or `batch`, default `record`).

For `record` mode, each message payload is converted from a single `ConsumerRecord`; 

for mode `batch` the payload is a list of objects which are converted from all the `ConsumerRecord` s returned by the consumer poll.

As with the batched `@KafkaListener`, the `KafkaHeaders.RECEIVED_MESSAGE_KEY`, `KafkaHeaders.RECEIVED_PARTITION_ID`, `KafkaHeaders.RECEIVED_TOPIC` and `KafkaHeaders.OFFSET` headers are also lists with, positions corresponding to the position in the payload.

An example of xml configuration variant is shown here:

```
<int-kafka:message-driven-channel-adapter
        id="kafkaListener"
        listener-container="container1"
        auto-startup="false"
        phase="100"
        send-timeout="5000"
        mode="record"
        retry-template="template"
        recovery-callback="callback"
        error-message-strategy="ems"
        channel="someChannel"
        error-channel="errorChannel" />

<bean id="container1" class="org.springframework.kafka.listener.KafkaMessageListenerContainer">
    <constructor-arg>
        <bean class="org.springframework.kafka.core.DefaultKafkaConsumerFactory">
            <constructor-arg>
                <map>
                <entry key="bootstrap.servers" value="localhost:9092" />
                ...
                </map>
            </constructor-arg>
        </bean>
    </constructor-arg>
    <constructor-arg>
        <bean class="org.springframework.kafka.listener.config.ContainerProperties">
            <constructor-arg name="topics" value="foo" />
        </bean>
    </constructor-arg>

</bean>
```

When using Java Configuration:

```
@Bean
public KafkaMessageDrivenChannelAdapter<String, String>
            adapter(KafkaMessageListenerContainer<String, String> container) {
    KafkaMessageDrivenChannelAdapter<String, String> kafkaMessageDrivenChannelAdapter =
            new KafkaMessageDrivenChannelAdapter<>(container, ListenerMode.record);
    kafkaMessageDrivenChannelAdapter.setOutputChannel(received());
    return kafkaMessageDrivenChannelAdapter;
}

@Bean
public KafkaMessageListenerContainer<String, String> container() throws Exception {
    ContainerProperties properties = new ContainerProperties(this.topic);
    // set more properties
    return new KafkaMessageListenerContainer<>(consumerFactory(), properties);
}

@Bean
public ConsumerFactory<String, String> consumerFactory() {
    Map<String, Object> props = new HashMap<>();
    props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, this.brokerAddress);
    // set more properties
    return new DefaultKafkaConsumerFactory<>(props);
}
```

When using Spring Integration Java DSL:

```
@Bean
public IntegrationFlow topic1ListenerFromKafkaFlow() {
    return IntegrationFlows
            .from(Kafka.messageDrivenChannelAdapter(consumerFactory(),
                    KafkaMessageDrivenChannelAdapter.ListenerMode.record, TEST_TOPIC1)
                    .configureListenerContainer(c ->
                            c.ackMode(AbstractMessageListenerContainer.AckMode.MANUAL)
                                    .id("topic1ListenerContainer"))
                    .recoveryCallback(new ErrorMessageSendingRecoverer(errorChannel(),
                            new RawRecordHeaderErrorMessageStrategy()))
                    .retryTemplate(new RetryTemplate())
                    .filterInRetry(true))
            .filter(Message.class, m ->
                            m.getHeaders().get(KafkaHeaders.RECEIVED_MESSAGE_KEY, Integer.class) < 101,
                    f -> f.throwExceptionOnRejection(true))
            .<String, String>transform(String::toUpperCase)
            .channel(c -> c.queue("listeningFromKafkaResults1"))
            .get();
}
```

Received messages will have certain headers populated.Refer to the `KafkaHeaders` class for more information.

| ![[Important]](https://docs.spring.io/spring-kafka/reference/htmlsingle/images/important.png) | Important |
| ---------------------------------------- | --------- |
| The `Consumer` object (in the `kafka_consumer` header) is not thread-safe; you must only invoke its methods on the thread that calls the listener within the adapter; if you hand off the message to another thread, you must not call its methods. |           |

When a `retry-template` is provided, delivery failures will be retried according to its retry policy.An `error-channel` is not allowed in this case.The `recovery-callback` can be used to handle the error when retries are exhausted.In most cases, this will be an `ErrorMessageSendingRecoverer` which will send the `ErrorMessage` to a channel.

When building `ErrorMessage` (for use in the `error-channel` or `recovery-callback`), you can customize the error message using the `error-message-strategy` property.By default, a `RawRecordHeaderErrorMessageStrategy` is used; providing access to the converted message as well as the raw `ConsumerRecord`.

### 5.1.4 Message Conversion

A `StringJsonMessageConverter` is provided, see [Section 4.1.4, “Serialization/Deserialization and Message Conversion”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#serdes) for more information.

当使用这个转换器驱动通道适配器，你可以指定你想要转换的incoming payload的类型。

这可以在adapter设置payload type属性(`payloadType` 性能)达到。

When using this converter with a message-driven channel adapter, you can specify the type to which you want the incoming payload to be converted.This is achieved by setting the `payload-type` attribute (`payloadType` property) on the adapter.

```
<int-kafka:message-driven-channel-adapter
        id="kafkaListener"
        listener-container="container1"
        auto-startup="false"
        phase="100"
        send-timeout="5000"
        channel="nullChannel"
        message-converter="messageConverter"
        payload-type="com.example.Foo"
        error-channel="errorChannel" />

<bean id="messageConverter"
    class="org.springframework.kafka.support.converter.MessagingMessageConverter"/>
```

```
@Bean
public KafkaMessageDrivenChannelAdapter<String, String>
            adapter(KafkaMessageListenerContainer<String, String> container) {
    KafkaMessageDrivenChannelAdapter<String, String> kafkaMessageDrivenChannelAdapter =
            new KafkaMessageDrivenChannelAdapter<>(container, ListenerMode.record);
    kafkaMessageDrivenChannelAdapter.setOutputChannel(received());
    kafkaMessageDrivenChannelAdapter.setMessageConverter(converter());
    kafkaMessageDrivenChannelAdapter.setPayloadType(Foo.class);
    return kafkaMessageDrivenChannelAdapter;
}
```

### 5.1.5 What’s New in Spring Integration for Apache Kafka

See the [Spring for Apache Kafka Project Page](https://projects.spring.io/spring-kafka/) for a matrix of compatible `spring-kafka` and `kafka-clients` versions.

#### 2.1.x

The 2.1.x branch introduced the following changes:

- Update to `spring-kafka` 1.1.x; including support of batch payloads
- Support `sync` outbound requests via XML configuration
- Support `payload-type` for inbound channel adapters
- Support for Enhanced Error handling for the inbound channel adapter (2.1.1)
- Support for send success/failure messages (2.1.2)

#### 2.2.x

The 2.2.x branch introduced the following changes:

- Update to `spring-kafka` 1.2.x

#### 2.3.x

The 2.3.x branch introduced the following changes:

- Update to `spring-kafka` 1.3.x; including support for transactions and header mapping provided by `kafka-clients` 0.11.0.0
- Support for record timestamps

#### 3.0.x

- Update to `spring-kafka` 2.1.x and `kafka-clients` 1.0.0
- Support `ConsumerAwareMessageListener` (`Consumer` is available in a message header)
- Update to Spring Integration 5.0 and Java 8
- Moved Java DSL to main project

# 6. Other Resources

In addition to this reference documentation, there exist a number of other resources that may help you learn aboutSpring and Apache Kafka.

- [Apache Kafka Project Home Page](https://kafka.apache.org/)
- [Spring for Apache Kafka Home Page](https://projects.spring.io/spring-kafka/)
- [Spring for Apache Kafka GitHub Repository](https://github.com/spring-projects/spring-kafka)
- [Spring Integration Kafka Extension GitHub Repository](https://github.com/spring-projects/spring-integration-kafka)

# Appendix A. Change History

## A.1 Changes Between 1.3 an 2.0

### A.1.1 Spring Framework and Java Versions

The Spring for Apache Kafka project now requires Spring Framework 5.0 and Java 8.

### A.1.2 @KafkaListener Changes

You can now annotate `@KafkaListener` methods (and classes, and `@KafkaHandler` methods) with `@SendTo`.If the method returns a result, it is forwarded to the specified topic.See [the section called “Forwarding Listener Results using @SendTo”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#annotation-send-to) for more information.

### A.1.3 Message Listeners

Message listeners can now be aware of the `Consumer` object.See [the section called “Message Listeners”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#message-listeners) for more information.

### A.1.4 ConsumerAwareRebalanceListener

Rebalance listeners can now access the `Consumer` object during rebalance notifications.See [the section called “Rebalance Listeners”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#rebalance-listeners) for more information.

### A.1.5 @EmbeddedKafka Annotation

For convenience a test class level `@EmbeddedKafka` annotation is provided with the purpose to register `KafkaEmbedded` as a bean.See [Section 4.3, “Testing Applications”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#testing) for more information.=== Changes Between 1.2 and 1.3

### A.1.6 Support for Transactions

The 0.11.0.0 client library added support for transactions; the `KafkaTransactionManager` and other support for transactions has been added.See [the section called “Transactions”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#transactions) for more information.

### A.1.7 Support for Headers

The 0.11.0.0 client library added support for message headers; these can now be mapped to/from `spring-messaging` `MessageHeaders`.See [Section 4.1.5, “Message Headers”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#headers) for more information.

### A.1.8 Creating Topics

The 0.11.0.0 client library provides an `AdminClient` which can be used to create topics.The `KafkaAdmin` uses this client to automatically add topics defined as `@Bean` s.

### A.1.9 Support for Kafka timestamps

`KafkaTemplate` now supports API to add records with timestamps.New `KafkaHeaders` have been introduced regarding `timestamp` support.Also new `KafkaConditions.timestamp()` and `KafkaMatchers.hasTimestamp()` testing utilities have been added.See [the section called “KafkaTemplate”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#kafka-template), [the section called “@KafkaListener Annotation”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#kafka-listener-annotation) and [Section 4.3, “Testing Applications”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#testing) for more details.

### A.1.10 @KafkaListener Changes

You can now configure a `KafkaListenerErrorHandler` to handle exceptions.See [Section 4.1.7, “Handling Exceptions”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#annotation-error-handling) for more information.

By default, the `@KafkaListener` `id` property is now used as the `group.id` property, overriding the property configured in the consumer factory (if present).Further, you can explicitly configure the `groupId` on the annotation.Previously, you would have needed a separate container factory (and consumer factory) to use different `group.id` s for listeners.To restore the previous behavior of using the factory configured `group.id`, set the `idIsGroup` property on the annotation to `false`.

### A.1.11 @EmbeddedKafka Annotation

For convenience a test class level `@EmbeddedKafka` annotation is provided with the purpose to register `KafkaEmbedded` as a bean.See [Section 4.3, “Testing Applications”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#testing) for more information.

### A.1.12 Kerberos Configuration

Support for configuring Kerberos is now provided.See [Section 4.1.8, “Kerberos”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#kerberos) for more information.

## A.2 Changes between 1.1 and 1.2

This version uses the 0.10.2.x client.

## A.3 Changes between 1.0 and 1.1

### A.3.1 Kafka Client

This version uses the Apache Kafka 0.10.x.x client.

### A.3.2 Batch Listeners

Listeners can be configured to receive the entire batch of messages returned by the `consumer.poll()` operation, rather than one at a time.

### A.3.3 Null Payloads

Null payloads are used to "delete" keys when using log compaction.

### A.3.4 Initial Offset

When explicitly assigning partitions, you can now configure the initial offset relative to the current position for the consumer group, rather than absolute or relative to the current end.

### A.3.5 Seek

You can now seek the position of each topic/partition.This can be used to set the initial position during initialization when group management is in use and Kafka assigns the partitions.You can also seek when an idle container is detected, or at any arbitrary point in your application’s execution.See [the section called “Seeking to a Specific Offset”](https://docs.spring.io/spring-kafka/reference/htmlsingle/#seek) for more information.