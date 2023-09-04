---
title: SpringCloud Alibaba-Sentinel - 学习笔记
date: 2022-03-21 09:03:58
tags: [SpringCloud,Java,SpringBoot]
categories: Spring
---

# SpringCloud Alibaba-Sentinel - 学习笔记

##  熔断降级和限流

### 1.什么是熔断？

A 服务调用 B 服务的某个功能，由于网络不稳定或者 B 服务压力大，导致业务处理能力降低响应慢。如果这样的次数太多，可以直接把 B 服务断了（A 不再请求 B），凡是调用 B 的直接返回降级数据，不必等待 B 的超长执行。这样 B 的故障问题就不会级联影响到 A。

### 2.什么是降级？

整个网站处于流量高峰期，服务器压力剧增，可以根据当前业务情况及流量，对一些服务和页面进行尤策略的降级停止服务，所有的调用直接返回降级数据。以此缓解服务器资源的压力，以保证核心业务的正常运行，同时也保证了客户和大部分客户能得到正确的响应。

### 3. 什么是限流

对打入服务的请求流量进行控制，使服务能够承担不超过自己能力的流量压力。

<more/>

### 4. 异同

- 相同点
  1. 为了保证集群大部分服务的可用性和可靠性，防止崩溃，关闭特定业务以保证资源可用
  2. 用户最终都是体验到某个功能不可用
- 不同点
  1. 熔断时被调用方故障，触发的系统主动规则
  2. 降级是基于全局考虑，停止一些正常服务，释放资源

## 整合SpringBoot
导入依赖
```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```
**注意在依赖导入的时候，可能会产生 `com.alibaba.fastjson` 依赖冲突，好像是引入了两个这个依赖，把它忽略就好**

> ```xml
> <!--    用于限流、熔断、降级    -->
> <dependency>
> <groupId>com.alibaba.cloud</groupId>
> <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
> 
> <exclusions>
>  <exclusion>
>    <groupId>com.alibaba</groupId>
>    <artifactId>fastjson</artifactId>
>  </exclusion>
> </exclusions>
> </dependency>
> ```

下载对应版本的 sentinel-dashboard 【https://github.com/alibaba/Sentinel/releases】尽量与你项目中下载的sentinel版本一致，否则可能会有其它问题

