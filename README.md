# Apache Kafka - Complete Guide

## Introduction

Apache Kafka is a distributed event streaming platform used for high-performance data pipelines, streaming analytics, data integration, and mission-critical applications. Kafka is horizontally scalable, fault-tolerant, fast, and runs in production in thousands of companies.

**Use Cases:**

* Messaging
* Website Activity Tracking
* Metrics Collection and Monitoring
* Log Aggregation
* Stream Processing

## Kafka Architecture

### Core Concepts

* **Producer**: Sends messages to a Kafka topic.
* **Consumer**: Reads messages from a Kafka topic.
* **Broker**: A Kafka server that stores and serves messages.
* **Topic**: A category/feed name to which messages are published.
* **Partition**: Topics are split into partitions for scalability.
* **Zookeeper**: Manages Kafka brokers and cluster state (in Kafka <= 2.x).

### Message Flow

```
Producer --> Kafka Topic (partitioned) --> Consumer Group
```

## Kafka Setup

### Prerequisites

* Java 8+
* Apache Kafka binary ([https://kafka.apache.org/downloads](https://kafka.apache.org/downloads))

### Steps

```bash
# 1. Start Zookeeper (required in Kafka <= 2.x)
bin/zookeeper-server-start.sh config/zookeeper.properties

# 2. Start Kafka broker
bin/kafka-server-start.sh config/server.properties
```

### Create Topic

```bash
bin/kafka-topics.sh --create \
--topic test-topic \
--bootstrap-server localhost:9092 \
--partitions 3 \
--replication-factor 1
```

## Kafka CLI Commands

### List Topics

```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

### Describe Topic

```bash
bin/kafka-topics.sh --describe --topic test-topic --bootstrap-server localhost:9092
```

### Produce Messages

```bash
bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092
```

### Consume Messages

```bash
bin/kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server localhost:9092
```

## Kafka with Spring Boot

### Dependencies (Maven)

```xml
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
</dependency>
```

## `application.properties` Configuration

```properties
# Kafka bootstrap servers
spring.kafka.bootstrap-servers=localhost:9092

# Producer configuration
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=org.apache.kafka.common.serialization.StringSerializer

# Consumer configuration
spring.kafka.consumer.key-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.value-deserializer=org.apache.kafka.common.serialization.StringDeserializer
spring.kafka.consumer.group-id=dev-group
spring.kafka.consumer.auto-offset-reset=earliest
```

### Producer Configuration

```java
@Configuration
public class KafkaProducerConfig {
    @Bean
    public ProducerFactory<String, String> producerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        config.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        return new DefaultKafkaProducerFactory<>(config);
    }

    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
}
```

### Consumer Configuration

```java
@Configuration
@EnableKafka
public class KafkaConsumerConfig {
    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> config = new HashMap<>();
        config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        config.put(ConsumerConfig.GROUP_ID_CONFIG, "group_id");
        config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(config);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}
```

### Producer Example

```java
@Service
public class KafkaProducer {
    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    public void sendMessage(String msg) {
        kafkaTemplate.send("test-topic", msg);
    }
}
```

### Consumer Example

```java
@Service
public class KafkaConsumer {
    @KafkaListener(topics = "test-topic", groupId = "group_id")
    public void listen(String message) {
        System.out.println("Received: " + message);
    }
}
```

## Summary

Apache Kafka is a powerful distributed streaming platform used to build real-time data pipelines and applications. This guide covered its architecture, setup, CLI usage, and integration with Spring Boot.
