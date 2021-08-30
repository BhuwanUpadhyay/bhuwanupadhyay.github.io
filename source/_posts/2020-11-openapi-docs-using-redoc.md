---
title: OpenAPI Docs Using ReDoc
date: 2020-11-25 01:31:55
cover: /gallery/media/openapi-docs-using-redoc.png
categories:
- OpenAPI
- ReDoc
tags:
- openapi 
- redoc
---

How to use ReDoc for OpenAPI? In this blog post, I'm going to show you the practical way to use ReDoc for OpenAPI Documentation.

<!-- more -->

In recent times, **Rest APIs** are de-facto standards for client-server and service-to-service communication.

As a business grows, the number of APIs also increases and, after some time it becomes very hard to manage API without using any comprehensive tool and standard. For this reason, the idea of OpenAPI came into the market and, which is a broadly adopted industry standard for describing modern APIs.

The process of defining API is very simple in OpenAPI that just required a single file in `json` or `yaml` format, but once the number of API increases then real problems starting to appear in the development and maintenance of this single file.

> A clever person solves a problem.
> A wise person AVOIDS it. <br/>
> — Albert Einstein

[ReDoc](https://github.com/Redocly/redoc) allows us to document API
into [multiple file definitions](https://redoc.ly/docs/resources/multi-file-definitions/) in standard structure and also provide CLI tool [@redocly/openapi-cli](https://github.com/Redocly/openapi-cli) that can use for validating definition, to generate docs and export OpenAPI definition.

## Demo > How to do?

In this demo, we will see how to use OpenAPI and ReDoc together for documentation.

### 1. Installation

First of all, you must have [nodejs](https://nodejs.org/) installed on your machine. Then install [create-openapi-repo](https://github.com/Redocly/create-openapi-repo) globally:

```bash
npm install -g create-openapi-repo
```

### 2. Project Setup

Run the following command in your preferred folder to create OpenAPI multiple files and folder structure.

```bash
create-openapi-repo
```

_**Example:**_

![](/gallery/media/openapi-docs-using-redoc-1.png)

### 3. Defining APIs

For example, we have to create services for user management and product management then possible APIs are the following:

```
|_**User Service**_                            | _**Product Service**_                              |
|----------------------------------------------|----------------------------------------------------|
| - POST `/users` to create a new user         | - POST `/products` to create a new product         |
| - GET `/users` to get all users              | - GET `/products` to get all products              |
| - GET `/users/{username}` to get user        | - GET `/products/{productId}` to get product       |
| - PUT `/users/{username}` to edit user       | - PUT `/products/{productId}` to edit product      |
| - DELETE `/users/{username}` to delete user  | - DELETE `/products/{productId}` to delete product |
```

API definitions using multiple files and folders structure with ReDoc:

```
openapi/
┣ code_samples/
┃ ┣ C#/
┃ ┃ ┗ echo/
┃ ┃   ┗ post.cs
┃ ┣ PHP/
┃ ┃ ┗ echo/
┃ ┃   ┗ post.php
┃ ┗ README.md
┣ components/
┃ ┣ headers/
┃ ┃ ┗ PageInfo.yaml
┃ ┣ schemas/
┃ ┃ ┣ Email.yaml
┃ ┃ ┣ Product.yaml
┃ ┃ ┗ User.yaml
┃ ┣ securitySchemes/
┃ ┃ ┣ api_key.yaml
┃ ┃ ┣ basic_auth.yaml
┃ ┃ ┗ main_auth.yaml
┃ ┗ README.md
┣ paths/
┃ ┣ products.yaml
┃ ┣ products@{productId}.yaml
┃ ┣ users.yaml
┃ ┗ users@{username}.yaml
┣ README.md
┗ openapi.yaml
```
In the main [openapi yaml](https://raw.githubusercontent.com/bhuwanupadhyay/codes/main/openapi-docs-using-redoc/openapi.yaml) file we just need to link our separate definitions with API paths.

```yaml
paths:
  '/users':
    $ref: 'paths/users.yaml'
  '/users/{username}':
    $ref: 'paths/users@{username}.yaml'
  '/products':
    $ref: 'paths/products.yaml'
  '/products/{productId}':
    $ref: 'paths/products@{productId}.yaml'
```

> _**Why multiple files and folders?**_
>
> If you don't use multiple files and folders structure to manage definitions then
> you will end with the big file like [dist.yaml](https://raw.githubusercontent.com/bhuwanupadhyay/codes/main/openapi-docs-using-redoc/dist.yaml).
> After looking at it, you might think it is not bad but this is a demo which has only 8 APIs
> which may not be in a real-world application. You might have 50 or 100 APIs then you will face the real problem of managing single file
> so that I recommend multiple files and folder structure to manage API definition.
> - Easy to organize.
> - Easy for development.
> - Easy to re-use of objects to avoid duplication.

### 4. Lint, Build and Generate Docs

- To Lint

```bash
npm run test
```

- To Build

```bash
npm run build
```

- To Generate Docs

```bash
npm run start
```

### 5. Source Code > [Github Link](https://github.com/bhuwanupadhyay/codes/tree/main/openapi-docs-using-redoc)

## Conclusion

In summary, managing a single OpenAPI definition file is a daunting task instead by using ReDoc we can organize API development in a better way, and also maintenance becomes very easy.

## Reference
- [https://github.com/Redocly/redoc](https://github.com/Redocly/redoc)
- [https://github.com/Redocly/openapi-cli](https://github.com/Redocly/openapi-cli)
- [https://github.com/Redocly/create-openapi-repo](https://github.com/Redocly/create-openapi-repo)
