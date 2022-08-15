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
{% highlight java %}
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
{% endhighlight %}

## @Mock

- @Mock là annotation của thư viện mockito, nó sẽ tạo ra một object giả của class được mock mà không cần quan tâm đến class thật. 
Do đó, khi gọi bất kỳ method nào của class thì đều không thực hiện (nếu là void method) hoặc return defaul 

Ví dụ
{% highlight java %}
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
{% endhighlight %}

Run test trên sẽ cho kết quả in toàn bộ bean của Spring được khởi tạo  
{% endhighlight %}
$ ./mvnw clean test
...
[INFO] Running io.github.huypva.junit5mock.MockTest
Start test!
End test.
...
{% endhighlight %}

## @InjectMock

- @InjectMock sẽ tạo object thật từ class được mock
Khác với @Mock, khi gọi method của object được inject mock thì sẽ thực hiện method thực của nó

{% highlight java %}
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
{% endhighlight %}

Kết quả test case
{% highlight text %}
Start test!
Component B do nothing
End test.
...
{% endhighlight %} 

- Khi tạo object với @InjectMock, object này sẽ inject tất cả các object đã được khởi tạo bởi ***@Mock*** và ***@Spy***

{% highlight java %}
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
{% endhighlight %}

Kết quả test case  
{% highlight text %}
Service A do ServiceA
{% endhighlight %}

## @Spy

- @Spy là khởi tạo một object thật của một class, mọi sự thay đổi sẽ thay thực hiện thật trên object này

{% highlight java %}
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
{% endhighlight %}

Kết quả test case
{% highlight text %}
Start test!
Component B do Spy
End test.
{% endhighlight %}

- Sử dụng ***@Spy*** khi muốn inject một object thật vào object test (bởi @InjectMocks)

{% highlight java %}
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
{% endhighlight %}

Kết quả test case  
{% highlight text %}
Component B do nothing
Service A do Test
{% endhighlight %}

## @MockBean

- @MockBean là annotation của Spring framework, một thay thế của @Mock 

{% highlight java %}
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
{% endhighlight %}

Kết quả giống như @Mock
{% highlight text %}
Start test!
End test.
{% endhighlight %}

