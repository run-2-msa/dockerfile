### confluent
https://docs.confluent.io/platform/current/platform-quickstart.html#step-2-create-ak-topics-for-storing-your-data

### confluent connect sink jdbc
https://docs.confluent.io/kafka-connectors/jdbc/current/sink-connector/overview.html

### wurstmeister/kafka-docker
https://github.com/wurstmeister/kafka-docker



### single broker
- ip address 직접 지정
```yaml
version: '3.7'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    # image: confluentinc/cp-zookeeper:latest
    container_name: zookeeper
    ports:
      - "2181:2181"
    networks:
      my-network:
        ipv4_address: 172.18.0.100
  kafka:
    container_name: kafka
    image: wurstmeister/kafka
    # image: confluentinc/cp-kafka:latest
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 172.18.0.101
      KAFKA_CREATE_TOPICS: "test:1:1"
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - zookeeper
    networks:
      my-network:
        ipv4_address: 172.18.0.101
networks:
  my-network:
    external: true
    name: suhan-network # 서브넷  172.18.0.0/255.255.0.0  / 게이트 웨이 172.18.0.1
```
docker compose up -d

![](https://velog.velcdn.com/images/develing1991/post/854a426c-1789-42e4-b69c-5ff27dbbb554/image.png)


