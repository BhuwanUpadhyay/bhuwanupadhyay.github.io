---
title: SSL in spring boot application
cover: /images/fsqs-tips-tricks-notes.png
date: 2020-07-06T10:08:22.921Z
categories: [Spring Boot]
tags: [ssl, wireshark]
---

How to enable SSL in Spring Boot?

<!--more--> 

> Secure Sockets Layer, is an encryption-based Internet security protocol. The primary reason why SSL is used is to keep sensitive information sent across the Internet encrypted so that only the intended recipient can access it. This is important because the information you send on the Internet is passed from computer to computer to get to the destination server. Any computer in between you and the server can see your credit card numbers, usernames and passwords, and other sensitive information if it is not encrypted with an SSL certificate. When an SSL certificate is used, the information becomes unreadable to everyone except for the server you are sending the information to. This protects it from hackers and identity thieves. 
> 
> -« [sslshopper.com](https://www.sslshopper.com/why-ssl-the-purpose-of-using-ssl-certificates.html) »-

## How to hack SSL less Spring Boot Application?

Let's create a simple spring boot application that will have a `POST` method to create an order and `GET` method to get orders.

- Run the following to create order microservice spring boot application:

```bash
curl https://start.spring.io/starter.tgz -d dependencies=actuator,webflux -d language=java -d platformVersion=2.3.1.RELEASE -d javaVersion=14 -d baseDir=ssl-in-spring-boot-application | tar -xzvf -
```

- Create `OrderController.java` and add the following text:

```java
record Order(@JsonProperty("item")String item,
             @JsonProperty("quantity")Integer quantity) {

    public Order {
        if (quantity < 1) {
            throw new IllegalArgumentException("Quantity should be positive number.");
        }
    }
}

@RestController
public class OrderController {
    private final List<Order> orders = new ArrayList<>();

    @PostMapping("/orders")
    public Order create(@RequestBody final Order request) {
        this.orders.add(request);
        return this.orders.get(this.orders.size() - 1);
    }

    @GetMapping("/orders")
    public List<Order> findAll() {
        return this.orders;
    }
}
```

To start the application simply run the command: `mvn spring-boot:run` and verify with the command: `http :8080/actuator/health`.

_**Hack Time!**_

> Wireshark is the widely-used network protocol analyzer. It lets you see what’s happening on your network at a microscopic level. 
> 
> -« [wireshark.org](https://www.wireshark.org/) »-

Go through the following command and steps to install Wireshark in ubuntu.
```bash
# 1. Add ppa repository and install
sudo add-apt-repository ppa:wireshark-dev/stable
sudo apt-get update
sudo apt-get install wireshark

# 2. Should non-superusers be able to capture packets? - Yes

# 3. Give permission
sudo chmod +x /usr/bin/dumpcap
# 4. Logout and login again
# 5. Launch wireshark
wireshark
```

In the Wireshark window select `Loopback: lo` interface, right-click on it, and hit `Start capture` option. As a result, Wireshark started to capture all network packets but we need data that is transferring between the Spring Boot server (in port 8080) and [httpie](https://httpie.org/) client to get it we have to filter `tcp.port == 8080` and hit enter.

Now, let's make `POST` http request on localhost 8080.

```bash
echo '{"item": "hack-item", "quantity": 20}' | http POST :8080/orders
```

Go back to Wireshark you will see a list of packets that is captured when you make this request. Among them, select `POST /orders`, right-click on it and select  `Follow -> TCP Stream` and you will see all values that were sent by the client on this request. The output looks like below:

```
POST /orders HTTP/1.1
Host: localhost:8080
User-Agent: HTTPie/0.9.8
Accept-Encoding: gzip, deflate
Accept: application/json, */*
Connection: keep-alive
Content-Type: application/json
Content-Length: 38

{"item": "hack-item", "quantity": 20}
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 34

{"item":"hack-item","quantity":20}
```

****

## SSL in Spring Boot

Spring Boot provides a set of a declarative [server.ssl.* properties](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-configure-ssl). We can use those properties to configure HTTPS.

### Self-signed Certificate

The tools like [OpenSSL](https://www.openssl.org/), [Keytool](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html), etc. helps to generate a self-signed certificate. For example, to generate a key store (key bag) with keytool:

```bash
keytool -genkey \
          -alias Spring Boot \
          -storetype PKCS12 \
          -keyalg RSA \
          -keysize 2048 \
          -keystore keystore.p12 \
          -validity 4000
```

Here, we store the keys using [PKCS #12](https://en.wikipedia.org/wiki/PKCS_12) which is like a collection of keys such as private keys and public key certifications. We can look inside our generated keystore (keystore.p12) with OpenSSL.

```bash
openssl pkcs12 -info -in keystore.p12
```

### Enable HTTPS

You should copy your generated keystore.p12 and create new `application-ssl.properties` file in your Spring boot application resources with follwing text:

```properties
server.port=8443
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=mypass
server.ssl.keyStoreType=PKCS12
server.ssl.keyAlias=Spring Boot
```

Again, you need to start the application for this simply run the command: `mvn spring-boot:run -Dspring-boot.run.profiles=ssl` and verify with the command: `http --verify=no https://localhost:8443/actuator/health`.

_**Hack Time!**_

This time we have to add a filter in Wireshark `tcp.port == 8443` and hit enter.

Now, let's make `POST` http request on localhost 8080.

```bash
echo '{"item": "hack-item", "quantity": 20}' | http --verify=no POST https://localhost:8443/orders
```

Go back to Wireshark you will see a list of packets that is captured when you make this request. Among them select `Application Data`, right-click on it, and select  `Follow -> TCP Stream` and you will see encrypted values that were sent by the client on this request. The output looks like below:

```
...........XP4..1.'
.....U....... X..U.w:.. .b.e..."....G...".........M.z..J.V.......,.0.+./...................$.(.#.'.
...    ...........k.g.9.3.............=.<.5./.....]........    localhost.........
........ <more>....
```


## That's all folks.

Thanks for reading! Code example on [Github](https://github.com/BhuwanUpadhyay/ssl-in-spring-boot-application).

## References
- https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-configure-ssl
- https://www.sslshopper.com/why-ssl-the-purpose-of-using-ssl-certificates.html
- https://www.youtube.com/watch?v=r0l_54thSYU
- https://wiki.wireshark.org/CaptureSetup/Loopback
