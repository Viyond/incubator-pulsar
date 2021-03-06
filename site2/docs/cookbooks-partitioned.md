---
id: cookbooks-partitioned
title: Non-persistent messaging
sidebar_label: Partitioned Topics
---

By default, Pulsar topics are served by a single broker. Using only a single broker, however, limits a topic's maximum throughput. *Partitioned topics* are a special type of topic that can span multiple brokers and thus allow for much higher throughput. For an explanation of how partitioned topics work, see the [Partitioned Topics](getting-started-concepts-and-architecture.md#partitioned-topics) concepts.

You can [publish](#publishing-to-partitioned-topics) to partitioned topics using Pulsar's client libraries and you can [create and manage](#managing-partitioned-topics) partitioned topics using Pulsar's [admin API](admin-api-overview.md).

## Publishing to partitioned topics

When publishing to partitioned topics, the only difference from non-partitioned topics is that you need to specify a [routing mode](getting-started-concepts-and-architecture.md#routing-modes) when you create a new [producer](reference-terminology.md#producer). Examples for [Java](#java) are below.

### Java

Publishing messages to partitioned topics in the Java client works much like [publishing to normal topics](client-libraries-java.md#using-producers). The difference is that you need to specify either one of the currently available message routers or a custom router.

#### Routing mode

You can specify the routing mode in the ProducerConfiguration object that you use to configure your producer. You have three options:

* `SinglePartition`
* `RoundRobinPartition`
* `CustomPartition`

Here's an example:

```java
String pulsarBrokerRootUrl = "pulsar://localhost:6650";
String topic = "persistent://my-tenant/my-namespace/my-topic";

PulsarClient client = PulsarClient.create(pulsarBrokerRootUrl);
ProducerConfiguration config = new ProducerConfiguration();
config.setMessageRoutingMode(ProducerConfiguration.MessageRoutingMode.SinglePartition);
Producer producer = client.createProducer(topic, config);
producer.send("Partitioned topic message".getBytes());
```

#### Custom message router

To use a custom message router, you need to provide an implementation of the {@inject: javadoc:MessageRouter:/client/org/apache/pulsar/client/api/MessageRouter} interface, which has just one `choosePartition` method:

```java
public interface MessageRouter extends Serializable {
    int choosePartition(Message msg);
}
```

Here's a (not very useful!) router that routes every message to partition 10:

```java
public class AlwaysTenRouter implements MessageRouter {
    public int choosePartition(Message msg) {
        return 10;
    }
}
```

With that implementation in hand, you can send

```java
String pulsarBrokerRootUrl = "pulsar://localhost:6650";
String topic = "persistent://my-property/my-cluster-my-namespace/my-topic";

PulsarClient client = PulsarClient.create(pulsarBrokerRootUrl);
ProducerConfiguration config = new ProducerConfiguration();
config.setMessageRouter(AlwaysTenRouter);
Producer producer = client.createProducer(topic, config);
producer.send("Partitioned topic message".getBytes());
```

## Managing partitioned topics

You can use Pulsar's [admin API](admin-api-overview.md) to create and manage [partitioned topics](admin-api-partitioned-topics.md).
