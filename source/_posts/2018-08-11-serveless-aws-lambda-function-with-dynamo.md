---
title: Serverless AWS Lambda function with dynamo db
date: 2018-08-11T10:08:22.921Z
categories: [Serverless]
tags: [awslambda, dynamodb]
cover: /images/serverless-aws-lambda-function-with-dynamo-db/featured.png
---

This article demonstrates, how we can use [serverless framework](https://serverless.com/) to deploy lambda function with dynamo DB.
<!-- more -->

We will create project from scratch and deploy to aws using serverless framework.

## Code Example
This article is accompanied by working example code [on GitHub](https://github.com/bhuwanupadhyay/serverless-aws-lambda-function-with-dynamodb).

## Create Handler Functions
Let's consider we have to expose APIs to manage an order:
 - Put order into the db (DynamoDB)
 - Retrieve orders from the db (DynamoDB).
 
To achieve these two operations in serverless, we need two lambda functions with http event type:
 - POST : Store order into the db 
 - GET  : Fetch orders to the client
  
## AWS Lambda Proxy Integration I/O format

Before write a code, let's understand input and output format supported by 
aws lambda proxy integration in API gateway. You can check on following links: 
 - [Input Format](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-input-format)
 - [Output Format](https://docs.aws.amazon.com/apigateway/latest/developerguide/set-up-lambda-proxy-integrations.html#api-gateway-simple-proxy-for-lambda-output-format)

According to AWS documentation, java class for input format is:
```java
@Data
public class ApiGatewayRequest {

    private String resource;
    private String path;
    private String httpMethod;
    private Map<String, String> headers;
    private Map<String, String> queryStringParameters;
    private Map<String, String> pathParameters;
    private Map<String, String> stageVariables;
    private Map<String, Object> requestContext;
    private String body;
    private boolean isBase64Encoded;

    @SneakyThrows
    <T> T toBody(Class<T> valueType) {
        return MAPPER.readValue(body, valueType);
    }
}
```
and java class for out format with its builder class to simply construction object simple:
```java
@Getter
public class ApiGatewayResponse {

    public static final ObjectMapper MAPPER = new ObjectMapper();

    private final int statusCode;
    private final String body;
    private final Map<String, String> headers = new HashMap<>();
    private final boolean isBase64Encoded;

    private ApiGatewayResponse(int statusCode, String body) {
        this.statusCode = statusCode;
        this.body = body;
        this.setHeaders();
        this.isBase64Encoded = false;
    }

    public static ApiGatewayResponse bad(String message) {
        return build(HttpStatus.SC_BAD_REQUEST, message);
    }

    public static <T> ApiGatewayResponse ok(T body) {
        return build(HttpStatus.SC_OK, body);
    }

    static ApiGatewayResponse serverError(String message) {
        return build(HttpStatus.SC_INTERNAL_SERVER_ERROR, message);
    }

    @SneakyThrows
    private static <T> ApiGatewayResponse build(int statusCode, T body) {
        return new ApiGatewayResponse(statusCode, MAPPER.writeValueAsString(body));
    }

    private void setHeaders() {
        headers.put("X-Powered-By", "BhuwanUpadhyay");
        headers.put("Content-Type", "application/json");
        headers.put("Access-Control-Allow-Origin", "*");
    }
}
```

### AWS Dynamo DB Persistence
To persist customer orders into the dynamo db we need to define data object which match with 
our table structure in dynamo.
```java
@Data
@DynamoDBTable(tableName = "Order")
public class Order {

    @DynamoDBHashKey
    private String orderId;
    private String description;
    private String customer;
}
```
 
 Now, we need a repository to interact with dynamo db.
```java
public class OrderRepository {

    private final DynamoDBMapper mapper;

    public OrderRepository() {
        mapper = new DynamoDBMapper(AmazonDynamoDBClientBuilder.standard().build());
    }

    public List<Order> getOrders() {
        PaginatedScanList<Order> orders = mapper.scan(Order.class, new DynamoDBScanExpression());
        return new ArrayList<>(orders);
    }

    public Order save(Order order) {
        mapper.save(order);
        return mapper.load(order);
    }

    public Optional<Order> findByOrderId(String orderId) {
        return Optional.ofNullable(mapper.load(Order.class, orderId));
    }
}
```
  
### AWS Serverless Lambda Function  
Now, before create lambda function i will create one abstract class which encapsulate common
behaviour of all lambda function which are going to create on this demo.

```java
public abstract class HttpEventHandler<T> implements RequestHandler<ApiGatewayRequest, ApiGatewayResponse> {

    @Override
    public ApiGatewayResponse handleRequest(ApiGatewayRequest input, Context context) {
        try {
            return this.handle(toRequestIfNoVoidType(input), context.getLogger());
        } catch (Exception e) {
            return ApiGatewayResponse.serverError(ExceptionUtils.getStackTrace(e));
        }
    }

    protected abstract ApiGatewayResponse handle(T request, LambdaLogger log);

    private T toRequestIfNoVoidType(ApiGatewayRequest input) {
        final Class<T> bodyType = getBodyType();
        return !bodyType.equals(Void.class) ? input.toBody(bodyType) : null;
    }

    @SuppressWarnings("unchecked")
    private Class<T> getBodyType() {
        try {
            String typeName = ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments()[0].getTypeName();
            return (Class<T>) Class.forName(typeName);
        } catch (Exception e) {
            throw new RuntimeException("Class is not parametrized with generic type!!! Please use extends <> ", e);
        }
    }

}
```

Now, time come to create our serverless lambda function. 
 - `OrderProcessHandler`

```java
public class OrderProcessHandler extends HttpEventHandler<Order> {

    private final OrderRepository repository = new OrderRepository();

    @Override
    protected ApiGatewayResponse handle(Order request, LambdaLogger log) {
        Optional<Order> order = repository.findByOrderId(request.getOrderId());
        return order.
                map(o -> bad(String.format("Order already exist with id %s", o.getOrderId()))).
                orElse(ok(repository.save(request)));
    }

} 
```

 - `FetchOrderHandler`
```java
public class FetchOrderHandler extends HttpEventHandler<Void> {

    private final OrderRepository repository = new OrderRepository();

    @Override
    protected ApiGatewayResponse handle(Void request, LambdaLogger log) {
        log.log("Fetching orders....");
        return ApiGatewayResponse.ok(repository.getOrders());
    }
}
```

We have done with coding so far.

Next step is launch our function into aws cloud platform and evaluate the result.

## Serverless Framework
Severless framework simply deployment process for lambda function. To use it we need to create `serverless.yml`
file in our project and configure our lambda function and DynamoDB tables as follows:

```yaml
service: order-service

provider:
  name: aws
  runtime: java8
  stage: ${opt:stage, 'dev'}
  iamRoleStatements:
  - Effect: "Allow"
    Resource: "*"
    Action:
    - "dynamodb:*"

package:
  artifact: target/app-jar-with-dependencies.jar

functions:
  order-process:
    handler: bhuwanupadhyay.serverless.order.OrderProcessHandler
    events:
    - http:
        path: orders
        method: post
        cors: true

  fetch-order:
    handler: bhuwanupadhyay.serverless.order.FetchOrderHandler
    events:
    - http:
        path: orders
        method: get
        cors: true

resources:
  Resources:
    Order:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: Order
        AttributeDefinitions:
        - AttributeName: orderId
          AttributeType: S
        KeySchema:
        - AttributeName: orderId
          KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 1
          WriteCapacityUnits: 1
```

## `make build`
When you run the command `make run` then it will create uber jar with its all needed dependencies at runtime.

For this you have configured `maven-assembly-plugin` as below:
```xml
<plugin>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <!-- get all project dependencies -->
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
    </configuration>
    <executions>
        <execution>
            <id>make-assembly</id>
            <!-- bind to the packaging phase -->
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## `make deploy`
When you run the command `make deploy` then it will deploy the lambda function to AWS and perform tests to verify those deployed functions.
