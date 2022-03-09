---
title: "Kafka"
date: 2021-08-01T17:59:44+08:00
draft: true
author: "spider"
toc: true
tags:
  - kafka
---
深入理解 Kafka

``消息系统、存储系统、流式处理平台``

## 基本概念


经典Kafka体系架构包括若干Producer、若干 Broker、若干 Consumer，以及一个 Zookeeper 集群（可插拔）
Zookeeper 负责集群元数据管理、控制器的选举
Producer 将消息发送到 Broker
Broker 负责将收到的消息存储到磁盘中，代理服务
Consumer 负责从 Broker 订阅并消费消息

Kafka 消息都是以主题为单位进行归类，生产者负责将消息发送到特定的主题(发送到Kafka 集群中的每一条消息都需要指定一个主题)
主题是逻辑上的概念，还可以细分为多个分区，一个分区只属于单个主题，也叫做主题分区(Topic Pariition)。

同一个主题下的不同分区包含的消息是不同的，分区在存储层面可以看作是一个可追加的日志文件，消息在追加到分区日志文件的时候都会分配一个特定的偏移量
offset 是消息在分区中的唯一标识，Kafka 通过它来保证消息在分区内的顺序性，不过offset并不跨越分区，所以保证的是分区有序，
分区解决的主题的IO，创建主题可以通过指定参数设置分区个数，也可以在主题创建完成修改分区的数量，通过增加分区数量实现水平拓展。

Kakfa 为分区引入了多副本 (Replica)机制，通过增加副本数量可以提升容灾能力。
同一分区的不同副本中保存的是相通的消息（在同一时刻，副本之间并非完全一样），副本之间是一主多从的关系
leader 节点负责处理读写请求，follower 副本只负责与 leader 副本的消息同步
副本处于不同的 broker 中，当 leader 副本出现故障时，从 follower 副本中重新选举新的 leader 副本对外提供服务
Kafka 通过多副本机制实现了故障的自动转移，当 Kafka 集群中某个 broker 失效时仍然能够保证服务可用。

Kafka 消费端也具备一定的容灾能力，Consumer 使用拉（Pull）模式从服务端拉取消息，并且保存消费的具体位置，当消费者宕机后恢复上线时可以根据之前保存的消费位置重新拉取需要的消息进行消费,不会造成消息丢失。

AR  分区中的所有副本 Assigned Replicas
ISR 与leader副本保持一定程度同步的副本(包括leader副本在内)组成 In-Sync Replicas
OSR 与leader 副本同步滞后过多的副本组成 Out-of-Sync
AR=ISR+OSR 正常情况下 OSR=nil
leader 副本负责维护和跟踪ISR集合中所有 follower 副本的滞后状态，当 follower 副本落后太多或者失效时，leader 副本会把他从 ISR 集合中剔除，如果 OSR 集合中有 follower 副本追上了 leader 副本，那么 leader 副本会把他从 OSR 集合转移到 ISR 集合。
默认情况下，只有在 ISR 集合中的副本才有资格被选举为新的 leader，OSR 集合中的副本则没有任何机会(可以修改配置改变)。

ISR 与 HW、LEO
HW (high watermark),高水位，标识了一个特定的消息偏移量，消费者只能拉取到这个 offset 之前的消息，后面消息不可见
LEO(Log End Offset) 标识当前日志文件中下一条代写入消息的 Offset，LEO 大小相当于当前日志分区中最后一条消息的 Offset +1
分区ISR 集合中的每个副本都会维护自身的 LEO，而ISR集合中最小的LEO极为HW,对消费者而言只能消费HW之前的消息。

HW 通过 follower 副本同步完数据保证了数据的一致，即不是完全的同步复制也不是单纯的异步复制


# 生产者

## kafka服务端参数配置

zookeeper.connect
broker 要链接的 zookeeper 集群的服务地址(包含端口号)，必填无默认值
chroot 路径 c1:2181,c2:2181/kafka

listeners
broker 监听客户端连接的地址列表，客户端连接 broker 的入口地址列表
格式： protocol1://hostname0:port0,protocol2://hostname1:port1
protocoal 协议类型 支持类型 PLAINTEXT,SSL,SASL_SSL
未开启安全认证，使用简单的 PLAINTEXT ，hostname 主机名默认为空，不知定主机名默认绑定默认网卡（127.0.0.1 就可能无法对外提供服务）。

advertised.listeners 主要用于 Iaas(Infrastructure as a Service)环境，公网只配置这个
listeners 绑定私网 broker 间通信

broker.id
指定集群中 broker 唯一标示，默认-1,如果没有设置自动生成一个，和 meta.properties、服务端参数 broker.id.generation.enable 、reserved.broker.max.id 有关


log.dir log.dirs
kafka 把所有的消息报存在磁盘上，log.dir 配置kafka 日志文件存放的根目录
log.dir 配置单个根目录，log.dirs 用来配置多个根目录(逗号分隔)
log.dirs 优先级高，没有配置 log.dirs 会以 log.dir 配置为准



message.max.bytes
broker 所能接受消息的最大值，默认值为 1000012(B)，～ 976.6kb
如果 Producer 发送的消息大于这个参数设置的值，Producer 会报出 RecordToolLargeException 异常
修改此参数还要考虑 max.request.size 客户端参数、max.message.bytes(topic 端参数)等参数的影响
为避免修改此参数而引起的级联影响，建议考虑分拆消息的可行性。


## 生产者客户端

1.配置生产者客户端参数及创建相应的生产者实例
2.构建待发送的消息
3.发送消息
4.关闭生产者实例

KafkaProducer 是线程安全的，send 方法异步发送，消息存储到缓冲区会立即返回
发送的结果是一个RecordMetadata，指定记录被发送到的分区，它被分配的偏移量和记录的时间戳。
producer.send(record).get() 阻塞实现同步发送
get(long timeout,TimeUnit unit) Future 实现超时同步阻塞
异常：
1.NetWorkException
2.LeaderNotAvailiableException 分区leader 副本不可用，发生在副本下线，新leader 副本上线前，需要重试恢复
3.UnknowTopicOrPartitionExceptop
3.NotEnoughReplicasException
4.NotCoordinatorException
5.RecordTooLargeException 不会进行重试，直接抛出异常
properties 配置 ProducerConfig.RETRIES_CONFIG
消息发送模式
1.发后即忘 fire-and-forget
2.同步 sync
3.异步


```java
producer.send(record, (metadata, exception) -> {
                //metadata, exception 是互斥的
                if (exception != null) {
                    //do something log...
                    exception.printStackTrace();
                } else {
                    //
                    System.out.println(metadata.topic() + "-" + metadata.partition() + ":" + metadata.offset());
                }
            });
//对于同一个分区，如果消息recored1 先发送，Kafka 可以保证对应的 callback 先调用
producer.send(record1,callBack1)
producer.send(record2,callBack2)
//会阻塞等待之前所有的发送请求完成后在关闭 KafkaProduce
producer.close(10, TimeUnit.SECONDS);//超时关闭
```

## ProducerRecord

```
构建消息，创建 ProducerRecord对象，topic,value 属性是必填项
针对不同消息需要构建不同的 ProducerRecord 对象
 /**
         * public class ProducerRecord<K, V> {
         *     private final String topic; 主题
         *     private final Integer partition; 分区号
         *     private final Headers headers; 消息同步
         *     private final K key; 键 指定消息的键，可以用来计算分区号，可以对消息进行二次归类
         *     private final V value; 值
         *     private final Long timestamp; 消息的时间戳
         *}
         */
同一个 key 的消息会被划分到同一个分区         
```

## 序列化

