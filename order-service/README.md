## 코드 변경 사항

<br><br>
### order-service
- 프로듀서 브로커 ip address 변경
- 테스트 코드 주석, sink 부분과 producer하는 부분 원복

![](https://velog.velcdn.com/images/develing1991/post/6a687087-4262-4a7a-894d-94342700d385/image.png)

![](https://velog.velcdn.com/images/develing1991/post/6cdf7c9f-6f85-4eaa-b0f3-c5667f6a07eb/image.png)

<br><br><br><br>

## Docker

### dockerfile - 내부에서 jar 빌드
```
# https://hub.docker.com/_/maven/tags?page=1&name=openjdk-17-
FROM maven:3.8.5-openjdk-17-slim AS build
WORKDIR /home/app

# 캐싱
COPY ./pom.xml /home/app/pom.xml
COPY ./src/main/java/com/example/orderservice/OrderServiceApplication.java /home/app/src/main/java/com/example/orderservice/OrderServiceApplication.java
RUN mvn -f /home/app/pom.xml clean compile package -DskipTests=true

# 캐싱
COPY . /home/app
RUN mvn -f /home/app/pom.xml clean compile package -DskipTests=true

FROM openjdk:17-ea-17-jdk-slim
COPY --from=build /home/app/target/*.jar OrderService.jar
ENTRYPOINT [ "java", "-jar", "OrderService.jar"]
```

```bash
# 이미지 생성
docker build -t completed0728/order-service:1.0 .

# 실행
docker run -d --network suhan-network --name order-service -e "spring.config.import=optional:configserver:http://config-server:8888" -e "spring.rabbitmq.host=rabbitmq" -e "management.zipkin.tracing.endpoint=http://zipkin:9411/api/v2/spans" -e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" -e "spring.datasource.url=jdbc:mysql://mysql:3306/msa?useSSL=false&useUnicode=true&allowPublicKeyRetrieval=true" -e "logging.file=/api-logs/order-ws.log" completed0728/order-service:1.0
```



docker run -d --network suhan-network --name order-service \
-e "spring.config.import=optional:configserver:http://config-server:8888" \
-e "spring.rabbitmq.host=rabbitmq" \
-e "management.zipkin.tracing.endpoint=http://zipkin:9411/api/v2/spans" \
-e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" \
-e "spring.datasource.url=jdbc:mysql://mysql:3306/msa?useSSL=false&useUnicode=true&allowPublicKeyRetrieval=true" \
-e "logging.file=/api-logs/order-ws.log" \
completed0728/order-service:1.0

![](https://velog.velcdn.com/images/develing1991/post/77013edb-7973-46e5-84b0-031499feca6a/image.png)



docker network inspect suhan-network

![](https://velog.velcdn.com/images/develing1991/post/ebac5ac7-39d5-49e0-8d97-8fb2dfe47196/image.png)


<br><br>

### docker compose

```bash
# 이미지 생성
docker build -t completed0728/order-service:1.0 .

# 컴포즈 업
docker compose up -d
```

```yaml
version: '3.7'
services:
  order-service:
    container_name: order-service
    image: completed0728/order-service:1.0
    environment:
      spring.rabbitmq.host: rabbitmq
      spring.config.import: optional:configserver:http://config-server:8888
      management.zipkin.tracing.endpoint: http://zipkin:9411/api/v2/spans
      eureka.client.serviceUrl.defaultZone: http://naming-server:8761/eureka
      spring.datasource.url: jdbc:mysql://mysql:3306/msa?useSSL=false&useUnicode=true&allowPublicKeyRetrieval=true
      logging.file: /api-logs/order-ws.log
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```