# ConfigServer 설정

## 1. 개요
- 마이크로 서비스 환경에서 어플리케이션이 늘어남에 따라 환경설정 파일도 늘어나기때문에 환경설정 파일을 Spring Cloud 를통해 한곳에서 관리한다.
- 서비스의 중단없이 환경설정 파일을 변경할수있다.

## 2. 설정방법
### 1. vehicle-config-server 프로젝트 ( server )
1. pom.xml dependency 추가
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
2. application.yml Config 파일 git 주소 추가
    ```yaml
    spring:
        application:
            name: vehicle-config-server

        cloud:
            config:
                server:
                    git:
                        uri: https://github.com/austine-namgung/config-server.git
                        username: austine-namgung
                        password: osc1234!
                # clone-on-start: true
        security:
            user:
                name: root
                password: 1234
            

    ```
3. Application.java 파일에 @EnableConfigServer 어노테이션 추가
    ```java
    package com.example.vehicleconfigserver;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.config.server.EnableConfigServer;

    @SpringBootApplication
    @EnableConfigServer
    public class VehicleConfigServerApplication {

        public static void main(String[] args) {
            SpringApplication.run(VehicleConfigServerApplication.class, args);
        }

    }


    ```
### 2. vehicle-fe 프로젝트 ( client )
1. pom.xml dependency 추가
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
2. bootstrap.yml server URL 설정 , spring actuator url 노출 추가
    ```yaml
    spring: 
        profiles:
            active: local
        application:
            name: vehicle-fe
        cloud:
            config:
                uri: http://root:1234@localhost:8888

        management:
            endpoints:  
                web:
                    exposure:
                        include: '*'
            

    ```
3. Application.java 파일에 @EnableDiscoveryClient 어노테이션 추가
    ```java
    package com.example.vehiclefe;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

    @SpringBootApplication
    @EnableDiscoveryClient
    public class VehicleFeApplication {

        public static void main(String[] args) {
            SpringApplication.run(VehicleFeApplication.class, args);
        }

    }



    ```

## 3. 결과화면
- localhost:8888 로 접속후 로그인 (applcation.yml 에설정한값 root/1234 )
    ![!](./Config2.png "")


- ex) vehicle-api 의 local properties 를 보고싶으면
    http://localhost:8888/vehicle-api/local 로 접속

- config-server 의 내용을 변경후 http://localhost:8009/configTest 호출
- actuator/refresh 호출후 config 내용 변경 확인
    ```shell
    curl -X POST http://localhost:8081/actuator/refresh
    ```
    ![!](./Config3.PNG "")