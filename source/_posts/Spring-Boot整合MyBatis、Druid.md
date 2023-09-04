title: Spring Boot整合MyBatis、Druid
categories: 随笔
tags: [SpringBoot,JAVA]
date: 2022-01-02 18:26:56
---
# Spring Boot整合MyBatis、Druid

## 配置

### pom

在`pom.xml`文件中引入mybatis、druid、mysq的依赖

```xml
<!--mybatis 依赖-->
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>${spring-boot.mybatis}</version>
</dependency>
<!--mysql 连接依赖-->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
<!--druid连接池-->
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid-spring-boot-starter</artifactId>
  <version>1.2.5</version>
</dependency>
```



### nacos

在nacos中对对应的配置进行编辑

```yaml
# Spring
spring: 
  redis:
    host: 47.115.149.74
    port: 6379
    password: 
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://ip/cainaer_cloud?useUnicode=true&characterEncoding=utf8&zeroDateTimeBehavior=convertToNull&useSSL=true&serverTimezone=GMT%2B8
    username: # 你的账号
    password: # 你的密码
    # 阿里连接池
    druid:
      initial-size: 2
      min-idle: 1
      max-active: 20
      filters: stat,wall
      web-stat-filter:
          enabled: true
      stat-view-servlet:
          enabled: true


# Mybatis配置
mybatis-plus:
  # 搜索指定包别名
  type-aliases-package: # 你的包名
  # 配置mapper的扫描，找到所有的mapper.xml映射文件
  mapper-locations: classpath:mapper/**/*.xml
  configuration:
    # 是否将sql打印到控制面板(该配置会将sql语句和查询的结果都打印到控制台)
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

# swagger 配置
```

### 启动项目

以上配置完成后启动项目，启动过程中如果出现以下日志，那就证明配置成功了

![截屏2021-03-10 16.02.09](http://qiniu-note-image.ctong.top/note/images/202112271055844.png)

执行一段curd代码打印日志如下，那么druid正常使用

```log
JDBC Connection [com.alibaba.druid.proxy.jdbc.ConnectionProxyImpl@7a95ba84] will not be managed by Spring
```