```
生产者需要用序列化器把对象换成字节数组才能通过网络发送给 Kafka，消费端需要用反序列化器把从 Kafka 中收到的字节数组转换成相应的对象
序列化器 数据类型 String,ByteArray,ByteBuffer,Bytes,Double,Integer,Long
```

```java
    Serializer 接口
		/**
     * Configure this class.
     * @param configs configs in key/value pairs
     * @param isKey whether is for key or value
     */
		void configure(Map<String, ?> configs, boolean isKey);

    /**
     * Convert {@code data} into a byte array.
     *
     * @param topic topic associated with data
     * @param data typed data
     * @return serialized bytes
     */
    byte[] serialize(String topic, T data);

    /**
     * Close this serializer.
     * 实现需要保证幂等性
     * This method must be idempotent as it may be called multiple times.
     */
    @Override
    void close();
```

## 分区器 partitioner

```
消息在通过 send 方法发往 broker 过程中，有可能需要经过拦截器 interceptor、序列化器 serializer 、分区器 partitioner 的一系列作用后才能真正被发往 broker，拦截器不必须，序列化器是必须的。
消息经过序列化之后就需要确定他发往的分区，如果消息 ProducerRecord 中指定了 partition 字段，就不需要分区器作用，没有指定就需要依赖分区器，根据 key 计算 partition 值，分区器的作用就是为消息分配分区。
默认分区器 org.apache.kafka.clients.producer.internals.DefaultPartitioner
```

```java
   implements Paritioner -> implements Configurable ,method  config 获取配置信息、初始化
		/**
     * Compute the partition for the given record.
     *
     * @param topic The topic name
     * @param key The key to partition on (or null if no key)
     * @param keyBytes The serialized key to partition on( or null if no key)
     * @param value The value to partition on or null
     * @param valueBytes The serialized value to partition on or null
     * @param cluster The current cluster metadata
     */
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster);

    /**
     * This is called when partitioner is closed.
     */
    public void close();
```

```java
 /**
     * Compute the partition for the given record.
     *
     * @param topic The topic name
     * @param key The key to partition on (or null if no key)
     * @param keyBytes serialized key to partition on (or null if no key)
     * @param value The value to partition on or null
     * @param valueBytes serialized value to partition on or null
     * @param cluster The current cluster metadata
     */
    public int partition(String topic, Object key, byte[] keyBytes, Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        if (keyBytes == null) {
            int nextValue = nextValue(topic);
            List<PartitionInfo> availablePartitions = cluster.availablePartitionsForTopic(topic);
            // keyBytes 为 null，并且有可用分区，计算得到分区仅为可用分区中的任意一个
            if (availablePartitions.size() > 0) {
                int part = Utils.toPositive(nextValue) % availablePartitions.size();
                return availablePartitions.get(part).partition();
            } else {
                // no partitions are available, give a non-available partition
                return Utils.toPositive(nextValue) % numPartitions;
            }
        } else {
            // hash the keyBytes to choose a partition
            // 序列化后的 key 不为空，使用 murmurhash2 对 keyBytes 进行哈希计算分区号，拥有相同分区号被写入一个分区
            return Utils.toPositive(Utils.murmur2(keyBytes)) % numPartitions;
        }
    }
```

```
不改变主题分区数量情况下，key 与分区之间的映射可以保持不变
一旦主题增加了分区，就难以保证 key 与分区之间的映射关系
实现 Partitioner 接口，可以使得 当 key 不存在是选择非可用的分区 ？？？？？
```

## 拦截器 interceptor

```
生产者、消费者拦截器，可以在发送前做一些准备，eg 按照某个规则过滤不符合要求的消息，修改消息的内容，也可以在发送回调逻辑前做一些定制化
```

## producerInterceptor

```java
implements org.apache.kafka.clients.producer.ProducerInterceptor
这三个方法抛出的异常都会被捕获并且记录到日志中，不会向上传递

  //将消息序列化和计算分区之前进行定制化操作，一般不要修改消息 ProductRecord topic、key、partition
  //修改 key 不仅影响分区，还以影响 broker 端日志压缩
	public ProducerRecord<K, V> onSend(ProducerRecord<K, V> record);
	// 在消息被应答 acknowledgement 之前或消息被发送失败时调用生产者拦截器的 onAcknowledgement() 方法
	// 优先于用户设定的 callBack 执行，方法运行于 producer 的 io 线程中，所以逻辑越简单越好
  public void onAcknowledgement(RecordMetadata metadata, Exception exception);
  public void close();
```

```java
//拦截器,可以多个构成拦截链,如果某个执行失败，下一个拦截器会接着从上一个执行成功的拦截器继续执行
        properties.put(ProducerConfig.INTERCEPTOR_CLASSES_CONFIG, CompanyInterceptorPlus.class.getName() +
                "," + CompanyInterceptorPrefix.class.getName());

/**
 * 自定义拦截器
 */
public class CompanyInterceptorPrefix implements ProducerInterceptor<String, String> {

    private volatile long sendSuccess = 0;
    private volatile long sendFailure = 0;

    @Override
    public ProducerRecord onSend(ProducerRecord record) {
        String modifiedValue = "prefix-" + record.value();
        return new ProducerRecord<>(record.topic(),
                record.partition(), record.timestamp(), record.key(), modifiedValue, record.headers());
    }

    @Override
    public void onAcknowledgement(RecordMetadata metadata, Exception exception) {
        if (Objects.isNull(exception)) {
            sendSuccess++;
        } else {
            sendFailure++;
        }
    }

    @Override
    public void close() {
        double successRatio = (double) sendSuccess / (sendFailure + sendSuccess);
        System.out.println("[INFO] 发送成功率=" + String.format("%f", successRatio * 100) + "%");
    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}

```

## 整体架构


主线程
->
KafkaProducer ->1.拦截器->2.序列化器->3.分区器->4.消息累加器 RecordAccumulator
sender 线程
->
sender->1.创建Request->2.缓存->3.提交给 Selector 准备发送 -> 4.[kafka-cluster]->5.selector - response->6.清理

主线程由 KafkaProducer 创建消息，通过拦截器、序列化器、分区器，缓存到消息累加器(RecordAccumulator)
sender 线程负责从 RecordAccmulator 中获取消息并发送到 Kafka
RecordAccumulator 主要用来缓存消息以便 sender 线程可以批量发送，进而减少网络传输的消耗以提升性能
RecordAccumulator 缓存的大小可以通过生产者客户端参数 buffer.memory 配置，默认 32MB

1.如果生产者发送消息的速度超过发送到服务器的速度，生产者可能空间不足，send() 调用要么阻塞要么抛出异常，取决于 max.block.ms 默认60s
2.主线程中发送过来的消息都会被追加到 RecordAccumulator 的某个双端队列中，RecordAccumulator 内部为每个分区都维护了一个双端队列，队列中的内容就是 ProducerBatch,Deque<ProducerBatch>
3.消息写入缓存时，追加到双端队列的尾部
4.Sender 读取消息时，从双端的头部读取，ProducerBatch 由多个 ProducerRecord 组成
5.可以调大 buffer.memory 增加整体的吞吐量
6.Kafka 生产者客户端，通过 java.io.ByteBuffer 实现消息的内存创建和释放，RecordAcumulator 内部由一个 BufferPool
复用ByteBuffer实现缓存的高效利用，避免频繁创建和释放。
BufferPool 只针对特定大小的 ByteBuffer 进行管理，其他大小的 ByteBuffer 不会缓存进 BufferPool,batch.size 默认 16kb。

