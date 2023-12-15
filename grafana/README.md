https://hub.docker.com/r/prom/prometheus

### prometheus.yml 디렉토리 이동
![](https://velog.velcdn.com/images/develing1991/post/bac69d92-ba62-4be5-b1fc-a6211459cdae/image.png)

- D:\prometheus 디렉토리로 변경
  job 등록했던 prometheus.yml만 사용

<br><br>

### prometheus.yml 수정

- localhost에서 gateway-service 네이밍으로 변경

![](https://velog.velcdn.com/images/develing1991/post/52b8484c-868e-4d6d-b0a4-061a5fbe8ba4/image.png)

![](https://velog.velcdn.com/images/develing1991/post/32aa725c-467f-48f2-b151-3bd37b9a5004/image.png)

<br><br>

### dockerfile

- docker run -d -p 9090:9090 -v D:/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml --network suhan-network --name prometheus prom/prometheus

<br><br>

- docker network inspect suhan-network

![](https://velog.velcdn.com/images/develing1991/post/be1b781c-1ba7-4c37-9954-1fae82be1779/image.png)


### 볼륨 마운트 내용 확인

![](https://velog.velcdn.com/images/develing1991/post/cf22f9cc-23e0-4084-b74d-5f8ff870c953/image.png)


<br><br>
### docker compose

- docker compose up -d

```yaml
version: '3.7'
services:
  prometheus:
    container_name: prometheus
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - D:\prometheus\prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```
