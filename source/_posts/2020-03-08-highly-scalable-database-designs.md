---
title: Highly scalable database designs
date: 2020-03-08T10:08:22.921Z
categories: [Microservices]
tags: [scalable-database-designs, postgres-replication]
cover: /images/highly-scalable-database-designs/featured.png
---

While architecting cloud-native applications, you need to ensure that your system is highly available, performant, scalable, fault-tolerant, and has the capability to recover from a disaster scenario. In this article, Samir Behara discusses the options available when designing the database architecture to achieve scalability.

<!-- more -->

For further detail, reading see an article on [Reg Gate](https://www.red-gate.com/simple-talk/cloud/cloud-data/designing-highly-scalable-database-architectures/)

In this post, I will explain how we can deploy highly scalable PostgresSQL databases using Docker. 
Also, I will talk about how to connect our microservices (based on Spring Boot) with scaled database deployments via api gateway by using practical examples.

## Example Scenario

![](/images/highly-scalable-database-designs/employees-highly-scalable-databases.png)


## Query APIs - [on Github](https://github.com/BhuwanUpadhyay/employees/tree/13-highly-scalable-database-designs/employees-query)
```kotlin
@Bean
public RouterFunction<ServerResponse> employeeRoutes(EmployeeHandler handler) {
	return route(GET("/employees/{id}").and(accept(APPLICATION_JSON)), handler::getEmployee)
	  .andRoute(GET("/employees").and(accept(APPLICATION_JSON)), handler::listEmployee);
}
```

## Command APIs - [on Github](https://github.com/BhuwanUpadhyay/employees/tree/13-highly-scalable-database-designs/employees-command)
```kotlin
@Bean
public RouterFunction<ServerResponse> employeeRoutes(EmployeeHandler handler) {
	return route(DELETE("/employees/{id}").and(accept(APPLICATION_JSON)), handler::deleteEmployee)
	   .andRoute(PUT("/employees/{id}").and(accept(APPLICATION_JSON)), handler::updateEmployee)
	   .andRoute(POST("/employees").and(accept(APPLICATION_JSON)), handler::createEmployee);
}
```

## API Gateway with KrakenD - [on Github](https://github.com/BhuwanUpadhyay/highly-scalable-database-designs/blob/master/docker/krakend/krakend.json)
KrakenD is a stateless, distributed, high-performance API Gateway that helps you 
effortlessly adopt microservices. The configuration file for KrakenD to address our requirements:

```json
{
  "version": 2,
  "endpoints": [
  {
    "endpoint": "/v1/employees",
    "method": "GET",
    "backend": [
    {
      "url_pattern": "/employees",
      "method": "GET",
      "host": [
        "http://query:8080"
      ]
    }
    ]
  },
  {
    "endpoint": "/v1/employees/{id}",
    "method": "GET",
    "backend": [
    {
      "url_pattern": "/employees/{id}",
      "method": "GET",
      "host": [
        "http://query:8080"
      ]
    }
    ]
  },
  {
    "endpoint": "/v1/employees",
    "method": "POST",
    "backend": [
    {
      "url_pattern": "/employees",
      "method": "POST",
      "host": [
        "http://command:8080"
      ]
    }
    ]
  },
  {
    "endpoint": "/v1/employees/{id}",
    "method": "DELETE",
    "backend": [
    {
      "url_pattern": "/employees/{id}",
      "method": "DELETE",
      "host": [
        "http://command:8080"
      ]
    }
    ]
  },
  {
    "endpoint": "/v1/employees/{id}",
    "method": "PUT",
    "backend": [
    {
      "url_pattern": "/employees/{id}",
      "method": "PUT",
      "host": [
        "http://command:8080"
      ]
    }
    ]
  }
  ],
  "extra_config": {
  }
}
```

## Docker Stack - [on Github](https://github.com/BhuwanUpadhyay/highly-scalable-database-designs/blob/master/docker-compose.stack.yml)

You can define number of replicas as you need for high scalable database design. 

```yaml
...
services:
  # -------------------------------------------------
  # Application Backend Databases
  # -------------------------------------------------

  replica:
    image: bitnami/postgresql:12
    ...
    deploy:
      replicas: 3
    ...  
  primary:
    image: bitnami/postgresql:12
    ...  
    deploy:
      replicas: 1
    ...

  # -------------------------------------------------
  # Api Gateway
  # -------------------------------------------------

  gateway:
    image: devopsfaith/krakend:1.1.1-alpine
    ...  
    deploy:
      replicas: 1
    ...

  # -------------------------------------------------
  # Applications
  # -------------------------------------------------

  command:
    image: bhuwanupadhyay/employees-command
    ...  
    deploy:
      replicas: 1
    ...

  query:
    image: bhuwanupadhyay/employees-query
    ...  
    deploy:
      replicas: 1
    ...

  ui:
    image: bhuwanupadhyay/employees-ui
    ...  
    deploy:
      replicas: 1
    ...
```

## Deploy in Docker Swarm

Let's build the docker images that needed on this scenario.
Firstly, clone the [github repo](https://github.com/BhuwanUpadhyay/employees)
and checkout the branch `13-highly-scalable-database-designs` then run following commands:

```
make build &&  docker-compose build --no-cache
```

To enable a docker swarm in your machine:

```
docker swarm init
```

To deploy using [docker-compose.stack.yml](https://github.com/BhuwanUpadhyay/highly-scalable-database-designs/blob/master/docker-compose.stack.yml)

```
docker stack deploy --compose-file docker-compose.stack.yml highly-scalable-db
```

## Testing APIs

For testing, we need a host and port for api gateway. The default port for api gateway is `8080`,
so to identify a host ip address lets run docker network inspect command for `gateway`. 

```
docker network inspect highly-scalable-db_gateway | grep 'IP\"'
```

```
export API_HOST=<IP Address>
export API_HOST_PORT=8080
```

Install `httpie` tool on your machine. [Install Documentation](https://httpie.org/doc#installation)

### Create New Employee
```
http POST $API_HOST:$API_HOST_PORT/v1/employees name="Bhuwan Prasad Upadhyay"
```

### List Employees
```
http $API_HOST:$API_HOST_PORT/v1/employees
```

### Get Employee by Id
```
http $API_HOST:$API_HOST_PORT/v1/employees/<employeeId>
```

### Update an Employee
```
http PUT $API_HOST:$API_HOST_PORT/v1/employees/<employeeId> name="Bandana Poudyal"
```

### Delete an Employee
```
http DELETE $API_HOST:$API_HOST_PORT/v1/employees/<employeeId>
```