ProducerBatch 大小和 batch.size 关系
1.当一条消息(ProducerRecord) 流入 RecordAccmulator 时，会先寻找与消息分区所对应的双端队列(如果没有新建)
2.在从双端队列尾部获取一个 ProducerBatch(没有则新建)，查看 ProducerBatch 中是否还可以写入这个 ProducerRecord
3.如果可以则写入，不可以则需要创建一个新的 ProducerBatch
4.新建 ProducerRecord 时评估这条消息大小是否超过 batch.size 大西哦啊，如果不超过就以 batch.szie 大小来创建 ProducerBatch
5.这样使用完这段内存区域后，可以通过 BufferPool 的管理来进行复用，如果超过，就以评估的大小来创建 ProducerRecord，这段内存区域不会被复用。

sender 从 RecordAccumulator 中获取缓存的消息之后，会进一步将原本<分区,Deque<ProducerBatch> 的保存形式转变成<Node，List<ProducerBatch> 形式，其中 node 表示 Kafka 集群的 broker 节点，对于网络链接来说，生产者客户端是与具体的 broker 节点建立的链接，也就是向具体的 broker 节点发送消息，而并不关心消息属于哪一个分区
对于 KafkaProducer 的应用逻辑而言，只关注向哪个分区发送哪些消息，在这里需要做一个应用逻辑层面到网络IO层面的转换

转换成<Node，List<ProducerBatch>> 的形式之后，Sender 还会进一步封装成<Node,Request> 形式，这样就可以将Request(各种协议请求) 请求发往各个Node了
请求在从 sender 线程发往 kafka 之前还会保存到 InFlightRequests 中，InFlightRequests 保存对象的具体形式为<Map<NodeId,Deque<Request>>,主要作用是缓存已经发出去但还没有收到响应的请(NodeId 是一个String类型，表示节点的Id编号)
通过 max.in.flight.requests.per.connection 限制每个链接最多缓存的请求数，默认 5
超过限制请求数就不能再向这个链接发送更多的请求了，除非有缓存的请求收到了响应
通过比较 Deque<Request> 的 size 和限制链接参数大小判断对应的 Node 中是否已经堆积很多未响应的消息，排查Node节点负载/网络故障，继续发送可能会增大请求超时的可能。


### BufferPool

```java
public class BufferPool {

    static final String WAIT_TIME_SENSOR_NAME = "bufferpool-wait-time";
		//整个 pool 大小
    private final long totalMemory;
    //指定ByteBuffer 大小
    private final int poolableSize;
    //多线程并发分配和回收ByteBuffe
    private final ReentrantLock lock;
    //缓存指定大小的ByteBuffer对象
    private final Deque<ByteBuffer> free;
    //记录因申请不到足够空间而阻塞的线程，实际记录的是阻塞线程对应的Condition对象
    private final Deque<Condition> waiters;
    /** Total available memory is the sum of nonPooledAvailableMemory and the number of byte buffers in free * poolableSize.  */
    //可用的空间大小，totalMemory 减去free 列表中所有ByteBuffer的大小
    private long nonPooledAvailableMemory;
    private final Metrics metrics;
    private final Time time;
    private final Sensor waitTime;
}
```

```java
 /**
     * Allocate a buffer of the given size. This method blocks if there is not enough memory and the buffer pool
     * is configured with blocking mode.
     *
     * @param size The buffer size to allocate in bytes
     * @param maxTimeToBlockMs The maximum time in milliseconds to block for buffer memory to be available
     * @return The buffer
     * @throws InterruptedException If the thread is interrupted while blocked
     * @throws IllegalArgumentException if size is larger than the total memory controlled by the pool (and hence we would block
     *         forever)
     */
    public ByteBuffer allocate(int size, long maxTimeToBlockMs) throws InterruptedException {
        //消息大小大于 totalMemory 直接抛出异常
        if (size > this.totalMemory)
            throw new IllegalArgumentException("Attempt to allocate " + size
                                               + " bytes, but there is a hard limit of "
                                               + this.totalMemory
                                               + " on memory allocations.");

        ByteBuffer buffer = null;
        this.lock.lock();
        try {
            // check if we have a free buffer of the right size pooled
            // 如果请求的是poolableSize 大小的ByteBuffer，且free 中有空余的ByteBuffer
            if (size == poolableSize && !this.free.isEmpty())
                return this.free.pollFirst();

            // now check if the request is immediately satisfiable with the
            // memory on hand or if we need to block
            int freeListSize = freeSize() * this.poolableSize;
            //空闲内存 + 缓存队列内存 >= 消息大小
            if (this.nonPooledAvailableMemory + freeListSize >= size) {
                // we have enough unallocated or pooled memory to immediately
                // satisfy the request, but need to allocate the buffer
                //需要空闲内存大于消息才进行分配
                freeUp(size);
                this.nonPooledAvailableMemory -= size;
            } else {
                // we are out of memory and will have to block
                int accumulated = 0;
                Condition moreMemory = this.lock.newCondition();
                try {
                    long remainingTimeToBlockNs = TimeUnit.MILLISECONDS.toNanos(maxTimeToBlockMs);
                    this.waiters.addLast(moreMemory);
                    // loop over and over until we have a buffer or have reserved
                    // enough memory to allocate one
                    while (accumulated < size) {
                        long startWaitNs = time.nanoseconds();
                        long timeNs;
                        boolean waitingTimeElapsed;
                        try {
                            waitingTimeElapsed = !moreMemory.await(remainingTimeToBlockNs, TimeUnit.NANOSECONDS);
                        } finally {
                            long endWaitNs = time.nanoseconds();
                            timeNs = Math.max(0L, endWaitNs - startWaitNs);
                            this.waitTime.record(timeNs, time.milliseconds());
                        }

                        if (waitingTimeElapsed) {
                            throw new TimeoutException("Failed to allocate memory within the configured max blocking time " + maxTimeToBlockMs + " ms.");
                        }

                        remainingTimeToBlockNs -= timeNs;

                        // check if we can satisfy this request from the free list,
                        // otherwise allocate memory
                        if (accumulated == 0 && size == this.poolableSize && !this.free.isEmpty()) {
                            // just grab a buffer from the free list
                            buffer = this.free.pollFirst();
                            accumulated = size;
                        } else {
                            // we'll need to allocate memory, but we may only get
                            // part of what we need on this iteration
                            freeUp(size - accumulated);
                            int got = (int) Math.min(size - accumulated, this.nonPooledAvailableMemory);
                            this.nonPooledAvailableMemory -= got;
                            accumulated += got;
                        }
                    }
                    // Don't reclaim memory on throwable since nothing was thrown
                    accumulated = 0;
                } finally {
                    // When this loop was not able to successfully terminate don't loose available memory
                    this.nonPooledAvailableMemory += accumulated;
                    this.waiters.remove(moreMemory);
                }
            }
        } finally {
            // signal any additional waiters if there is more memory left
            // over for them
            try {
                if (!(this.nonPooledAvailableMemory == 0 && this.free.isEmpty()) && !this.waiters.isEmpty())
                    this.waiters.peekFirst().signal();
            } finally {
                // Another finally... otherwise find bugs complains
                lock.unlock();
            }
        }

        if (buffer == null)
            return safeAllocateByteBuffer(size);
        else
            return buffer;
    }
```

### 元数据更新


InFlightRequests 可以通过每个Node 未确认的请求获得 leastLoadedNode (负载最小)选择 leastLoadedNode 发送请求可以使它尽快发出，避免网络拥塞等异常而影响整体的进度。
leastLoadedNode 概念可以应用于多个场景，比如元数据请求，消费者组播协议的交互
最简单的创建 ProducerRecord 只指定 tpoic 和 record，只有主题名称，KafkaProducer 要将此消息追加到指定主题的某个分区对应的 leader 副本之前，需要首先知道主题的分区数量，然后经过计算得出(或者指定)目标分区，之后 KafkaProducer 需要知道目标分区的 leader 副本所在的 broker 节点的地址、端口等信息才能叫建立链接，最终才能将消息发送到 Kafka，这一过程中的所需要的信息都属于元数据的信息。
客户端只配置了 broker 节点的地址，不需要配置所有的节点地址，客户端可以自己发现其他 broker 节点的地址，这个过程也属于元数据相关的更新操作，分区数量和leader 副本的分布都会动态变化，客户端也要动态捕捉变化。

