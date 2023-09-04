title: SpringSession 基本原理
categories: Spring
tags: [Spring,Session,SpringBoot,SpringSession,JAVA,SpringCloud,原理解析]
date: 2022-02-15 06:12:21
---
# SpringSession 基本原理

1. `@EnableRedisHttpSession` 引入了一个 `RedishttpSessionConfiguration` 配置

   ```java
   @Import(RedisHttpSessionConfiguration.class)
   ```

2. 在 `RedishttpSessionConfiguration` 配置中注册了一个 `RedisIndexedSessionRepository` bean。这是一个 Redis 操作 Session 的bean。

   > org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration#sessionRepository
<!--more-->
3. `RedisHttpSessionConfiguration` 继承了 `SpringHttpSessionConfiguration` 这个配置

4. 在 `SpringHttpSessionConfiguration` 配置中注册了一个 `SessionRepositoryFilter` bean。

   > org.springframework.session.config.annotation.web.http.SpringHttpSessionConfiguration#springSessionRepositoryFilter

5. `SessionRepositoryFilter` 继承了一个 `Filter` 。由此得出，该 Bean 是一个 **Servlet 过滤器**，每个请求过来都需要经过这个过滤器，而这个过滤器调用了 `doFilterInternal` 方法。

经过以上步骤可以判断出，`doFilterInternal` 是SpringSession的核心代码。

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
      throws ServletException, IOException {
   request.setAttribute(SESSION_REPOSITORY_ATTR, this.sessionRepository);

   SessionRepositoryRequestWrapper wrappedRequest = new SessionRepositoryRequestWrapper(request, response);
   SessionRepositoryResponseWrapper wrappedResponse = new SessionRepositoryResponseWrapper(wrappedRequest,
         response);

   try {
      filterChain.doFilter(wrappedRequest, wrappedResponse);
   }
   finally {
      wrappedRequest.commitSession();
   }
}
```

## getSession

```java
@GetMapping("")
public String indexPage(HttpSession session) {}

@GetMapping("")
public String indexPage(HttpServletRequest req){
  HttpSession session = req.getSession();
} 
```

1. 调用 `getSession()` 后会调用 `SessionRepositoryRequestWrapper` 中的 `getSession(boolean)` 方法

2. 在当前 request 上下文中获取一个 session，如果有那么将这个session返回

   ```java
   HttpSessionWrapper currentSession = getCurrentSession();
   ```

3. 如果没有，那么就通过 `getRequestedSession()` 方法获取一个session。

   > org.springframework.session.web.http.SessionRepositoryFilter.SessionRepositoryRequestWrapper#getRequestedSession

   ```java
   S requestedSession = getRequestedSession();
   ```

   1. 查询sessionid，这个sessionid其实就是在 **cookie 中拿到的**

      ```java
      List<String> sessionIds = SessionRepositoryFilter.this.httpSessionIdResolver.resolveSessionIds(this);
      ```

   2. 遍历看看哪一个 sessionid 可以拿到session

   3. 通过 `SessionRepository` 根据这个sessionid去 redis 查

   4. 如果有则将session缓存、退出循环并标记已找到session

4. 如果获取到session那么久将session返回，如果没有就创建一个 RedisSession

   ```java
   S session = SessionRepositoryFilter.this.sessionRepository.createSession();
   currentSession = new HttpSessionWrapper(session, getServletContext());
   return currentSession;
   ```

   > org.springframework.session.SessionRepository#createSession 

   最终创建了一个 `RedisSession` 返回

   ```java
   public RedisSession createSession() {
      MapSession cached = new MapSession();
      if (this.defaultMaxInactiveInterval != null) {
         cached.setMaxInactiveInterval(Duration.ofSeconds(this.defaultMaxInactiveInterval));
      }
      RedisSession session = new RedisSession(cached, true);
      session.flushImmediateIfNecessary();
      return session;
   }
   ```

## setAttribute

`setAttribute` 方法调用之后，并不会立刻保存到 redis，而是先缓存到本地的一个

`MapSession` ，在请求结束之后才调用包装后的 request 中的 `commitSession` 方法保存

> org.springframework.session.web.http.SessionRepositoryFilter.SessionRepositoryRequestWrapper#commitSession

```java
@Override
public void setAttribute(String attributeName, Object attributeValue) {
   this.cached.setAttribute(attributeName, attributeValue);
   this.delta.put(getSessionAttrNameKey(attributeName), attributeValue);
   flushImmediateIfNecessary();
}
```

如果需要设置后立刻保存，那么可以添加以下配置（不是）：

```properties
spring.session.redis.flush-mode=immediate
```

是需要在启用SpringSession时开启（特喵的骗人...）

```java
@EnableRedisHttpSession(flushMode = FlushMode.IMMEDIATE)
```

添加这个配置后 `flushImmediateIfNecessary` 就能够生效

```java
private void flushImmediateIfNecessary() {
   if (RedisIndexedSessionRepository.this.flushMode == FlushMode.IMMEDIATE) {
      save();
   }
}
```

## getAttribute

其实就是在本地缓存的 `MapSession` 里拿数据

```java
private final MapSession cached;

private Map<String, Object> delta = new HashMap<>();

@Override
public Object getAttribute(String name) {
  checkState();
  return this.session.getAttribute(name);
}

@Override
public <T> T getAttribute(String attributeName) {
  T attributeValue = this.cached.getAttribute(attributeName);
  if (attributeValue != null
      && RedisIndexedSessionRepository.this.saveMode.equals(SaveMode.ON_GET_ATTRIBUTE)) {
    this.delta.put(getSessionAttrNameKey(attributeName), attributeValue);
  }
  return attributeValue;
}
```