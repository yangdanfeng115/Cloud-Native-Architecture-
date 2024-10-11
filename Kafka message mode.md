Here’s how to implement Publish-Subscribe Model and Point-to-Point Model in Kafka using Java. These examples demonstrate how to configure consumers and producers for each model.

1. Publish-Subscribe Model (Pub-Sub)
In this model, each consumer is independent (they don’t share a consumer group). Every consumer receives all the messages published to the topic.

Pub-Sub Consumer Code:

```java

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class PubSubConsumer {
    public static void main(String[] args) {
        // Create consumer configuration
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

        // No group.id for Publish-Subscribe pattern
        // Uncomment this line if you want each consumer to have a unique group
        // properties.put(ConsumerConfig.GROUP_ID_CONFIG, "consumer-group-" + UUID.randomUUID().toString());

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
        consumer.subscribe(Collections.singletonList("orders"));  // Subscribe to the topic

        // Poll and consume messages
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("Received message: key = %s, value = %s%n", record.key(), record.value());
            }
        }
    }
}
```
Explanation:
Each consumer here is independent and does not belong to a consumer group (group.id is not set), meaning each consumer will receive all messages from the orders topic.
Start multiple instances of this consumer to see the publish-subscribe behavior. Each instance will get a copy of every message.

**2. Point-to-Point Model (P2P)**

In this model, consumers are part of the same consumer group. Kafka will distribute messages among them. Each message is only consumed by one consumer within the group.

Point-to-Point Consumer Code:

```java
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class PointToPointConsumer {
    public static void main(String[] args) {
        // Create consumer configuration
        Properties properties = new Properties();
        properties.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        properties.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());

        // Configure the group.id to ensure consumers are in the same group
        properties.put(ConsumerConfig.GROUP_ID_CONFIG, "order-consumer-group");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
        consumer.subscribe(Collections.singletonList("orders"));  // Subscribe to the topic

        // Poll and consume messages
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
            for (ConsumerRecord<String, String> record : records) {
                System.out.printf("Received message: key = %s, value = %s%n", record.key(), record.value());
            }
        }
    }
}
```
Explanation:
Consumers here are part of the same consumer group (order-consumer-group), meaning each message published to the topic will only be processed by one consumer in the group.
If you run multiple instances of this consumer, they will share the workload by dividing the messages.

**3. Producer Code (Same for Both Models)**

Regardless of whether you use Pub-Sub or Point-to-Point, the producer code remains the same. The producer sends messages to a topic (orders in this example), and how the messages are distributed depends on the consumer configuration.

Producer Code:
```java

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;

import java.util.Properties;

public class OrderProducer {
    public static void main(String[] args) {
        // Create producer configuration
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

        // Send 10 messages to the "orders" topic
        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> record = new ProducerRecord<>("orders", "key" + i, "Order " + i);
            producer.send(record);
            System.out.printf("Sent message: key = %s, value = %s%n", "key" + i, "Order " + i);
        }

        // Close producer
        producer.close();
    }
}
```
**Explanation:**
The producer sends messages to the orders topic.
You can run the producer once, and it will produce messages that will be consumed either in Pub-Sub mode (where all consumers get every message) or Point-to-Point mode (where messages are distributed across consumers in the same group).

**Key Points:**

**Publish-Subscribe Model:**

Multiple consumers independently subscribe to a topic.
Each consumer receives all messages.
To achieve this, do not set the group.id for consumers, or set unique group IDs for each consumer.


**Point-to-Point Model:**

Multiple consumers belong to the same consumer group.
Kafka distributes messages across the consumers in that group.
To achieve this, ensure consumers have the same group.id.
Both approaches demonstrate how Kafka can handle different messaging patterns depending on how the consumers are configured.