元数据(集群的元数据)
1.集群中主题
2.主题的分区
3.每个分区的副本分配在哪个节点上
4.follower 副本分配在哪些节点上
5.哪些副本在在 AR,ISR等集合中
6.集合中有哪些节点，控制器节点是哪一个

当客户端中没有需要使用的元数据时(没有指定的主题信息时)，或者超过 metadata.max.age.ms(默认300000 5分钟)都会引起元数据的更新操作。
元数据的更新是在客户端内部进行的，对客户端的外部使用者不可见
当需要更新元数据时，会先挑选出 leastLoadedNode，然后向这个 Node 发送 MetadateRequest 请求来获取具体的元数据信息
更新操作是由 sender 线程发起的，创建完 MetadateRequest 之后同样会存入 inFlightRequests，之后步骤就和发送消息时类似
元数据虽然由 sender 线程负责更新，但是主线程也需要读取这些信息，这里的数据同步时通过 synchronized 和 final 关键字保证。

### 重要的生产者参数

#### ack (字符串类型)

指定分区中必须要有多少副本收到这条消息，之后生产者才会认为这条消息是成功写入的，涉及到消息的可靠性和吞吐量
acks=1 (默认)，只要分区的leader 副本成功写入，就会收到来自服务端的成功响应
如果消息无法写入leader副本(副本崩溃、重新选举过程)，生产者就会收到一个错误响应，为了避免消息丢失，生产者可以选择重发消息
如果消息写入leader副本并返回成功响应给生产者，且在被其他follower 副本拉取之前 leader 副本崩溃，消息还是会丢失，因为新选举的leader 副本中并没有这条消息，acks=1 是消息可靠性和吞吐量之间的折中方案

acks=0 生产者发送消息之后不需要等待任何服务端响应
如果消息从发送到写入kafka 的过程出现异常，导致kafka 没有收到这条消息，生产者也无从得知，消息就丢失了
acks=0 可以达到最大吞吐量

acks=-1 或者 all，生产者在消息发送成功之后，需要等待ISR中的所有副本都成功写入消息之后才能够收到来自服务端的成功响应
可以达到最强可靠性，但是消息不一定可靠，如果ISR中可能只有 leader 副本，就退化成了 acks=1 的情况
获得更高的消息可靠性需要配合 min.insync.replicas 等参数的联动

#### max.request.size

限制生产者客户端能发送的消息最大值，默认为 1MB
参数涉及到 broker 端的 message.max.bytes 参数

#### retries 和 retry.backoff.ms

retries 配置生产者重试次数，默认0
可恢复的异常 网络抖动、leader 副本选举可以重试
不可恢复的异常 max.request.size 就不行
retry.backoff.ms 默认 100,设定两次重试之间的时间间隔，避免无效的重试。

Kafka 可以保证同一分区消息是有序的
如果生产者按照一定的顺序发送消息，消息会顺序的写入分区，消费者可以按照同样的顺序消费他们
eg: 订阅mysql binlog
如果 retries 参数配置成非零值，并且 max.in.flight.requests.per.connection(每个链接最多缓存的请求数)参数配置大于1就会出现错序
需要保证顺序的消息场景，max.in.flight.requests.per.connection 配置为 1，retries 设置 0，舍弃吞吐量保证顺序。

#### compression.type

消息压缩方式，默认 none 消息不会被压缩
可选 gzip snappy lz4
时间换空间，如果对时延有一定的要求，不推荐对消息进行压缩

#### connections.max.idle.ms

多久之后关闭闲置的链接，默认是 540000(ms) 9 分钟

#### linger.ms

指定生产者发送 ProducerBatch 之前等待更多消息(ProducerRecord)加入 ProducerBatch 的时间，默认值为 0
生产者会被等在 ProducerBatch 被填满或等待时间超过 linger.ms 值时发出去
增大这个参数会增加消息的延迟，但是同时能提升一定的吞吐量
TCP[Nagle] TODO

#### receive.buffer.bytes

设置 socket 接收消息缓冲区(SO_RECBUF)的大小，默认值是 32768B，32kb
如果设置为 -1，使用操作系统默认值
如果Producer 与 Kafka 处于不同的机房，可以适当调大。

#### send.buffer.bytes

设置 Socket 发送消息缓冲区 (SO_SUDBUF)大小，默认是 131072(B)，128kb
如果设置为 -1，使用操作系统默认值


#### request.timeout.ms

参数配置 Producer 等待请求响应的最长时间，默认值为 30000(ms)
请求超时之后可以选择进行重试
【这个参数需要比 broker 端参数 replica.lag.time.max.ms 值要大，减少因客户端重试而引起的消息重复概率】

# 消费者

### 消费者组

每个消费者都对应一个消费者组，消息发布到主题后，只会投递给订阅他的每个消费组中的一个消费者
消费者只能消费所分配的分区中的消息，每个分区只能被一个消费组的一个消费者所消费
消费者与消费者组模型可以让整体的消费能力具备横向伸缩性，可以增加或减少消费者个数来提高或降低整体的消费能力
分区固定，如果消费者过多就会有消费者分配不到任何分区
消费者客户端参数 partition.assignment.strategy

消费组是一个逻辑上的概念，每一个消费者只隶属于一个消费者组。
消费者在进行消费前需要指定其所书消费组的名称，可以通过消费者客户端参数 group.id 来配置，默认值为空字符串
消费者是实际的应用实例，可以是一个线程/进程/可以在不同机器实例上

### 消息投递模式

点对点(P2P point-to-point)模式和发布订阅模式(Pub/Sub)模式
点对点模式基于队列的，消息生产者发送消息到队列，消息消费者从队列中接受消息
发布订阅模式定义了如何向一个内容节点发布和订阅消息，这个内容节点称为主题,主题可以认为是消息传递的中介，使得消息的订阅者和发布者互相保持独立，不需要进行接触就可以保证消息的传递，发布订阅模式在消息的一对多广播时采用

Kafka 消费者组的模式同时支持两种消息投递模式
1.如果所有的消费者都隶属于同一个消费组，所有的消息都会被均衡的投递给每一个消费者
2.如果所有的消费者都隶属于不同的消费组，所有的消息都会被广播给所有的消费者，相当于发布订阅模式

### 消费者客户端

1.配置消费者客户端参数及床啊斤相应的消费者实例
2.订阅主题
3.拉取消息
4.提交消费位移
5.关闭消费者实例

### 必要参数设置

1.bootstrap.servers 链接Kafka集群地址
2.group.id 消费者隶属消费组
3.key.deserializer value.deserializer 反序列化字节数组消息
4.client.id 客户端id(客户端不设置，KafkaConsumer 会自动生成一个非空字符串)

### 订阅主题与分区

```java
一个消费者可以订阅多个主题
subscribe(collection)
1.list
2.pattern，listener
3.pattern
subscribe 订阅两次不同的主题，以最后一次为准
pattern 正则订阅可以消费到新添加的主题

订阅分区
assign(Collection<TopicPartition> topicPartitions)
KafkaConsumer.partitionsFor() 可以查询指定主题的元数据信息
public class PartitionInfo {

    private final String topic;
    private final int partition;
    private final Node leader; //leader 副本位置
    private final Node[] replicas; // AR
    private final Node[] inSyncReplicas; // ISR
    private final Node[] offlineReplicas; //OSR
 }
//取消订阅
1.unsubscribe()
2.subscribe(new ArrayList<String>())  
3.assign(new ArrayList<TopicPartition>())

集合订阅、正则订阅、订阅分区代表了三种不同的订阅状态，状态互斥，一个消费者中只能使用其中一种。
subscribe 方法订阅的主题具有消费者自动在均衡的功能，在多个消费者增加或减少时，分区分配关系会自动调整，以实现消费负载均衡及故障转移
assign 不具备消费者自动均衡的功能  
```

