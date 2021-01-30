### spring 中的分布式锁

[官方文档](https://docs.spring.io/spring-integration/docs/5.3.1.RELEASE/reference/html/redis.html#redis-lock-registry)

pom.xml

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-integration</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.integration</groupId>
    <artifactId>spring-integration-redis</artifactId>
</dependency>
```

配置文件

```java
spring:
  redis:
    host: redis.dev.victorplus.cn
    port: 6379
    password: XXXXXXXXXXX
    timeout: 1M
    database: 4
```

java配置

```java
@Bean
public RedisLockRegistry redisLockRegistry(RedisConnectionFactory redisConnectionFactory) {
    return new RedisLockRegistry(redisConnectionFactory, "boss-creidt2-query", DEFAULT_EXPIRE_AFTER.toMillis());
}
```

调用代码

```java
@Resource
private RedisLockRegistry redisLockRegistry;


Lock lock = redisLockRegistry.obtain(key);
try {
    if (lock.tryLock()) {    
        // todo 这里这处理业务逻辑
    } else {
        throw new RuntimeException("获取锁失败: "+key);
    }
} finally {
    lock.unlock();
}
```

