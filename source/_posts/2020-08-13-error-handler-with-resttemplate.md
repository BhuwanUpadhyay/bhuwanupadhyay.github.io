---
title: Error Handler with RestTemplate
cover: /images/fsqs-tips-tricks-notes.png
date: 2020-08-13T10:08:22.921Z
categories: [Spring]
tags: [rest-template, error-handler]
---

In this code snippet, I will show how to implement and inject the ResponseErrorHandler interface in a RestTemplate instance - to gracefully handle HTTP errors returned by remote APIs.

<!-- more -->

> The second law of thermodynamics, in principle, states that a closed system's disorder cannot be reduced, it can only remain unchanged or increase. A measure of this disorder is entropy. This law also seems plausible for software systems; as a system is modified, its disorder, or entropy, tends to increase. This is known as software entropy. - wikipedia

```java
class ClientRestException extends RuntimeException {

    public ClientRestException(String message) {
        super(message);
    }
}

class ClientRestTemplateErrorHandler extends DefaultResponseErrorHandler {

    @Override
    public void handleError(ClientHttpResponse response) throws IOException {
        if (response.getStatusCode().is4xxClientError() || response.getStatusCode().is5xxServerError()) {
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(response.getBody()))) {
                String httpBodyResponse = reader.lines().collect(Collectors.joining(""));
                throw new ClientRestException(httpBodyResponse);
            }
        }
    }
}

record OrderInfo(@JsonProperty("customerId") String customerId,
                 @JsonProperty("itemId") String itemId,
                 @JsonProperty("quantity") Integer quantity) {

}

@Service
class OrderServiceClient {

    private final String orderServiceUrl;
    private final RestTemplate restTemplate;

    public OrderServiceClient(@Value("${order-service.url:http://localhost:8080}") String orderServiceUrl) {
        this.orderServiceUrl = orderServiceUrl;
        this.restTemplate = new RestTemplate();
        this.restTemplate.setErrorHandler(new ClientRestTemplateErrorHandler());
    }

    public OrderInfo getOrderInfo(String orderId) {
        URI uri = UriComponentsBuilder.fromHttpUrl(orderServiceUrl)
                .path("/orders/{orderId}").queryParam("expired", "NO").build(orderId);

        RequestEntity<Void> getByOrderId = RequestEntity.get(uri).build();

        return restTemplate.exchange(getByOrderId, OrderInfo.class).getBody();
    }
}

@SpringBootApplication
public class ErrorHandlerWithRestTemplate {

    private final OrderServiceClient orderServiceClient;

    public ErrorHandlerWithRestTemplate(OrderServiceClient orderServiceClient) {
        this.orderServiceClient = orderServiceClient;
    }

    public static void main(String[] args) {
        SpringApplication.run(ErrorHandlerWithRestTemplate.class, args);
    }

    @EventListener
    public void on(ApplicationReadyEvent event) {
        orderServiceClient.getOrderInfo("123");
    }

}
```
