---
layout: post
title: Json Utils
date: 2021-10-11 10:00:20 +0700
description: Thao tác với string json
img: java/json_utils.png
tags: [Java, Json]
categories: [Java]
source: https://github.com/huypva/json-utils-example
---

> Hướng dẫn thao tác Json thông qua Gson hoặc Jackson 

## Interface

- Một số thao tác Json cơ bản
    - String toJson(Object obj): chuyển đổi một Java object sang chuỗi String
    - T fromJson(String json, Class<T> clazz): convert String thành một Java object
    - T fromJson(String json, TypeToken<T> type): convert String thành một collection
    - Thay đổi param name bằng annotation
    
## Gson

- Thêm dependency `gson` trong pom.xml

```xml
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
        <version>2.8.6</version>
    </dependency>
```

- Tạo và sử dụng thông qua class `Gson`

```java
public class GsonUtils {
  private static final Gson GSON = new GsonBuilder().create();

  private GsonUtils() {}

  public static <T> T fromJson(String json, Class<T> clazz) {
    return GSON.fromJson(json, clazz);
  }

  public static <T> T fromJson(String json, TypeToken<T> type) {
    return GSON.fromJson(json, type.getType());
  }

  public static String toJson(Object obj) {
    return GSON.toJson(obj);
  }
}
```

- Sử dụng annotation *SerializedName* lên object cần thao tác để thay đổi param name

```java
public class GsonAnnotationObject {

  @SerializedName("object_id")
  public int objectId;

  @SerializedName("object_name")
  public String objectName;

}
```

Sử dụng 

```java
GsonAnnotationObject annotationObject = new GsonAnnotationObject(1, "A");
System.out.println(GsonUtils.toJson(annotationObject));
```

Output:

```text
{"object_id":1,"object_name":"A"}
```

## Jackson

- Thêm dependency `jackson-databind` trong pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.12.5</version>
    </dependency>
</dependencies>
```

- Tạo và sử dụng thông qua class `ObjectMapper`

```java
public class JacksonUtils {

  private static final ObjectMapper OBJECT_MAPPER = new ObjectMapper();

  public static <T> T fromJson(String json, Class<T> clazz) throws JacksonParserException {
    try {
      return OBJECT_MAPPER.readValue(json, clazz);
    } catch (JsonProcessingException e) {
      throw new JacksonParserException(e);
    }
  }

  public static <T> T fromJson(String json, TypeReference<T> type) throws JacksonParserException {
    try {
      return OBJECT_MAPPER.readValue(json, type);
    } catch (JsonProcessingException e) {
      throw new JacksonParserException(e);
    }
  }

  public static String toJson(Object obj) throws JacksonParserException {
    try {
      return OBJECT_MAPPER.writeValueAsString(obj);
    } catch (JsonProcessingException e) {
      throw new JacksonParserException(e);
    }
  }

}
```

- Sử dụng annotation *JsonAlias* lên object cần thao tác để thay đổi param name

```java
@AllArgsConstructor
@NoArgsConstructor
public class JacksonAnnotationObject {

  @JsonAlias("object_id")
  public int objectId;

  @JsonAlias("object_name")
  public String objectName;

}
```

Sử dụng 

```java
GsonAnnotationObject annotationObject = new GsonAnnotationObject(1, "A");
System.out.println(GsonUtils.toJson(annotationObject));
```

Output:

```text
{"object_id":1,"object_name":"A"}
```