---
layout: post
title: Spring Boot + Sonar Qube
date: 2021-10-18 10:00:20 +0700
description: Hướng dẫn tích hợp SonarQube trong SpringBoot application
img: unit_test/sonar_qube/sonar_qube.png
tags: [Spring Boot, Unit Test, Sonar Qube]
categories: [Unit Test]
source: https://github.com/huypva/unit-test-sonar-qube-example
---

> Hướng dẫn tích hợp SonarQube trong SpringBoot application

## Cài đặt SonarQube

- Tạo docker-compose file chạy SonarQube 

```yaml
version: "3.4"
services:
  sonarqube:
    image: sonarqube:7.9.2-community
    container_name: sonarqube
    restart: unless-stopped
    environment:
      - SONARQUBE_JDBC_USERNAME=sonar
      - SONARQUBE_JDBC_PASSWORD=v07IGCFCF83Z95NX
      - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonarqube
    ports:
      - "9000:9000"
      - "9092:9092"
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_bundled-plugins:/opt/sonarqube/lib/bundled-plugins
    networks:
      unit-test-sonar-qube-network:

  db:
    image: postgres:12.1
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=v07IGCFCF83Z95NX
      - POSTGRES_DB=sonarqube
    volumes:
      - sonarqube_db:/var/lib/postgresql
      # This needs explicit mapping due to https://github.com/docker-library/postgres/blob/4e48e3228a30763913ece952c611e5e9b95c8759/Dockerfile.template#L52
      - postgresql_data:/var/lib/postgresql/data
    networks:
      unit-test-sonar-qube-network:

volumes:
  postgresql_data:
  sonarqube_bundled-plugins:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_db:
  sonarqube_extensions:

networks:
  unit-test-sonar-qube-network:
```

## Tạo token

- Truy cập SonarQube tại địa chỉ http://localhost:9000/ và login theo tài khoản admin/admin

Open setting account:

<div align="center">
    <img src="/assets/img/unit_test/sonar_qube/setting.png"/>
</div>

Tạo token:

<div align="center">
    <img src="/assets/img/unit_test/sonar_qube/create_token.png"/>
</div>

Lưu lại kết qua token:

<div align="center">
    <img src="/assets/img/unit_test/sonar_qube/result.png"/>
</div>

## Setup report trong project

- Thêm plugin jacoco trong file pom.xml của project

```xml
    <properties>
        <jacoco.version>0.8.6</jacoco.version>
    </properties>
    <!--  ...  -->
    <plugin>
        <groupId>org.jacoco</groupId>
        <artifactId>jacoco-maven-plugin</artifactId>
        <version>${jacoco.version}</version>
        <executions>
            <execution>
                <id>check</id>
                <phase>test</phase>
                <goals>
                    <goal>check</goal>
                </goals>
                <configuration>
                    <rules>
                        <rule>
                            <limits>
                                <limit>
                                    <counter>LINE</counter>
                                    <value>COVEREDRATIO</value>
                                    <minimum>0%</minimum>
                                </limit>
                            </limits>
                        </rule>
                    </rules>
                </configuration>
            </execution>
            <execution>
                <goals>
                    <goal>prepare-agent</goal>
                </goals>
            </execution>
            <execution>
                <id>generate-code-coverage-report</id>
                <phase>test</phase>
                <goals>
                    <goal>report</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
```

- Thêm file lombok.config để bỏ qua lombok khi report SonarQube

```text
config.stopBubbling = true
lombok.addLombokGeneratedAnnotation = true
```

- Chạy unit test và report trên sonar với giá trị sonar.login bằng token đã lưu bên trên

```shell
$ ../mvnw clean package
$ ../mvnw sonar:sonar \
  -Dsonar.projectKey=io.codebyexample:unit-test-sonar-qube \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=b229f28876ad85e6cdb5a30ef7e1331f66a8a2c9
```