### 反序列化

implements Deserializer
configure
deserialize(String topic,byte[] data) 如果data为null,处理直接返回null不抛出异常
close

``
使用自定义的序列化/反序列化器，会增加生产者何消费者之间的耦合度，系统升级换代的时候很容易出错，上下游兼容性也是问题
``

### 消息消费

```java
Kakfa 中消费是基于拉模式的，消费者主动向服务端发起请求来拉取消息，需要不断轮询，重复 poll，拉取订阅主题分区上的一组消息
poll(final Duration timeout) 控制阻塞时间,消费者缓冲区里面没有可用数据也会阻塞，不能超过 long 0x7fffffffffffffffL
返回的消息集合中的 ConsumerRecord
public class ConsumerRecord<K, V> {
    public static final long NO_TIMESTAMP = RecordBatch.NO_TIMESTAMP;
    public static final int NULL_SIZE = -1;
    public static final int NULL_CHECKSUM = -1;

    private final String topic;
    private final int partition;
    private final long offset;
    private final long timestamp;
    private final TimestampType timestampType; //时间戳类型 CreateTime,LogAppendTime
    private final int serializedKeySize;
    private final int serializedValueSize;
    private final Headers headers;
    private final K key;
    private final V value;

    private volatile Long checksum;//CRC32校验值
 }
 返回的消息集合 ConsuermRecords 提供 iterator() 遍历消息
records(TopicPartition) 获取消息集中的指定分区消息
records(topic) 按照主题消费消息
count 统计消息个数
isEmpty
empty
```

### 位移提交

Kafka 分区每条消息都有 offset，表示消息在分区里面的位置
消费者使用 offset 表示消费到分区中的某个消息的位置
每次 poll 返回的是还没有被消费过的消息集(前提是存储在Kafka中并且没有异常发生)，需要将消费位移持久化，并且新的消费者加入在均衡要分给新的消费者
在旧消费者客户端中，消费位移是存储在zk 中的，新消费客户端消费位移存储在 Kafka 内部的主题 _consumer_offsets 中，消费者消费完消息后需要执行消费位置的提交。

### 消息丢失/重复消费

poll 完立即提交，消费发生异常，消息丢失，拉取后继数据
poll 完，等待消费完提交，消费过程发生异常，恢复再次重新拉取
乱序提交

Kafka 中默认的消费位移提交方式是自动提交，有消费者客户端参数 enable.auto.commit 配置，默认值为 true，定期提交
定期周期有客户端参数 auto.commit.interval.ms 配置，默认 5s,前提是 enable.auto.commit 参数为 true
自动位移提交是在 poll 里面完成的，每次真正向服务端发起拉取请求之前会检查是否可以进行位移提交，如果可以就提交上一次轮询的位移 [TODO]
减少位移提交时间间隔来减小重复消息的窗口大小，但是不能避免重复消息的发生，会使位移提交更加频繁

手动提交
enable.auto.commit false
同步提交/异步提交
KafkaConsumer commitSync() commitAsync
无参 commitSync 提交消费位移的频率和拉取批次消息、处理批次消息的频率是一样的，只能提交当前poll 批次对应 position
commit(final Map<TopicPartition,OffsetAndMetadata> offsets) 细粒度指定分区
offset 参数用来提交指定分区的位移，

异步提交在执行的时候消费者线程不会被阻塞，可能在提交位移的结果还没有返回之前就开始了新一次的拉取操作，可以使得消费者的性能得到加强
commitAsync()
commitAsync(OffsetCommitCallback callback)
commitAsync(final Map<TopicPartition,OffsetAndMetadata> offsets, OffSetCommitCallback callback)
位移提交完会回调 OffsetCommitCallback onComplete() 方法

异步提交失败
如果失败，其他消费异步提交成功，重试提交成功，这个时候发生异常或者在均衡，offset 前移，导致重复消费
解决
设置一个递增序号维护异步提交的顺序，每次提交位移增加，遇到失败需要重试的时候，检查提交的位移和序号大小，如果小雨，说明有更大的offset 已经提交了，不需要对本次进行重试，如果二者相等可以进行重试。【TODO】
防止消费者正常退出/发生再均衡，finally 使用同步提交方式兜底

### 控制/关闭

KafkaConsumer pause resume 暂停、恢复 TopPartition 拉取操作
paused() 返回被暂停的分区集合
wakeup() 线程安全，退出 poll ，抛出 WakeupException 异常，不需要处理异常，只是一种跳出循环的方式
跳出循环一定要显示的关闭动作以及高释放资源占用，close 方法提供超时方法

### 指定位移消费

当一个新的消费组建立的时候/消费组内的一个新消费者订阅了新的主题，没有可以查找的消费位移，
当 _consumer_offses 主题中有关这个消费组的位移信息过期而被删除后，也没有可以查找的消费位移
Kafka中每当消费者找不到所记录的消费位移时，就会根据消费者客户端参数 auto.offset.reset 的配置来决定从何处开始进行消费
默认值为 latest，表示从分区末尾开始消费消息
设置为 earliest ，消费者会从起始处，从0开始消费
位移越界也会出发 auto.offset.reset 参数的执行
设置为 "none",出现查不到消费位移的时候，既不从最新的消息位置处开始消费，也不从最早的消息位置处开始消费，抛出 NoOffsetForPartitionException
在找不到消费位移的时候或者位移越界的情况下，通过 auto.offset.reset 参数只能在比较粗粒度的从头或末尾开始消费

### seek

seek(TopicaPartition topic,Long offset)
重置消费者分配到的分区的固定位置，要先 poll 分配到分区
KafkaConsumer.assignment() 判断是否分配到了相应的分区
如果消费组内的消费者在启动的时候能够找到消费位移，除非发生位移越界，否则 auto.offset.reset 参数不会奏效，这个时候想要指定开头或末尾开始消费，就需要 seek

