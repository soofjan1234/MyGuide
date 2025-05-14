------

# Kafka

> 作者：hou
------
## 1.1 Kafka 核心概念

1. **Producer**   
2. **Consumer**  
3. **Broker**  
- 可以看作是一个独立的 Kafka 实例。多个 Kafka Broker 组成一个 Kafka Cluster  

4. **Topic** 
5. **Partition**  
- Partition 属于 Topic 的一部分。一个 Topic 可以有多个 Partition。并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上  

## 1.3 Kafka 消息顺序保证

1. **1 个 Topic 只对应一个 Partition**   

2. **发送消息时指定 key/Partition**   

## 1.4 Kafka 分区与消费者

多个消费者可以同时消费一个 Partition，但每个 Partition 只能被一个消费者组中的一个消费者消费  

### 1.4.1 为什么同组不能消费同一个partition

Kafka 里的 Offset（位移） 就相当于“当前读到第几页”，它存储在集群里供组内所有人共享。只有一个人在更新这一章的“读书进度”，才能保证进度不会被两个人的操作互相覆盖。

## 1.5 Kafka 消息不丢失保证

1. **生产者丢失消息**  
- 使用 `send` 方法并添加回调函数来确认消息是否成功发送  

2. **消费者丢失消息**  
- 消息被添加到分区时会分配一个 `offset`，消费者拉取到消息后手动提交 `offset`，避免消息丢失  

3. **Kafka 丢失消息**  
- 设置生产者的 `acks=all`，所有 ISR 列表的副本全部收到消息时，生产者才会接收到来自服务器的响应  
- 设置 `replication.factor >= 3`，每个 Partition 至少有 3 个副本  
- 设置 `min.insync.replicas > 1`，至少要被写入到 2 个副本才算是被成功发送  

## 1.6 Kafka 避免重复消费

1. **幂等校验**  
- 消费消息服务做幂等校验，比如 Redis 的 `set`、MySQL 的主键等天然的幂等功能。这种方法最有效  

2. **事务机制**  
- 使用 Kafka 的事务机制来保证消息的 Exactly-Once 语义  

3. **消息去重**  
- 在消费者端维护一个已消费消息的缓存，通过消息的唯一标识进行去重