title: spring-cloud远程调用
categories: 随笔
tags: [SpringCloud,随笔]
date: 2022-01-02 18:27:00
---
## 笔记

在微服务中，若想要使用远程调用，需要引入`spring-cloud-starter-openfeign`（在使用注册中心的环境下）

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
  <version>xxx</version>
</dependency>
```

由于open-feign是声明式的远程调用，所以需要编写一个接口，并且告诉SpringCloud这个接口需要调用远程服务。这个接口我放在公共模块下的`feign`中。

```java
package top.ctong.gulimall.common.feign;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import top.ctong.gulimall.common.utils.R;

import java.util.Map;
@FeignClient("gulimall-coupon")
@RequestMapping("/coupon/coupon")
public interface CouponFeignService {

    /**
     * 通过自定义参数查询列表
     * @param params 自定义参数
     * @return R
     * @author Clover You
     * @date 2021/11/17 09:11
     */
    @RequestMapping("/list")
    R list(@RequestParam Map<String, Object> params);
}

```

- `@FeignClient("xxx")` 该注解用于告诉SpringCloud这是一个远程调用接口，其中的`value`是你在注册中心中注册的服务名。

> 接口方法签名需要与被调用的远程服务签名一致，例如上面哪个代码要调用的远程服务：
>
> ```java
> package top.ctong.gulimall.coupon.controller;
> 
> @RestController
> @RequestMapping("coupon/coupon")
> public class CouponController {
>     @Autowired
>     private CouponService couponService;
>     /**
>      * 列表
>      */
>     @RequestMapping("/list")
>     //@RequiresPermissions("coupon:coupon:list")
>     public R list(@RequestParam Map<String, Object> params){
>         PageUtils page = couponService.queryPage(params);
>         return R.ok().put("page", page);
>     }
> }
> 
> ```
>
> 

接着还要使用`@EnableFeignClients`开启当前需要使用远程调用的这个服务的远程调用，并且指定你的远程调用接口在哪个包下。

```java
@EnableFeignClients(basePackages = "top.ctong.gulimall.common.feign")
@EnableDiscoveryClient
@MapperScan("top.ctong.gulimall.member.dao")
@SpringBootApplication
public class GulimallMemberApplication {

    public static void main(String[] args) {
        SpringApplication.run(GulimallMemberApplication.class, args);
    }

}
```

- `@EnableFeignClients` 该注解用于开启当前服务的远程调用功能
  - `basePackages` 用于指定远程调用接口所在的包，方便服务启动的时候可以快速扫描到。他可以接收多个包名，因为它是一个`String[]`



最后在需要远程调用时注入对应的远程调用接口就好

```java
package top.ctong.gulimall.member.controller;

@RestController
@RequestMapping("member/member")
public class MemberController {

    private final CouponFeignService couponFeignService;

    @Autowired
    public MemberController(CouponFeignService couponFeignService, MemberService memberService) {
        this.couponFeignService = couponFeignService;
        this.memberService = memberService;
    }
  
    @RequestMapping("/testFeignInvoke")
    public R testFeignInvoke() {
        Map<String, Object> parem = new HashMap<>(10);
        return couponFeignService.list(parem);
    }
}

```

## 错误(nacos)

如果在启动时出现 `No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-loadbalancer?` 错误，那么就是你的SpringCloud版本比较高，在高版本的SpringCloud中已经不再使用 `spring-cloud-starter-netflix-ribbon` 了，而是使用 `spring-cloud-starter-loadbalancer` 。而nacos还是使用的 `spring-cloud-starter-netflix-ribbon`。

在`pom.xml`文件中引入 `spring-cloud-starter-loadbalancer` 再启动就没毛病了。

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-loadbalancer</artifactId>
  <version>3.0.4</version>
</dependency>
```

在测试远程调用中发生 `AbstractMethodError` 异常。需要在 `pom.xml` 中**排除** nacos 中引入的 **ribbon** 。否则 `spring-cloud-starter-loadbalancer` 无法工作。

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
  <exclusions>
    <exclusion>
      <groupId>com.netflix.ribbon</groupId>
      <artifactId>ribbon</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```




