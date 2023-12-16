### depends on (order)
개별 실행 시 순서
- rabbitmq
- config-server
- naming-server
- gateway-service
- mysql
- zipkin
- kafka
- prometheus
- grafana


### 여기 부턴 순서 상관 없음
- user-service
- order-service
- catalog-service


```yaml
docker compose up -d

# after user-service restart
curl -f http://localhost:8000/actuator/busrefresh
```


### healthcheck

https://docs.docker.com/compose/compose-file/05-services/#healthcheck

https://devlifetestcase.tistory.com/88

https://superuser.com/questions/1080239/run-powershell-command-from-cmd