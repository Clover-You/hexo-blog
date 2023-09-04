---
title: Sleuth + Zipkin 服务链路追踪 - 学习笔记
date: 2022-03-21 20:11:00
tags: [Java,SpringCloud,微服务,学习笔记]
categories: Spring
---

# Sleuth + Zipkin 服务链路追踪 - 学习笔记

微服务框架是一个分布式架构，它按业务划分服务单元，一个分布式系统往往有很多个服务单元。由于服务单元数量众多、业务的复杂性，如果出现了错误和异常，很难去定位。主要体现在：一个请求可能需要调用多个服务，而内部服务的调用复杂性，决定了问题难以定位。所以微服务架构中，必须实现分布式链路追踪，去跟进一个请求到底有哪些服务参与，参与的顺序又是怎样的，从而达到每个请求的步骤清晰可见，出了问题可以很快定位。

链路追踪组件有 Google 的 Dapper、Twitter 的 Zipkin 以及阿里的Eagleeye（鹰眼）等

## 基本术语

- Span（跨度）：基本工作单元，发送一个远程调度任务，就会产生一个 Span，Span 是一个 64 位 ID 唯一标识的，Trace 是用另一个 64 位 ID 唯一标识的，Span 还有其他数据信息，比如摘要、时间戳事件、Span 的 ID以及进度 ID。
- Trace（跟踪）：一系列 Span 组成的一个树状结构。请求一个微服务系统的 API 接口，这个 API 接口需要需要调用多个微服务，调用每个微服务都会产生一个新的 Span，所有由这个请求产生的 Span 组成了这个 Trace。
- Annotation（标注）：用来及时记录一个事件的，一些核心注解用来定义一个请求的开始和结束
  - `cs` Client Sent：客户端发送一个请求，这个注解描述了这个 Span 的开始。
  - `sr` Server Received：服务端获得请求并准备开始处理它，如果将其 `sr` 减去 `cs` 的时间戳便可得到网络传输的时间。
  - `ss` Server Sent（服务端发送响应）：该注解表明请求处理的完成（当请求返回客户端），如果 `ss` 的时间戳减去 `sr` 的时间戳，就可以得到服务器请求的时间。
  - `cr` Client Received（客户端接收响应）：此时 Span 的结束，如果 `cr` 的时间戳减去 `cs` 的时间戳便可以得到整个请求所消耗的时间。

## 整合Sleuth

服务提供者与消费者都需要导入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-sleuth -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```

开启日志

```yaml
logging:
  level:
    top.ctong.gulimall: info
    org.springframwork.cloud.openfeign: debug
    org.springframwork.cloud.sleuth: debug
```

开启日志后发送一次远程调用，控制台打印链路信息

```
WARN [gulimall-seckill,424a8cbc7003a8ab,c5ff8faf9ba3f49c]
```

- `gulimall-seckill` 是服务名
- `424a8cbc7003a8ab` 是 TraceId，一条链路中只有一个 TraceId
- `c5ff8faf9ba3f49c` 是 SpanId，链路中的基本工作单元 ID

## 整合 Zipkin 可视化观察

通过 Sleuth 产生的调用链监控信息，可以得知微服务之间的调用链路，但监控信息只输出到控制台不方便查看。可以使用图形化工具-Zipkin。Zipkin 是 Twitter 开源的分布式跟踪系统，主要用来收集系统的时序数据，从而追踪系统的调用问题。

[官方文档传送门：https://zipkin.io/pages/quickstart](https://zipkin.io/pages/quickstart)

![Zipkin原理图](https://qiniu-note-image.ctong.top/note/images/202203211520956.png)

docker 安装 zipkin 服务器

```
docker run -d -p 9411:9411 openzipkin/zipkin
```

然后在服务中导入 zipkin 依赖，这个依赖同时包含了 sleuth，我们可以不用自己引入 zleuth

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

之后在服务中加入以下配置

```yaml
spring:
  # 服务链路追踪
  zipkin:
    # 服务地址
    base-url: http://172.16.156.128:9411/
    # 关闭服务发现，否则 SpringCloud 会把 zipkin 的 url 当作服务名
    discovery-client-enabled: false
    sender:
      # 使用http进行传输
      type: web
  sleuth:
    sampler:
      # 设置抽样采集器，1=100%
      probability: 1
```

配置好一切后，重启服务把所有的功能都测试一遍再检查 zipkin

**服务依赖关系**

![依赖](https://qiniu-note-image.ctong.top/note/images/202203211840644.png)

**调用链路**

![调用链路](https://qiniu-note-image.ctong.top/note/images/202203212011433.png)

> 链路追踪最重要的是分析请求

**注意：所有信息在服务重启后丢失，可以保存到ElasticSearch或者MySQL**
