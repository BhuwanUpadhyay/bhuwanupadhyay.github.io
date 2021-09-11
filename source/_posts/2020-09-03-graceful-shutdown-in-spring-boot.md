---
title: Graceful shutdown in Spring Boot
cover: /images/fsqs-tips-tricks-notes.png
date: 2020-09-03T10:08:22.921Z
categories:
  - Spring Boot
tags:
  - graceful-shutdown
---
Upgrades are inevitable in production with the time, consider you have to relocate currently running service in different clusters which require mostly termination of running application and then move to the target cluster.

<!-- more -->

> What will happen with the ongoing requests that are not responded yet by service?

If we did a hard shutdown then the server stopped immediately, no response will receive by the client which might provide a bizarre user experience. It’s important to respond to all ongoing requests properly by service before getting killed. Now graceful shutdown comes into play, the service will block the new requests and will wait for ongoing requests to complete.

> Code is like humor. When you have to explain it, it’s bad.  
> ─ Cory House  

In Spring Boot version 2.3 graceful shutdown is implemented out of the box; when you enabled graceful shutdown, the web server will no longer permit new requests and will wait for a grace period for active requests to complete. Here you can find an example for how to enable a graceful shutdown in spring boot application.

## Define Controller

```java  
@RestController
@RequestMapping("/orders")
public class OrderController {

    @GetMapping
    public List<String> getOrders() throws InterruptedException {
        TimeUnit.SECONDS.sleep(5);
        return List.of("order-1", "order-2");
    }
}
```

## Enable a graceful shutdown

```properties  
server.shutdown=graceful  
```

The grace period can be configured using `spring.lifecycle.timeout-per-shutdown-phase`.

## How to test?

Run your application then call API `http://localhost:8080/orders`, the response will take 5 secs, 
before 5 secs try to stop running the application. As a result, the server will not stop until your current request is not complete.

## References

- https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.3-Release-Notes#graceful-shutdown
