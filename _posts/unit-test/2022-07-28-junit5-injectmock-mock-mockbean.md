---
layout: post
title: JUnit5 - @InjectMock, @Mock vs @MockBean
date: 2022-07-28 12:00:20 +0700
description: Giải thích các annotation @InjectMock, @Mock vs @MockBean
img: unit_test/junit_5.png
tags: [Spring Boot, JUnit 5, Mockito]
categories: [Unit Test]
source: https://github.com/huypva/junit5-mock-example
---

> Giải thích các annotation @InjectMock, @Mock vs @MockBean

Ví dụ project sau
```java
@Setter
@Component
public class ComponentB  {

  private String action = "nothing";

  public void doSomething() {
    System.out.println("Component B do " + action);
  }
}

@Service
public class ServiceA {

  @Autowired
  ComponentB componentB;

  public void action(String actionName) {
    componentB.doSomething(actionName);
    System.out.println("Service A do " + actionName);
  }
}

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

## @Mock

- @Mock là annotation của thư viện mockito, nó sẽ tạo ra một object giả của class được mock mà không cần quan tâm đến class thật. 
Do đó, khi gọi bất kỳ method nào của class thì đều không thực hiện (nếu là void method) hoặc return defaul 

Ví dụ
```java
@ExtendWith(MockitoExtension.class)
class MockTest {

  @Mock
  ComponentB componentB;

  @Test
  void action() {
    System.out.println("Start test!");
    componentB.doSomething();
    System.out.println("End test.");
  }
}
```

Run test trên sẽ cho kết quả in toàn bộ bean của Spring được khởi tạo  
```
$ ./mvnw clean test
...
[INFO] Running io.github.huypva.junit5mock.MockTest
Start test!
End test.
...
```

## @InjectMock

- @InjectMock sẽ tạo object thật từ class được mock
Khác với @Mock, khi gọi method của object được inject mock thì sẽ thực hiện method thực của nó

```java
@ExtendWith(MockitoExtension.class)
class InjectMockTest {

  @InjectMocks
  ComponentB componentB;

  @Test
  void action() {
    System.out.println("Start test!");
    componentB.doSomething();
    System.out.println("End test.");
  }
}
```

Kết quả test case
```text
Start test!
Component B do nothing
End test.
...
``` 

- Khi tạo object với @InjectMock, object này sẽ inject tất cả các object đã được khởi tạo bởi ***@Mock*** và ***@Spy***

```java
@ExtendWith(MockitoExtension.class)
class InjectMockTest {

  @InjectMocks
  ServiceA serviceA;

  @Mock
  ComponentB componentB;

  @Test
  void action() {
    serviceA.action("ServiceA");
  }
}
```

Kết quả test case  
```text
Service A do ServiceA
```

## @Spy

- @Spy là khởi tạo một object thật của một class, mọi sự thay đổi sẽ thay thực hiện thật trên object này

```java
@ExtendWith(MockitoExtension.class)
class SpyTest {

  @Spy
  ComponentB componentB;

  @Test
  void action() {
    componentB.setAction("Spy");
    System.out.println("Start test!");
    componentB.doSomething();
    System.out.println("End test.");
  }
}
```

Kết quả test case
```text
Start test!
Component B do Spy
End test.
```

- Sử dụng ***@Spy*** khi muốn inject một object thật vào object test (bởi @InjectMocks)

```java
@ExtendWith(MockitoExtension.class)
class InjectSpyTest {

  @InjectMocks
  ServiceA serviceA;

  @Spy
  ComponentB componentB;

  @Test
  void action() {
    serviceA.action("Test");
  }
}
```

Kết quả test case  
```text
Component B do nothing
Service A do Test
```

## @MockBean

- @MockBean là annotation của Spring framework, một thay thế của @Mock 

```java
@SpringBootTest
class MockBeanTest {

  @MockBean
  ComponentB componentB;

  @Test
  void action() {
    System.out.println("Start test!");
    componentB.doSomething();
    System.out.println("End test.");
  }
}
```

Kết quả giống như @Mock
```
Start test!
End test.
```

