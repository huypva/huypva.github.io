---
layout: post
title: Spring Cloud + Sleuth
date: 2021-11-20 10:00:20 +0700
description: Hướng dẫn tích hợp Spring Cloud Sleuth trong ứng dụng Spring Boot
img: spring_cloud/sleuth.png
tags: [Spring Cloud, MySQL]
categories: [Spring Cloud]
source: https://github.com/huypva/spring-cloud-sleuth-example
---

> Hướng dẫn tích hợp [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) trong ứng dụng Spring Boot

## Example

<div align="center">
    <img src="/assets/img/spring_cloud/sleuth_example.png"/>
</div>

- Trong ví dụ này bao gồm 3 service
    - ServiceA: một service public 2 api và gọi vào ServiceB, ServiceC
    - ServiceB: http service
    - ServiceC: grpc service
    
    
- Thêm dependency [spring-cloud-starter-sleuth][1] trong mỗi service

```xml
<properties>
    <spring-cloud.version>2020.0.4</spring-cloud.version>
</properties>

<dependencies>
    ...
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-sleuth</artifactId>
    </dependency>
    ...
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## Service A  

- Thêm dependencies sử dụng trong grpc
    - [spring-cloud-starter-openfeign][2]: http client
    - [grpc-spring-boot-starter][3]: grpc client 
    
```xml
<properties>
    <net.devh.grpc.starter.version>2.12.0.RELEASE</net.devh.grpc.starter.version>
