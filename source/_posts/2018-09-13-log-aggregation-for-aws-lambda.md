---
title: Log aggregation for AWS Lambda
date: 2018-09-13T10:08:22.921Z
categories: [Serverless]
tags: [logging, awslambda]
cover: /images/fsqs-tips-tricks-notes.png
---

ELK is the acronym for three open source projects: Elasticsearch, Logstash, and Kibana. Elasticsearch is a search and analytics engine. Logstash is a server‑side data processing pipeline that ingests data from multiple sources simultaneously, transforms it, and then sends it to a "stash" like Elasticsearch. Kibana lets users visualize data with charts and graphs in Elasticsearch.
<!-- more -->

This article demonstrates, how we can use [ELK Stack](https://www.elastic.co/what-is/elk-stack) with lambda function for log aggregation.

> I get paid for produce code that works, not for writing tests. - Kent Back

## Example Scenario

In the post, I explained an approach of using a Lambda function and Kinesis Stream to ship all your lambda logs from
CloudWatch logs to a log aggregation service such as **ELK Stack**-(Elasticsearch, Logstash, Kibana).

![](/images/log-aggregation-for-aws-lambda/featured.png)

## Create Handler Functions
Let's consider we have to expose APIs to manage an order with ELK:
 - Put order into the db (DynamoDB)
 - Retrieve orders from the db (DynamoDB).
 - Send CloudWatch logs from Kinesis Stream to ELK


To achieve these two operations in serverless, we need three lambda functions:
 - POST : Store order into the db  - [on Github](https://github.com/BhuwanUpadhyay/log-aggregation-for-aws-lambda/tree/master/functions/create-order)
 - GET  : Fetch orders to the client - [on Github](https://github.com/BhuwanUpadhyay/log-aggregation-for-aws-lambda/tree/master/functions/get-orders)
 - KinesisLogsStream  : Ship logs to ELK - [on Github](https://github.com/BhuwanUpadhyay/log-aggregation-for-aws-lambda/tree/master/functions/ship-logs-to-elk)


## Serverless Framework
Severless framework simply deployment process for lambda function. 
To use it we need to create `serverless.yml` file in our project and configure our lambda function , DynamoDB, Kinesis Stream as follows:

Note: `logstash_host`, `logstash_port` and `token` configure with your ELK environment.
  
```yaml
service:
  name: order-apis

plugins:
- serverless-pseudo-parameters
- serverless-iam-roles-per-function

custom:
  region: ${opt:region, self:provider.region}
  stage: ${opt:stage}
  prefix: ${self:service}-${self:custom.stage}
  dynamodb_table: ${self:custom.prefix}-order
  dynamodb_arn: arn:aws:dynamodb:${self:custom.region}:*:table/${self:custom.dynamodb_table}

provider:
  name: aws
  runtime: nodejs8.10
  region: ${opt:region, 'us-east-1'}
  timeout: 30
  iamRoleStatements:
  - Effect: Allow
    Action:
    - dynamodb:Query
    - dynamodb:Scan
    - dynamodb:GetItem
    - dynamodb:PutItem
    - dynamodb:UpdateItem
    - dynamodb:DeleteItem
    - dynamodb:DescribeTable
    Resource: ${self:custom.dynamodb_arn}
  environment:
    DYNAMO_TABLE: ${self:custom.dynamodb_table}

package:
  exclude:
  - .idea/**
  - .git/**
  - tmp/**

functions:

  create-order:
    handler: functions/create-order/handler.handle
    description: Create new order
    events:
    - http:
        path: orders
        method: post

  get-orders:
    handler: functions/get-orders/handler.handle
    description: Get orders
    events:
    - http:
        path: orders
        method: get

  ship-logs-to-elk:
    handler: functions/ship-logs-to-elk/handler.handle
    description: Sends CloudWatch logs from Kinesis to ELK
    events:
    - stream:
        type: kinesis
        arn:
          Fn::GetAtt:
          - KinesisLogsStream
          - Arn
    environment:
      logstash_host: listener.logz.io #<INSERT VALUE HERE>
      logstash_port: 5050 #<INSERT VALUE HERE>
      token: <INSERT VALUE HERE>

resources:
  Resources:
    Order:
      Type: AWS::DynamoDB::Table
      Properties:
        TableName: ${self:custom.dynamodb_table}
        AttributeDefinitions:
        - AttributeName: orderId
          AttributeType: S
        KeySchema:
        - AttributeName: orderId
          KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 1
          WriteCapacityUnits: 1
    KinesisLogsStream:
      Type: AWS::Kinesis::Stream
      Properties:
        Name: ${self:custom.prefix}-logs
        ShardCount: 1
    CWLtoKinesisRole:
      Type: AWS::IAM::Role
      Properties:
        RoleName: ${self:custom.prefix}-CWLtoKinesisRole
        AssumeRolePolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Effect: Allow
            Principal:
              Service: logs.${self:custom.region}.amazonaws.com
            Action: sts:AssumeRole
    PermissionsPolicyForCWL:
      Type: AWS::IAM::Policy
      DependsOn:
      - CWLtoKinesisRole
      - KinesisLogsStream
      Properties:
        PolicyName: ${self:custom.prefix}-PermissionsPolicyForCWL
        Roles:
        - Ref: CWLtoKinesisRole
        PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Effect: Allow
            Action: kinesis:PutRecord
            Resource:
              Fn::GetAtt:
              - KinesisLogsStream
              - Arn
          - Effect: Allow
            Action: iam:PassRole
            Resource:
              Fn::GetAtt:
              - CWLtoKinesisRole
              - Arn
    CWLtoKinesisSubscription:
      Type: AWS::Logs::SubscriptionFilter
      DependsOn:
      - CWLtoKinesisRole
      - PermissionsPolicyForCWL
      - KinesisLogsStream
      Properties:
        DestinationArn:
          Fn::GetAtt:
          - KinesisLogsStream
          - Arn
        RoleArn:
          Fn::GetAtt:
          - CWLtoKinesisRole
          - Arn
        FilterPattern: ""
        LogGroupName: /aws/lambda/${self:custom.prefix}-create-order
```

## `make deploy`

When you run the command `make deploy` then it will deploy the lambda function to AWS and perform test on lambda function
and cloud watch logs will be shipped to the ELK.
