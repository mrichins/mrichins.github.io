---
title: "Testing Kafka-Based Microservices"
layout: post
date: 2017-01-05 12:01
image: /assets/images/markdown.jpg
headerImage: false
tag:
-
blog: true
star: true
author: masonrichins
description: A Brief Description of Approaches I've Used in Testing Kafka-Based Microservices
---

## Introduction to Kafka Microservices

Usually, services in a large enterprise application use HTTP requests to pass information between each other. While this generally works okay for one-off information transfers between two services, it is not ideal for applications where stream processing of large volumes of requests. Sometimes requests will arbitrarily fail, processing of messages might fail for external reasons that just need a retry ,or maybe you want to create a swarm of services that work as a group of consumers or producers. While you can work around HTTP for these specific use cases, Kafka (along with other message broker systems like RabbitMQ) is becoming an increasingly popular tool for tackling problems, and its not hard to see why.

Although Kafka is basically an advanced pub/sub messaging system, the consumers and producers have access to delivery features that are not found elsewhere. For example, the people at [Confluent](https://www.confluent.io/) have included several tools such as a REST Proxy, Kafka-Connect, and a Schema Registry.

Although I could spend the rest of this post discussing the benefits of using Kafka, it still has problems, and no technology is able to validate all of its own business logic without external testing. So onto the main part of this post...



## So then how do we test it?

Kafka microservices are the integration point between multiple services. Before even worrying about testing the kafka messages, I would hope that there is extensive testing within each service for handling messages that it has subscribed to, and posting messages when appropriate. However, just because these are tested, it does not mean that there should not be testing on the messages themselves against multiple deployed services.

In order to test the messages that are coming through the Kafka message busses, I find that setting up an independent consumer that also subscribes to the topics used by the rest of the system is a great, non-intrusive way of verifying that messages are getting passed and that certain domain transformations are happening as data passes through various services.

## Setting up a basic consumer

Setting up a consumer is actually very easy using the Kafka consumer. As a quick example, you can set up a simple consumer, and begin polling for messages with just this code taken from the Apache documents:

```java
Properties props = new Properties();
     props.put("bootstrap.servers", "localhost:9092");
     props.put("group.id", "test");
     props.put("enable.auto.commit", "true");
     props.put("auto.commit.interval.ms", "1000");
     props.put("session.timeout.ms", "30000");
     props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
     KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
     consumer.subscribe(Arrays.asList("foo", "bar"));
     while (true) {
         ConsumerRecords<String, String> records = consumer.poll(100);
         for (ConsumerRecord<String, String> record : records)
             System.out.printf("offset = %d, key = %s, value = %s", record.offset(), record.key(), record.value());
     }
```

While most of the properties are generic, some of them are very important to understand while setting up a consumer for testing purposes. For instance, the 'group.id' property assigns the consumer to a group. Kafka will (in most installations) ensure that each group will consume a message exactly once. That is fantastic for a production environment, but unfortunate if you are going to be testing different messages on the same Kafka topic, or would like to be able to re-run my tests against the same set of messages. I have found that using a timestamp to make sure that this property is always unique provides a good enough solution. There is also a property for how often to commit the offset. Simply described, this is how often the consumer would indicate that it has successfully processed a message.

## Beyond just reading messages

Once we have a consumer reading your Kafka message streams hooked into your testing framework, then we can start actually making assertions on what our system is doing. While it might be tempting to write assertions that methods within your service publish messages to a Kafka topic, all you're doing at that point is testing the confluent framework itself.

Instead, it is valuable to be able to check the flow of messages through your system. This follows some of the principals of [Hexagonal Architecture](http://alistair.cockburn.us/Hexagonal+architecture) where we concern ourself with inputs and outputs of a microservice, rather than looking deeply into the application logic. In essence, these tests explore the integration between your components that use Kafka and ensure that your data flows correctly.

So lets explore how we would test for the following requirement:

```
"given a database insert to a the users table"
"when kafka-connect publishes the data message to Kafka"
"then the user service publishes a 'user-created' message'"
```

Now we just need simple method to validate that a topic had any messages at all, and map this across all of the topics that I expected to see any messages on.

```scala
  def verifyTopicHasRecords(topic: String) = {
  assert(pollTopicForRecords[String, String](topic).count() > 0, s" $topic did not have records")
}
```

So in our test setup we would insert whatever data into the database, then verify that all of the topics that we expect to see messages have them (assuming a starting empty state of the Kafka message bus). If my database insert does not result in this flow of messages, then I know that something has prevented message propagation i.e. if I don't see that 'user-created' method, I know that my user service has failed to process the kafka-connect message correctly. Additionally the specificity of the error means that you can easily identify the point of failure, because you might have a much longer string of events to test.

## Building Onward

Later on you can drill down to increase the specificity on the tests depending on the topic and expected values. Maybe you expect the 'user-created' event to have applied some transformation, or for some correlation identifier to exist and correspond across specific events and commands. With Kafka-Based Microservices testing your Kafka messages gives you an extremely robust testing framework.

There are some housekeeping tasks that I did not have the space to explore in detail, but it is quite simple to get the. One of the main issues I ended up running into was a way to purge all messages from Kafka to get into a test-ready state. There were some options to purge a running Kafka deployment available from Confluent, but at the time none of them seemed to completely purge the Kafka instance, but that has hopefully been resolved. Our short term solution was to bounce Kafka, but that was a heavyweight solution to a lightweight problem.

## (Optional Additional Information) Additional Notes on Confluent Kafka

The REST Proxy allows you to interface with the Kafka bus rather than having to configure it on startup every time, it also allows you to post messages to topics without a true producer. Kafka connect allows you to both bootstrap data from an existing data source, or to convert changes in a database to messages processed by the services. Lastly, the schema registry, which defines the objects that are getting passed between services, replaces a need for robust integration testing between services, provided you use its build-in compatibility checking tools. Together, all of these tools give teams the ability to easily and confidently pass messages between services using the Kafka bus.
