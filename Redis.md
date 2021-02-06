# Redis

NoSQL(NoSQL = Not Only SQL )，意即“不仅仅是SQL”，泛指非关系型的数据库。

CAP原则又称CAP定理，指的是在一个分布式系统中， Consistency /kənˈsɪstənsi/（一致性）、 Availability /əˌveɪləˈbɪləti/（可用性）、Partition tolerance /pɑːrˈtɪʃn/ /ˈtɑːlərəns/ （分区容错性），三者不可得兼。

**分区容错性（P）**：大多数分布式系统都分布在多个子网络，每个子网络就叫做一个区（partition）。分区容错的意思是，区间通信可能失败。

**一致性（C）**：在分布式系统中的所有数据备份，在同一时刻是否同样的值。意思是，写操作之后的读操作，必须返回该值。

**可用性（A）**：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。只要收到用户的请求，服务器就必须给出回应。
用户可以选择向 G1 或 G2 发起读操作。不管是哪台服务器，只要收到请求，就必须告诉用户，到底是 v0 还是 v1，否则就不满足可用性。



## 常见NoSQL数据库

* K/V键值类型：memcached，redis
* 文档类型数据库：mongodb，couchdb
* 列存储数据库类型：Cassandra，HBase
* 图关系类型数据库：Neo4j，InfoGrid 朋友圈社交，广告推荐系统

Redis 与其他 key - value 缓存产品有以下三个特点：

* Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
* Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
* Redis支持数据的备份，即master-slave模式的数据备份。



## redis持久化

Redis 为了内部数据的安全考虑，会把本身的数据以文件的形式保存到硬盘中一份，在服务器重启之后，会重新自动把硬盘的数据恢复到内存（redis）里面。数据保存到硬盘的过程就称为“持久化”。

Redis的两种持久化方式：

1. RDB
2. AOF

### 1. RDB

RDB 是 Redis 默认的持久化方案，也称为快照。在指定的时间间隔内，执行指定次数的写操作，则会将内存中的数据写入到磁盘中。即在指定目录下生成一个dump.rdb文件。Redis 重启会通过加载dump.rdb文件恢复数据。

#### 1.1 RDB核心规则配置

```shell
save 900 1
save 300 10
save 60 10000
```

说明：save <指定时间间隔> <执行指定次数更新操作>，满足条件就将内存中的数据同步到硬盘中。配置默认是 900秒内有超过一个key被修改，300秒内有超过10个key被修改以及60秒内有超过10000个key被修改，则将内存中的数据快照写入磁盘。
若不想用RDB方案，可以把 save "" 的注释打开，下面三个注释。

#### 1.2 备份文件名

默认的备份文件：

```shell
dbfilename dump.rdb
```

#### 1.3 存放目录

默认配置：

```shell
dir ./
```

#### 1.4 触发RDB快照

1. 在指定的时间间隔内，执行指定次数的写操作
2. 执行save（阻塞， 只管保存快照，其他的等待） 或者是bgsave （异步）命令
3. 执行flushall 命令，清空数据库所有数据。
4. 执行shutdown 命令，保证服务器正常关闭且不丢失任何数据。

#### 1.5 RDB 的优缺点

优点：

1. 适合大规模的数据恢复。
2. 如果业务对数据完整性和一致性要求不高，RDB是很好的选择。

缺点：

1. 数据的完整性和一致性不高，因为RDB可能在最后一次备份时宕机了。
2. 备份时占用内存，因为Redis 在备份时会独立创建一个子进程，将数据写入到一个临时文件（此时内存中的数据是原来的两倍哦），最后再将临时文件替换之前的备份文件。
   所以Redis 的持久化和数据的恢复要选择在夜深人静的时候执行是比较合理的。

### 2. AOF

Redis 默认不开启。它的出现是为了弥补RDB的不足（数据的不一致性），所以它采用日志的形式来记录每个写操作，并追加到文件中。Redis 重启的会根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作。

#### 2.1 AOF开启

redis 默认关闭，开启需要手动把no改为yes

```shell
appendonly no
```

#### 2.2 日志文件名

```shell
appendfilename "appendonly.aof"
```

#### 2.3 更新日志条件

```shell
# appendfsync always
appendfsync everysec
# appendfsync no
```

说明：

* always：同步持久化，每次发生数据变化会立刻写入到磁盘中。性能较差当数据完整性比较好（慢，安全）
* everysec：默认推荐，每秒异步记录一次（默认值）
* no：不同步

#### 2.4 AOF 的优缺点

优点：数据的完整性和一致性更高
缺点：因为AOF记录的内容多，文件会越来越大，数据恢复也会越来越慢。



## java中使用Redis

### 1. 引入redis的启动器依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 2. application.yml

```yml
spring:
  # redis的配置
  redis:
    host: localhost
    database: 1
    port: 6379
```

### 3. 配置类

```java
@Configuration
/**
 * @EnableCaching
 * 作用：开启注解式缓存
 */
@EnableCaching
public class RedisConfig {

    /**
     * 1.配置redisTemplate
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(jackson2JsonRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(jackson2JsonRedisSerializer());
        template.afterPropertiesSet();
        return template;
    }

    /**
     * 2.json序列化
     */
    @Bean
    public RedisSerializer<Object> jackson2JsonRedisSerializer() {
        //使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值
        Jackson2JsonRedisSerializer serializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper mapper = new ObjectMapper();
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        mapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(mapper);
        return serializer;
    }

    /**
     * 配置缓存管理器
     */
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {
        // 生成一个默认配置，通过config对象即可对缓存进行自定义配置
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig();
        // 设置缓存的默认过期时间，也是使用Duration设置
        config = config.entryTtl(Duration.ofMinutes(30))
                // 设置 key为string序列化
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                // 设置value为json序列化
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer()))
                // 不缓存空值
                .disableCachingNullValues();

        // 设置一个初始化的缓存空间set集合
        Set<String> cacheNames = new HashSet<>();
        cacheNames.add("default:cache");
        cacheNames.add("user:login:cache");

        // 对每个缓存空间应用不同的配置
        Map<String, RedisCacheConfiguration> configMap = new HashMap<>();
        configMap.put("default:cache", config);
        configMap.put("user:login:cache", config.entryTtl(Duration.ofMinutes(120)));

        // 使用自定义的缓存配置初始化一个cacheManager
        RedisCacheManager cacheManager = RedisCacheManager.builder(redisConnectionFactory)
                // 一定要先调用该方法设置初始化的缓存名，再初始化相关的配置
                .initialCacheNames(cacheNames)
                .withInitialCacheConfigurations(configMap)
                .build();
        return cacheManager;
    }

    @Bean
    public RedisUtil redisUtil(){
        return new RedisUtil();
    }
}
```

### 4. 使用与测试

```java
@Service
public class CacheService {

    @Cacheable(value = "user:login:cache", key = "#username", unless = "#result eq null")
    public User cacheUser(String username) {
        System.out.println("cacheUser..." + username);
        if(StringUtils.isEmpty(username)) {
            return null;
        }
        User user = new User(1, username, "123", "13012312312");
        return user;
    }
}
```

