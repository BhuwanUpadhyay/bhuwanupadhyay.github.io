---
title: RSocket with Spring Boot
cover: /images/fsqs-tips-tricks-notes.png
date: 2020-08-17T10:08:22.921Z
categories: [Spring Boot]
tags: [rsocket]
---

How to use RSocket in Spring Boot?

<!-- more -->

## RSocket

> It is a connection-oriented, message-driven protocol with built-in flow control at the application level. It works in a browser equally as well as on a server. In fact, a web browser can serve traffic to backend microservices. It is also binary. It works equally well with text and binary data, and the payloads can be fragmented. It models all the interactions that you do in your application as network primitives. This means you can stream data or do Pub/Sub without having to setup an application queue.
>
> -- [InfoQ](https://www.infoq.com/articles/give-rest-a-rest-rsocket/)

In this article, you’ll discover how to do request-response with RSocket using Spring Boot.

## Step 1: Setup the Server Code

### The Project File
 
In the project’s `pom.xml` file, you need to add the dependency for rsocket.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-rsocket</artifactId>
</dependency>
```

### The Application Properties

In the `application.properties` file, add the following text:

```properties
spring.rsocket.server.port=7000
```

### The Message Class

```java
record Message(
        @JsonProperty("source") String source,
        @JsonProperty("message") String message,
        @JsonProperty("createdAt") long createdAt
) {
    Message {
        createdAt = Instant.now().getEpochSecond();
    }
}
```

### The Controller Class

```java
@Controller
@Slf4j
class RSocketController {

    @MessageMapping("app-socket")
    public Message appSocket(Message message) {
        log.info("Received request: {}", message);
        return message;
    }
}
```

That’s it for code. Let’s try it.

## Step 2: Start The Spring Boot RSocket Server

```shell script
./mvnw clean package spring-boot:run
```

## Step 4: Send A Command To The Server With The RSocket CLI

Next, download [RSocket Client CLI](https://github.com/making/rsc) and test cli:

```shell script
wget -O rsc.jar https://github.com/making/rsc/releases/download/0.5.0/rsc-0.5.0.jar

java -jar rsc.jar --help
```

Next, you’ll send a message to the running server using the RSocket client:

```shell script
java -jar rsc.jar --debug --request --data "{\"source\":\"LeadByExamples\",\"message\":\"Hello, RSocket!\"}" --route app-socket tcp://localhost:7000
``` 

### The Client Output

```
2020-08-17 08:14:10.584 DEBUG --- [actor-tcp-nio-1] i.r.FrameLogger : sending -> 
Frame => Stream ID: 1 Type: REQUEST_RESPONSE Flags: 0b100000000 Length: 74
Metadata:
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 0a 61 70 70 2d 73 6f 63 6b 65 74                |.app-socket     |
+--------+-------------------------------------------------+----------------+
Data:
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 7b 22 73 6f 75 72 63 65 22 3a 22 4c 65 61 64 42 |{"source":"LeadB|
|00000010| 79 45 78 61 6d 70 6c 65 73 22 2c 22 6d 65 73 73 |yExamples","mess|
|00000020| 61 67 65 22 3a 22 48 65 6c 6c 6f 2c 20 52 53 6f |age":"Hello, RSo|
|00000030| 63 6b 65 74 22 7d                               |cket"}          |
+--------+-------------------------------------------------+----------------+
2020-08-17 08:14:10.657 DEBUG --- [actor-tcp-nio-1] i.r.FrameLogger : receiving -> 
Frame => Stream ID: 1 Type: NEXT_COMPLETE Flags: 0b1100000 Length: 83
Data:
         +-------------------------------------------------+
         |  0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f |
+--------+-------------------------------------------------+----------------+
|00000000| 7b 22 73 6f 75 72 63 65 22 3a 22 4c 65 61 64 42 |{"source":"LeadB|
|00000010| 79 45 78 61 6d 70 6c 65 73 22 2c 22 6d 65 73 73 |yExamples","mess|
|00000020| 61 67 65 22 3a 22 48 65 6c 6c 6f 2c 20 52 53 6f |age":"Hello, RSo|
|00000030| 63 6b 65 74 22 2c 22 63 72 65 61 74 65 64 41 74 |cket","createdAt|
|00000040| 22 3a 31 35 39 37 36 33 31 33 35 30 7d          |":1597631350}   |
+--------+-------------------------------------------------+----------------+
{"source":"LeadByExamples","message":"Hello, RSocket","createdAt":1597631350}
```
