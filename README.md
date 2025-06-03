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

### Common Assumptions

* Kafka is downloaded and extracted (e.g., in `~/kafka_2.13-3.6.1/`)
* Kafka is running on `localhost:9092`
* Zookeeper is running (if using pre-KRaft versions)
* Java is installed and in your `PATH`

### Folder Structure

Assume the Kafka folder is referred to as:

* **Linux/macOS**: `~/kafka`
* **Windows**: `C:\kafka`
  (or the full path where you extracted Kafka)

### 1. Start Zookeeper (if not using KRaft mode)

#### Mac/Linux

```bash
bin/zookeeper-server-start.sh config/zookeeper.properties
```

#### Windows

```cmd
bin\windows\zookeeper-server-start.bat config\zookeeper.properties
```

### 2. Start Kafka Broker

#### Mac/Linux

```bash
bin/kafka-server-start.sh config/server.properties
```

#### Windows

```cmd
bin\windows\kafka-server-start.bat config\server.properties
```

### 3. Create a Topic

#### Mac/Linux

```bash
bin/kafka-topics.sh --create --topic my-topic \
  --bootstrap-server localhost:9092 \
  --partitions 1 --replication-factor 1
```

#### Windows

```cmd
bin\windows\kafka-topics.bat --create --topic my-topic ^
  --bootstrap-server localhost:9092 ^
  --partitions 1 --replication-factor 1
```

### 4. List Topics

#### Mac/Linux

```bash
bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

#### Windows

```cmd
bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
```

### 5. Produce Messages

#### Mac/Linux

```bash
bin/kafka-console-producer.sh --topic my-topic --bootstrap-server localhost:9092
```

#### Windows

```cmd
bin\windows\kafka-console-producer.bat --topic my-topic --bootstrap-server localhost:9092
```

* Start typing messages and press `Enter` after each one.

### 6. Consume Messages

#### Mac/Linux

```bash
bin/kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:9092
```

#### Windows

```cmd
bin\windows\kafka-console-consumer.bat --topic my-topic --from-beginning --bootstrap-server localhost:9092
```

### 7. Delete Topic (Optional)

#### Mac/Linux

```bash
bin/kafka-topics.sh --delete --topic my-topic --bootstrap-server localhost:9092
```

#### Windows

```cmd
bin\windows\kafka-topics.bat --delete --topic my-topic --bootstrap-server localhost:9092
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

In Kafka (using Java/Spring or plain Kafka Java API), we can send messages both **synchronously** and **asynchronously** using the `KafkaProducer` API or Spring's `KafkaTemplate`.

## 1. **Using Kafka Producer (Plain Java API)**

### Setup

Make sure to include the Kafka client dependency:

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.6.1</version>
</dependency>
```

### Producer Config

```java
Properties props = new Properties();
props.put("bootstrap.servers", "localhost:9092");
props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
```

### Sync Send

```java
try {
    RecordMetadata metadata = producer.send(new ProducerRecord<>("my-topic", "key", "Hello Kafka")).get(); // Blocks
    System.out.printf("Sent sync: partition=%d, offset=%d%n", metadata.partition(), metadata.offset());
} catch (Exception e) {
    e.printStackTrace();
}
```

> `.get()` blocks until the message is acknowledged or failed.

### Async Send

```java
producer.send(new ProducerRecord<>("my-topic", "key", "Hello Kafka"), (metadata, exception) -> {
    if (exception == null) {
        System.out.printf("Sent async: partition=%d, offset=%d%n", metadata.partition(), metadata.offset());
    } else {
        exception.printStackTrace();
    }
});
```

> Async uses a callback and does **not block**.

## 2. **Using KafkaTemplate (Spring Boot)**

In Spring Boot, `KafkaTemplate` abstracts this and supports both sync and async.

### Auto Configuration

Spring Boot auto-configures `KafkaTemplate` if `spring-kafka` is in your `pom.xml`.

### Async Send (default)

```java
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

public void sendAsync() {
    kafkaTemplate.send("my-topic", "Hello from Kafka (async)")
        .addCallback(
            result -> {
                if (result != null) {
                    System.out.println("Sent async to partition: " + result.getRecordMetadata().partition());
                }
            },
            ex -> System.err.println("Send failed: " + ex.getMessage())
        );
}
```

### Sync Send

```java
public void sendSync() {
    try {
        SendResult<String, String> result = kafkaTemplate
            .send("my-topic", "Hello from Kafka (sync)")
            .get(); // blocks

        System.out.println("Sent sync to partition: " + result.getRecordMetadata().partition());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

> `.get()` on the returned `ListenableFuture` blocks until the broker acknowledges.

## Notes

| Sync vs Async | Characteristics                                                        |
| ------------- | ---------------------------------------------------------------------- |
| Sync        | Blocks the thread, waits for acknowledgment, easier to handle failures |
| Async       | Non-blocking, better performance, needs callback to handle errors      |

## Summary

Apache Kafka is a powerful distributed streaming platform used to build real-time data pipelines and applications. This guide covered its architecture, setup, CLI usage, and integration with Spring Boot.
