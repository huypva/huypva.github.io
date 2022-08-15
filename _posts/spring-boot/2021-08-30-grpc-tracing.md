---
layout: post
title: Tracing gRPC API by Jaeger
date: 2021-08-30 10:00:20 +0700
description: Tracing gRPC API by Jaeger
img: spring_boot/jeager.png
tags: [Spring Boot]
categories: [Spring Boot]
source: https://github.com/huypva/grpc-tracing-example
---

> Hướng dẫn tích hợp OpenTracing tracing Grpc API trong Spring Boot application 

Thư viện sử dụng:
- [grpc-spring-boot-starter](https://github.com/yidongnan/grpc-spring-boot-starter){:target="_blank"} - Spring Boot starter module for gRPC framework
- [opentracing-contrib/java-spring-cloud](https://github.com/opentracing-contrib/java-spring-cloud){:target="_blank"} - OpenTracing instrumentation for Spring Boot
- [opentracing-contrib/java-grpc](https://github.com/opentracing-contrib/java-grpc){:target="_blank"} - OpenTracing instrumentation for gRPC

Định nghĩa dependency trong pom.xml
{% highlight xml %}
    <properties>
        <com.google.api.grpc.version>1.18.0</com.google.api.grpc.version>
        <com.google.protobuf.version>3.7.1</com.google.protobuf.version>
        <grpc.springboot.starter.version>2.12.0.RELEASE</grpc.springboot.starter.version>

        <io.grpc.version>1.37.0</io.grpc.version>

        <kr.motd.maven.version>1.5.0.Final</kr.motd.maven.version>
        <org.xolstice.maven.plugins.version>0.6.1</org.xolstice.maven.plugins.version>

        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        #server service only
        <dependency>
            <groupId>net.devh</groupId>
            <artifactId>grpc-spring-boot-starter</artifactId>
            <version>${grpc.springboot.starter.version}</version>
        </dependency>
        #client service only
        <dependency>
            <groupId>net.devh</groupId>
            <artifactId>grpc-client-spring-boot-starter</artifactId>
            <version>${grpc.springboot.starter.version}</version>
        </dependency>
        #both client and server
        <dependency>
            <groupId>io.opentracing.contrib</groupId>
            <artifactId>opentracing-grpc</artifactId>
            <version>0.2.3</version>
        </dependency>
        <dependency>
            <groupId>io.opentracing.contrib</groupId>
            <artifactId>opentracing-spring-jaeger-cloud-starter</artifactId>
            <version>3.1.2</version>
        </dependency>
    </dependencies>
{% endhighlight %}

- Cấu hình OpenTracing trong application.yml

{% highlight yaml %}
opentracing.jaeger:
  service-name: grpc-server
  enabled: true
  udp-sender:
    host: localhost
    port: 6831
  log-spans: true
  enable-b3-propagation: false
  probabilistic-sampler:
    sampling-rate: 1.0
{% endhighlight %} 

## Server

- Tạo grpc Controler

{% highlight java %}
@GrpcService
public class GrpcController extends SimpleGrpc.SimpleImplBase {
 
  @Override
  public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
    HelloReply reply = HelloReply.newBuilder().setMessage("Hello ==> " + req.getName()).build();
    responseObserver.onNext(reply);
    responseObserver.onCompleted();
  }
 
}
{% endhighlight %} 

- Tạo và intercept `TracingServerInterceptor` vào GrpcControler thông qua annotation `GrpcGlobalServerInterceptor`

{% highlight java %}
  @Autowired
  private Tracer tracer;
 
  @GrpcGlobalServerInterceptor
  TracingServerInterceptor tracingServerInterceptor() {
    TracingServerInterceptor tracingInterceptor = TracingServerInterceptor
      .newBuilder()
      .withTracer(tracer)
      .withStreaming()
      .withVerbosity()
      .withTracedAttributes(ServerRequestAttribute.HEADERS,
          ServerRequestAttribute.METHOD_TYPE)
      .build();
 
    return tracingInterceptor;
  }
{% endhighlight %}

## Client

- Tạo grpc-client

{% highlight java %}
  @GrpcClient("service-name")
  private SimpleBlockingStub simpleStub;
   
  @Override
  public String hello(String name) {
    try {
      final HelloReply response = this.simpleStub.sayHello(HelloRequest.newBuilder().setName(name).build());
      return response.getMessage();
    } catch (final StatusRuntimeException e) {
      return "FAILED with " + e.getStatus().getCode().name();
    }
  }
{% endhighlight %}

- Tạo `TracingClientInterceptor` và intercept vào grpc-client thông qua annotation `GrpcGlobalClientInterceptor`

{% highlight java %}
  @Autowired
  private Tracer tracer;
 
  @GrpcGlobalClientInterceptor
  TracingClientInterceptor tracingServerInterceptor() {
    TracingClientInterceptor tracingInterceptor = TracingClientInterceptor
      .newBuilder()
      .withTracer(tracer)
      .withStreaming()
      .withTracedAttributes(ClientRequestAttribute.ALL_CALL_OPTIONS, ClientRequestAttribute.HEADERS)
      .build();
 
    return tracingInterceptor;
  }
{% endhighlight %}