```java
package dev.spider.kafka.consumer;

import dev.spider.kafka.serializer.CompanyDeserializer;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.OffsetAndTimestamp;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringSerializer;
import org.assertj.core.util.Lists;
import org.assertj.core.util.Maps;
import org.assertj.core.util.Sets;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;

import java.time.Duration;
import java.util.*;
import java.util.stream.Collectors;

public class SeekConsumer {

    private static final String brokerList = "c1:9092";
    private static final String topic = "topic-demo";
    private static final String groupId = "group.demo";

    private Properties properties;

    @BeforeAll
    public Properties initConfig() {
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, CompanyDeserializer.class.getName());
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "group-demo");
        properties.put(ConsumerConfig.CLIENT_ID_CONFIG, "producer.client.id.demo");
        this.properties = properties;
        return properties;
    }

    @Test
    public void seek(String[] args) {
        KafkaConsumer consumer = new KafkaConsumer<String, String>(initConfig());
        consumer.subscribe(Arrays.asList(brokerList));
        Set<TopicPartition> topicPartitions = Sets.newHashSet();
        while (topicPartitions.size() == 0) {
            consumer.poll(Duration.ofMillis(100));
            topicPartitions = consumer.assignment();
        }
        //获取指定分区末尾消息位置
        //request.timeout.ms 设置等待获取超时时间
        Map<TopicPartition, Long> endOffsets = consumer.endOffsets(topicPartitions);
        //获取分区起始位置
        Map<TopicPartition, Long> beginOffsets = consumer.beginningOffsets(topicPartitions);

        //直接从分区头/尾重置消费位移
        consumer.seekToBeginning(topicPartitions);
        consumer.seekToEnd(topicPartitions.stream().filter(topicPartition -> topicPartition.equals("")).
                collect(Collectors.toList()));
        for (TopicPartition tp : topicPartitions) {
            consumer.seek(tp, endOffsets.get(tp));
        }

        //根据时间点消费
        Map<TopicPartition, Long> timestampToSearch = Maps.newHashMap(Lists.newArrayList(topicPartitions).get(0), 100L);


        Map<TopicPartition, Long> tpTimeMap = new HashMap<>();
        for (TopicPartition topicPartition : topicPartitions) {
            tpTimeMap.put(topicPartition, System.currentTimeMillis() - 1 * 24 * 3600 * 1000);
        }
        Map<TopicPartition, OffsetAndTimestamp> offsetsForTimesMap = consumer.offsetsForTimes(tpTimeMap);
        for (TopicPartition topicPartition : topicPartitions) {
            OffsetAndTimestamp offsetAndTimestamp = offsetsForTimesMap.get(topicPartition);
            if (Objects.nonNull(offsetAndTimestamp)) {
                consumer.seek(topicPartition, offsetAndTimestamp.offset());
            }
        }

    }
}

```

### 再均衡

再均衡是指分区的所属权从一个消费者转移到另一个消费者的行为，为消费组具备高可用和伸缩性提供保障，可以方便安全地添加/删除消费者组内的消费者
再均衡发生期间，消费组内的消费者是无法读取消息的，当一个分区被重新分配给另一个消费者时，消费者当前的状态也会丢失
消费者消费完一个分区中部分消息时还没来得及提交消费位移时发生了再均衡，这个分区又被分配给了消费组内其他消费者，原来消费完的消息会被重新消费一遍，发上重复消费，应避免不必要的再均衡的发生

```
ConsumerRebalancListener 再均衡监听器用来设定发生均衡动作前后的一些准备或收尾动作
```

```java
        consumer.subscribe(Arrays.asList(brokerList), new ConsumerRebalanceListener() {
            /**
             * 在再均衡开始之前和消费者停止读取消息之后被调用
             * 可以处理消费位移的提交避免一些不必要的重复消费发生
             * @param partitions 再均衡前所分配到的分区
             */
            @Override
            public void onPartitionsRevoked(Collection<TopicPartition> partitions) {
                for (TopicPartition partition : partitions) {
                    consumer.committed(partition);
                    currentOffset.clear();
                }
            }

            /**
             * 重新分配分区之后和消费者开始读取消息之前被调用
             * @param partitions 再均衡后分配到的分区
             */
            @Override
            public void onPartitionsAssigned(Collection<TopicPartition> partitions) {
                //do st
            }
        });
```

### 拦截器

```java
消费者拦截器主要在消费消息或在提交消费位移时进行一些定制化操作
/**
 * 自定义消费者拦截器
 * 拦截器链 按照配置的顺序拦截
 * 如果在拦截器链中某个拦截器执行失败，下一个拦截器会从上一个执行成功的拦截器继续执行
 */
public class ConsumerInterceptionTTL implements ConsumerInterceptor<String, String> {

    private static final long EXPIRE_INTERVAL = 10 * 1000;

    /**
     * 在 poll 方法返回之前
     * 如果发生异常会被捕获记录到日志中，不向上传递
     *
     * @param records
     * @return
     */
    @Override
    public ConsumerRecords<String, String> onConsume(ConsumerRecords<String, String> records) {
        long now = System.currentTimeMillis();
        Map<TopicPartition, List<ConsumerRecord<String, String>>> newRecords = new HashMap<>();
        for (TopicPartition tp : records.partitions()) {
            List<ConsumerRecord<String, String>> tpRecords = records.records(tp);
            List<ConsumerRecord<String, String>> newTpRecords = Lists.newArrayList();
            for (ConsumerRecord<String, String> record : tpRecords) {
                if (now - record.timestamp() < EXPIRE_INTERVAL) {
                    newTpRecords.add(record);
                }
            }
            if (!newTpRecords.isEmpty()) {
                newRecords.put(tp, newTpRecords);
            }
        }
        return null;
    }

    /**
     * 提交完消息位移之后调用，可以用来记录跟踪提交的位移消息(eg:针对异步commit)
     *
     * @param offsets
     */
    @Override
    public void onCommit(Map<TopicPartition, OffsetAndMetadata> offsets) {
        offsets.forEach((tp, offset) -> {
            System.out.println("MSG commit " + tp + ":" + offset.offset());
        });
    }

    /**
     * close resource
     */
    @Override
    public void close() {

    }

    @Override
    public void configure(Map<String, ?> configs) {

    }
}

```

### 多线程

KafkaProducer 是线程安全的，KafkaConsumer 非线程安全
KakfaConsumer 定义了 acquire()方法，用来检测当前是否只有一个线程在操作，若有其他线程正在操作，会抛出ConcurrentModifcationException
KafkaConsumer 中每个公用方法在执行前所要执行动作之前都会调用 acquire() 方法，只有 wakeup() 是例外

```java
    // currentThread holds the threadId of the current thread accessing KafkaConsumer
    // and is used to prevent multi-threaded access
    private final AtomicLong currentThread = new AtomicLong(NO_CURRENT_THREAD);

		/**
     * Acquire the light lock protecting this consumer from multi-threaded access. Instead of blocking
     * when the lock is not available, however, we just throw an exception (since multi-threaded usage is not
     * supported).
     * @throws ConcurrentModificationException if another thread already has the lock
     * 线程计数标记检测线程是否发生了并发操作，
     */
    private void acquire() {
        long threadId = Thread.currentThread().getId();
        if (threadId != currentThread.get() && !currentThread.compareAndSet(NO_CURRENT_THREAD, threadId))
            throw new ConcurrentModificationException("KafkaConsumer is not safe for multi-threaded access");
        refcount.incrementAndGet();
    }

		/**
     * Release the light lock protecting the consumer from multi-threaded access.
     * 释放锁
     */
    private void release() {
        if (refcount.decrementAndGet() == 0)
            currentThread.set(NO_CURRENT_THREAD);
    }
```

Kafka 消息保留机制，有些消息有可能在被消费之前就被清理了，从而造成消息的丢失 【TODO?】
多线程整体提高消费能力

```java
#1.每个线程实例化一个 KafkaConsumer 对象，并发受限于分区的实际个数，消费线程大于分区数时就会闲置
public class FirstMultiThreadConsumer {
    public static final String brokerList = "c2:9092,c1:9092,c3:9092";
    public static final String topic = "topic-demo";
    public static final String groupId = "group.demo";

    public static Properties initConfig() {
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, CompanyDeserializer.class.getName());
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "group-demo");
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
        properties.put(ConsumerConfig.CLIENT_ID_CONFIG, "producer.client.id.demo");
        return properties;
    }

    public static void main(String[] args) {
        Properties properties = initConfig();
        int consumerThreadNum = 4;
        for (int i = 0; i < consumerThreadNum; i++) {
            new KafkaConsumerThread(properties, topic).start();
        }
        //设置和分区大小一致的线程
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer(properties);
        List<PartitionInfo> partitionInfos = kafkaConsumer.partitionsFor(topic);
        int partition = partitionInfos.get(0).partition();
        for (int i = 0; i < partition - 1; i++) {
            new KafkaConsumerThread(properties, topic).start();
        }
    }

    /**
     * 每个线程顺序消费一个主题分区
     * 每个线程需要维护一个
     */
    public static class KafkaConsumerThread extends Thread {

        private KafkaConsumer<String, String> kafkaConsumer;

        public KafkaConsumerThread(Properties properties, String topic) {
            this.kafkaConsumer = new KafkaConsumer<>(properties);
            this.kafkaConsumer.subscribe(Lists.newArrayList(topic));
        }

        @Override
        public void run() {
            try {
                while (true) {
                    ConsumerRecords<String, String> records = kafkaConsumer.poll(Duration.ofMillis(100));
                    for (ConsumerRecord<String, String> record : records) {
                        //do st
                        //如果这部分处理耗时，整体消费能力下降
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
                //do st
            } finally {
                kafkaConsumer.close();
            }
        }
    }
}

```

