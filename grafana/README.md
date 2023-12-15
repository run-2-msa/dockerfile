https://grafana.com/grafana/download?pg=get&platform=docker&plcmt=selfmanaged-box1-cta1

### dockerfile

- 그라파나.. 대시보드 값 셋팅 매번 하기 힘들기 때문에 volume으로 내꺼 마운트 한다

- docker run -d -p 3000:3000 --network suhan-network -v D:/grafana/data:/var/lib/grafana --name grafana grafana/grafana

<br><br>

- docker network inspect suhan-network

![](https://velog.velcdn.com/images/develing1991/post/9a72f0ca-733a-40aa-90d1-fef1aad3ea45/image.png)

### docker compose

- docker compose up -d

```yaml
version: '3.7'
services:
  grafana:
    container_name: grafana
    image: grafana/grafana
    ports:
      - "3000:3000"
    volumes:
      - D:\grafana\data:/var/lib/grafana
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```

### 그라파나 접속 및 데이터소스 변경
- localhost -> prometheus

![](https://velog.velcdn.com/images/develing1991/post/09dc77f1-fc04-44ec-a4c9-6cdec720787c/image.png)

### 그라파나 대시보드 접속 확인
![](https://velog.velcdn.com/images/develing1991/post/493f1917-e1f2-46bc-a277-0de4f8666d67/image.png)

