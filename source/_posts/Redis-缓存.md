title: Redis 缓存
categories: 其它
tags: [微服务,Redis,JAVA]
date: 2022-01-04 15:31:00
---
# 缓存

为了系统性能的提升，一般会将部分数据加入缓存中，加速访问。而 db 承担数据落盘工作。

 哪些数据适合放入缓存？

- 即时性、数据一致性要求不高的
- 访问量大而且更新频率不高的数据（读多，写少）



![请求流程](http://qiniu-note-image.ctong.top/note/images/202112301501739.png)



# 整合Redis

在 SpringBoot 工程中引入 Redis 场景

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

在配置文件中配置 Redis，Redis 的属性配置都在 `org.springframework.boot.autoconfigure.data.redis.RedisProperties`

```yml
spring:
  # 配置 redis
  redis:
    host: 172.16.34.128
```

使用 `StringRedisTemplate` 操作 redis

```java
@SpringBootTest
@Slf4j
@DisplayName("Redis 测试")
public class RedisTest {

  @Autowired
  private StringRedisTemplate redisTemplate;

  @Test
  @DisplayName("redis 操作简单值")
  void opsForValueTest() {
    // 操作简单值
    ValueOperations<String, String> ops = redisTemplate.opsForValue();
    // 保存
    ops.set("hello", "world_" + UUID.randomUUID());
    // 查询
    String hello = ops.get("hello");
    log.info(hello); // world_9f7fb10f-23fa-424d-a6e6-2de794852fed
  }
}
```



# 缓存穿透

- 缓存穿透是指查询一个一定不存在的数据，由于缓存是不命中，将去查询数据库，但是数据库也无此记录，我们没有将这次的查询 null 写入缓存，这将导致这个不存在的数据每次请求都要去存储层查询，失去了缓存的意义。
- 该问题存在极大风险，如果利用不存在的数据进行攻击，数据库瞬时压力增大，最终导致崩溃
- 可以将一个null结果缓存，并加入短暂的过期时间。

```java
public Map<String, List<Catalog2Vo>> getCatalogJson() {
  // 从缓存中获取数据
  String catalogJson = redisTemplate.opsForValue().get("catalogJson");
  if (!StringUtils.hasText(catalogJson)) {
    // 从数据库中获取数据
    Map<String, List<Catalog2Vo>> catalogJsonFormDb = getCatalogJsonFormDB();
    // 解决缓存穿透
    if (catalogJsonFormDb == null || catalogJsonFormDb.isEmpty()) {
      redisTemplate.opsForValue().set("catalogJson", "{}", 30000);
      return new HashMap<>(0);
    } else {
      String jsonString = JSON.toJSONString(catalogJsonFormDb);
      redisTemplate.opsForValue().set("catalogJson", jsonString);
      return catalogJsonFormDb;
    }
  }
  // 将缓存的数据转为对象返回
  return JSON.parseObject(catalogJson, new TypeReference<Map<String, List<Catalog2Vo>>>(){});
}
```



# 缓存雪崩

- 缓存雪崩是指在我们设置缓存时 key 采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到 DB，DB 瞬时压力过重导致崩溃。
- 在原有的实效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，很难引发集体失效的事件



# 缓存击穿

- 对一些设置了过期时间的 key，如果这些 key 可能会在某些时间点被超高并发访问，那么这就是一种非常 “热点” 的数据。
- 如果这个 key 在大量请求同时进来前正好失效，那么所有对这个 key 的数据查询都落到 DB，这被称为缓存击穿
- 如果要解决这个问题，那么可以加锁，大量并发只让一个去查，其他人等着，查到后释放锁，其他人获取到锁，先查缓存就会有数据，不用去db。

```java
/**
 * getCatalogJsonLock 锁对象
 */
private final Object getCatalogJsonLock = new Object();

@Override
public Map<String, List<Catalog2Vo>> getCatalogJson() {
  // 从缓存中获取数据
  String catalogJson = redisTemplate.opsForValue().get("catalogJson");
  if (!StringUtils.hasText(catalogJson)) {
    // 防止击穿
    synchronized (getCatalogJsonLock) {
      // 确认缓存数据
      String confirmCache = redisTemplate.opsForValue().get("catalogJson");
      if (!StringUtils.hasText(confirmCache)) {
        // 从数据库中获取数据
        Map<String, List<Catalog2Vo>> catalogJsonFormDb = getCatalogJsonFormDB();
        // 解决缓存穿透
        if (catalogJsonFormDb == null || catalogJsonFormDb.isEmpty()) {
          redisTemplate.opsForValue().set("catalogJson", "{}", 1, TimeUnit.DAYS);
          return new HashMap<>(0);
        } else {
          String jsonString = JSON.toJSONString(catalogJsonFormDb);
          redisTemplate.opsForValue().set("catalogJson", jsonString);
          return catalogJsonFormDb;
        }
      }
    }
  }
  // 将缓存的数据转为对象返回
  return JSON.parseObject(catalogJson, new TypeReference<Map<String, List<Catalog2Vo>>>() {
  });
}
```

在分布式系统中，我们应该使用分布式锁，因为本地锁只能锁住当前进程

![分布式锁](http://qiniu-note-image.ctong.top/note/images/202112301707003.png)



# 分布式锁

在发送 `set` 命令时携带 `NX` 选项可以进行占锁，如果拿不到锁则返回 `nil` 。

> `SET KEY VAL NX EX` key 不存在的情况下才设置密钥

```java
ValueOperations<String, String> ops = redisTemplate.opsForValue();
// 占用分布式锁 set EX NX
Boolean lock = ops.setIfAbsent("product_catalog_lock", "lock val");

if(lock) {
  // 抢到分布式锁
} else {
  // 没抢到锁，使用自旋方式重试
  Thread.sleep(100);
  return getCatalogJsonWithRedisLock();// 自旋代码
}

```

防止当前抢到锁的服务 “断水断电” 后导致死锁问题，我们应该给这个锁一个短暂的过期时间。

```java
redisTemplate.expire("product_catalog_lock", 30, TimeUnit.SECONDS);
```

为了防止程序在设置过期时间代码执行前 “断水断电”，我们需要的是抢占锁和设置过期时间这是一个原子操作。所以需要在抢锁时设置而不是另外设置过期时间。

```java
Boolean lock = ops.setIfAbsent("product_catalog_lock","lock val", 30, TimeUnit.SECONDS);
```

当业务完成后释放分布式锁，其实就是把当前存在的 `KEY` 删了。

```java
redisTemplate.delete("product_catalog_lock");
```

但是这样还有一个问题，这个问题在官网就给出了解释和答案

> **注意:** 下面这种设计模式并不推荐用来实现redis分布式锁。应该参考[the Redlock algorithm](http://redis.io/topics/distlock)的实现，因为这个方法只是复杂一点，但是却能保证更好的使用效果。
>
> 命令 `SET resource-name anystring NX EX max-lock-time` 是一种用 Redis 来实现锁机制的简单方法。
>
> 如果上述命令返回`OK`，那么客户端就可以获得锁（如果上述命令返回Nil，那么客户端可以在一段时间之后重新尝试），并且可以通过[DEL](http://www.redis.cn/commands/del.html)命令来释放锁。
>
> 客户端加锁之后，如果没有主动释放，会在过期时间之后自动释放。
>
> 可以通过如下优化使得上面的锁系统变得更加鲁棒：
>
> - 不要设置固定的字符串，而是设置为随机的大字符串，可以称为token。
> - 通过脚步删除指定锁的key，而不是[DEL](http://www.redis.cn/commands/del.html)命令。
>
> 上述优化方法会避免下述场景：a客户端获得的锁（键key）已经由于过期时间到了被redis服务器删除，但是这个时候a客户端还去执行[DEL](http://www.redis.cn/commands/del.html)命令。而b客户端已经在a设置的过期时间之后重新获取了这个同样key的锁，那么a执行[DEL](http://www.redis.cn/commands/del.html)就会释放了b客户端加好的锁。
>
> 解锁脚本(Lua)的一个例子将类似于以下：
>
> ```lua
> if redis.call("get",KEYS[1]) == ARGV[1]
> then
>     return redis.call("del",KEYS[1])
> else
>     return 0
> end
> ```
>
> 这个脚本执行方式如下：
>
> EVAL …script… 1 resource-name token-value

其实就是怕我们设置的过期时间短，而业务代码比预期的久，导致业务没完成锁就过期并被别的线程占用，当业务执行完后删除锁，把别人的锁给删了。

这种情况需要将锁保存的值设置为一个随机的大长字符串「token」，删除时判断当前锁是不是自己的，如果是再删。这判断和删除也需要是一个原子操作，因为 IO 操作是需要时间的。

在抢占锁时设置一个 UUID 

```java
String uuid = UUID.randomUUID().toString();
Boolean lock = ops.setIfAbsent("product_catalog_lock", uuid, 30, TimeUnit.SECONDS);
```

删除锁通过Lua脚步达到原子操作

```java
String luaScript = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
DefaultRedisScript<Long> redisScript = new DefaultRedisScript<>(luaScript, Long.class);
redisTemplate.execute(redisScript, Collections.singletonList("product_catalog_lock"), uuid);
```


