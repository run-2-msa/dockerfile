![](https://velog.velcdn.com/images/develing1991/post/becc6e6c-875c-4df8-891d-da68d9be2bb0/image.png)

![](https://velog.velcdn.com/images/develing1991/post/6f93cf48-261c-41e3-8860-9cd4de50fb3a/image.png)

- 공개키 복사 옮기기
- 도커파일 작성 (공개키 도커 내부 루트에 복사)
- 버전 1.0으로 변경

<br><br>

![](https://velog.velcdn.com/images/develing1991/post/2174bee7-a4c2-4f62-9530-d6a35db417ad/image.png)

- 공개키를 도커의 루트에 복사했기 때문에 키 찾을 경로 루트로 변경
  ![](https://velog.velcdn.com/images/develing1991/post/7988d53a-e2b5-4209-acae-88b2880c5be2/image.png)
- local-repo-config의 uri를 로컬에서 github로 변경 private 프로젝트니 username,과 password(토큰) 작성
  ![](https://velog.velcdn.com/images/develing1991/post/1e5307df-f299-4f4a-8390-9b61b841ef9f/image.png)
  토큰 발급시 어떤 레포지토리인지 잘확인...


### application-encrypt.yaml
```yaml
encrypt:
  key-store:
    # location: file:///D:/msa/keystore/client.jks
    location: file:/client.jks # 컨테이너 루트에 복사한 키파일
    password: test1234
    alias: client_public # 공개키 자신의 private_key로 복호화
    secret: test1234
```
### application-config.yaml
```yaml
spring:
  cloud:
    config:
      server:
        git:
#          uri: file:///D:/msa/local-repo-config # http://localhost:8888/common/default
          uri: https://github.com/run-2-msa/local-repo-config.git # public은 username, password 필요 없음
          username: Develing1991 # private github일 때 Develing1991
          password: [ token ] # private github일 때 토큰 값
```


### config-server -> jar 파일로 컴파일

- 도커 파일 client.jks 키파일 경로와 내 로컬의 경로가 다르기 때문에 (못 찾아서)
- 빌드 중에 테스트 까지 하면 어차피 컴파일 에러 나니까... 테스트는 스킵
```
mvn clean compile package -DskipTests=true
```
![](https://velog.velcdn.com/images/develing1991/post/7b37c03d-ccb1-42d9-9e43-21b2c6a1353a/image.png)


<br><br>

### Dockerfile 빌드
```javascript
FROM openjdk:17-ea-17-jdk-slim
VOLUME /tmp
COPY client.jks client.jks
COPY target/config-server-1.0.jar ConfigServer.jar
ENTRYPOINT [ "java", "-jar", "ConfigServer.jar"]
```

```
docker build -t completed0728/config-server:1.0 .
```

![](https://velog.velcdn.com/images/develing1991/post/969be7d9-d811-4523-9594-6def32bfd284/image.png)

<br><br>

### 생성한 도커 이미지 확인
```
docker images
```
![](https://velog.velcdn.com/images/develing1991/post/66a0dceb-1fe8-4e7f-914b-5d571b2c1484/image.png)


<br><br>

### rabbitmq 확인 사항

![](https://velog.velcdn.com/images/develing1991/post/4819a8e2-c3a4-45cd-90c7-8bf0634542ff/image.png)


- 컨테이너 내부 이기 때문에 `localhost`대신 `rabbitmq` 컨테이너가 할당 받은 `rabbitmq`의 Ip address `172.18.0.2`를 사용해도 되지만
- 결국 이 IP address도 변경될 수 있기 때문에 `rabbitmq`이름으로 사용할 것.
  실행 커맨드 옵션에 직접 지정 `-e "spring.rabbitmq.host=rabbitmq"`

![](https://velog.velcdn.com/images/develing1991/post/790868a9-ac38-4a97-8078-753c2995fecc/image.png)

** docker run**

- docker run -d -p 8888:8888 --network suhan-network -e "spring.rabbitmq.host=rabbitmq" -e "spring.profiles.active=default" --name config-server completed0728/config-server:1.0

```bash
docker run -d -p 8888:8888 --network suhan-network -e "spring.rabbitmq.host=rabbitmq" -e "spring.profiles.active=default" --name config-server completed0728/config-server:1.0
```

<br><br>

![](https://velog.velcdn.com/images/develing1991/post/095d611f-86fb-4538-80c6-4e43a60c23ca/image.png)

docker ps -a 또는 데스크탑 확인
<br><br>


![](https://velog.velcdn.com/images/develing1991/post/d2af1594-b00b-4db2-afcb-60eb55fa6448/image.png)

네트워크 확인
```
docker network inspect suhan-network
```

<br><br>

![](https://velog.velcdn.com/images/develing1991/post/4289b6e4-193a-433e-b1ec-5a41467c124a/image.png)

로그 확인 (정상 실행)
```
docker logs config-server
```

<br><br>

### config 데이터 확인

![](https://velog.velcdn.com/images/develing1991/post/190b0146-b510-40a8-ab44-baf0e8b3c8e6/image.png)

<br><br>

## 실행 방법 2. docker-compose
- 저장소 저장 까지는 아니더라도 ** 일단 기본 build해서 이미지가 있어야 함 **
### Dockerfile
- `COPY client.jks client.jks 삭제`
```javascript
FROM openjdk:17-ea-17-jdk-slim
VOLUME /tmp
COPY target/config-server-1.0.jar ConfigServer.jar
ENTRYPOINT [ "java", "-jar", "ConfigServer.jar"]
```
```java
docker build -t completed0728/user-service:1.0 . # 빌드
// docker push completed0728/config-server:1.0 # 업로드
// docker pull completed0728/config-server:1.0 # 다운로드
```

### docker-compose.yaml
- volumes로 로컬 pc에 있는 client.jks를 /경로에 사용하게 추가
- 이렇게 compose.yaml로 작성 하다보니 드는 생각이..
  도커 이미지 자체는 도커 허브에 올릴 수도 있기 때문에(private 일지라도..)
  언제나 키는 내 로컬 pc에서 끌어다 쓰는게 보안적으로 유리한거 같다
```yaml
version: '3.7'
services:
  config-server:
    container_name: config-server
    image: completed0728/config-server:1.0
    ports:
      - "8888:8888"
    environment:
      spring.rabbitmq.host: rabbitmq
      spring.profiles.active: default
#    depends_on:
#      - rabbitmq
    volumes:
      - D:\msa\keystore\client.jks:/client.jks # <- 이 방법 나름 굿
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```
```
docker compose up -d
```

### 커스텀 브릿지 네트워크 확인

```bash
docker network inspect suhan-network
```

![](https://velog.velcdn.com/images/develing1991/post/7fa75528-a8fd-45f8-9cc7-4570c65adc08/image.png)

<br><br>

### 컨테이너 확인

- 차이점을 발견했는데 docker run 명령어와 다른 점은 docker compose는 여러 이미지를 묶다보니
  Docker Desktop 자체에서도 표현을 아코디언 형식으로 보여준다

![](https://velog.velcdn.com/images/develing1991/post/3f6d6f5f-4d4f-44e0-8f61-6779e4d2884b/image.png)


<br><br>
### 로그 확인

![](https://velog.velcdn.com/images/develing1991/post/6c27647f-93af-4935-8759-7da7ec5956d1/image.png)

### config 데이터 확인

![](https://velog.velcdn.com/images/develing1991/post/190b0146-b510-40a8-ab44-baf0e8b3c8e6/image.png)