</properties>
<dependencies>
    <dependency>
        <groupId>net.devh</groupId>
        <artifactId>grpc-spring-boot-starter</artifactId>
        <version>${net.devh.grpc.starter.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    <dependency>
        <groupId>io.zipkin.brave</groupId>
        <artifactId>brave-instrumentation-grpc</artifactId>
    </dependency>
</dependencies>
```

Mặc định OpenFeign tích hợp sẵn sleuth, còn với gRPC cần sử dụng thêm dependency [brave-instrumentation-grpc][4]

- Cấu hình sleuth trong application.yml

```yaml
spring:
  sleuth:
    propagation:
      type: B3 #,W3C
    sampler:
      probability: 1.0
```

- Cấu hình service B và C trong application.yml

```yaml
serviceb:
  url: http://localhost:8082
  path:
    greet: /greet/{id}
grpc:
  client:
    servicec:
      address: static://localhost:8183
      enableKeepAlive: true
      keepAliveWithoutCalls: true
      negotiationType: plaintext
```

- Sử dụng FeignClient để tạo HTTP client gọi đến ServiceB 
    
```java
@FeignClient(value = "serviceb", url = "${serviceb.url}")
public interface ServiceBClient {

  @RequestMapping(method = RequestMethod.GET, value = "${serviceb.path.greet}")
  public MessageB greet(@PathVariable(name = "id") int userId);

}
```

- Viết gRPC client gọi đến ServiceC

```java
import net.devh.boot.grpc.client.inject.GrpcClient;
...
  @GrpcClient("servicec")
  private ServiceCBlockingStub serviceCBlockingStub;

  @Override
  public MessageC greet(int id) {
    GreetRequest request = GreetRequest.newBuilder()
        .setId(id)
        .build();

    GreetResponse response = this.serviceCBlockingStub.greet(request);
    return new MessageC(response.getMessage());
  }
```

## Service B

- Cấu hình port HTTP

```yaml
server:
  port : 8082
```

- Viết Rest controller đơn giản

```java
@RestController
public class Controller {

  @GetMapping("/greet/{id}")
  public MessageB greet(@PathVariable(name = "id") int id) {
    log.info("ServiceB.greet");
    return new MessageB("MessageB greet " + id);
  }
}
```

## ServiceC

- Cấu hình port gRPC trong file application.yml

```yaml
grpc:
  server:
    port: 8183
```

- Viết gRPC controller

```java
import net.devh.boot.grpc.server.service.GrpcService;
...
@GrpcService
public class GrpcController extends ServiceCGrpc.ServiceCImplBase {

  @Override
  public void greet(GreetRequest request, StreamObserver<GreetResponse> responseObserver) {
    log.info("ServiceC.greet " + request.getId());
    GreetResponse response = GreetResponse.newBuilder()
        .setMessage("Message C")
        .build();

    responseObserver.onNext(response);
    responseObserver.onCompleted();
  }

}
```

## Test sleuth

- Start 3 service A, B, C

- Gửi request test đến ServiceA, trong ServiceA sẽ gọi đến ServiceB

```shell
$ curl http://localhost:8081/greetb/1
```

Log service A:
```text
2021-11-20 21:01:03.834  INFO [service-A,48c90d4d61668476,48c90d4d61668476] 2433 --- [nio-8081-exec-1] i.c.servicea.entrypoint.Controller       : ServiceA.greetB
```
và service B
```text
2021-11-20 21:01:03.939  INFO [service-B,48c90d4d61668476,c73d7b8f5c34adc6] 2429 --- [nio-8082-exec-1] i.c.serviceb.entrypoint.Controller       : ServiceB.greet
```
chung TraceId 48c90d4d61668476

- Gửi request test đến ServiceA, trong ServiceA sẽ gọi đến ServiceC

```shell
$ curl http://localhost:8081/greetc/1
```

Log service A:
```text
2021-11-20 21:02:54.170  INFO [service-A,9b6f0e1e5e5ca2da,9b6f0e1e5e5ca2da] 2433 --- [nio-8081-exec-2] i.c.servicea.entrypoint.Controller       : ServiceA.greetC
```
và service C
```text
2021-11-20 21:02:54.340  INFO [service-C,9b6f0e1e5e5ca2da,9cd977fdc2849bb4] 2100 --- [ault-executor-2] i.c.servicec.entrypoint.GrpcController   : ServiceC.greet 1
```
chung TraceId 9b6f0e1e5e5ca2da

## Report lên Zipkin, Jaeger

- Setup Zipkin bằng docker-compose

```yaml
version: "3.4"
services:
  zipkin:
    image: openzipkin/zipkin
    ports:
      - 9411:9411
```

Run docker-compose

```shell script
docker-compose up -d
```

- Trong mỗi service, thêm thông tin dependency 

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-sleuth-zipkin</artifactId>
    </dependency>
</dependencies>
```

- Thêm cấu hình zipkin trong application.yml

```
spring:
  zipkin:
    base-url: http://localhost:9411
```

- Send request và mở Zipkin UI tại địa chỉ <http://localhost:9411/> lên xem report
<div align="center">
    <img src="/assets/img/spring_cloud/zipkin_service_b.png"/>
</div>
<div align="center">
    <img src="/assets/img/spring_cloud/zipkin_service_c.png"/>
</div>

- Ngoài ra, có thể sử dụng Jeager server để hiện thị report
    - Setup Jeager server bằng docker-compose và cho phép nhận report theo format của zipkin
    
```yaml
version: "3.4"
services:
  jaeger:
    image: jaegertracing/all-in-one:1.17
    ports:
      - 5775:5775/udp
      - 6831:6831/udp
      - 6832:6832/udp
      - 5778:5778
      - 16686:16686
      - 14268:14268
      - 14250:14250
      - 9411:9411
    environment:
      - COLLECTOR_ZIPKIN_HTTP_PORT=9411
```

Test và xem kết quả trên Jeager UI tại địa chỉ <http://localhost:16686/>

<div align="center">
    <img src="/assets/img/spring_cloud/jaeger_a_b.png"/>
</div>
<div align="center">
    <img src="/assets/img/spring_cloud/jaeger_a_c.png"/>
</div>

## Reference
- <https://spring.io/projects/spring-cloud-sleuth>
- <https://www.baeldung.com/spring-cloud-sleuth-single-application>
- <https://docs.spring.io/spring-cloud-sleuth/docs/current-SNAPSHOT/reference/html/integrations.html#sleuth-openfeign-integration>
- <https://github.com/yidongnan/grpc-spring-boot-starter>

[1]: https://cloud.spring.io/spring-cloud-sleuth
[2]: https://docs.spring.io/spring-cloud-sleuth/docs/current-SNAPSHOT/reference/html/integrations.html#sleuth-openfeign-integration
[3]: https://yidongnan.github.io/grpc-spring-boot-starter/en/brave.html
[4]: https://github.com/openzipkin/brave/tree/master/instrumentation/grpc