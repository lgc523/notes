---
title: "Kafka Apply"
date: 2021-08-16T22:12:38+08:00
draft: true
toc: true
images:
tags: 
  - kafka
  - middle-aware
---

## Kafka 应用

### cli

### 消费组管理

``kafka-consumer-groups.sh,查看变更消费组消息``



--list 列出所有的消费组名称
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

--describe 展示某一个消费者组的信息

bin/kafak-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group groupNameXXX

消费组的状态 state
Dead、Empty、PreparingRebalance、CompletingRebalance、Stable
	--member 列出消费组内消费者成员信息
	--verbose 列出每个消费者成员的分配情况

--delete 删除一个置顶的消费组，如果有消费者成员正在运行，删除操作会失败。

​	

### 消费者位移管理

消费组内没有正在运行的消费者成员，kafka-consumer-groups.sh 提供了重置消费者组内消费位移的功能，具体是通过 reset-offsets 指令类型的参数来实施的

```
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group groupNameXXX --topic /--all--topics --reset-offsets --to-earliest/--to--latest --execute
```



### 调整位置的更多选择

- by-duration <<String:duration>>> 将消费者位移重置到距离当前时间指定间隔的最早位移处，duration 格式为 PnDTnHnMns

- from-file <String:path to CSV file> 将消费位移重置到 CSV 文件中指定的位置

- shift-by <<Long:number-of-offset>> 可以把位移调整到当前位移 + number-of-offsets 处，可以为负数

- to-current 将消费位移调整到当前位置处

- to-datetime <<String,datetime>> 将消费位移调整到大于给定时间的最早位移处，

  datetime 格式为 ”YYYY-MM-DDTHH:mm:SS.sss“

```te
#将当前消费位移往前调整 10，不执行
bin/kafak-consumer-groups.sh --bootstrap-server localhost:9092 --group groupNamexxx --topic topName --reset-offsets --shift-by -10 --dry-run
#将消费位移调整为当前位移并将结果输出到控制台，不执行
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group groupNameXXX --topic topicName --reset-offsets --shift-by -20 --export --dry-run
#通过 from-file 参数从 offsets.csv 文件中获取位移重置策略，并且执行
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group groupNameXXX --topic topicName --reset-offsets --from-file offsets.csv --execute
#文件格式
topicName,2,99
topicName,1,97
topicName,0,90
```



### 手动删除消息

当一个分区被创建时，它的起始位置 logStartoffset 为 0，可以通过 KafkaConsumer 中的 beginningOffsets() 方法来查看分区的起始位置

```java
package dev.spider.kafka.consumer;

import lombok.Data;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.PartitionInfo;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInstance;

import java.util.List;
import java.util.Map;
import java.util.Properties;

import static java.util.stream.Collectors.toList;

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@Data
class KafkaConsumerShTest {
    private static final String brokerList = "c1:9092";
    private static final String topic = "topic-0-2";
    private static final String groupId = "group.demo";

    private Properties properties;

    @BeforeAll
    void initConfig() {
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, brokerList);
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.CLIENT_ID_CONFIG, "producer.client.id.demo");
        this.properties = properties;
    }

    @Test
    void testViewPartitionOffset() {
        //设置消费组的名称
        properties.put("group.id", groupId);
        //创建一个消费者客户端实例
        KafkaConsumer<String, String> kafkaConsumer = new KafkaConsumer<>(properties);
        List<PartitionInfo> partitions = kafkaConsumer.partitionsFor("topic-0-2");
        List<TopicPartition> tpList = partitions.stream().map(pInfo -> new TopicPartition(pInfo.topic(), 		  						pInfo.partition())).collect(toList());
        Map<TopicPartition, Long> beginningOffsets = kafkaConsumer.beginningOffsets(tpList);
        System.out.println(beginningOffsets);
    }
}

```

```
{topic-0-2-1=0, topic-0-2-2=0, topic-0-2-0=0}
```



使用 Kafka-delete-records.sh 来删除部分消息，在执行具体的删除动作之前需要先配置一个 JSON 文件，用来指定所要删除消息的分区及对应的位置，分别删除主题 topic-monitor 下分区 0 中偏移量为 10，分区1 中偏移量为 11 和分区 2 中偏移量为 12 的消息


```json
{
  "partitions":[
    {
      "topic":"topic-monitor",
      "partition":0,
      "offset":10
    },
    {
      "topic":"topci-monitor",
      "paritition":1,
      "offset":11
    },
    {
      "topic":"topic-monitor",
      "partition":"2",
      "offset":12
    }
  ],
  "version":1
}
```

将上面内容保存为 delete.json , 就可以通过 kakfa-delete-records.sh 脚本中的 offset-json-file 参数来指定这个  json 文件

```
bin/kafka-delete-records.sh --bootstrap-server localhost:9092 --offset-json-file delete.json
```

Kafka-delete-records.sh 内部通过调用 KafkaAdminClient 中的 deleteRecords() 方法来实现

```java
public DeleteRecordsResult deleteRecords(Map<TopicPartition, RecordsTODelete> recordsToDelete)
```

deleteRecords 方法最终还要通过发送 DeleteRecordsRequest 请求来通知 Kafka 晚上相应的 “删除”动作，

Kakfa 在收到 DeleteRecordsRequest 请求之后，会将指定分区的 logSrartOffset 置为相应的请求值，最终的删除动作还是交由日志删除任务来完成

### Kafka Connect
