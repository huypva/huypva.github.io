---
layout: post
title: Spring Boot + JUnit 5
date: 2021-10-21 10:00:20 +0700
description: Hướng dẫn thực hiện Unit Test trong SpringBoot application
img: unit_test/junit_5.png
tags: [Spring Boot, Unit Test, JUnit 5]
categories: [Unit Test]
source: https://github.com/huypva/unit-test-junit-5-example
---

> Hướng dẫn thực hiện UnitTest trong SpringBoot application

## Dependency

- Thêm dependency trong pom.xml để thực hiện UnitTest

```html
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Trong Spring Boot version 2.2.6 đã có sẵn thư viện cho junit 5

<div align="center">
    <img src="/assets/img/junit_5_libs.png"/>
</div>

Nếu không bạn có thể add thêm dependency như sau

```html
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>5.5.2</version>
        <scope>test</scope>
    </dependency>
```

## Tạo class Test

- Mỗi class cần thực hiện Unit Test với class test tên có post fix là "Test"

- Thêm `@ExtendWith(SpringExtension.class)` (Thay cho `@RunWith(SpringRunner.class)` của Junit4) trên đầu mỗi class test

- Lấy bean của class cần test bằng cách
    - Thêm `@Import([Class_cần_test].class)` trên đầu class
    - Sử dụng `@Autowired` để lấy bean class cần test 
    
Ví dụ:

```java
@ExtendWith(SpringExtension.class)
@Import(RandomIdProvider.class)
class RandomIdProviderTest {

  @Autowired
  RandomIdProvider randomIdProvider;

  @Test
  void genId() {
    assertThat(randomIdProvider).isNotNull();
    assertThat(randomIdProvider.genId()).isGreaterThan(Integer.MIN_VALUE);
    assertThat(randomIdProvider.genId()).isLessThan(Integer.MAX_VALUE);
  }
}
```

## Tạo method test

- Quy tắt đặt tên method test
> method_StateUnderTest_ExpectedBehavior

Ví dụ: greet_HelloWorldWithId1_ReturnGreeting

- Nội dụng 1 method test bao gồm 3 thành phần
    - Mock usecase
    - Expected value
    - Run method & assertions (Sử dụng org.junit.jupiter.api.Assertions thay cho org.junit.Assert)

Ví dụ:

```java
  @Test
  void greet_HelloWorldWithId1_ReturnGreeting() {
    Mockito.when(randomIdProvider.genId()).thenReturn(1);

    Greeting valueDefault = new Greeting(1, "Hello World!");

    Greeting greeting = greetUseCase.greet("World");
    Assertions.assertThat(valueDefault).usingRecursiveComparison().isEqualTo(greeting);
  }
```

## Assertions

- Assert 2 value

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
//...
assertEquals(expectValue, actualValue);
```

- Assert 2 object

```java
import static org.assertj.core.api.Assertions.assertThat;
//...
assertThat(expectObject).usingRecursiveComparison().isEqualTo(actualObject);
```

## Một số lưu ý

- Một method test chỉ test cho 1 trường hợp, nó phải nhỏ, dễ đọc

- Không viết trực tiếp code business logic trong mã test, thay vì đó nên hard code giá trị luôn

Ví dụ: Thay vì

```
Assertions.assertEquals(String.format("Msg: %s", param1), sampleObject.getMessage());
```

nên

```
Assertions.assertEquals("Msg abc", sampleObject.getMessage());
```

- Chỉ mock các object/class/bean mà mình viết trong project , không mock third party lib

- Không viết unit test cho các thư viện bên thứ 3

Ví dụ: nếu bạn sử dụng Lombok để generate code getter, setter thì tạo file lombok.config với nội dụng bên dưới để bỏ qua viết unit test

```
config.stopBubbling = true
lombok.addLombokGeneratedAnnotation = true
```