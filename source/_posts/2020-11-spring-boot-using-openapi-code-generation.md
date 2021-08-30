---
title: Spring Boot App Using Java Code Generated From OpenAPI
date: 2020-11-19 01:31:55
cover: /gallery/media/spring-boot-using-openapi-code-generation.png
categories:
- OpenAPI
- Spring Boot
tags:
- spring-boot
- openapi
---

How to use OpenAPI to generate code for Spring Boot? You couldn't find a better place then. I'm going to show you the practical way to generate code and use in spring boot.

<!-- more -->

Over the last three years, my team, and I have been using OpenAPI for API definitions across the microservices. The main benefits of OpenAPI specification: __provides a single source for API contracts__ and __easy to render docs by using the same specification__. Our team not only benefited from these principle advantages but additionally, we used code generation tools from OpenAPI to generate code for the server application (Spring Boot) as well as for the client application (Angular).

## Demo > How to do?

In this demo, we will see how to build the Spring Boot App using OpenAPI and code generation together.

### 1. Installation

First of all, you must have [nodejs](https://nodejs.org/) and [jdk](https://sdkman.io/jdks) installed on your machine.
Then install [@openapitools/openapi-generator-cli](https://www.npmjs.com/package/@openapitools/openapi-generator-cli) globally:

```bash
# install the latest version of "openapi-generator-cli"
npm install @openapitools/openapi-generator-cli -g
# use a specific version of "openapi-generator-cli"
openapi-generator-cli version-manager set 4.3.1
```

### 2. Project Setup

Before writing the code we need to agree on our service API. In this demo, I used the definition that I published in my previous [article](https://bhuwanupadhyay.github.io/2020/11/openapi-docs-using-redoc/) which specification located in [github.com/bhuwanupadhyay/codes](https://raw.githubusercontent.com/bhuwanupadhyay/codes/main/openapi-docs-using-redoc/dist.yaml).

Open the terminal where you want to generate code and run the following command to save the specification.

```bash
wget \
 https://raw.githubusercontent.com/bhuwanupadhyay/codes/main/openapi-docs-using-redoc/dist.yaml \
 -O openapi.yaml
```

I used the following configuration of code generation where you can modify according to your need. The use of each property is described on [openapi-generator](https://openapi-generator.tech/docs/generators/spring) site.

```yaml
generatorName: 'spring'
groupId: 'io.github.bhuwanupadhyay'
artifactId: 'spring-boot-using-openapi-code-generation'
apiPackage: 'io.github.bhuwanupadhyay.demo.interfaces.rest'
modelPackage: 'io.github.bhuwanupadhyay.demo.interfaces.rest.dto'
artifactVersion: '1.0.0'
library: 'spring-boot'
inputSpec: 'openapi.yaml'
outputDir: 'demo'
additionalProperties:
  java8: 'true'
  dateLibrary: 'java8'
  serializableModel: 'true'
  serializationLibrary: 'jackson'
  interfaceOnly: 'true'
  skipDefaultInterface: 'true'
  prependFormOrBodyParameters: 'true'
  useTags: 'true'
  bigDecimalAsString: 'true'
  booleanGetterPrefix: 'is'
```

To generate code:

```bash
openapi-generator-cli batch config.yaml
```

The generated minimum structure for java code looks like as below:

```
rest/
┣ dto/
┃ ┣ Product.java
┃ ┗ User.java
┣ ApiUtil.java
┣ ProductApi.java
┗ UserApi.java
```

### 3. API Implementation

The generated code has skeleton API interfaces on which implementation needs to provide from us.

For example, the implementation for Product API Endpoints:

```java
@RestController
class WebProductApi implements ProductApi {
 
   private final ProductRepository repository;
 
   WebProductApi(ProductRepository repository) {
       this.repository = repository;
   }
 
   @Override
   public ResponseEntity<Void> createProduct(Product product) {
       repository.save(toEntity(product));
       return ResponseEntity.ok().build();
   }
 
   @Override
   public ResponseEntity<Void> deleteProduct(String productId) {
       return applyVoid(repository, productId, e -> repository.deleteById(e.getProductId()));
   }
 
   @Override
   public ResponseEntity<Product> getProductById(String productId) {
       return applyResult(repository, productId, this::toResource);
   }
 
   @Override
   public ResponseEntity<List<Product>> getProducts(Integer page, Integer size, String sort) {
       PageRequest request = PageRequest.of(page, size, Sort.by(sort));
       Page<ProductEntity> all = repository.findAll(request);
       List<Product> list = all.stream().map(this::toResource).collect(Collectors.toList());
       return ResponseEntity.status(HttpStatus.OK).headers(pageHeaders(all)).body(list);
   }
 
   @Override
   public ResponseEntity<Void> updateProduct(Product product, String productId) {
       return applyVoid(repository, productId, e -> {
           e.with(product.getProductName());
           repository.save(e);
       });
   }
 
   private Product toResource(ProductEntity entity) {
       return new Product().productId(entity.getProductId()).productName(entity.getProductName());
   }
 
   private ProductEntity toEntity(Product product) {
       ProductEntity entity = new ProductEntity(product.getProductId());
       entity.with(product.getProductName());
       return entity;
   }
 
}
```

### 4. Advantages

There are the following advantages of generated code from the specification.

- Consistent API contracts.
- Fail-fast & easy to maintain.
- Better control of the API interface code.
- Remove overhead of maintaining POJOs.

### 5. Source Code > [Github Link](https://github.com/bhuwanupadhyay/codes/tree/main/spring-boot-using-openapi-code-generation)

## Conclusion

In summary, the code generation from OpenAPI specification is a very good option to build error-free server code within a short period. I recommend you also give a try and see the results.

## Reference
- [https://openapi-generator.tech/](https://openapi-generator.tech/)
- [https://openapi-generator.tech/docs/generators/spring](https://openapi-generator.tech/docs/generators/spring)
