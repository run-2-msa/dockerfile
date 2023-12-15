### dockerfile

![](https://velog.velcdn.com/images/develing1991/post/25b26785-566c-4388-b122-bad0cd65757e/image.png)

도커파일 생성 및 버전 1.0 변경

<br><br>


### 기본 테스트 코드 있어서 의존성 추가
```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```


- mvn clean compile package -DskipTests=true

- docker build -t completed0728/gateway-service:1.0 .


- docker run -d -p "8000:8000" --network suhan-network -e "spring.config.import=optional:configserver:http://config-server:8888" -e "spring.rabbitmq.host=rabbitmq" -e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" --name gateway-service completed0728/gateway-service:1.0

```bash
docker run -d -p "8000:8000" --network suhan-network -e "spring.config.import=optional:configserver:http://config-server:8888" -e "spring.rabbitmq.host=rabbitmq" -e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" --name gateway-service completed0728/gateway-service:1.0
```

- docker network inspect suhan-network

![](https://velog.velcdn.com/images/develing1991/post/48e84bca-60f4-49ed-9633-b99eecc57451/image.png)

### docker compose

```yaml
version: '3.7'
services:
  gateway-service:
    container_name: gateway-service
    image: completed0728/gateway-service:1.0
    ports:
      - "8000:8000"
    environment:
      spring.rabbitmq.host: rabbitmq
      spring.config.import: optional:configserver:http://config-server:8888
      eureka.client.serviceUrl.defaultZone: http://naming-server:8761/eureka
    #    depends_on:
    #      - rabbitmq
    #      - config-server
    #      - naming-server
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```

