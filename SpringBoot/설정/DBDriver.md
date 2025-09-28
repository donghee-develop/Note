# JPA + DB
```yml
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

# JPA
```yml
spring:
  jpa:
    open-in-view: false
    hibernate:
      ddl-auto: validate
    properties:
      hibernate.default_batch_fetch_size: 100
```

# JWT (시크릿 생성 사이트)
```yaml
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

# 레디스
```yaml
spring:
  data:
    redis:
      host: ${redis-local-url}
      port: 6379
      timeout: 3000
      password: ${redis-local-password} # 필수 아님
```

# mybatis
```yaml
mybatis:
  # mybatis-config.xml 위치
  config-location: classpath:mybatis/configuration.xml
  
  
  # Config (xml로 가능)
  type-aliases-package: com.example.board.domain
  mapper-locations: classpath:mybatis/mapper/**/*.xml
  configuration:
    map-underscore-to-camel-case: true # 대소문자 구분 없이 매핑
    cache-enabled: false # 캐시 사용 시 SqlSession 값 공유, 사용 시 데이터 동기화 잘 해야 함
    lazy-loading-enabled: true
    multiple-result-sets-enabled: true # 프로시저 관련 동시 리턴
    use-generated-keys: true
    default-executor-type: SIMPLE
    jdbc-type-for-null: NULL


```
