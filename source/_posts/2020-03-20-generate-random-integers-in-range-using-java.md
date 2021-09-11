---
title: Generate random integers in range using Java
date: 2020-03-20 12:14:00 Z
categories: [Java]
tags: [random]
cover: /images/fsqs-tips-tricks-notes.png
---

Sometimes, one might need to assign a random value to a variable. Random numbers within a specific range of type integer, float, double, long, boolean can be generated in Java.

<!-- more -->

There are three methods to generate random numbers in Java.

## 1. By using - [Math.random](https://docs.oracle.com/javase/8/docs/api/java/lang/Math.html#random--)

This `Math.random()` gives a random double from `0.0` (inclusive) to `1.0` (exclusive).

For generating random numbers within a range using `Math.random()`, see the example below:

```java
public class Randomizer {
    
    /**
    * @param min - minimum range value
    * @param max - maximum range value
    * @return random integer between min (inclusive) to max (inclusive) with uniform probability
    */    
    public static int next(int min, int max) {
        if (min >= max) {
        	throw new IllegalArgumentException("max must be greater than min");
        }

        return (int)(Math.random() * ((max - min) + 1)) + min;
    }

}
```

## 2. By using - [java.util.Random](https://docs.oracle.com/javase/8/docs/api/java/util/Random.html)

For generating random numbers within a range using `java.util.Random`, see the example below:

```java
public class Randomizer {
    
    /**
    * @param min - minimum range value
    * @param max - maximum range value
    * @return random integer between min (inclusive) to max (inclusive) with uniform probability
    */    
    public static int next(int min, int max) {
        if (min >= max) {
        	throw new IllegalArgumentException("max must be greater than min");
        }

        Random r = new Random();
        return r.nextInt((max - min) + 1) + min;
    }

}
```

The `Random.nextInt(n)` is more efficient than `Math.random() * n`, read this [post](https://community.oracle.com/message/6596485#thread-message-6596485).

In addition, `Math.random()` is thread safe by itself but if you want to generate numbers using `Random`
class then `ThreadLocalRandom` is more preferable which thread safe.

## 3. [Java 8] By using - [Random.ints](https://docs.oracle.com/javase/8/docs/api/java/util/Random.html#ints-int-int-)

For generating random numbers within a range using `Random.ints`, see the example below:

```java
public class Randomizer {
    
    /**
    * @param min - minimum range value
    * @param max - maximum range value
    * @return random integer between min (inclusive) to max (inclusive) with uniform probability
    */    
    public static int next(int min, int max) {
        if (min >= max) {
        	throw new IllegalArgumentException("max must be greater than min");
        }
        
        Random r = new Random();
		return r.ints(min, (max + 1)).limit(1).findFirst().getAsInt();
    }

}
```

Note: To generates random integers in a range between 15 (inclusive) and 20 (exclusive), with stream size of 3.

```java
new Random().ints(3, 15, 20).forEach(System.out::println);
```
