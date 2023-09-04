title: SpringBoot Actuator - 学习笔记
categories: Spring
tags: [Spring,学习笔记,SpringMvc,JAVA,SpringBoot]
date: 2022-01-02 18:08:41
---
## 指标监控

[官网相关文档](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints)

### SpringBoot Actuator

未来每一个微服务在云上部署后，需要对其进行监控、追踪、审计、控制等。SpringBoot就抽取了Actuator场景，使得我门每个微服务快速引用即可获得生产级别的应用监控、审计等功能。

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```



#### 使用

- 引入actuator场景

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```

- 访问http://localhost:8080/actuator/\*\*

- Web下默认暴露的端点只有`health`和`info`，可以在配置中暴露所有端点信息

  ```yaml
  # actuator 监控指标
  management:
    endpoints:
      enabled-by-default: true # 默认开启所有监控端点
      web:
        exposure:
          include: '*' # 以web方式暴露所有监控端点
  ```



#### 可视化

https://github.com/codecentric/spring-boot-admin



### Actuator Endpoint

常用的Endpoint有

- Health
  监控状况
- Metrics
  运行时指标
- Loggers
  日志记录



#### Health Endpoint

健康检查端点，一般用在云平台，平台会定时检查应用的健康状况。需要Health Endpoint为平台返回当前应用的一系列组件健康状况集合。

- health endpoint 返回的结果，应该是一系列健康检查后的一个汇总报告
- 很多的健康检查默认已经自动配好了，比如：数据库，redis等
- 可以自定义添加健康检查机制



开启指定端点详细信息

```yaml
# actuator 监控指标
management:
  endpoint:
    health:
      show-details: always # 显示详细
```

![截屏2021-07-14 15.04.50](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-07-14%2015.04.50.png)





#### Metrics Endpoint

提供详细的、层级的、空间指标信息，这些信息可以被pull（主动推送）或者push（被动获取）方式得到：

- 通过Metrics对接多种监控系统
- 简化核心Metrics开发
- 添加自定义Metrics或者扩展已有Metrics



![截屏2021-07-14 15.10.09](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-07-14%2015.10.09.png)



![截屏2021-07-14 15.11.13](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-07-14%2015.11.13.png)



  ### 管理Endpoint



#### 开启与禁用Endpoint

默认情况下，endpoint是全部开启的，若需要指定开启，需要全部关闭后再开启指定端点

```yaml
# actuator 监控指标
management:
  endpoints:
    enabled-by-default: false # 关闭全部端点
  endpoint:
    health:
      show-details: always # 显示详细信息
      enabled: true # 开启端点
    info:
      enabled: true
    metrics:
      enabled: true
```





### 定制Endpoint



#### 定制Health信息

实现`HealthIndicator`接口或者`AbstractHealthIndicator`抽象类。

```java
@Component
public class ComponentHealthIndicator extends AbstractHealthIndicator implements Serializable {

  private static final long serialVersionUID = 5066467685531996717L;

  public ComponentHealthIndicator() {
    // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
  }

  @Override
  protected void doHealthCheck(Health.Builder builder) throws Exception {
    Map<String, Object> map = new HashMap<>();
    if (false) {
      builder.status(Status.UP);
      map.put("count", 1);
      map.put("ms", 100);
    } else {
      builder.status(Status.OUT_OF_SERVICE);
      map.put("err", "连接超时");
      map.put("ms", 3000);
    }
    builder.withDetail("code", 200).withDetails(map);
  }

}
```

![image-20210714154234818](http://qiniu-note-image.ctong.top/note/images/image-20210714154234818.png)



#### 定制info信息

定制info有两种方法，一种是在application.yaml配置文件中定义 

```yaml
info:
  version: @project.version@ # 获取pom文件中的版本信息
  autor: Clover
  age: 19
```

![截屏2021-07-14 15.45.20](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-07-14%2015.45.20.png)



第二种方式，可以实现`InfoContributor`接口的方式实现，第二种优先级高于配置文件方式。

```java
@Component
public class AppInfoContributor implements Serializable, InfoContributor {

  private static final long serialVersionUID = -2489956740668578470L;

  public AppInfoContributor() {
    // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
  }

  @Override
  public void contribute(Info.Builder builder) {
    Map<String, Object> info = new HashMap<>();
    info.put("author", "Clover.You");
    info.put("age", 19);
    info.put("version", "1.0.0.1");
    builder.withDetails(info);
  }

}
```



#### 定制Metrics信息

##### 增加定制Metrics

```java
@Slf4j
@Controller
public class LoginController implements Serializable {

  private static final long serialVersionUID = -2783841528321793923L;

  // 计数指标
  private Counter counter;

  public LoginController(MeterRegistry meter) {
    // 注册自定义计数指标，记录登录方法调用了几次
    counter = meter.counter("com.ctong.learnspringboot.controller.LoginController.login");
  }

  @PostMapping("/login")
  public String login(HttpSession session, User user) {
    // 增加计数
    counter.increment();
    session.setAttribute("loginUser", user);
    return "redirect:/index.html";
  }

}
```

注册成功后，访问localhost:8080/actuator/metrics就可以看见我们自定义的新指标

![image-20210714162110608](http://qiniu-note-image.ctong.top/note/images/image-20210714162110608.png)



计数会随着`counter.increment();`方法的调用而增加

![截屏2021-07-14 16.22.11](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-07-14%2016.22.11.png)



#### 定义Endpoint

SpringBoot给我们定义了非常多的监控端点，一旦我们引入 了复杂场景，可能需要自定义监控端点。

```java
@Component
@Endpoint(id = "diy")
@Slf4j
public class MyEndpoint implements Serializable {

  private static final long serialVersionUID = 1370368576988354134L;

  public MyEndpoint() {
    // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
  }

  @ReadOperation
  public Map getDockerInfo() {
    return Collections.singletonMap("docker", "docker started...");
  }

  @WriteOperation
  public void stopDocker() {
    log.info("docker stopped....");
  }

}
```



   