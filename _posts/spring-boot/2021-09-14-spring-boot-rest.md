---
layout: post
title: Spring Boot - RESTful API
date: 2021-09-14 10:00:20 +0700
description: Hướng dẫn xây dựng RESTful service bằng Spring Boot
img: spring_boot/spring_boot_restful.png
tags: [Spring Boot, RESTful]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-rest-example
---

> Hướng dẫn xây dựng RESTful service bằng Spring Boot

## Server side
### Configuration

- Thêm dependency trong pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
``` 

- Thêm cấu hình port listen cho server trong file application.yml

```yml
server:
  port : 8082
```

### Write controller

- Viết class controller bằng cách sử dụng *RestController*

```java
@RestController
@RequestMapping("/api")
public class Controller {

  @Autowired
  GreetUseCase greetUseCase;

  @GetMapping("/get-mapping")
  public Greeting getMapping() {
    return greetUseCase.greet(0, "GetMapping");
  }
}
```

*RequestMapping* và *GetMapping* tạo nên path của api (Ví dụ: /api/get-mapping) 

### Mapping request

- Tạo mapping lấy variable từ path của api

```java
  @GetMapping("/path-variable/{user_id}")
  public Greeting pathVariable(@PathVariable(name = "user_id") int userId) {
    return greetUseCase.greet(userId, "PathVariable");
  }
```

- Tạo api nhận request param

```java
  @GetMapping("/request-param")
  public Greeting requestParam(@RequestParam(name = "user_id") int userId) {
    return greetUseCase.greet(userId, "RequestParam");
  }
```

- Tạo post request

```java
  @PostMapping("/request-body")
  public Greeting requestBody(@RequestBody User user) {
    return greetUseCase.greet(user.getUserId(), "RequestBody");
  }
```

## Client side

- Thư viện sử dụng
  + [OpenFeign](https://github.com/OpenFeign/feign){:target="_blank"}: a HTTP client 
  + [spring-cloud-openfeign](https://spring.io/projects/spring-cloud-openfeign){:target="_blank"}: a REST client for Spring Boot apps

- Thêm dependency trong file pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
        <version>2.2.3.RELEASE</version>
    </dependency>
</dependencies>
```

- Cấu hình url, path trong file application.yml

```yml
greeting-service:
  url: http://localhost:8082
  path:
    get-mapping: /api/get-mapping
    path-variable: /api/path-variable/{user_id}
    request-param: /api/request-param
    request-body: /api/request-body
```

- Sử dụng *FeignClient* annotation để tạo RestClient

```java
@FeignClient(value = "greeting", url = "${greeting-service.url}")
public interface GreetingRestClient {

  @RequestMapping(method = RequestMethod.GET, value = "${greeting-service.path.get-mapping}")
  public Greeting getMapping();

  @RequestMapping(method = RequestMethod.GET, value = "${greeting-service.path.path-variable}")
  public Greeting pathVariable(@PathVariable(name = "user_id") int userId);

  @RequestMapping(method = RequestMethod.GET, value = "${greeting-service.path.request-param}")
  public Greeting requestParam(@RequestParam(name = "user_id") int userId);

  @RequestMapping(method = RequestMethod.POST, value ="${greeting-service.path.request-body}")
  public Greeting requestBody(@RequestBody User user);

}
```

- Autowired *GreetingRestClient* bean để giao tiếp server side

```java
  @Autowired
  GreetingRestClient greetingRestClient;

  public String getMapping() {
    Greeting getResponse = greetingRestClient.getMapping();
    log.info(getResponse.toString());

    Greeting pathVariableResponse = greetingRestClient.pathVariable(1);
    log.info(pathVariableResponse.toString());

    Greeting requestParamResponse = greetingRestClient.requestParam(1);
    log.info(requestParamResponse.toString());

    Greeting requestBodyResponse = greetingRestClient.requestBody(new User(1));
    log.info(requestBodyResponse.toString());

    return "Hello world!";
  }
```