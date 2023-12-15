### Dockerfile

![](https://velog.velcdn.com/images/develing1991/post/7d676261-97e0-46e3-962f-c3c7d1e56f55/image.png)

도커파일 생성 및 버전 1.0 변경

<br><br>

![](https://velog.velcdn.com/images/develing1991/post/03107a24-e24f-4202-acbc-777b4ee85535/image.png)

- mvn clean compile package -DskipTests=true
  <br><br>

![](https://velog.velcdn.com/images/develing1991/post/43cd3339-a525-4101-90e6-8a4742321ef4/image.png)

- docker build -t completed0728/naming-server:1.0 .
  <br><br>
  생략
  // docker images
  // docker push completed0728/naming-server:1.0
  // docker images rmi naming-server:1.0
  // docker pull completed0728/naming-server:1.0
  <br><br>


#### rabbitmq, config-server 순으로 기동 된 상태에서

- docker run -d -p "8761:8761" --network suhan-network -e "spring.config.import=optional:configserver:http://config-server:8888" --name naming-server completed0728/naming-server:1.0

```bash
docker run -d -p "8761:8761" --network suhan-network -e "spring.config.import=optional:configserver:http://config-server:8888" --name naming-server completed0728/naming-server:1.0
```

![](https://velog.velcdn.com/images/develing1991/post/dcd5d90a-57c7-4689-a322-924c12818058/image.png)

<br><br>

- docker network inspect suhan-network

![](https://velog.velcdn.com/images/develing1991/post/30e99e30-6a1e-4d83-b11a-4c611be430d6/image.png)


### 테스트

![](https://velog.velcdn.com/images/develing1991/post/6e9afa4d-30fb-40ed-a9ac-3c4c0c294304/image.png)


### docker compose

```yaml
version: '3.7'
services:
  naming-server:
    container_name: naming-server
    image: completed0728/naming-server:1.0
    ports:
      - "8761:8761"
    environment:
      spring.config.import: optional:configserver:http://config-server:8888
    #    depends_on:
    #      - config-server
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```