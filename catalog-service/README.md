## 코드 변경 사항

<br><br>
### catalog-service
- 프로듀서 컨슈머 ip address 변경

![](https://velog.velcdn.com/images/develing1991/post/fcd433a3-c736-4f85-92a0-bb01483b29bb/image.png)

<br><br><br><br>

## Docker

### dockerfile - 내부에서 jar 빌드
```
# https://hub.docker.com/_/maven/tags?page=1&name=openjdk-17-
FROM maven:3.8.5-openjdk-17-slim AS build
WORKDIR /home/app

# 캐싱
COPY ./pom.xml /home/app/pom.xml
COPY ./src/main/java/com/example/catalogservice/CatalogServiceApplication.java /home/app/src/main/java/com/example/catalogservice/CatalogServiceApplication.java
RUN mvn -f /home/app/pom.xml clean compile package -DskipTests=true

# 캐싱
COPY . /home/app
RUN mvn -f /home/app/pom.xml clean compile package -DskipTests=true

FROM openjdk:17-ea-17-jdk-slim
COPY --from=build /home/app/target/*.jar CatalogService.jar
ENTRYPOINT [ "java", "-jar", "CatalogService.jar"]
```

```bash
# 이미지 생성
docker build -t completed0728/catalog-service:1.0 .

# 실행
docker run -d --network suhan-network --name catalog-service -e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" -e "spring.datasource.url=jdbc:mysql://mysql:3306/msa?useSSL=false&useUnicode=true&allowPublicKeyRetrieval=true" -e "logging.file=/api-logs/catalog-ws.log" completed0728/catalog-service:1.0
```

docker run -d --network suhan-network --name catalog-service \
-e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" \
-e "spring.datasource.url=jdbc:mysql://mysql:3306/msa?useSSL=false&useUnicode=true&allowPublicKeyRetrieval=true" \
-e "logging.file=/api-logs/catalog-ws.log" \
completed0728/catalog-service:1.0

![](https://velog.velcdn.com/images/develing1991/post/c2a404e0-ed2c-429a-b0cc-35a3a7ae2c9b/image.png)




docker network inspect suhan-network

![](https://velog.velcdn.com/images/develing1991/post/a41c5a07-bf26-42a6-8a2d-f745a1f7a2f8/image.png)



<br><br>

- 이 후 추가
  zipkin, config, rabbitmq..

```
# 실행
docker run -d --network suhan-network --name catalog-service -e "spring.config.import=optional:configserver:http://config-server:8888" -e "spring.rabbitmq.host=rabbitmq" -e "management.zipkin.tracing.endpoint=http://zipkin:9411/api/v2/spans" -e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" -e "spring.datasource.url=jdbc:mysql://mysql:3306/msa?useSSL=false&useUnicode=true&allowPublicKeyRetrieval=true" -e "logging.file=/api-logs/catalog-ws.log" completed0728/catalog-service:1.0
```

docker run -d --network suhan-network --name catalog-service \
-e "spring.config.import=optional:configserver:http://config-server:8888" \
-e "spring.rabbitmq.host=rabbitmq" \
-e "management.zipkin.tracing.endpoint=http://zipkin:9411/api/v2/spans" \
-e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" \
-e "spring.datasource.url=jdbc:mysql://mysql:3306/msa?useSSL=false&useUnicode=true&allowPublicKeyRetrieval=true" \
-e "logging.file=/api-logs/catalog-ws.log" \
completed0728/catalog-service:1.0


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
  catalog-service:
    container_name: catalog-service
    image: completed0728/catalog-service:1.0
    environment:
      eureka.client.serviceUrl.defaultZone: http://naming-server:8761/eureka
      spring.datasource.url: jdbc:mysql://mysql:3306/msa?useSSL=false&useUnicode=true&allowPublicKeyRetrieval=true
      logging.file: /api-logs/catalog-ws.log
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```