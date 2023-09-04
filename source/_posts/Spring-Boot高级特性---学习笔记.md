title: Spring Boot高级特性 - 学习笔记
categories: Spring
tags: [SpringBoot,Spring,学习笔记,SpringMvc,JAVA]
date: 2022-01-02 18:07:36
---
## Profile功能

为了方便多环境适配，SpringBoot简化了profile功能



### application-profile功能

- 默认配置文件 application.yaml，任何时候都会加载

- 指定环境配置文件application-**env**.yaml

- 激活指定环境

  - 配置文件激活

  - 命令行激活

    ```
    java -jar xxx.jar --spring.profiles.active=profile
    ```

    > 修改配置文件的任意值，命令行优先

- 默认配置与环境配置同时生效

- 同名配置项，profile配置优先

在配置文件中指定环境

```yaml
spring:
  profiles:
    active: pro
```



### @Profile条件装配

```java
@Profile("pro")
@Component
@ConfigurationProperties(prefix = "person")
@Data
public class Boss implements Person {

  private String name;

  private Integer age;

}
```

`@Profile("pro")`当环境为`pro`时，这个类才会被注册到容器中。



### profile分组

```yaml
spring:
  profiles:
    active: diy
    group:
      diy:
        - dev
        - pro
```

分组之后可以使用`spring.profiles.active`将这个组中的配置环境全部加载。



## 外部化配置



### 外部配置源

常用：Java属性文件、YAML文件、环境变量、命令行参数



### 配置文件查找位置

外部化配置文件可以从以下几个路径进行查找，后面加载的配置会覆盖前面的配置  

1. classpath根路径
2. classpath根路径下config目录
3. jar包当前目录
4. jar包当前目录的lconfig目录
5. config子目录的直接子目录



### 配置文件加载顺序

1. 当前jar包内部的application.properties和application.yaml
2. 当前jar包内部的application-{profile}.properties和application-{profile}.yaml
3. 引用的外部jar包的application.properties和application.yaml
4. 引用的外部jar包的application-{profile}.properties和application-{profile}.yaml



> 指定环境优先，外部优先，后面的可以覆盖前面的同名配置项



## 自定义starter



### starter启动原理

