---
title: How to convert java object to properties format
cover: /images/fsqs-tips-tricks-notes.png
date: 2021-09-14T10:08:22.921Z
categories:
  - FasterXML
tags:
  - java-to-props
  - faster-xml
---

How to convert java object to properties format?

<!-- more -->

# Dependencies

```xml
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-properties</artifactId>
    <version>2.12.5</version>
</dependency>
```

## Java Object To Properties Map

```java

public class PropsUtils {

    /**
     * 
     * Convert java object to properties: all fields, getter methods and is methods into properties map.
     * 
     * @param object
     * @param <T>
     * @return
     */
    public <T> Map<String, String> toProperties(T object) {
        JavaPropsMapper mapper = JavaPropsMapper.builder().build();
        JavaPropsSchema javaPropsSchema = JavaPropsSchema.emptySchema().withWriteIndexUsingMarkers(true);
        return mapper.writeValueAsMap(entity, javaPropsSchema);
    }


    /**
     *
     * Convert java object to properties: all fields only into properties map.
     *
     * @param object
     * @param <T>
     * @return
     */
    public <T> Map<String, String> toPropertiesOnlyFields(T object) {
        JavaPropsMapper mapper = JavaPropsMapper.builder()
                .visibility(PropertyAccessor.FIELD, JsonAutoDetect.Visibility.ANY)
                .visibility(PropertyAccessor.GETTER, JsonAutoDetect.Visibility.NONE)
                .visibility(PropertyAccessor.IS_GETTER, JsonAutoDetect.Visibility.NONE).build();
        JavaPropsSchema javaPropsSchema = JavaPropsSchema.emptySchema().withWriteIndexUsingMarkers(true);
        return mapper.writeValueAsMap(entity, javaPropsSchema);
    }
    
}
```
