---
title: AWS Lambda load testing with large scale data using Gatling
date: 2018-09-12T10:08:22.921Z
categories: [Serverless]
tags: [load-testing, gatling]
cover: /images/aws-lambda-load-testing-with-large-scale-data-using-gatling/featured.png
---

Load testing is a type of non-functional testing. A load test is type of software testing which is conducted to understand the behavior of the application under a specific expected load. Load testing is performed to determine a system's behavior under both normal and at peak conditions.
<!-- more -->

> No amount of testing can prove a software right, a single test can prove a software wrong. - Amir Ghahrai

## Example Scenario

Let's say, We have to perform **Two Million** HTTP requests load in AWS Lambda function via API Gateway.
Obviously, to perform 2M requests we need to run load testing in cluster mode.
[Gatling Frontline](https://gatling.io/gatling-frontline/) support cluster load testing directly. With Gatling Community Version we need to do some extra work to achieve cluster mode load testing.

## Code Example
This article is accompanied by working example code [on GitHub](https://github.com/bhuwanupadhyay/aws-lambda-load-testing-with-large-scale-data-using-gatling)

## Create Handler Functions - [on Github](https://github.com/bhuwanupadhyay/aws-lambda-load-testing-with-large-scale-data-using-gatling/blob/master/src/createOrder.js)
Let’s consider we have to expose APIs to manage an order:

- Put order into the db (DynamoDB)

To achieve these two operations in serverless, we need two lambda functions with http event type:

- POST : Store order into the db

## Load Testing - [on Github](https://github.com/BhuwanUpadhyay/aws-lambda-load-testing-with-large-scale-data-using-gatling/blob/master/load-testing/src/test/scala/testing/LoadTestingSimulation.scala)

I used [gatling-aws-maven-plugin](https://github.com/electronicarts/gatling-aws-maven-plugin) to run 20 EC2 instances together, which runs load testing parallel.
For two millions http requests by using 20 instances together, in gatling our plan will be like below:
```bash
REPETITION -> 2000  
NO_OF_USERS -> 50
-----------------------------------------------
TOTAL REQUEST = 2000 * 50 = 100000
WITH 20 INSTANCES = 100000 * 20 = 2000000 = 2M
```

Now our load testing simulation will looks like:

```scala
import java.util.UUID.randomUUID
import io.gatling.core.Predef._
import io.gatling.core.structure.ScenarioBuilder
import io.gatling.http.Predef._

class LoadTestingSimulation extends Simulation {

  private val ENDPOINT = "https://xxxxx.execute-api.ap-southeast-2.amazonaws.com/test"
  private val BODY: String = "{\"customerId\" : \"@@customerId@@\" }"

  private val scn: ScenarioBuilder = scenario("Load Testing")
    .repeat(2000) {
      exec(
        http("initiation")
          .post(ENDPOINT + "/orders")
          .check(status.is(200))
          .body(StringBody(session => BODY.replaceAll("@@customerId@@", newId)))
          .header("Content-type", "text/xml")
      )
    }

  setUp(scn.inject(atOnceUsers(50))
    .protocols(http
      .baseURL(ENDPOINT)
      .acceptHeader("text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8")
      .acceptEncodingHeader("gzip, deflate")
      .acceptLanguageHeader("en-US,en;q=0.5")
      .userAgentHeader("Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:16.0) Gecko/20100101 Firefox/16.0")))

  private def newId = {
    randomUUID().toString.substring(0, 33)
  }

}
```

## `make deploy`

When you run the command `make deploy` then it will deploy the lambda function to AWS.

## `make test`
To run the command `make test` provide correct AWS `accessKey` and `secretKey` in [aws.properties](https://github.com/BhuwanUpadhyay/aws-lambda-load-testing-with-large-scale-data-using-gatling/blob/master/load-testing/aws.properties).
Also, change values in [runLoadtestRemotely.sh](https://github.com/BhuwanUpadhyay/aws-lambda-load-testing-with-large-scale-data-using-gatling/blob/master/load-testing/runLoadtestRemotely.sh) 
according to you configuration.

```
S3_BUCKET=load-testing-s3-bucket
EC2_KEY_PAIR=loadtest-keypair
NO_OF_INSTANCES=20
EC2_ENDPOINT=https://ec2.us-west-1.amazonaws.com
EC2_SECURITY_GROUP_ID=sg-xxxxxxx
AMI_ID=ami-0782017a917e973e7
SSH_USER=ec2-user
```

Then you can run `make test` and analyze the result.
