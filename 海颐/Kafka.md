# Kafka

### 一、概述

Apache Kafka是一个分布式发布 - 订阅消息系统和一个强大的队列，可以处理大量的数据，并能够将消息从一个端点传递到另一个端点

#### 消息模式

**点对点**

在点对点系统中，消息被保留在队列中，可以有一个或多个消费者消费队列中的消息，但特定消息只能由最多一个消费者消费，一旦消费者读取队列中的消息，它就从该队列中消失 (如订单处理系统)

**发布-订阅**

在发布-订阅系统中，消息被保留在主题中。消费者可以订阅一个或多个主题并使用该主题中的所有消息。

#### 好处

- 可靠性：Kafka是分布式、分区、复制和容错的
- 可扩展性：Kafka消息传递系统轻松缩放，无需停机
- 耐用性：Kafka使用"分布式提交日志"，这意味着消息会尽可能快的保留在磁盘上，因此它是持久的
- 性能：Kafka对于发布和订阅消息都具有高吞吐量，及时存储了许多TB的消息，也能保持稳定的性能

Kafka速度非常快，并能保证零停机和零数据丢失

#### 相关概念

1、Producer (生产者)：消息的发送者

2、Consumer (消费者)：消息的使用者和接受者

3、Broker：Kafka集群中有很多台Server，每台Server都可以存储消息，将每台Server成为一个Kafka实例，也叫作Broker

4、Topic (主题)：一个topic里保存的是同一类消息，相当于消息的分类

5、Partition (分区)：每个topic都可以分成多个partition，每个partition在存储层面是append log文件，任何发布到此partition的消息都会被追加到log文件的稳步

6、Offset (偏移量)：一个分区对应一个磁盘上的文件，而消息在文件中的位置就是偏移量，long型 (Kafka无其他额外的索引机制来存储offset，只能顺序读写)



**分区原因**

Kafka基于文件进行存储，当文件内容大到一定程度时，很容易达到单个磁盘的容量上限，因此，采用分区的方法，一个分区对应一个文件，这样就可以将数据分别存储到不同的server上，另外这样做也可以实现负载均衡，容纳更多的消费者



#### 发布-订阅消息的工作流程

1. 生产者定期想主题发送消息
2. Kafka代理存储为该特定主题配置的分区中的所有消息。它确保消息在分区之间平等共享。如果生产者发送两个消息并且有两个分区，Kafka将在第一分区中存储一个消息，在第二分区中存储第二个消息
3. 消费者订阅特定主题
4. 一旦消费者订阅主题，Kafka将向消费者提供主题的当前偏移，并且还将偏移保存在Zookeeper系统中
5. 消费者将定期请求Kafka里的新消息
6. 一旦Kafka收到来自生产者的消息，它将这些消息转发给消费者
7. 消费者将收到消息并进行处理
8. 一旦消息被处理，消费者将向Kafka代理发送确认
9. 一旦Kafka收到确认，它将偏移量更新，并在Zookeeper中更新它。由于偏移在Zookeeper中维护，消费者可以正确的读取下一封邮件，即使在服务器暴力期间
10. 以上流程将重复，直到消费者停止请求
11. 消费者可以随时回退 / 跳到所需的主题偏移量，并阅读所有后续的消息

#### 消息队列 / 用户组的工作流

订阅具有相同 Group ID 的主题的消费者被认为是单个组，消息在它们之间共享

1. 生产者以固定间隔向某个主题发送消息
2. Kafka存储在为该特定主题配置的分区中的所有消息，类似于前面的方案
3. 单个消费者订阅特定主题，假设主题为 Topic-01 ， Group ID 为 Group-01
4. Kafka以与发布-订阅消息相同的方式与消费者交互，知道新消费者以相同的 Group ID 订阅相同主题 Topic-01
5. 一旦新消费者到达，Kafka将其操作切换到共享模式，并在两个消费者之间共享数据。此共享将继续，直到用户数达到该特定主题配置的分区数
6. 一旦消费者的数量超过分区的数量，新消费者将不收接收到任何进一步的消息，直到现有消费者取消订阅任何一个消费者。出现这种情况是因为Kafka中的每个消费者将被分配至少一个分区，并且一旦所有分区被分配给现有消费者，新消费者必须等待

此功能也称为使用者组。



### 二、安装步骤

#### 1、安装Java 8

#### 2、安装Zookeeper