```java
#2.消息处理模块多线程处理
public class ThirdMultiThreadConsumer {
    public static final String brokerList = "c2:9092,c1:9092,c3:9092";
    public static final String topic = "topic-demo";
    public static final String groupId = "group.demo";
    public static Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();


    public static Properties initConfig() {
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, CompanyDeserializer.class.getName());
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "group-demo");
        properties.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, true);
        properties.put(ConsumerConfig.CLIENT_ID_CONFIG, "producer.client.id.demo");
        return properties;
    }

    public static void main(String[] args) {
        Properties properties = initConfig();
        KafkaConsumerThread kafkaConsumerThread = new KafkaConsumerThread(properties, topic, Runtime.getRuntime().availableProcessors());
        kafkaConsumerThread.start();

    }

    public static class KafkaConsumerThread extends Thread {
        private KafkaConsumer<String, String> kafkaConsumer;
        private ExecutorService executorService;
        private Integer threadNumber;


        public KafkaConsumerThread(Properties properties, String topic, int threadNumber) {
            KafkaConsumer kafkaConsumer = new KafkaConsumer<>(properties);
            kafkaConsumer.subscribe(Collections.singleton(topic));
            this.threadNumber = threadNumber;
            this.executorService = new ThreadPoolExecutor(threadNumber, threadNumber, 0L, TimeUnit.MILLISECONDS, new ArrayBlockingQueue<>(100), new ThreadPoolExecutor.CallerRunsPolicy());
        }

        @Override
        public void run() {
            try {
                while (true) {
                    ConsumerRecords<String, String> records = this.kafkaConsumer.poll(Duration.ofMillis(100));
                    if (!records.isEmpty()) {
                        executorService.submit(new RecordHandler(records));
                        synchronized (offsets) {
                            if (!offsets.isEmpty()) {
                                kafkaConsumer.commitSync(offsets);
                            }
                        }
                    }
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                kafkaConsumer.close();
            }
        }
    }

    public static class RecordHandler extends Thread {
        public final ConsumerRecords<String, String> records;

        public RecordHandler(ConsumerRecords<String, String> records) {
            this.records = records;
        }

        @Override
        public void run() {
            //consumer records
            for (TopicPartition tp : records.partitions()) {
                List<ConsumerRecord<String, String>> tpRecords = this.records.records(tp);
                //处理tpRecords
                long lastConsumedOffset = tpRecords.get(tpRecords.size() - 1).offset();
                synchronized (offsets) {
                    if (!offsets.containsKey(tp)) {
                        offsets.put(tp, new OffsetAndMetadata(lastConsumedOffset + 1));
                    } else {
                        long position = offsets.get(tp).offset();
                        if (position < lastConsumedOffset + 1) {
                            offsets.put(tp, new OffsetAndMetadata(lastConsumedOffset + 1));
                        }
                    }
                }
            }
        }
    }
}
```



### 重要的消费者参数

fetch.min.bytes
配置 consumer 在一次拉取请求中能从 Kafka 中拉取的最小数据量，默认值为 1(B)，Kafka 在收到 Consumer 的拉取请求时，如果返回给 Consumer 的数据量小于这个参数配置的值，就需要等待，知道数据量满足这个参数的配置大小

fetch.max.bytes
默认值 52428800(B),50MB，如果这个参数设置的值比任何一条写入 Kafka 中的消息要小，也是会返回的(仅是在非空分区第一条消息)，以确保消费者继续工作

fetch.max.wait.ms
指定Kafka 等待时间，默认是 500ms，没有足够多的消息会阻塞500ms发送给Consumer

max.partition.fetch.bytes
从每个分区里返回给 Consumer 的最大数据量，默认值是 1048576(B),1MB
如果设置的参数比消息的大小要小，也不会导致无法消费，为了保证消费逻辑的正常运转不做强硬的限制

max.poll.records
Consumer 一次拉取请求中拉取的最大消息数，默认值500

connections.max.idle.ms
设置多久关闭闲置的链接，默认值是 540000(ms),9分钟

exclude.internal.topics
Kafka 中有两个内部的主题，_consumer_offsets 和 _transaction_state
exclude.internal.topics 用来指定 Kafka 中的主题是否可以向消费者公开，默认 true,只能使用 subscribe(Collection) 方式而不是 subscribe(Pattern) 当时来订阅内部主题，设置为 false 则没有这个限制

receive.buffer.bytes
这个参数谁 socket 接受消息缓冲区( SO_SNDBUF) 的大小，默认值 131072(B)，128KB
-1 使用操作系统的默认值

request.timeout.ms
Consuemr 等待请求响应的最长时间，默认值为 30000(ms)

metadata.max.age.ms
配置元数据的过期时间，默认值为 300000(ms),5分钟
如果元数据在此参数限定的时间内没有进行更新，则会被强制更新，即使没有任何分区变化或有新的 broker 加入

reconnect.backoff.ms
配置尝试重试链接指定主机之前的等待时间，避免频繁链接主机，默认时间 50(ms)，适用于消费者向 broker 发送的所有请求

retry.backoff.ms
用来配置尝试重新发送失败的请求到指定的主题分区之前的等待，避免在某些故障情况情况下频繁的重复发送，默认值为 100(ms)

isolation.level
配置消费者的事务隔离级别，字符串类型，表示消费者所消费到的位置
read_uncommitted 默认配置，可以消费到 HW(high watermark)
read_committed 消费者忽略事务为提交的消息，只能消费到 LSO(LastStableOffset)

## 主题与分区

### 主题的管理

kafka-topics.sh 命令
实质是调用 exec $(dirname $0)/kafka-run-class.sh kafka.admin.TopicCommand "$@"
管理主题还可以通过 KafkaAdminClient(通过发送 CreateTopicRequest、DeleteAdminClient) 等请求来实现
还可以通过直接操作日志文件和zookeeper 节点来实现。

### 创建主题

broker 端配置参数 auto.create.topics.enable 设置为 true
当生产者向一个尚未创建的主题发送消息时
会自动创建一个分区数为 num.partitions(默认值为1)、副本因子为 default.replication.factor(默认值为1)
当一个消费者开始从未知主题读取消息时，或则会当任意一个客户端向未知主题发送元数据请求时
都会按照 num.partitions、default.replication.factor 创建一个相应的主题
auto.create.topics.enable 参数设置为 true，会增加主题的管理和维护的难度
bin/kafka-topics.sh --zookeeper localhost:2181/kafka create --topic tpName --partitions 3 -replication-factor 3
kakfa 会在 log.dirs 目录下创建相应的主题分区文件夹,默认 /tmp/kafka-logs/
文件夹以 topic-partition-分区编号 命名，一共会有 3*3 个日志文件夹分布在 brokre 机器上
			 partition
			 						replica -> log
topic  partition  replica -> log
									replica -> log
			 partition