- starter-pom引入autoconfigurer包
  ![截屏2021-07-18 15.15.25](http://qiniu-note-image.ctong.top/note/images/202112271049837.png)

- autoConfigure包中配置使用**META-INF/spring.factories**中**EnableAutoConfiguration**的值，使项目启动加载指定的自动配置类。

  ```yaml
  # auto configure
  org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  top.ctong.springbootstartautoconfigure.auto.HelloServiceAutoConfiguration
  ```

- 编写自动配置类**xxxAutoConfigration -> xxxProperties**

  - `@Configuration`
  - `@Conditional`
  - `@EnableConfigurationProperties`
  - `@Bean`

  ```java
  @Configuration
  @ConditionalOnMissingBean(HelloService.class)// 如果之前通过奇奇怪怪的方法注册了一个service，那么就不使用这个配置类了
  @EnableConfigurationProperties(HelloProperties.class)// 当这个配置类生效后，自动将这个properties放在容器中，如果没有开启这个配置，那么在service中的自动注入可能会找不到指定Bean
  public class HelloServiceAutoConfiguration {
  
    @Bean
    public HelloService helloService() {
      HelloService service = new HelloService();
      return service;
    }
  }
  ```

### 操作过程

[源码GitHub地址](https://github.com/YouChuantong/learn-customer-spring-boot-starter)



新建一个空工程在指定文件夹中

![截屏2021-07-18 16.56.14](http://qiniu-note-image.ctong.top/note/images/202112271049845.png)



新建成功一个空的工程后，会自动打开一个添加模块的面板，先需要添加一个starter工程「其实就是一个普通的maven工程」。

> starter工程一般是空的，只有一些基本的结构，但这并不绝对。

![截屏2021-07-18 16.58.02](http://qiniu-note-image.ctong.top/note/images/202112271049851.png)

完成之后可以根据需求删除不需要的文件夹，保留`pom.xml`文件即可。之后再新建一个模块，它是一个maven工程

起个名字吧！就叫`spring-boot-hello-service-starter-autoconfigure`

![截屏2021-07-18 17.07.04](http://qiniu-note-image.ctong.top/note/images/202112271049860.png)

然后在customer-starter项目中引入刚刚创建的工程

```xml
<dependencies>
    <dependency>
        <groupId>top.ctong</groupId>
        <artifactId>spring-boot-hello-service-starter-autoconfigure</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>	
</dependencies>
```

在spring-boot-hello-service-starter-autoconfigure工程中创建`top.ctong.springboothelloservicestarterautoconfigure.servlet.HelloServlet`、

`top.ctong.springboothelloservicestarterautoconfigure.bean.HelloProperties`

`top.ctong.springboothelloservicestarterautoconfigure.HelloServiceAutoConfigration`

创建这3个文件后在service中编写逻辑，例如我有一个请求，这个请求接受一个参数并读取配置文件中配置`HelloProperties`的内容。

需要`HelloProperties`绑定配置文件，但不用在这注入到容器中，此时会提示错误，忽略即可。

```java
@ConfigurationProperties(prefix = "clover.hello")
public class HelloProperties {

  private String prefix;

  private String suffix;

  public String getPrefix() {
    return prefix;
  }

  public void setPrefix(String prefix) {
    this.prefix = prefix;
  }

  public String getSuffix() {
    return suffix;
  }

  public void setSuffix(String suffix) {
    this.suffix = suffix;
  }

}
```

现在编写service中的内容。**此时HelloProperties已经和配置文件绑定，但还未注入到容器，所以执行时`@Autowired`会出错**。

```java
public class HelloServlet {

    @Autowired
    private HelloProperties helloProperties;

    public String sayHello(String userName) {
        return helloProperties.getPrefix()+" : " + userName + " >> " + helloProperties.getSuffix();
    }
}

```

最后编写自动配置类

```java
@Configuration
@EnableConfigurationProperties(HelloProperties.class)
@ConditionalOnMissingBean(HelloService.class)
public class HelloServiceAutoConfiguration implements Serializable {

  @Bean
  public HelloService helloService() {
    return new HelloService();
  }

}
```

- `@EnableConfigurationProperties(HelloProperties.class)` 

  当这个配置类生效后，将这个properties注入到容器中

- `@ConditionalOnMissingBean(HelloService.class)`
  如果已经有一个不知道通过什么奇奇怪怪的手段注入到容器中的Bean，那么这个自动配置类就不生效



最后需要添加一个**spring.factories**开启这个自动配置类

```yaml
# auto configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
top.ctong.springboothelloservicestarterautoconfigure.auto.HelloServiceAutoConfiguration
```

> 在resources/MATE-INF下



完成之后需要将我们编写的starter打包并安装到本地maven库，先操作`spring-boot-hello-service-starter-autoconfigure`

最后操作`customer-starter`这样自定义的starter就好了。



现在新建一个SpringWeb项目测试，在这个项目中引入刚刚自定义的starter

```xml
<dependencies>
  <dependency>
    <groupId>top.ctong</groupId>
    <artifactId>customer-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
  </dependency>
</dependencies>
```



在测试项目的配置文件中添加需要的配置

```yaml
clover.hello.prefix=CLOVER
clover.hello.suffix=CLOVER NP~~
```

编写一个Controller测试

```java
@RestController
public class HelloController {

  public final HelloService helloService;

  @Autowired
  public HelloController(HelloService helloService) {
    this.helloService = helloService;
  }

  @RequestMapping("/hello")
  public String sayHello() {
    return helloService.sayHello("Clover");
  }
}
```

![截屏2021-07-18 18.36.25](http://qiniu-note-image.ctong.top/note/images/202112271049864.png)



## SpringBoot 原理

Spring原理、SpringMVC原理、自动配置原理、SpringBoot原理



### SpringApplication创建初始化流程

- 创建SpringApplication

  ```java
  public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return new SpringApplication(primarySources).run(args);
  }
  ```

  > `org.springframework.boot.SpringApplication#run(java.lang.Class<?>[], java.lang.String[])`

  - 保存当前应用程序信息

    ```java
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
      this.webApplicationType = WebApplicationType.deduceFromClasspath();
    }
    ```

    > `org.springframework.boot.SpringApplication#SpringApplication(org.springframework.core.io.ResourceLoader, java.lang.Class<?>...)`

  - 创建初始启动器：`Bootstrapper.class`
    去所有的spring.factories文件中找`org.springframework.boot.Bootstrapper`

    ```java
    public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
      this.bootstrapRegistryInitializers = getBootstrapRegistryInitializersFromSpringFactories();
    }
    ```

  - 在所有spring.factories中找所有的`ApplicationContextInitializer`

  - 在所有spring.factories中找所有的`ApplicationListener`

- 运行`SpringApplication`

  > `org.springframework.boot.SpringApplication#run(java.lang.String...)`

  - 监控整个应用启动/停止：StopWatch 

    - 记录应用启动时间

  - 创建引导上下文

    > `org.springframework.boot.SpringApplication#createBootstrapContext`

    - 创建默认上下文

      ```java
      DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
      ```

      

    - 获取之前保存的所有`org.springframework.boot.SpringApplication#bootstrapRegistryInitializers`，然后遍历它，调用每一项中的`initialize`方法，因为它是`bootstrapRegistryInitializers`,是一个`org.springframework.boot.BootstrapRegistryInitializer`接口

      ```java
      @FunctionalInterface
      public interface BootstrapRegistryInitializer {
      
        void initialize(BootstrapRegistry registry);
      
      }
      ```

      给`initialize`方法传入上一步创建的上下文`DefaultBootstrapContext`来完成对引导启动器上下文环境设置

      ```java
      this.bootstrapRegistryInitializers.forEach((initializer) -> initializer.initialize(bootstrapContext));
      ```

  - 让当前应用进入headlless模式

  - 获取并保存运行时监听器

    ```java
    SpringApplicationRunListeners listeners = getRunListeners(args);
    ```

  - 调用所有监听器的`starting`方法
    通知所有监听器，当前项目正在启动

    > `org.springframework.boot.SpringApplicationRunListener#starting(org.springframework.boot.ConfigurableBootstrapContext)`

  - 保存命令行参数 

    ```java
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
    ```

  - 准备环境

    ```java
    ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
    ```

    > `org.springframework.boot.SpringApplication#prepareEnvironment`

    - 准备一个基础环境，如果不存在则创建

      ```java
      ConfigurableEnvironment environment = getOrCreateEnvironment();
      ```

    - 配置环境信息

      ```java
      configureEnvironment(environment, applicationArguments.getSourceArgs());
      ```

      - 配置转换器

        ```java
        if (this.addConversionService) {
         environment.setConversionService(new ApplicationConversionService());
        }
        ```

      - 加载外部配置信息，读取所有配置源属性值

        ```java
        configurePropertySources(environment, args);
        ```

    - 绑定环境信息

      ```java
      ConfigurationPropertySources.attach(environment);
      ```

    - 调用所有监听器`environmentPrepared`方法

      > 此时所有环境已经准备完成

      ```java
      listeners.environmentPrepared(bootstrapContext, environment);
      ```

    - 判断如果没有自定义环境信息就使用默认的环境

      ```java
      if (!this.isCustomEnvironment) {
        environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
                                                                                               deduceEnvironmentClass());
      }
      ```

    - 最后再把环境信息合并

      ```java
      ConfigurationPropertySources.attach(environment);
      ```

  - **创建IOC容器**

    ```java
    context = createApplicationContext();
    ```

    根据项目类型进行创建

    ```java
    ApplicationContextFactory DEFAULT = (webApplicationType) -> {
      try {
        switch (webApplicationType) {
          case SERVLET:
            return new AnnotationConfigServletWebServerApplicationContext();
          case REACTIVE:
            return new AnnotationConfigReactiveWebServerApplicationContext();
          default:
            return new AnnotationConfigApplicationContext();
        }
      }
      catch (Exception ex) {
        throw new IllegalStateException("Unable create a default ApplicationContext instance, "
                                        + "you may need a custom ApplicationContextFactory", ex);
      }
    };
    
    ConfigurableApplicationContext create(WebApplicationType webApplicationType);
    ```

  - 准备IOC容器上下文信息

    ```java
    prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
    ```

    > org.springframework.boot.SpringApplication#prepareContext

    - 保存前面准备好的基础环境到容器

      ```java
      context.setEnvironment(environment);
      ```

    - 在这一步之前，IOC默认配置都准备得差不多了，这一步的目的是使用`ApplicationContextInitializer`来对进行扩展。

      ```java
      protected void applyInitializers(ConfigurableApplicationContext context) {
        for (ApplicationContextInitializer initializer : getInitializers()) {
          Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(),
                                                                          ApplicationContextInitializer.class);
          Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
          initializer.initialize(context);
        }
      }
      ```

    - 在容器准备完后再调用所有`SpringApplicationRunListener`中的`contextPrepared(context)`方法

    - 获取Bean工厂，**这个工厂是在容器new的过程中被初始化的，具体在`org.springframework.context.support.GenericApplicationContext#GenericApplicationContext()`**

      ```java
      ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
      ```

    - 将命令行参数组册到容器

      ```java
      beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
      ```

    - `org.springframework.boot.SpringApplication#prepareContext`这个方法准备完成之后再调用所有的监听器中的`contextLoaded(context)`方法。

  - 刷新容器

    ```java
    refreshContext(context);
    ```

    - 创建容器中的所有组件

  - 记录容器启动完成花费的时间

    ```java
    stopWatch.stop();
    ```

  - 容器启动完成之后调用所有监听器的`started(context)`方法

    ```java
    listeners.started(context);
    ```

  - 调用所有runners

    ```java
    callRunners(context, applicationArguments);
    ```

    - 获取容器中的`ApplicationRunner`

      ```java
      runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
      ```

    - 获取容器中的`CommandLineRunner`

      ```java
      runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
      ```

    - 合并所有runner并且安装`@Order`进行排序

      ```java
      AnnotationAwareOrderComparator.sort(runners);
      ```

    - 遍历所有的runner调用`run`方法

      ```java
      for (Object runner : new LinkedHashSet<>(runners)) {
        if (runner instanceof ApplicationRunner) {
          callRunner((ApplicationRunner) runner, args);
        }
        if (runner instanceof CommandLineRunner) {
          callRunner((CommandLineRunner) runner, args);
        }
      }
      ```

      ```java
      private void callRunner(ApplicationRunner runner, ApplicationArguments args) {
        try {
          (runner).run(args);
        }
        catch (Exception ex) {
          throw new IllegalStateException("Failed to execute ApplicationRunner", ex);
        }
      }
      ```

  - 如果以上有异常，会调用所有监听器的`listeners.failed(context, exception);`方法

      ```java
      try {...} catch (Throwable ex) {
        handleRunFailure(context, ex, listeners);
        throw new IllegalStateException(ex);
      }
      ```

  - 没有任何异常发生时，会调用所有监听器的running方法
  
      ```java
      listeners.running(context);
      ```
  
  - 如果`listeners.running(context);`中有异常，会继续调用
  
      ```java
      handleRunFailure(context, ex, null);
      ```
  
  - 到此，SpringBoot启动成功



## 自定义监听组件



### ApplicationContextInitializer

这需要到`spring.factories`中注册

```java
@Slf4j
public class MyApplicationContextInitializer
        implements Serializable, ApplicationContextInitializer<ConfigurableApplicationContext> {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        log.info("容器初始化...");
    }

}
```

```yaml
org.springframework.context.ApplicationContextInitializer=\
  com.example.springapplication.listener.MyApplicationContextInitializer
```



### ApplicationListener

这需要到`spring.factories`中注册

```java
@Slf4j
public class MyApplicationListener implements Serializable, ApplicationListener<ApplicationEvent> {

    private static final long serialVersionUID = 6443715384736562018L;

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        log.info("MyApplicationListener： 绑定事件");
    }

}
```

```yaml
org.springframework.context.ApplicationListener=\
  com.example.springapplication.listener.MyApplicationListener
```



### SpringApplicationRunListener

这需要到`spring.factories`中注册

```java
@Slf4j
public class MySpringApplicationRunListener implements Serializable, SpringApplicationRunListener {

  private static final long serialVersionUID = -8494070955501588597L;
  
  private SpringApplication app;

  public MySpringApplicationRunListener(SpringApplication app, String[] args) {
		this.app = app;
  }


  @Override
  public void starting(ConfigurableBootstrapContext bootstrapContext) {
    log.info("MySpringApplicationRunListener starting方法执行");
    SpringApplicationRunListener.super.starting(bootstrapContext);
  }

  @Override
  public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext,
                                  ConfigurableEnvironment environment) {
    log.info("MySpringApplicationRunListener environmentPrepared方法执行");
    SpringApplicationRunListener.super.environmentPrepared(bootstrapContext, environment);
  }
}
```

```java
org.springframework.boot.SpringApplicationRunListener=\
  com.example.springapplication.listener.MySpringApplicationRunListener
```





### ApplicationRunner

在底层`ApplicationRunner`是从容器中获取，所以需要注册到容器

```java
@Component
@Slf4j
public class MyApplicationRunner {

    @Bean
    public ApplicationRunner applicationRunner() {
      return args -> log.info("applicationRunner runner...执行");
    }

}
```



### CommandLineRunner

```java
@Component
@Slf4j
public class MyApplicationRunner {

		@Bean
    public CommandLineRunner commandLineRunner() {
        return args -> log.info("commandLineRunner run...执行");
    }

}
```