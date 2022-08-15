---
layout: post
title: Spring Boot + Mockito
date: 2021-10-22 10:00:20 +0700
description: Hướng dẫn sử dụng Mockito trong SpringBoot application
img: unit_test/mockito.png
tags: [Spring Boot, Unit Test, Mockito]
categories: [Unit Test]
source: https://github.com/huypva/unit-test-mockito-example
---

> Hướng dẫn sử dụng Mockito trong SpringBoot application

## Dependency
- Thêm dependency trong pom.xml để thực hiện UnitTest

{% highlight html %}
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
  </dependency>
</dependencies>
{% endhighlight %}

Trong Spring Boot version 2.2.6 trở lên đã có sẵn thư viện cho junit 5
<div align="center">
  <img src="/assets/img/unit_test/junit_5_libs.png"/>
</div>

## Mock bean
- Sử dụng annotation ***@Mockbean*** để mock bean  
{% highlight java %}
@ExtendWith(SpringExtension.class)
class UserCaseTest {

  @MockBean
  DataProvider dataProvider;

  @Autowired
  UserCase userCase;

  @Test
  void returnMethod() {
    Mockito.when(dataProvider.getCount()).thenReturn(2);

    Assertions.assertEquals(2, userCase.returnMethod());
  }
}
{% endhighlight %}

## Mock static class
- Sử dụng ***MockedStatic*** như sau
  
{% highlight java %}
  try (MockedStatic<[Class cần mock]> mockedStatic = Mockito.mockStatic([Class cần mock].class); ) {
    //mock function bằng cách sử dụng mockedStatic 
  }
{% endhighlight %}

Ví dụ: Giả sử cần test class ***MockStatic*** sử dụng static class ***StaticClass***  
{% highlight java %}
public class MockStatic {

  public String doMockStatic() {
    return "Static: " + StaticClass.doStaticMethod(1);
  }
}
{% endhighlight %}

Viết unit test như sau  
{% highlight java %}
@ExtendWith(SpringExtension.class)
class MockStaticTest {

  @Test
  void doMockStatic() {
    try (MockedStatic<StaticClass> mockedStatic = Mockito.mockStatic(StaticClass.class); ) {
      mockedStatic.when(() -> StaticClass.doStaticMethod(1)).thenReturn("1");

      MockStatic mockStatic = new MockStatic();
      Assertions.assertEquals("Static: 1", mockStatic.doMockStatic());
    }
  }
}
{% endhighlight %}

## Mock return function 

- Giả lập function theo format
  
{% highlight java %}
Mockito.when(...).thenReturn(...);
{% endhighlight %}

Ví dụ  
{% highlight java %}
@Test
void returnMethod_Count_Return2() {
Mockito.when(dataProvider.getCount()).thenReturn(2);

Assertions.assertEquals(2, userCase.returnMethod());
}
{% endhighlight %}

## Mock void function
- Giả lập void function theo format

{% highlight java %}
Mockito.doNothing().when(...);
{% endhighlight %}

Ví dụ  
{% highlight java %}
@Test
void voidMethod_() {
Mockito.doNothing().when(dataProvider).increase(1);
//...
}
{% endhighlight %}
