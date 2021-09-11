---
title: Enforce onion architecture in your code
cover: /images/fsqs-tips-tricks-notes.png
date: 2020-08-14T10:08:22.921Z
categories: [ArchUnit]
tags: [architecture-testing]
---

ArchUnit's main focus is to automatically test architecture and coding rules. In this code snippet, I will show how to use ArchUnit to enforce architecture in your code.

<!-- more -->

> To create something exceptional, your mindset must be relentlessly focused on the smallest detail. - Giorgio Armani

```xml
<dependency>
    <groupId>com.tngtech.archunit</groupId>
    <artifactId>archunit-junit5</artifactId>
    <!-- <artifactId>archunit-junit4</artifactId> -->
    <version>0.14.1</version>
    <scope>test</scope>
</dependency>
```
## Bounded Context-Package Artifacts

![](/images/enforce-archx-in-testing.png)

## ArchUnit Test
```java
@AnalyzeClasses(packages = CodingRuleTest.PACKAGE)
class CodingRuleTest {

    public static final String PACKAGE = "io.retailstore.cart";

    @ArchTest
    private final ArchRule classes_are_under_packages = ArchRuleDefinition.classes()
            .should()
            .resideInAnyPackage(PACKAGE, "..application..", "..domain..", "..infrastructure..", "..interfaces..");

    @ArchTest
    private final ArchRule onion_dependencies_are_respected = Architectures
            .onionArchitecture()
            .domainModels("..domain.model..")
            .domainServices("..domain.services..")
            .applicationServices("..application.commandservices..", "..application.queryservices..")
            .adapter("outbound", "..infrastructure..", "..application.outboundservices..")
            .adapter("inbound", "..interfaces..");

}

```
