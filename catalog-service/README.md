## 코드 변경 사항

### local-repo-config.git
- 도커의 gateway ip address / subnet 확인해서 변경
- 도커의 mysql 컨테이너 이름으로 변경
- docker network inspect suhan-network

![](https://velog.velcdn.com/images/develing1991/post/958b5501-204a-4028-af68-f3b733de8661/image.png)

<br><br>
### user-service
- ** gateway-service - Ip address 체크 로직 변경 **
- `@Bean` 안에서 환경변수 사용하기 위해 `EnvironmentAware` 구현
- `@Override setEnvironment()`

```java
public class SecurityConfig implements EnvironmentAware {
	
    private Environment env;
    
    @Override
    public void setEnvironment(final Environment env) {
        this.env = env;
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    	...중략
    }
    
    private AuthorizationDecision hasIpAddress(Supplier<Authentication> authentication, RequestAuthorizationContext object) {

        String ALLOWED_IP_ADDRESS = env.getProperty("gateway.ip"); // 
        String SUBNET = env.getProperty("gateway.subnet");	//
        IpAddressMatcher ALLOWED_IP_ADDRESS_MATCHER = new IpAddressMatcher(ALLOWED_IP_ADDRESS + SUBNET);
        return new AuthorizationDecision(ALLOWED_IP_ADDRESS_MATCHER.matches(object.getRequest()));
    }
}
```

<br><br><br><br>

## Docker

### dockerfile - 외부에서 jar 빌드 copy
- 로컬 머신에 무언가를 빌드해서 Docker 이미지로 복사해서 가져오는 방식은 비추
  다른 사람의 머신에서 실행했을 때 출력이 다소 달라질 가능성이 있기 때문
```yaml
FROM openjdk:17-ea-17-jdk-slim
VOLUME /tmp
COPY target/user-service-1.0.jar UserService.jar
ENTRYPOINT [ "java", "-jar", "UserService.jar"]
```

```bash
# jar 빌드
mvn clean compile package -DskipTests=true

# 이미지 생성
docker build -t completed0728/user-service:1.0 .

# 실행
docker run -d --network suhan-network --name user-service -e "spring.config.import=optional:configserver:http://config-server:8888" -e "spring.rabbitmq.host=rabbitmq" -e "management.zipkin.tracing.endpoint=http://zipkin:9411/api/v2/spans" -e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" -e "logging.file=/api-logs/users-ws.log" completed0728/user-service:1.0
```

docker run -d --network suhan-network --name user-service \
-e "spring.config.import=optional:configserver:http://config-server:8888" \
-e "spring.rabbitmq.host=rabbitmq" \
-e "management.zipkin.tracing.endpoint=http://zipkin:9411/api/v2/spans" \
-e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" \
-e "logging.file=/api-logs/users-ws.log" \
completed0728/user-service:1.0

![](https://velog.velcdn.com/images/develing1991/post/41ddcd15-166e-4b28-b1e4-8a9b813ad2d4/image.png)


<br><br>

### dockerfile - 내부에서 jar 빌드 (추천)
- 대신 안에서 빌드할 때 pom.xml 변경사항이 없으면 캐싱 되도록
- 변경이 잦은 건 우리가 직접 작성하는 .java파일이니까..

```yaml
# https://hub.docker.com/_/maven/tags?page=1&name=openjdk-17-
FROM maven:3.8.5-openjdk-17-slim AS build
WORKDIR /home/app

# 캐싱
COPY ./pom.xml /home/app/pom.xml
COPY ./src/main/java/com/example/userservice/UserServiceApplication.java /home/app/src/main/java/com/example/userservice/UserServiceApplication.java
RUN mvn -f /home/app/pom.xml clean compile package -DskipTests=true

COPY . /home/app
RUN mvn -f /home/app/pom.xml clean compile package -DskipTests=true

FROM openjdk:17-ea-17-jdk-slim
COPY --from=build /home/app/target/*.jar UserService.jar
ENTRYPOINT [ "java", "-jar", "UserService.jar"]
```

![](https://velog.velcdn.com/images/develing1991/post/de2606ee-936b-4abd-80ba-de71a257a5a9/image.png)

- 캐시 사용 모습

![](https://velog.velcdn.com/images/develing1991/post/74bf8499-bb38-4fe3-819a-52627eaf1281/image.png)




```bash
# 이미지 생성
docker build -t completed0728/user-service:1.0 .

# 실행
docker run -d --network suhan-network --name user-service -e "spring.config.import=optional:configserver:http://config-server:8888" -e "spring.rabbitmq.host=rabbitmq" -e "management.zipkin.tracing.endpoint=http://zipkin:9411/api/v2/spans" -e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" -e "logging.file=/api-logs/users-ws.log" completed0728/user-service:1.0
```

docker run -d --network suhan-network --name user-service \
-e "spring.config.import=optional:configserver:http://config-server:8888" \
-e "spring.rabbitmq.host=rabbitmq" \
-e "management.zipkin.tracing.endpoint=http://zipkin:9411/api/v2/spans" \
-e "eureka.client.serviceUrl.defaultZone=http://naming-server:8761/eureka" \
-e "logging.file=/api-logs/users-ws.log" \
completed0728/user-service:1.0

![](https://velog.velcdn.com/images/develing1991/post/41ddcd15-166e-4b28-b1e4-8a9b813ad2d4/image.png)

### 확인

![](https://velog.velcdn.com/images/develing1991/post/87ce55fd-8daf-45df-b71b-6e2e07a0ca74/image.png)

### 테스트

![](https://velog.velcdn.com/images/develing1991/post/eb70422a-264f-442f-ace0-afd238ca7756/image.png)


<br><br>

### docker compose

```bash
# 이미지 생성
docker build -t completed0728/user-service:1.0 .

# 컴포즈 업
docker compose up -d
```

```yaml
version: '3.7'
services:
  user-service:
    container_name: user-service
    image: completed0728/user-service:1.0
    environment:
      spring.rabbitmq.host: rabbitmq
      spring.config.import: optional:configserver:http://config-server:8888
      management.zipkin.tracing.endpoint: http://zipkin:9411/api/v2/spans
      eureka.client.serviceUrl.defaultZone: http://naming-server:8761/eureka
      logging.file: /api-logs/users-ws.log
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```

### 테스트

![](https://velog.velcdn.com/images/develing1991/post/8260470d-3775-4f55-b14b-9f5de9b4ddc0/image.png)