主题和分区都是提供给上层的抽象，只有 log 才是物理的存在，同一个分区的多个副本必须在分布在不同 broker 中，这样才能提供有效的数据冗余。

当创建一个主题时，会在 zookeeper 的 /brokers/topics/ 目录下创建一个同名的实节点，记录该主题的分区副本分配方案
ls /kafka/brokers/topics

describe
bin/kafka-topics.sh --zookeeper localhost:2181/kafka --describe --topic topic-0-2
Topic:topic-0-2	PartitionCount:3	ReplicationFactor:3	Configs:
	Topic: topic-0-2	Partition: 0	Leader: 2	Replicas: 2,1,0	Isr: 2,1,0
	Topic: topic-0-2	Partition: 1	Leader: 0	Replicas: 0,2,1	Isr: 0,2,1
	Topic: topic-0-2	Partition: 2	Leader: 1	Replicas: 1,0,2	Isr: 1,0,2
PartitionCount 表示主题中分区个数
ReplicationFactor 表示副本因子
Configs 表示创建或修改主题时指定的参数配置
Leader 表示分区的leader副本所对应的 brokerId
Isr 表示分区的ISR集合
Replicas 表示分区所有的副本分配情况，AR 集合，数字表示 brokerId

replica-assignment 手动指定分区副本
--replica-assignment broker_Id_for_part1_replica1:broker_id_for_part1_replica2,...
根据分区号的数值大小按照从小到大顺序进行排列，分区内使用 : 隔开
bin/kafka-topics.sh --zookeeper localhost:2181/kafka --create --topic topic-replica-assignment --replica-assignment 2:0,0:1,1:2,2:1

partition 0 replica 2,0
partition 1 replica 0,1

同一个分区内的副本不能有重复      eg: 0:0,1:1
如果分区之间所指定的副本数要相同  eg: 0:1,0,0:1
分区不能跳过，',' 之间必须要有值
如果创建的主题存在，有冲突会报错，--if-not-exists 在创建时如果存在同名不做处理
kafka-topics.sh 脚本在创建主题时还会检测是否包含 "."/"_" 字符，内部埋点会根据主题来命名 metrics 名称，会将 "." 改成下划线"_"
eg: topic.1_2 和 topic_1.2 就会冲突

主题名称必须由大小写字母、数字、符号、".","-","_"组成，不能为空，不能只有点，并且长度不能超过 249
kafka 从 0.10.x 版本开始支持指定 broker 机架信息，如果指定机架信息，分区副本分配时会尽可能让分区副本分配到不同的机架上
broker 端参数 broker.rack=xxx 配置
集群中必须全部指定或者全部不指定机架信息，disabel-rack-aware 忽略机架信息
kafka-topics.sh --config xxx config 可以覆盖默认配置

```java
//命令创建
package dev.spider.kafka.command;

import kafka.admin.TopicCommand;

public class TopicCommandAction {


    public static void main(String[] args) {
        creatTopic();
    }

    static void creatTopic() {
        String[] options = new String[]{
                "--zookeeper", "localhost:2181/kafka",
                "--create",
                "--replication-factor", "1",
                "partitions", "1",
                "--topic", "topic-create-command"
        };
        TopicCommand.main(options);

    }
    //<dependency>
    //            <groupId>org.apache.kafka</groupId>
    //            <artifactId>kafka_2.11</artifactId>
    //            <version>2.0.0</version>
    //        </dependency>
}

```

### 分区副本的分配

创建主题时使用了 replica-assignment 参数就按照指定的方案进行分区副本的创建
如果有机架逻辑就采用机架信息分配策略
scala

### 查看主题

-list
--describe --topic x,y
	topics-with-overrides 找出所有包含覆盖配置的主题，列出包含了与集群不一样配置的主题
	under-replicated-partitions 包含失效副本的分区，可能正在进行同步、同步发生异常，此时 ISR<AR
	unavaulable-partitions 查看主题中没有 leader 副本的分区，分区已经处于离线状态，对于生产者和消费者处于不可用状态


### 修改主题

--alter
当主题中的消息包含 key 时，根据 key 计算分区的行为就会受到影响
当 topic-config 分区数为 1时，不管消息的 key 为何值，消息都会发往一个分区
当分区增加到 3 时，就会根据 key 来计算分区号，还会影响既定消息的顺序，基于 key 计算的主题，在一开始就要设定好分区数量
Kafka 只支持增加分区数，不支持减少分区数
--if-exists 忽略要修改的主题不存在
--config 可以搭配 --config  --delete-config

``
使用kafka-configs.sh 增删改操作，kafka-topics.sh 已经过时
``

### 配置管理

kafka-configs.sh 支持操作主题、broker、用户和客户端，运行时修改配置(alter、describe)
entity-type 参数指定配置的类型，entry-name 参数指定操作配置的名称
bin/kafka-configs.sh --zookeeper localhost:2181/kafka --describe --entity-type topics --entity-name topic-config

``
entity 只可以配置四个值，topics,brokers,clients,users
``

| Entity-type释义 | Entity-name释义           |
| --------------- | ------------------------- |
| topics          | 指定主题名称              |
| brokers         | 指定brokerid值，broker.id |
| clients         | 指定clientid,client.id    |
| users           | 指定用户名                |

### 删除主题

kafka-topics.sh 脚本中的delete
./kafka-topics.sh --zookeeper localhost:2181/kafka --delete --topic topic-name
broker 端 delete.topic.enable true 才能删除主题
删除内部主题、不存在主题会报错（_consumer.offsets、_transaction_state），可以使用 --if-exists 忽略
删除主题的行为本质只是在 zookeeper 中 /admin/delete_topics 路径下创建一个与待删除主题同名的节点，标记该主题为待删除的状态
真正删除主题的动作也是由kafka控制器负责完成

# 日志存储

### 文件目录布局

kafka 消息以 主题基本单位归纳，主题分为多个分区，分区中的每条消息都会被分配一个唯一的序列号，一个分区对应一个日志
为了防止日志过大，引入了日志分段 LogSegment，将log 切分为多个 LogSegment
Log 物理以文件夹形式存储，每个 LogSegment 对应于磁盘文件上的一个日志文件和两个索引文件，以及可能的其他文件(以.txnindex为后缀的事务索引文件)

```ri z
为了便于消息的检索，每个 LogSegment 中的日志文件(.log)，都有对应的两个索引文件
1.偏移量索引文件(.index)
2.时间戳索引文件(.timeindex)
每个 LogSegment 都有一个基准偏移量 bashOffset ,用来表示当前 LogSegment 中第一条消息的 offset
偏移量是一个 64 位的长整型数，日志文件和两个索引文件都是根据基准偏移量命名的，名称固定位20位数字，没有达到的位数用0填充
还会有一些 .deleted .cleaned .swap .snapshot .txnindx leader-epoch-checkpoint
```

### 日志格式演变

```
vo-v1-v2
消息集 存储、传输、压缩基本单元
```

### 消息压缩

```
将多条消息一起进行压缩，保证较好的压缩小效果，生产者到消费者两端都是压缩后的消息
日志中使用的压缩方式通过参数 compression.type 配置，默认是 producer ,保留生产者使用的压缩方式，还可以设置为 gzip,snappy,lz4
compression.type 不压缩设置为 uncomoressed

每个从生产者发出的消息集中的消息 offset 都是从 0 开始的
对 offset 的转换是在服务端进行的，客户端不需要做这个工作。
外层消息保存了内层消息的最后一条消息的绝对位移，绝对位移是相对于分区而言的
v1 比 v0 多了一个 timestamp 字段
外层消息
如果 timestamp 类型是 CreateTime 设置的是内层最大的时间戳
如果 timestamp 类型是 LogAppendTime 设置的是Kafka 服务器当前的时间戳
```

