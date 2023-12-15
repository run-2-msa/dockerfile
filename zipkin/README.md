### dockerfile

https://zipkin.io/pages/quickstart.html

- docker run -d -p 9411:9411 --network suhan-network --name zipkin openzipkin/zipkin

![](https://velog.velcdn.com/images/develing1991/post/a8148b37-8c05-480f-9968-fc6bcb731617/image.png)

### docker compose

- docker compose up -d

```yaml
version: '3.7'
services:
  zipkin:
    container_name: zipkin
    image: openzipkin/zipkin
    ports:
      - "9411:9411"
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```
