# Kafka 概述
> 以图像的方式清晰的描述 kafka 的主要概念：https://timothystepro.medium.com/visualizing-kafka-20bc384803e7

本文简单描述 Kafka 的架构，数据消费的流程。

## 什么是 Kafka？
Apache Kafka 是一种分布式数据存储，经过优化以实时提取和处理流数据。流数据是指由成千上万个数据源持续生成的数据，通常可以同时发送数据。数据流平台需要处理这些持续流入的数据，不断消费它们。
在日常使用中，Kafka 常被作为消息队列来使用。为了便于描述，下文将使用消息来指代 kafka 中存储的数据。

## 主要概念

### Producers（生产者）和 Consumer（消费者）
向 Kafka 发送消息的服务被称为 `producer` ，反之，消费 Kafka 中消息的服务被称为`consumer`。
一个`consumer`可能也是 `producer`，一个 `producer` 可能也是 `consumer`。比如存在服务 A，B，C。服务 A 向 Kafka 发送消息，服务 B 消费服务 A 上报的数据。并且，服务 B 会向 Kafka 发送消息，服务 C 消费服务 B 上报的消息。在微服务架构下，像服务 B 这样同时是 `producer` 和 `consumer` 的情况非常常见。

### Topic（主题）
`Topic` 是 Kafka 中消息具体被发送到的地方，`producer` 在发送消息时会指定一个 `topic`，Kafka 会将这个消息存放在指定的 `topic` 下。相应的，`consumer` 需要消费消息时也需要指定一个 `topic`，去拉取这个 `topic`  下的消息。

一个服务可以发送消息到多个 `topic`，也可以消费多个 `topic` 下的消息。

#### Partition（分区）
一个 `topic` 在外界看起来像是一个队列一样，`producer` 向其中添加数据，`consumer` 从队列中拿数据。
事实上 `topic` 并不是一个单独的类似队列的数据结构，而是由一个或多个（通常是多个）`partition` 组成的。一个 `partition` 类似一个队列，每个消息被发送到 `topic` 时会被添加到这个 `topic` 下的其中一个 `partition` 中。将 `topic` 拆分为 `partition` 使得 `topic` 具有很强的可扩展性。

### Consumer Group（消费者组）