---
layout: post
title: Spring Boot - gRPC API
date: 2021-09-17 10:00:20 +0700
description: Hướng dẫn xây dựng gRPC service bằng Spring Boot
img: spring_boot/spring_boot_grpc.png
tags: [Spring Boot, gRPC]
categories: [Spring Boot]
source: https://github.com/huypva/spring-boot-grpc-example
---

> Hướng dẫn xây dựng gRPC service bằng Spring Boot

- Thư viện sử dụng
  - [grpc-spring-boot-starter](https://github.com/yidongnan/grpc-spring-boot-starter){:target="_blank"}:  Spring Boot starter module for gRPC framework

## Build proto file 
- Tạo file proto và thêm trong thư mục /src/main/proto của ứng dụng.

Cấu trúc thư mục như sau:

```
.
├── src
│   ├── main
|   |   ├── java
|   |   |   ...
|   |   ├── proto
|   |   |   ├── greeting.proto
|   |   |   ...
|   |   ├── resources
|   |       ├── application.yml
|   |       ...
|   |
│   ├── test
│   ├── Dockerfile
│   ...
```

- Thêm build trong file pom.xml

```xml
<dependencies>
    <properties>
        <kr.motd.maven.version>1.5.0.Final</kr.motd.maven.version>
        <org.xolstice.maven.plugins.version>0.6.1</org.xolstice.maven.plugins.version>
    </properties>
    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>${kr.motd.maven.version}</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
    
            <!--   gRPC compilation   -->
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>${org.xolstice.maven.plugins.version}</version>
                <configuration>
                    <pluginArtifact>
                        io.grpc:protoc-gen-grpc-java:${io.grpc.version}:exe:${os.detected.classifier}
                    </pluginArtifact>
                    <pluginId>grpc-java</pluginId>
                    <protoSourceRoot>${basedir}/src/main/proto</protoSourceRoot>
                    <protocArtifact>
                        com.google.protobuf:protoc:${com.google.protobuf.version}:exe:${os.detected.classifier}
                    </protocArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

## Server side

- Thêm dependency trong pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>net.devh</groupId>
        <artifactId>grpc-spring-boot-starter</artifactId>
        <version>2.12.0.RELEASE</version>
    </dependency>
</dependencies>
``` 

- Thêm port gRPC trong file application.yml

```yml
grpc:
  server:
    port: 8082
```

- Sử dụng annotation *GrpcService* để tạo controller

```java
@GrpcService
public class Controller extends GreetingGrpc.GreetingImplBase {

  @Autowired
  GreetUseCase greetUseCase;

  @Override
  public void greet(GreetingRequest request, StreamObserver<GreetingResponse> responseObserver) {
    String message = greetUseCase.greet(request.getName());

    GreetingResponse response = GreetingResponse.newBuilder()
        .setMessage(message)
        .build();

    responseObserver.onNext(response);
    responseObserver.onCompleted();
  }

}
```

## Client side

- Thêm dependency trong file pom.xml

```xml
		<dependency>
			<groupId>net.devh</groupId>
			<artifactId>grpc-client-spring-boot-starter</artifactId>
			<version>2.12.0.RELEASE</version>
		</dependency>
```

- Cấu hình thông tin server trong application.yml

```yml
grpc:
  client:
    greeting:
      address: static://localhost:8082
      enableKeepAlive: true
      keepAliveWithoutCalls: true
      negotiationType: plaintext
```

- Sử dụng *GrpcClient* để tạo GrpcClient

```java
public class GreetingGrpcClient implements GreetingClient {

  @GrpcClient("greeting")
  private GreetingBlockingStub greetingBlockingStub;

  @Override
  public String greet(String name) {
    GreetingRequest greetingRequest = GreetingRequest.newBuilder().setName(name).build();

    GreetingResponse response = greetingBlockingStub.greet(greetingRequest);

    return response.getMessage();
  }
}
```