# Kafka Architecture

### Components

- **Producer**
- **Consumer**
- **Broker**
- **Topic / Partitions**
- **ZooKeeper / KRaft**

### Kafka Cluster

Multiple brokers working together for scalability and fault tolerance.

### ZooKeeper vs KRaft

- **ZooKeeper:** Legacy dependency for metadata management
- **KRaft:** Kafka's newer internal consensus protocol (no ZooKeeper)

<!-- ![Kafka Architecture](./assets/kafka-architecture.png) -->

> Kafka 3.x+ is moving toward KRaft mode (ZooKeeper-less).