# 캐싱
- 캐싱을 통해 자주 사용하는 조회에 대하여 검색 속도를 높일 수 있다. 그 방법 중에서 인메모리 캐시 사용과 레디스를 비교하고자 한다. 
속도 면에서는 인메모리 > 레디스 > 캐싱 없음 순으로 빠르다. 인메모리가 가장 빠른 이유는 레디스의 경우는 추가적으로 Redis 서버와의 통신이 필요하기에 외부를 한 번 거치지만 인메모리는 JVM 내부 저장 공간에 저장하기에 속도 면에서 가장 빠르다.



## 레디스는 속도가 더 느린데 왜 사용될까?
1. 분산 서버 환경에서 인스턴스 공유가 가능
2. 다양한 자료구조 방식 (Memcached는 String 타입만 지원)

### 사용법
- yaml 비밀번호는 선택값
```yaml
spring.data.redis.port=6379
spring.data.redis.host=localhost
spring.data.redis.password=1234
```

- config
```java
@Configuration
public class RedisConfig {
    @Value("${spring.data.redis.host}")
    private String host;

    @Value("${spring.data.redis.port}")
    private int port;

    @Value("${spring.data.redis.password}")
    private String redisPassword;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration config = new RedisStandaloneConfiguration(host, port);
        config.setPassword(redisPassword);
        return new LettuceConnectionFactory(config);
    }
    @Primary
    @Bean
    public RedisCacheManager redisCacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(10))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        return RedisCacheManager.builder(connectionFactory)
                .cacheDefaults(config)
                .build();
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        return template;
    }

}
```

인메모리 캐시 세팅
```java
@Configuration
@EnableCaching
public class CacheConfig {
    // ConcurrentMap + Caffeine 캐시 많이 씀

    @Bean
    public CacheManager caffeineCacheManager() {
        // 심플 써야 세팅 편함
        SimpleCacheManager cacheManager = new SimpleCacheManager();

        // 키워드를 카운팅하는 캐시
        CaffeineCache keywordCountMap = new CaffeineCache(
                "keywordCountMap",
                Caffeine.newBuilder()
                        .maximumSize(10000)
                        .build()
        );

        // 인기 검색어를 조회하는 캐시
        CaffeineCache popularKeywords = new CaffeineCache(
                "popularKeywords",
                Caffeine.newBuilder()
                        .maximumSize(10) // days 값 기준으로 최대 10개
                        .expireAfterWrite(5, TimeUnit.MINUTES) // TTL 5분
                        .build()
        );

        // 현재 캐시에 저장된 키워드 목록
        CaffeineCache keywordKeySet = new CaffeineCache(
                "keywordKeySet",
                Caffeine.newBuilder()
                        .maximumSize(1)
                        .build()
        );


        CaffeineCache businessSearchCacheInMemory = new CaffeineCache(
                "businessSearchCacheInMemory",
                Caffeine.newBuilder()
                        .maximumSize(10000)
                        .expireAfterWrite(30, TimeUnit.MINUTES) // 예: 30분 TTL
                        .build()
        );



        cacheManager.setCaches(Arrays.asList(keywordCountMap, popularKeywords, keywordKeySet, businessSearchCacheInMemory));
        return cacheManager;
    }
}
```

- 만약 동시에 두 개의 캐싱을 사용하는 경우 빈 등록 문제가 발생한다. @Primary 어노테잇녀이나 아래와 같이 @Cacheable 어노테이션을 통해 해결한다.
```java
@Cacheable(
        value = "businessSearchCacheInMemory",
        key = "#keyword + ':' + #page + ':' + #size",
        condition = "#keyword != null and !#keyword.isBlank()",
        cacheManager = "caffeineCacheManager"
)
```