### 도커 허브

https://hub.docker.com/_/rabbitmq/


### 실행 방법 1. bash
```bash
docker run -d --name rabbitmq --network suhan-network -p 15672:15672 -p 5672:5672 -p 15671:15671 -p 5671:5671 -p 4369:4369 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin123 rabbitmq:management
```

![](https://velog.velcdn.com/images/develing1991/post/6c1800b1-2383-442c-8c66-e3e524014dfe/image.png)


### 실행 방법 2. docker-compose
```yaml
version: '3.7'
services:
  rabbitmq:
    container_name: rabbitmq
    image: rabbitmq:management
    ports:
      - "5671:5671"
      - "5672:5672" # rabbitmq amqp port
      - "4369:4369"
      - "15671:15671"
      - "15672:15672" # ui manage port
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin123
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
![](https://velog.velcdn.com/images/develing1991/post/4a23ccdf-4190-4580-ab22-a3da086ac85f/image.png)

### 접속 테스트

![](https://velog.velcdn.com/images/develing1991/post/6c3c1f59-a2e3-41cf-aa85-bff1ffc05cc3/image.png)