![sentinel依赖版本](https://qiniu-note-image.ctong.top/note/images/202203181604926.png)

下载好 dashboard 后启动它，如果端口冲突就修改 `--server.prot` 参数
```
java -jar sentinel-dashboard-1.8.1.jar --server.prot=8080
```

启动 dashboard 后在需要保护的项目中加入配置
```properties
# 配置 sentinel dashboard 路径
spring.cloud.sentinel.transport.dashboard=172.16.156.128:8080
spring.cloud.sentinel.transport.port=8719
```
> 这里的 spring.cloud.sentinel.transport.port 端口配置会在应用对应的机器上启动一个 Http Server，该 Server 会与 Sentinel 控制台做交互。比如 Sentinel 控制台添加了一个限流规则，会把规则数据 push 给这个 Http Server 接收，Http Server 再将规则注册到 Sentinel 中。
>

访问你项目中任意一个路径再打开 dashboard 就可以看到对应的信息

![dashboard](https://qiniu-note-image.ctong.top/note/images/202203181609099.png)

## 流控

打开 dashboard 对指定路径设置流控规则

点击 【蔟点链路 ---> 流控】 然后对指定的路径进行设置

![流程01](https://qiniu-note-image.ctong.top/note/images/202203181613103.png)

设置好规则后点击新增即可

![流程02](https://qiniu-note-image.ctong.top/note/images/202203181615564.png)

以上规则是一秒只能处理一个请求，如果处理了多个，那么就直接返回失败，例如：

![流控演示](https://qiniu-note-image.ctong.top/note/images/202203181621625.gif)



点击【流控规则】可以看到你添加的所有规则，我刚刚添加的规则就可以在这看到

![规则列表](https://qiniu-note-image.ctong.top/note/images/202203181617998.png)

**但是现在有个问题，就是当我的服务重启后，规则就失效了**

![规则列表](https://qiniu-note-image.ctong.top/note/images/202203181625425.png)

### 自定义降级数据

Sentinel 的默认降级数据是一个 `Blocked by Sentinel (flow limiting)` 

> 默认是这个类实现：`com.alibaba.csp.sentinel.adapter.spring.webmvc.callback.DefaultBlockExceptionHandler`

如果需要自定义，可以在容器中放一个自定义的 `BlockExceptionHandler` 即可

```java
@Component
public static class CustomBlockException implements BlockExceptionHandler {
  @Override
  public void handle(HttpServletRequest httpServletRequest,
                     HttpServletResponse httpServletResponse,
                     BlockException e) throws Exception {
    httpServletResponse.setCharacterEncoding("UTF-8");
    httpServletResponse.setContentType(MediaType.APPLICATION_JSON_VALUE);
    PrintWriter writer = httpServletResponse.getWriter();
    R error = R.error(BizCodeEnum.TO_MANY_REQUEST);
    writer.println(JSON.toJSONString(error));
    log.info("限流: ===>> {}", error);
  }
}
```

![测试结果](https://qiniu-note-image.ctong.top/note/images/202203200856881.png)

**注意：由于我使用的是2021版本的，旧版需要使用 `WebCallbackManager.setUrlBlockHandler(xxx);`  来设置**

## 熔断降级

**默认当前环境就是微服务：SpringCloud + SpringCloudAlibaba + SpringFeign**

例如单A服务调用B服务时可能会出现错误导致一系列问题，在引入熔断保护后，在第一次调用失败时Sentinel会将B服务进行隔离（断路保护），下次再调用该服务时会直接执行降级方法。防止在出现问题后导致的服务崩溃现象。

### 调用方的熔断保护

例如我有这么一个远程接口

```java
@FeignClient(value = "gulimall-seckill")
public interface SeckillFeignService {
    /**
     * 通过商品id查询当前商品是否参与秒杀活动
     * @param skuId 商品id
     * @return R
     * @author Clover You
     * @email cloveryou02@163.com
     * @date 2022/3/16 9:25 上午
     */
    @GetMapping("/sku/seckill/{skuId}")
    R getSkuSeckillInfo(@PathVariable("skuId") Long skuId);

}
```

给 Feign 接口添加一个降级实现，注意该实现一定要添加到容器中

```java
package top.ctong.gulimall.product.feign.fallback;

import org.springframework.stereotype.Component;
import top.ctong.gulimall.common.exception.BizCodeEnum;
import top.ctong.gulimall.common.utils.R;
import top.ctong.gulimall.product.feign.SeckillFeignService;

@Component
public class SeckillFeignServiceFallBack implements SeckillFeignService {
    /**
     * 通过商品id查询当前商品是否参与秒杀活动
     * @param skuId 商品id
     * @return R
     * @author Clover You
     * @email cloveryou02@163.com
     * @date 2022/3/16 9:25 上午
     */
    @Override
    public R getSkuSeckillInfo(Long skuId) {
        return R.error(BizCodeEnum.UNKNOWN_EXCEPTION);
    }
}
```

添加完降级实现后再在远程接口中 `@FeignClient` 的 `fallback` 参数来指定这个降级实现

```java
@FeignClient(value = "gulimall-seckill", fallback = SeckillFeignServiceFallBack.class)
public interface SeckillFeignService {}
```

把远程服务停掉模拟宕机，再发送请求检查远程返回的数据

![降级数据](https://qiniu-note-image.ctong.top/note/images/202203201613419.png)

### 调用方手动指定降级策略

可以在 sentinel-dashboard 中手动指定降级规则，这种方式比较灵活，并且支持多种策略，例如 RT 策略，可以根据每秒调用比例来就行降级，例如每秒进来5个请求，这些请求如果都不在指定时间内完成（毫秒），那么就触发降级策略，过了指定保护后，会尝试几次去调用远程

在指定远程链路上点击【降级】

![image-20220320163541779](https://qiniu-note-image.ctong.top/note/images/202203201635927.png)

设置好规则后点击【新增】保存

![设置熔断规则](https://qiniu-note-image.ctong.top/note/images/202203201639927.png)

> 比例阈值：自己设定的 ， 慢调用次数 / 调用次数=比例阈值
>
> 来源于：https://blog.csdn.net/w1234567465/article/details/119649203

测试如下图

![熔断保护演示](https://qiniu-note-image.ctong.top/note/images/202203201707398.gif)

在指定时间内慢调次数达到比例阈，那么就开启熔断保护（降级处理并触发熔断回调），在保护期间，超过熔断时长后，会稍稍放一些请求过去，如果请求还是不达标，那么继续隔离。

## 自定义受保护资源

### 代码定义

任何一个我们需要保护的代码片段都是资源，可以使用 `SphU.entry("自定义资源名")` 定义一段资源，当被限流时 `try` 中被保护的资源并不会执行，而是抛出一个 `BlockException` 异常，我们只需要处理这个异常就好了。

```java
@GetMapping("/current-seckiil-skus")
@ResponseBody
public R getCurrentSeckiilSkus() {
    try (Entry entry = SphU.entry("current-seckiil-skus")) {
        List<SeckillSkuRedisTo> list = seckillService.getCurrentSeckiilSkus();
        return R.ok().setData(list);
    } catch (BlockException e) {
        // {code: 10003, msg: "当前服务器正忙"}
        return R.error(BizCodeEnum.TO_MANY_REQUEST);
    }
}
```

在对应的服务中点击【降级/限流规则】再新增一个对应的规则【新增降级规则】

![新增降级规则](https://qiniu-note-image.ctong.top/note/images/202203201928119.png)

**一定要注意资源名，一定要是 `SphU.entry` 里指定的资源名**，设置完后点【新增就好了】

![新增降级规则](https://qiniu-note-image.ctong.top/note/images/202203201931116.png)

就放结果好了，不搞gif了

![结果](https://qiniu-note-image.ctong.top/note/images/202203201942712.png)

### 注解定义资源

可以使用 sentinel 提供的 `@SentinelResource` 注解来定义一个资源，和使用代码定义的方式差不多，不过需要在本类指定一个降级处理方法。

```java
@GetMapping("/current-seckiil-skus")
@SentinelResource(value = "current-seckiil-skus", blockHandler = "blockHandler")
@ResponseBody
public R getCurrentSeckiilSkus() {
  List<SeckillSkuRedisTo> list = seckillService.getCurrentSeckiilSkus();
  return R.ok().setData(list);
}

// 降级处理方法
public R blockHandler(BlockException e) {
  log.info("限流了！！");
  return R.error(BizCodeEnum.TO_MANY_REQUEST);
}
```

如果不指定资源名称，那么也可以使用方法全类名作为资源名也是可以的

`top.ctong.gulimall.seckill.controller.SeckillController:getCurrentSeckiilSkus()`

**官方给出的注解参数**

`@SentinelResource` 用于定义资源，并提供可选的异常处理和 fallback 配置项。 `@SentinelResource` 注解包含以下属性：

- `value`：资源名称，必需项（不能为空）
- `entryType`：entry 类型，可选项（默认为 `EntryType.OUT`）
- `blockHandler` / `blockHandlerClass`: `blockHandler` 对应处理 `BlockException` 的函数名称，可选项。blockHandler 函数访问范围需要是 `public`，返回类型需要与原方法相匹配，参数类型需要和原方法相匹配并且最后加一个额外的参数，类型为 `BlockException`。blockHandler 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `blockHandlerClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。
- `fallback` / `fallbackClass`：fallback 函数名称，可选项，用于在抛出异常的时候提供 fallback 处理逻辑。fallback 函数可以针对所有类型的异常（除了 `exceptionsToIgnore` 里面排除掉的异常类型）进行处理。fallback 函数签名和位置要求：
  - 返回值类型必须与原函数返回值类型一致；
  - 方法参数列表需要和原函数一致，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  - fallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。
- `defaultFallback`（since 1.6.0）：默认的 fallback 函数名称，可选项，通常用于通用的 fallback 逻辑（即可以用于很多服务或方法）。默认 fallback 函数可以针对所有类型的异常（除了 `exceptionsToIgnore` 里面排除掉的异常类型）进行处理。若同时配置了 fallback 和 defaultFallback，则只有 fallback 会生效。defaultFallback 函数签名要求：
  - 返回值类型必须与原函数返回值类型一致；
  - 方法参数列表需要为空，或者可以额外多一个 `Throwable` 类型的参数用于接收对应的异常。
  - defaultFallback 函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 `fallbackClass` 为对应的类的 `Class` 对象，注意对应的函数必需为 static 函数，否则无法解析。
- `exceptionsToIgnore`（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。

1.8.0 版本开始，`defaultFallback` 支持在类级别进行配置。

> 注：1.6.0 之前的版本 fallback 函数只针对降级异常（`DegradeException`）进行处理，**不能针对业务异常进行处理**。

## 网关限流

使用网关限流效果更加明显，因为请求直接就在网关层拦截，根本就不需要转到其他服务。

Sentinel 支持目前主流的网关，如：Spring Cloud Gateway、Netflix和Zuul等。

可以根据请求属性进行解析，例如某个请求带或者没带某个属性进行限流和网关规则管理、网关规则检查、调用参数组装、自定义API分组管理、API路径匹配。

引入以下Maven依赖

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-alibaba-dependencies</artifactId>
  <version>2021.1</version>
</dependency>

<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-alibaba-sentinel-gateway</artifactId>
  <version>2021.1</version>
</dependency>

<dependency>
  <groupId>com.alibaba.csp</groupId>
  <artifactId>sentinel-spring-cloud-gateway-adapter</artifactId>
</dependency>
```

引入依赖后添加一些配置

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 172.16.156.128:8080
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

再发起一个请求稍等一会刷新 sentinel-dashboard 就可以看见对应模块了

Sentinel 1.6.3 引入了网关流控控制台的支持，用户可以直接在 Sentinel 控制台上查看 API Gateway 实时的 route 和自定义 API 分组监控，管理网关规则和 API 分组配置。

![网关控制台](https://qiniu-note-image.ctong.top/note/images/202203210811471.png)

对网关层新建一个流控规则，资源名可以是路由规则名，例如以下路由配置，`id` 就是路由规则名，同时可用于在 Sentinel 中作为资源名

```yaml
spring:
  application:
    name: gulimall-gateway
  cloud:
    gateway:
      routes:
        - id: ware_route
          uri: lb://gulimall-ware
          predicates:
            - Path=/api/ware/**
          filters:
            - RewritePath=/api/(?<path>.*),/$\{path}

        - id: coupon_route
          uri: lb://gulimall-coupon
          predicates:
            - Path=/api/coupon/**
          filters:
            - RewritePath=/api/(?<path>.*),/$\{path}
```

![新增网关流控规则](https://qiniu-note-image.ctong.top/note/images/202203210819802.png)

测试结果：

![结果](https://qiniu-note-image.ctong.top/note/images/202203210822392.png)

### 针对请求属性

同时也可以针对请求属性进行匹配，例如请求头、参数等信息。

如以下规则，如果携带了 `token` 参数，并且其值为 **1** ，那么就限制该请求，否则放行

![针对请求属性](https://qiniu-note-image.ctong.top/note/images/202203210828398.png)



![测试结果](https://qiniu-note-image.ctong.top/note/images/202203210830177.png)

### Api分组

可以通过将多个路径分成一组，然后对这个组统一进行限流

可以到【API管理】中点击【新增自定义API】进行设置

![API分组](https://qiniu-note-image.ctong.top/note/images/202203210835002.png)

新增好分组后再添加一个限流规则，注意这里的**API类型**选择API分组

![新建对分组的限流规则](https://qiniu-note-image.ctong.top/note/images/202203210838826.png)

再测试这两个API，都可以限制住

![测试结果](https://qiniu-note-image.ctong.top/note/images/202203210839309.png)

### 自定义回调数据

如果需要修改限流后的返回数据，需要使用 `GatewayCallbackManager` 来进行设置

```java
@Configuration
public class SentinelGatewayConfig {

    public SentinelGatewayConfig() {
        GatewayCallbackManager.setBlockHandler(new BlockRequestHandler() {
            @Override
            public Mono<ServerResponse> handleRequest(ServerWebExchange serverWebExchange, Throwable throwable) {
                R error = R.error(BizCodeEnum.TO_MANY_REQUEST);
                return ServerResponse.ok().body(Mono.just(error), R.class);
            }
        });
    }

}
```

![降级数据](https://qiniu-note-image.ctong.top/note/images/202203210855126.png)

### 参数相关

- `GatewayFlowRule`：网关限流规则，针对 API Gateway 的场景定制的限流规则，可以针对不同 route 或自定义的 API 分组进行限流，支持针对请求中的参数、Header、来源 IP 等进行定制化的限流。
- `ApiDefinition`：用户自定义的 API 定义分组，可以看做是一些 URL 匹配的组合。比如我们可以定义一个 API 叫 `my_api`，请求 path 模式为 `/foo/**` 和 `/baz/**` 的都归到 `my_api` 这个 API 分组下面。限流的时候可以针对这个自定义的 API 分组维度进行限流。

其中网关限流规则 `GatewayFlowRule` 的字段解释如下：

- `resource`：资源名称，可以是网关中的 route 名称或者用户自定义的 API 分组名称。
- `resourceMode`：规则是针对 API Gateway 的 route（`RESOURCE_MODE_ROUTE_ID`）还是用户在 Sentinel 中定义的 API 分组（`RESOURCE_MODE_CUSTOM_API_NAME`），默认是 route。
- `grade`：限流指标维度，同限流规则的 `grade` 字段。
- `count`：限流阈值
- `intervalSec`：统计时间窗口，单位是秒，默认是 1 秒。
- `controlBehavior`：流量整形的控制效果，同限流规则的 `controlBehavior` 字段，目前支持快速失败和匀速排队两种模式，默认是快速失败。
- `burst`：应对突发请求时额外允许的请求数目。
- `maxQueueingTimeoutMs`：匀速排队模式下的最长排队时间，单位是毫秒，仅在匀速排队模式下生效。
- `paramItem`：参数限流配置。若不提供，则代表不针对参数进行限流，该网关规则将会被转换成普通流控规则；否则会转换成热点规则。其中的字段：
  - `parseStrategy`：从请求中提取参数的策略，目前支持提取来源 IP（`PARAM_PARSE_STRATEGY_CLIENT_IP`）、Host（`PARAM_PARSE_STRATEGY_HOST`）、任意 Header（`PARAM_PARSE_STRATEGY_HEADER`）和任意 URL 参数（`PARAM_PARSE_STRATEGY_URL_PARAM`）四种模式。
  - `fieldName`：若提取策略选择 Header 模式或 URL 参数模式，则需要指定对应的 header 名称或 URL 参数名称。
  - `pattern`：参数值的匹配模式，只有匹配该模式的请求属性值会纳入统计和流控；若为空则统计该请求属性的所有值。（1.6.2 版本开始支持）
  - `matchStrategy`：参数值的匹配策略，目前支持精确匹配（`PARAM_MATCH_STRATEGY_EXACT`）、子串匹配（`PARAM_MATCH_STRATEGY_CONTAINS`）和正则匹配（`PARAM_MATCH_STRATEGY_REGEX`）。（1.6.2 版本开始支持）

> Url 限流可以根据 【自定义降级数据】章节的配置统一返回。其它方式需要指定
