# What is Kafka & What is Event-Driven Architecture?

## What is Kafka?

Apache Kafka is a **distributed event streaming platform** designed to handle high-throughput, low-latency data pipelines. It enables systems to publish and subscribe to streams of records in real time.

- Built for **horizontal scalability**
- Designed to be **fault-tolerant** and **durable**
- Supports **real-time stream processing**

Kafka is at the heart of modern data ecosystems.

## What is Event-Driven Architecture (EDA)?

Event-Driven Architecture is a software design pattern where components communicate by producing and consuming **events**.

### How it Works

1. **Producer** generates an event (e.g., "UserSignedUp")
2. Event is published to a **message broker** (e.g., Kafka)
3. One or more **Consumers** subscribe and react to that event


## Kafka's Role in EDA

Kafka acts as the **central nervous system** in an event-driven system:
- **Stores events durably**
- **Distributes events** to multiple systems
- Enables **decoupling** of microservices


## Benefits of EDA with Kafka

- Asynchronous communication
- Scalability and resilience
- Loose coupling between producers and consumers
- Better system observability and auditability

<!-- ![Event Driven Architecture](./assets/event-driven-architecture.png) -->