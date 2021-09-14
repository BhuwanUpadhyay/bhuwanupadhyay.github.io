---
title: Safely Exchanging Information Across Microservices
date: 2020-05-09T10:08:22.921Z
categories: [Spring Cloud]
tags: [spring-cloud-stream, avro-schema, schema-registry]
cover: https://github.com/bhuwanupadhyay/safely-exchanging-information-across-microservices/raw/master/assets/featured.png
---

Exchanging information between the microservices without breaking existing functionalities is a challenging task specially if your business model evolve with the time. It's very important to ensure new updates on the microservices should be seamless and have backward compatibility.

<!-- more -->

Specially, if our microservices communicate each other by using pub/sub architecture and have multiple
producers and consumers, it is necessary for all those microservices to agree on a contract that is based on a schema. Because, to accommodate new business requirements the message payload structure might needs to evolve, and the existing components are still required to continue to work.

 In this article, I will take you through how we can used evolving schemas to exchange information between microservices using Spring Boot and Spring Cloud.

## Avro Schema

Avro is used to define the schema for a message's payload. This schema describes the fields allowed in the payload, along with their data types. Avro bindings are used to serialize values before writing them, and to deserialize values after reading them. The usage of these bindings requires your applications to use the Avro data format, which means that each payload is associated with a schema.

In addition, Avro makes use of the Jackson APIs for parsing JSON. This is likely to be of interest to you if you are familiar with a JSON-based system.

## Schema Registry

For evolving schemas, we need to register them somewhere to share between the microservices without
any manual updates. This leads to the consumers have to read schema definitions from a registry and publisher needs to provide schema definitions to a registry. To address this philosophy, the concept of schema registry come into the picture.

