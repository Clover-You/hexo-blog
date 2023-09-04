title: 使用nacos作为配置中心统一管理配置
categories: 随笔
tags: [SpringBoot,SpringCloud,随笔,JAVA,微服务]
date: 2022-01-02 18:16:00
---
## 基础环境

引入所需依赖包

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

创建一个 `bootstrap.properties` 或 `bootstrap.yaml` 。其中指定项目名与配置中心服务器地址

```properties
spring.application.name=gulimall-coupon
spring.cloud.nacos.config.server-addr=127.0.0.1:8848
```

最后需要再配置中心添加一个数据集（Data Id），通常是「应用名.properties」。新建完后就可以给“**配置内容**”中添加任何配置，可以通过`@Value("${配置名}")`获取到配置。如果还需要动态刷新，可以给类加上 `@RefreshScope` 。

```java
@RefreshScope
public class CouponController {
    @Value("${author.name}")
    private String name;
}
```



![截屏2021-11-19 15.50.26](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-11-19%2015.50.26.png)

## dataID

在 Nacos Config Starter 中，dataId 的拼接格式如下

```
${prefix} - ${spring.profiles.active} . ${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。

- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)

  **注意，当 active profile 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}`.`${file-extension}`**

- `file-extension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension`来配置。 目前只支持 `properties` 类型。



## group

- `group` 默认为 `DEFAULT_GROUP`，可以通过 `spring.cloud.nacos.config.group` 配置。



## Nacos 作为配置中心的更多细节

### 命名空间

> 用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的 Group 或 DataID 的配置。Namespace 的常用场景之一是不同环境的配置的区分隔离。例如开发测试环境和生产环境的资源（如配置、服务）隔离等。

默认的命名空间是 `public(保留空间)` ：默认新增的所有配置都在 `public` 空间下。

例如有开发、测试、生产三个环境，此时需要将生产环境的配置切换到开发环境的配置，那么就可以使用命名空间快速切换不同的环境配置，需要在 `bootstrap` 配置中修改配置，配置如下

```properties
#命名空间ID
spring.cloud.nacos.config.namespace=24b7cf49-b115-4d1f-85a7-fdb44d9ec559
```



### 配置集

> 一组相关或者不相关的配置项的集合称为配置集。在系统中，一个配置文件就是一个配置集，包含了系统的各个方面的配置。例如，一个配置集可能包含了数据源、线程池、日志级别等配置项。



### 配置集ID

类似配置文件名，在Nacos中叫Data ID



### 配置分组

默认所有的配置集都属于：**DEFAULT_GROUP**

可以通过 `bootstrap` 配置文件中指定使用哪个配置分组

```properties
spring.cloud.nacos.config.group=DEFAULT_GROUP
```



---



> 每个微服务都创建自己的命名空间，使用配置分组区分环境