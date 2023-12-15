
### Dockerfile

- docker build -t completed0728/mysql:1.0 .

- docker run -d -p "3306:3306" --network suhan-network --restart=always -v D:\MYSQL:/var/lib/mysql --name mysql completed0728/mysql:1.0 --lower_case_table_names=1 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

```bash
docker run -d -p "3306:3306" --network suhan-network --restart=always --name mysql completed0728/mysql:1.0 --lower_case_table_names=1 --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

- docker network inspect suhan-network

![](https://velog.velcdn.com/images/develing1991/post/6fda4e0b-64b3-45e3-9abc-a998f2afaf5e/image.png)
### docker compose

- docker compose up -d

```yaml
version: "3.7"
services:
  mysql:
    image: mysql:8.0.26
    restart: always
    command:
      - --lower_case_table_names=1
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      - MYSQL_DATABASE=msa
      - MYSQL_ROOT_PASSWORD=root1234!!
      - TZ=Asia/Seoul
    volumes:
      - D:\MYSQL:/var/lib/mysql
    networks:
      my-network:
networks:
  my-network:
    external: true
    name: suhan-network
```