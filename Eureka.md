# Eureka 설정

## 1. 개요
- Eureka Server, Client 로 구성되며 Netflix에서 처음 구현하였습니다.

- 마이크로서비스들의 정보를 등록하고 로드밸런싱 및 모니터링, IP,Port로 어플리케이션을 찾는게 아닌 Applicaiton 명으로 서비스를 찾을수있다.

## 2. 설정방법
- common-api, vehicle-api, vehicle-fe 프로젝트의 Eureka Branch
- eureka-server 프로젝트 필요
### 1. Eureka Server
1. pom.xml dependency 추가
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    ```
2. application.yml 서버 정보 추가
    ```yaml
    server:
        port: 8761  
    eureka:
        instance:
            hostname: localhost
        client:
            registerWithEureka: false
            fetchRegistry: false
            serviceUrl:
            defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
    ```
3. Application.java 파일에 @EnableEurekaServer 어노테이션 추가
    ```java
    package com.example.eurekaserver;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

    @EnableEurekaServer
    @SpringBootApplication
    public class EurekaServerApplication {

        public static void main(String[] args) {
            SpringApplication.run(EurekaServerApplication.class, args);
        }

    }

    ```

### 2. Eureka Client
1. pom.xml dependency 추가
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    ```
2. application.yml 서버 정보 추가
    ```yaml
    api:
        vehicle:
            url: http://vehicle-api
        common:
            url: http://common-api

    eureka:
        client:
            serviceUrl:
                defaultZone: http://127.0.0.1:8761/eureka/

    ```
3. Application.java 파일에 @EnableDiscoveryClient 어노테이션 추가
    ```java
    package com.example.common;

    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
    import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

    @SpringBootApplication
    @EnableDiscoveryClient
    public class CommonApiApplication {

        public static void main(String[] args) {
            SpringApplication.run(CommonApiApplication.class, args);
        }

    }

    ```


## 3. 결과화면
1. Eureka Server 접속 화면
    ![!](./Eureka1.png "")
2. Eureka Client 에서 Application Name 으로 호출 성공
    - 호출코드 ( vehicleApiUrl = http://vehicle-api)
    ```java
    @Override
    public List<Vehicle> vehicleList() {

        List<Vehicle> vehicleList = webClientBuilder.build().get().uri(vehicleApiUrl+"/api/vehicles").retrieve().bodyToFlux(Vehicle.class)
                .collectList().block();
        List<Code> categoryList = commonCategoryList();
        List<Code> modelList = commonModelList();
        return mappedCodeNameVehicleList(vehicleList, categoryList, modelList);

    }
    ```
    - 호출완료
    ![!](./Eureka2.png "")
## 4. 참고