1. [下载Zookeeper](http://zookeeper.apache.org/releases.html)(zookeeper-3.4.6.tar.gz)

2. 解压

   ```shell
   # 解压到指定目录
   $ tar zxvf zookeeper-3.4.6.tar.gz -C /usr/local
   
   $ cd /usr/local
   
   # 重命名
   $ mv zookeeper-3.4.6 zookeeper
   
   $ cd zookeeper
   
   # 创建数据文件夹
   $ mkdir data
   
   # 复制 conf/zoo_sample.cfg 配置文件的内容
   $ cat conf/zoo_sample.cfg
   
   # 新建 zoo.cfg 配置文件，将上面复制的内容粘贴到此文件中，并修改以下参数
   $ vim conf/zoo.cfg
   # tickTime=2000
   # dataDir=/path/to/zookeeper/data
   # clientPort=2181
   # initLimit=5
   # syncLimit=2
   
   ```

3. 启动Zookeeper服务器

   ```shell
   # 启动服务器，启动成功将得到以下输出
   $ bin/zkServer.sh start
   JMX enabled by default
   Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
   Starting zookeeper ... STARTED
   
   # 启动CLI，启动成功将得到以下输出
   $ bin/zkCli.sh
   Connecting to localhost:2181
   ................
   ................
   ................
   Welcome to ZooKeeper!
   ................
   ................
   WATCHER::
   WatchedEvent state:SyncConnected type: None path:null
   ```

4. 停止Zookeeper服务器

   ```shell
   $ bin/zkServer.sh stop
   ```

#### 3、安装Kafka

1. [下载Kafka](https://kafka.apache.org/downloads.html) (如 kafka_2.12-2.8.0.tgz)

   - **注意**：勿下载版本为 kafka_\*\*-0.9.0.\*.tgz 的版本，此版本太低，它支持的最大request id为16，而新版SDK的 request id 可到18，若用新版SDK访问旧版本Kafka，将会导致数组下标越界异常

     ```shell
     [2018-11-23 15:35:14,958] ERROR Processor got uncaught exception. (kafka.network.Processor)
     java.lang.ArrayIndexOutOfBoundsException: 18
         at org.apache.kafka.common.protocol.ApiKeys.forId(ApiKeys.java:68)
         at org.apache.kafka.common.requests.AbstractRequest.getRequest(AbstractRequest.java:39)
         at kafka.network.RequestChannel$Request.<init>(RequestChannel.scala:79)
         at kafka.network.Processor$$anonfun$run$11.apply(SocketServer.scala:426)
         at kafka.network.Processor$$anonfun$run$11.apply(SocketServer.scala:421)
         at scala.collection.Iterator$class.foreach(Iterator.scala:727)
         at scala.collection.AbstractIterator.foreach(Iterator.scala:1157)
         at scala.collection.IterableLike$class.foreach(IterableLike.scala:72)
         at scala.collection.AbstractIterable.foreach(Iterable.scala:54)
         at kafka.network.Processor.run(SocketServer.scala:421)
         at java.lang.Thread.run(Thread.java:748)
     ```

2. 解压

   ```shell
   # 解压到指定目录
   $ tar zxvf kafka_2.12-2.8.0.tgz -C /usr/local
   
   $ cd /usr/local
   
   # 重命名
   $ mv kafka_2.12-2.8.0 kafka
   
   ```

3. 启动服务器

   ```shell
   # 如需配置其他网络访问该消息队列，则要添加 advertised.listeners 配置
   $ vim config/server.properties
   # 修改
   listeners=PLAINTEXT://192.168.15.130:9092
   # 添加
   advertised.listeners=PLAINTEXT://192.168.15.130:9092
   
   # 启动服务器，启动成功后，将看到类似如下输出
   $ bin/kafka-server.start.sh config/server.properties
   [2021-09-14 15:40:18,259] INFO KafkaConfig values: 
   	advertised.host.name = null
   	advertised.listeners = PLAINTEXT://192.168.15.130:9092
   	advertised.port = null
   	alter.config.policy.class.name = null
   	alter.log.dirs.replication.quota.window.num = 11
   	alter.log.dirs.replication.quota.window.size.seconds = 1
   	authorizer.class.name = 
   	auto.create.topics.enable = true
   	................
   	................
   ```

4. 停止服务器

   ```shell
   $ bin/kafka-server-stop.sh config/server.properties
   ```



























