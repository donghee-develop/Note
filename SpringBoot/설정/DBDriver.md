```yml
#DB
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://${database-local-url}/dbName
    username: ${database-local-username}
    password: ${database-local-password}

  jpa:
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        show_sql: true
        format_sql: true
        highlight_sql: true
```

```yml
# JPA
spring:
  jpa:
    open-in-view: false
    hibernate:
      ddl-auto: validate
    properties:
      hibernate.default_batch_fetch_size: 100
```

```yaml
# JWT
jwt:
  access-token:
    secret: jsdhfgsddfaskdfgkasdghfjkadkjfagdjadfgasjdfgaskdjfgkasjdasdfhjasdfg
    # 7일
    expire: 604800000
  refresh-token:
    secret: jsdhfgsddfaskdfgkasdghfjkadkjfagdjadfgasjdfgaskdjfgkasjdasdfhjasdfg
    # 7일
    expire: 604800000

---
spring.config.activate.on-profile: prod
jwt:
  access-token:
    expire: 3600000
  refresh-token:
    secret: ${refresh-token-prod-secret}
    expire: 604800000
```

```yaml
# redis
spring:
  data:
    redis:
      host: ${redis-local-url}
      port: 6379
      timeout: 3000
      password: ${redis-local-password} # 필수 아님
```