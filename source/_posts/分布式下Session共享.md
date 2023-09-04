title: 分布式下Session共享
categories: JAVA
tags: [Spring,SpringSession,随笔,JAVA,Session]
date: 2022-02-14 23:22:52
---
# 分布式下Session共享

首先聊聊session和cookie，session对象存储在服务器端节点内存中，cookie存储在客户端浏览器中。一般是客户端请求服务器，服务器端生成session对象，将session对象存储在jvm内存中，并且在响应头中放入sessionId响应给客户端，客户端收到响应后，将sessionid存储在本地。当浏览器第二次请求时会将本地cookie中存储的seesionId通过请求头的方式传递给服务器，这样服务器和客户端就能保持会话信息啦！如下图

![img](https://qiniu-note-image.ctong.top/note/images/202202141813781.jpg)

![img](https://qiniu-note-image.ctong.top/note/images/202202141813789.jpg)

那么为什么会出现分布式session问题呢，为了提高服务器端的负载能力，后台一般将服务器节点做集群，通过ngnix通过轮询的方式转发到目标服务器。打个比方，当浏览器首次访问A服务器生成session对象，然后在访问生成的session对象，如果正好被ngnix转发到了A服务器，那么没问题可以获取到session对象，如果不巧请求被转发到B服务器，由于之前生成的session对象在A服务器，B服务器根本没有生成session对象，很自然访问不到session对象。

> 上文来自[知乎https://zhuanlan.zhihu.com/p/95687710#:~:text=%E5%88%86%E5%B8%83%E5%BC%8Fsessi,ssion%E5%AF%B9%E8%B1%A1%E3%80%82](https://zhuanlan.zhihu.com/p/95687710#:~:text=%E5%88%86%E5%B8%83%E5%BC%8Fsessi,ssion%E5%AF%B9%E8%B1%A1%E3%80%82)

## 解决方案

### sessions 复制

session复制的优点是，web-server原生支持，只需要修改配置文件

而他的缺点也很明显：

- sessiong同步需要数据传输，占用大量网络带宽，降低了服务器群的业务处理能力
- 任意一台 web-server 保存的数据都是所有 web-server 的 session 总和，受到内存限制无法水平扩展更多的 web-server
- 大型分布式集群情况下，由所有 web-server 都全量保存数据，所以此方案不可取。

![截屏2022-02-14 下午7.05.56](https://qiniu-note-image.ctong.top/note/images/202202141905270.png)

### 客户端存储

客户端存储，其优点是，服务器不需要存储 session，用户保存自己的session信息到cookie中。节省服务端资源。

缺点是：

- 每次发起请求，都需要携带cookie中的完整信息，浪费网络带宽。
- session 数据放在 cookie 中，cookie 有长度**限制4k**，不能保存大量信息
- session 数据放在 cookie 中，存在泄露、篡改、盗取等安全隐患

![截屏2022-02-14 下午7.09.41](https://qiniu-note-image.ctong.top/note/images/202202141909184.png)

### hash一致性

优点：

- 只需要秀海 nginx 配置，不需要修改应用代码
- 负载均衡，只要hash属性值的分布是均匀的，多台 web-server 的负载是均匀的
- 可以支持 web-server 水平扩展

缺点：

- session 还是存在 web-server 中的，所以 web-server 重启可能导致部分 session 丢失，影响业务，如部分用户需要重新登录。
- 如果 web-server 水平扩展，rehash 后 session 重新分布，也会有一部分用户路由到不正确的 session

以上缺点问题不是很大，因为 session 本来都是有效期的，所以这两种反向代理的方式可以使用。

![截屏2022-02-14 下午7.21.57](https://qiniu-note-image.ctong.top/note/images/202202141922452.png)

### 统一存储

优点：

- 没有安全隐患
- 可以水平扩展，数据库/缓存水平切分即可
- Web-server 重启或者扩容都不会有 session 丢失

不足：

- 增加了一次网络调用，并且需要修改应用代码。如果将所有的 getSession 方法替换为从 Redis 查询数据的方式，Redis 获取数据比本地内存慢很多
- 上面缺点可以用 SpringSession 完美解决

![截屏2022-02-14 下午7.36.04](https://qiniu-note-image.ctong.top/note/images/202202141936301.png)

### SpringSession

SpringSession 的好处是，它替换掉了 Servlet 默认的 `HttpSession` 并且实现了原生的 `HttpSession` 。也就是说我们的代码不需要修改，只需要添加几个SpringSession的配置就可以使用它。

默认的 session 是在本地保存用户session（Map）。SpringSession是使用指定的store进行存储，我们可以指定redis、mongodb等。

![截屏2022-02-14 下午8.02.56](https://qiniu-note-image.ctong.top/note/images/202202142002916.png)

## SpringSession

## 整合

在对应服务中引入 SpringSession

```xml
<dependency>
  <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

之后再添加这个配置到你的配置文件中 `application.yaml/application.properties` ，用于指定存储服务类型

```properties
spring.session.store-type=redis
```

可以通过配置进行定制，如过期时间、存储前缀等：

```properties
server.servlet.session.timeout= # Session timeout. If a duration suffix is not specified, seconds is used.
spring.session.redis.flush-mode=on_save # Sessions flush mode.
spring.session.redis.namespace=spring:session # Namespace for keys used to store sessions
```

最后开启 SpringSession

```java
@EnableRedisHttpSession
@SpringBootApplication
public class GulimallAuthServerApplication;
```

## 使用

就正常使用我们的Session就可以了，因为SpringSession实现了HttpSession并替换掉 Servlet 默认的。

```java
@RequestMapping("/gitee/success")
public String giteeAuth(HttpSession session) throws Exception {
    R r = memberServerFeign.giteeLogin(giteeUserInfo);
    ...
    MemberRespVo memberData = r.getData(new TypeReference<MemberRespVo>() {});
    session.setAttribute("loginUser", memberData);

    log.info("登录成功 ====>> {}", memberData);

    return "redirect:xxx;
}
```

![截屏2022-02-14 下午9.16.36](https://qiniu-note-image.ctong.top/note/images/202202142116653.png)

##  注意

1. SpringSession 默认使用jdk存储，也就是java的序列化，如果存储的对象没有实现 `Serializable` 接口，那么会抛出 `SerializationException` 异常。

   ```java
   public class MemberRespVo implements Serializable {
      private static final long serialVersionUID = 1L;
   }
   ```

2. 在分布式系统下，session 是无法跨域取值的，session 设置时默认作用域是当前服务器域名，别的服务是无权使用的。需要将 cookie 中的 jsessionid 设置到父域名，提升作用域。

   ```java
   @Bean
   public CookieSerializer cookieSerializer() {
     DefaultCookieSerializer serializer = new DefaultCookieSerializer();
     serializer.setDomainName("gulimall.com");
     serializer.setCookieName("JSESSIONID");
     return serializer;
   }
   ```

3. SpringSession默认使用的是jdk序列化，这样的话就只能用Java程序进行操作了，如果其它语言也要使用，那么我们需要修改其配置，让它默认将数据序列化为JSON格式保存到Redis。

   ```java
   @Bean
   public RedisSerializer<?> springSessionDefaultRedisSerializer() {
     return new GenericJackson2JsonRedisSerializer();
   }
   ```

