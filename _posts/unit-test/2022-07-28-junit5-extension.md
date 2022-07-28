---
layout: post
title: JUnit5 - @SpringBootTest, @SpringExtension vs @MockitoExtension
date: 2022-07-28 10:00:20 +0700
description: Giải thích các cách khởi tạo môi trường test
img: unit_test/junit_5.png
tags: [Spring Boot, JUnit 5, Mockito]
categories: [Unit Test]
source: https://github.com/huypva/junit5-extension-example
---

> Giải thích các annotation @SpringBootTest, @ExtendWith khi viết unit test trong Spring Boot application

Ví dụ project sau
```java
@Service
public class ServiceA {

  @Autowired
  ComponentB componentB;

  public String greet(String name) {
    int id = componentB.genId();
    return String.format("Hello %s with id %d!", name, id);
  }
}

@Component
public class ComponentB  {

  public int genId() {
    Random random = new Random();
    return random.nextInt();
  }
}

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

## @SpringBootTest

- ***@SpringBootTest*** khởi tạo toàn bộ Spring Context bằng cách tìm kiếm class có gắn annotation @SpringBootApplication để thực hiện nó

Ví dụ
```java
@SpringBootTest
class SpringBootTestTest {

  @Autowired
  ApplicationContext ctx;

  @Test
  void printAllBean() {
    String[] beanNames = ctx.getBeanDefinitionNames();
    Arrays.sort(beanNames);
    for (String beanName : beanNames) {
      System.out.println("Bean: " + beanName);
    }
  }
}
```

Run Test trên sẽ cho kết quả in toàn bộ bean của Spring được khởi tạo  
```
...
Bean: server-org.springframework.boot.autoconfigure.web.ServerProperties
Bean: serviceA
Bean: servletWebServerFactoryCustomizer
Bean: simpleControllerHandlerAdapter
...
```

- Sau khi khởi tạo xong, có thể sử dụng ***@Autowired*** để lấy các bean trong Spring context ra test

```java
@SpringBootTest
class SpringBootTestTest {

  @Autowired
  ServiceA greetUseCase;

  @MockBean
  ComponentB randomIdProvider;

  @Test
  void greeting() {
    given(randomIdProvider.genId()).willReturn(1);

    String greeting = greetUseCase.greet("World");
    Assertions.assertEquals("Hello World with id 1!", greeting);
  }
}
```

## @ExtendWith(SpringExtension.class)

- Khởi tạo môi trường test do Spring quản lý, nhưng chỉ có vài bean cần thiết được tạo ra

```java
@ExtendWith(SpringExtension.class)
class SpringExtensionTest {

  @Autowired
  ApplicationContext ctx;

  @Test
  void printAllBean() {
    String[] beanNames = ctx.getBeanDefinitionNames();
    Arrays.sort(beanNames);
    for (String beanName : beanNames) {
      System.out.println("Bean: " + beanName);
    }
  }
}
```

Kết quả in ra các bean sau
```java
Bean: org.springframework.boot.test.mock.mockito.MockitoPostProcessor
Bean: org.springframework.boot.test.mock.mockito.MockitoPostProcessor$SpyPostProcessor
Bean: org.springframework.context.annotation.internalAutowiredAnnotationProcessor
Bean: org.springframework.context.annotation.internalCommonAnnotationProcessor
Bean: org.springframework.context.annotation.internalConfigurationAnnotationProcessor
Bean: org.springframework.context.event.internalEventListenerFactory
Bean: org.springframework.context.event.internalEventListenerProcessor
``` 

- Nếu cần test Một bean cần ***@Import*** class đó và sử dụng ***@Autowired*** để lấy bean ra test

```java
@Import(ServiceA.class)
@ExtendWith(SpringExtension.class)
class SpringExtensionTest {

  @Autowired
  ServiceA greetUseCase;

  @MockBean
  ComponentB randomIdProvider;

  @Test
  void greeting() {
    given(randomIdProvider.genId()).willReturn(1);

    String greeting = greetUseCase.greet("World");
    Assertions.assertEquals("Hello World with id 1!", greeting);
  }
}
```

## @ExtendWith(MockitoExtension.class)
- Tạo một môi trường test do Mockito quản lý. Môi trường này không phải của Spring sẽ không có bất kỳ bean nào được tạo ra, dù @Autowired ApplicationContext cũng báo lỗi ngay ***NullPointerException***  

- Sử dụng annotation ***@InjectMocks*** và ***@Mock*** để tạo ra các object cần test cũng như giải lập object để test

```java
@ExtendWith(MockitoExtension.class)
class ServiceAMockitoExtensionTest {

  @InjectMocks
  ServiceA greetUseCase;

  @Mock
  ComponentB randomIdProvider;

  @Test
  void greeting() {
    given(randomIdProvider.genId()).willReturn(1);

    String greeting = greetUseCase.greet("World");
    Assertions.assertEquals("Hello World with id 1!", greeting);
  }
}
```

## Conclusion

- Nếu chỉ muốn unit test logic của method thì chỉ nên sử dụng ***MockitoExtension***, sau đó tạo và giải lập các object để test

- Chỉ nên sử dụng ***SpringExtension*** và ***SpringBootTest*** trong trường hợp muốn integration test một vài component hoặc tất cả component, lúc đó cần thực hiện giải lập các công nghệ xung quanh mà project sử dụng