> [Spring Cloud Schema Registry](https://spring.io/projects/spring-cloud-schema-registry) provides support for schema evolution so that the data can be evolved over time and still work with older or newer producers and consumers and vice versa.

## Schema Registry Server

To use Spring Cloud Schema Registry Server in a Maven [Spring Boot](https://start.spring.io/) projects, we need to have `spring-cloud-schema-registry-server` from Spring Cloud in the project pom.xml:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-schema-registry-server</artifactId>
</dependency>
```

To enable schema registry server in spring boot, we need to use the annotation `@EnableSchemaRegistryServer` on the main application class:

```java
@Spring BootApplication
@EnableSchemaRegistryServer
public class SchemaRegistryApplication {

  public static void main(String[] args) {
    SpringApplication.run(SchemaRegistryApplication.class, args);
  }
}
```

Schema registry server uses `8990` as a default port for application and the example of http `GET` request to fetch schemas by its id will be looks like below:

```bash
curl -X GET http://localhost:8990/schemas/<id>
```

## Schema Registry Client

To use Spring Cloud Schema Registry Client in a Maven [Spring Boot](https://start.spring.io/) projects, we need to have `spring-cloud-schema-registry-client` from Spring Cloud in the project pom.xml:

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-schema-registry-client</artifactId>
</dependency>
```

To enable schema registry client in spring boot, we need to use the annotation `@EnableSchemaRegistryCLient` on the main application class:

```java
@Spring BootApplication
@EnableSchemaRegistryClient
public class OrderServiceApplication {

  public static void main(String[] args) {
    SpringApplication.run(OrderServiceApplication.class, args);
  }
}
```

## Story -- Exchanging Information

Let's consider we have to exchange messages between **Order Service** and **Payment Service**. Order and Payment microservices solution is below:

![](https://raw.githubusercontent.com/BhuwanUpadhyay/safely-exchanging-information-across-microservices/master/assets/example.png)

As an above diagram, we have three domain events that are **OrderPlaced**, **PaymentRequested** and **Payment Received**. A sequence diagram for order workflow as below:

![](/images/safely-exhange-message-order-domain.png)

## Create Avro Schemas

Let's create first version avro schemas for **PaymentRequested** and **Payment Received** because those domain events are not consumed inside bounded context.

- **PaymentRequestedV1** will publish by [Order Service](https://github.com/BhuwanUpadhyay/safely-exchanging-information-across-microservices/blob/master/order-service)

```
{
  "name": "PaymentRequestedV1",
  "type": "record",
  "namespace": "io.github.bhuwanupadhyay.schemas",
  "fields": [
    {
      "name": "orderId",
      "type": "string"
    },
    {
        "name": "orderAmount",
        "type": "string"
    }
  ]
}
```

- **PaymentReceivedV1** will publish by [Payment Service](https://github.com/BhuwanUpadhyay/safely-exchanging-information-across-microservices/blob/master/payment-service)

```
{
  "name": "PaymentReceivedV1",
  "type": "record",
  "namespace": "io.github.bhuwanupadhyay.schemas",
  "fields": [
    {
      "name": "orderId",
      "type": "string"
    },
    {
        "name": "paymentId",
        "type": "string"
    }
  ]
}
```

After sometime, business need to add `customerId` attribute on **PaymentRequested** message payload for **Order Service**. To do so we need to define a second version of schema i.e. **PaymentRequestedV2** and use it in the **Order Service**.

- **PaymentRequestedV2** will publish by [Order Service](https://github.com/BhuwanUpadhyay/safely-exchanging-information-across-microservices/blob/master/order-service) after upgrade.

```
{
  "name": "PaymentRequestedV2",
  "type": "record",
  "namespace": "io.github.bhuwanupadhyay.schemas",
  "fields": [
    {
      "name": "orderId",
      "type": "string"
    },
    {
        "name": "orderAmount",
        "type": "string"
    },
    {
      "name": "customerId",
      "type": "string",
      "default": ""
    }
  ]
}
```

## Order Service

Once schema upgraded for a message **PaymentRequested** then order service will publish the latest version i.e. **V2** to their consumers:  

```java
@Slf4j
@Service
@EnableBinding(OrderEventSource.class)
@RequiredArgsConstructor
public class OrderEventPublisherService {
  private final OrderEventSource orderEventSource;

  @TransactionalEventListener // Attach it to the transaction of the repository operation
  public void handlePaymentRequestedEvent(PaymentRequestedEvent paymentRequestedEvent) {
    LOG.info("Handling event [PaymentRequestedEvent].");
    final PaymentRequestedV2 paymentRequested =
        PaymentRequestedV2.newBuilder()
            .setOrderId(paymentRequestedEvent.getOrderId().getOrderId())
            .setOrderAmount(paymentRequestedEvent.getOrderAmount().asString())
            .setCustomerId(paymentRequestedEvent.getCustomerId().getCustomerId())
            .build();
    orderEventSource
        .paymentRequested()
        .send(MessageBuilder.withPayload(paymentRequested).build()); // Publish the event
    LOG.info("Successfully published the event [PaymentRequestedV2].");
  }
}
```

## Payment Service

Still, payment service is consuming old version i.e. **V1**  for a message **PaymentRequested**:

```java
@Slf4j
@Service
@RequiredArgsConstructor
@EnableBinding(PaymentEventSource.class) // Bind to the channel connection for the message
public class PaymentEventHandler {

  private final CreatePaymentCommandService createPaymentCommandService;

  // Listen to the stream of messages on the destination
  @StreamListener(target = PaymentEventSource.PAYMENT_REQUESTED_CHANNEL)
  public void receiveEvent(PaymentRequestedV1 paymentRequested) {
    LOG.info("Receive event [PaymentRequestedV1].");
    LOG.debug("Event payload {}.", paymentRequested);
    final CreatePaymentCommand createPaymentCommand = new CreatePaymentCommand();
    createPaymentCommand.setOrderId(paymentRequested.getOrderId().toString());
    createPaymentCommand.setOrderAmount(paymentRequested.getOrderAmount().toString());
    createPaymentCommandService.createPayment(createPaymentCommand);
    LOG.info("Successfully processed event [PaymentRequestedV1].");
  }
}
```

## Run Example

Use [docker-compose.yaml](https://github.com/BhuwanUpadhyay/safely-exchanging-information-across-microservices/blob/master/docker-compose.yaml) to run necessary infrastructure for microservices.

```bash
docker-compose up
```

Clone [Example Github Project](https://github.com/BhuwanUpadhyay/safely-exchanging-information-across-microservices) in your directory.

- Build

```bash
  make pull && make build
```

- Run Schema Registry -- (On New Terminal)

```bash
  make pull && make build
```

- Run Order Service -- (On New Terminal)

```bash
  make order_service
```

- Run Payment Service -- (On New Terminal)

```bash
  make payment_service
```

- Run Test - Perform following http request to test microservices.

```http
### Create New Order
POST http://localhost:8080/orders
Content-Type: application/json

{
  "itemId": "ITM00001",
  "quantity": 20,
  "customerId": "CUST00001"
}

### Get orders
GET http://localhost:8080/orders

### Get payments
GET http://localhost:8081/payments
```

## Conclusion

In the above example, microservices were able to communicate between each other seamlessly even we applied schema changes on producer service but not on consumer service.

Finally, we can exchange information between microservices safely by using schema registry and agree upon some sort of contracts between the microservice for message payloads.

You can find example on github: [<i class="fab fa-github"></i> Source Code](https://github.com/BhuwanUpadhyay/safely-exchanging-information-across-microservices)
