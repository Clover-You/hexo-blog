title: Spring Boot基础 - 学习笔记
categories: Spring
tags: [JAVA,SpringBoot,Spring,学习笔记,SpringMvc]
date: 2022-01-02 18:05:00
---
# Spring Boot2

> 2021/3/12 SpringBoot学习笔记

## 学习前准备

JDK版本官方推荐`v1.8`

```log
$ java -version 
openjdk version "1.8.0_282"
OpenJDK Runtime Environment (build 1.8.0_282-bre_2021_01_20_16_06-b00)
OpenJDK 64-Bit Server VM (build 25.282-b00, mixed mode)
```

检查是否安装了Maven

```log
$ mvn -version
Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
```

修改Maven的下载源与jdk版本，打开Maven安装路径，修改`conf/setting.xml`文件

```xml
<!-- 修改下载源为国内阿里源 -->
<mirrors>
  <mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
  </mirror>
</mirrors>

<!-- 使用jdk1.8 来进行项目编译-->
<profile>
  <profile>
    <id>jdk-1.8</id>
    <activation>
      <activeByDefault>true</activeByDefault>
      <jdk>1.8</jdk>
    </activation>
    <properties>
      <maven.compiler.source>1.8</maven.compiler.source>
      <maven.compiler.target>1.8</maven.compiler.target>
      <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
    </properties>
  </profile>        
</profile>
```



> SpringBoot2是基于jdk1.8编写，使用其他版本的jdk可能会出现问题，所以配置maven使用jdk1.8进行编译。



## HelloWorld

使用SpringBoot创建一个服务，浏览器对这个服务发送HTTP请求，响应`Hello, Spring Boot 2`



### 创建Maven工程

确认idea使用了我们自己的maven

![image-20210311225101834](http://qiniu-note-image.ctong.top/note/images/202112271053671.png)

点击：创建项目 ==>> maven ==>>  next ，输入你的项目路径以及名称后点击finish

![截屏2021-03-11 22.47.44](http://qiniu-note-image.ctong.top/note/images/202112271053682.png)

创建完成之后我们需要使用`Spring boot`的核心依赖`spring-boot-starter-parent`，在pom.xml文件中添加

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.4.3</version>
</parent>
```

> 当前使用的`Spring Boot`版本为`2.4.3`

若需要web开发，还需要导入相应的依赖

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
```



### 编写代码

在java文件夹下创建`Example`类

```java
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Example {
  
    public Example() { }
  
    public static void main(String[] args) {}
}
```

这里有个错误`Spring Boot Application in default package `，原因是当前这个启动类在默认包下，没有定义包，没有包自然就没有子包这个概念

```java
package com.upyou.hall;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Example {
  
    public Example() { }
  
    public static void main(String[] args) {}
}
```

![解决Spring Boot Application in default package错误](http://qiniu-note-image.ctong.top/note/images/202112271053691.png)

这样就行了

---

`@SpringBootApplication` 这个注解是把当前这个类标注为SpringBoot的启动类，通常也把标注了`@SpringBootApplication`注解的类称为主程序类...

当然，单单只有这个注解还不能让我们的主类跑起来，还需要通过SpringBoot提供的api`SpringApplication.run()`将这个类跑起来。

```java
@SpringBootApplication
public class Example {
  
    public Example() { }
  
    public static void main(String[] args) {
      SpringApplication.run(Example.class, args);
    }
}
```

> 一个模块只要有一个主程序类就行了

主类写好了之后无需过多的配置，可直接编写业务逻辑。

创建一个`controller`包，再创建一个`HelloController.java`文件

使用`@Controller`注解标注这个类，表示这是一个控制器

```java
package com.upyou.hall.controller;

import org.springframework.stereotype.Controller;

import java.io.Serializable;

/**
 * <p>
 * █████▒█      ██  ▄████▄   ██ ▄█▀     ██████╗ ██╗   ██╗ ██████╗
 * ▓██   ▒ ██  ▓██▒▒██▀ ▀█   ██▄█▒      ██╔══██╗██║   ██║██╔════╝
 * ▒████ ░▓██  ▒██░▒▓█    ▄ ▓███▄░      ██████╔╝██║   ██║██║  ███╗
 * ░▓█▒  ░▓▓█  ░██░▒▓▓▄ ▄██▒▓██ █▄      ██╔══██╗██║   ██║██║   ██║
 * ░▒█░   ▒▒█████▓ ▒ ▓███▀ ░▒██▒ █▄     ██████╔╝╚██████╔╝╚██████╔╝
 * ▒ ░   ░▒▓▒ ▒ ▒ ░ ░▒ ▒  ░▒ ▒▒ ▓▒     ╚═════╝  ╚═════╝  ╚═════╝
 * ░     ░░▒░ ░ ░   ░  ▒   ░ ░▒ ▒░
 * ░ ░    ░░░ ░ ░ ░        ░ ░░ ░
 * ░     ░ ░      ░  ░
 * <p>
 * Copyright Copyright 2021 UpYou.
 * </p>
 *
 * @author UpYou
 * @version V1.0
 * @file HelloController.java
 * @class HelloController
 * @description
 * @create 2021-03-13 21:39
 */
@Controller
public class HelloController implements Serializable {
  
    private static final long serialVersionUID = -8638687587976398521L;

    public HelloController() { }

}
```

在这个控制器需要处理`hello`请求，先声明一个方法处理`hello`并返回`'Hallo, Spring Boot2'`

```java
@Controller
public class HelloController implements Serializable {

  private static final long serialVersionUID = -8638687587976398521L;

  public HelloController() { }

  @ResponseBody
  @RequestMapping("/hallo")
  public String handleHallo() {
    return "Hello, Spring Boot2";
  }

}
```

`@RequestMapping`注解是当遇到**"/hallo"**这个请求时使用被标注的方法处理

`@ResponseBody`表示我们当前返回的数据是以字符串的方式写给浏览器



现在启动我们的程序并使用浏览器发送一个"/hallo"请求

![浏览器发送hallo请求](http://qiniu-note-image.ctong.top/note/images/202112271053692.png)

> 在未来我们一个控制器不可能只处理一个请求，可能都要给这些请求返回某些字符，如果不想每处理一个请求都标注`@ResponseBody`，可以将`@ResponseBody`标注在这个控制器上，这代表这个控制器下请求响应的所有数据都是直接写给浏览器的而不是跳转某个页面。当然，如果觉得使用两个注解麻烦，Spring还提供了`@RestController`可以替代这两个注解
>
> ```java
> @RestController
> public class HelloController implements Serializable {}
> ```



## 简化配置

SprintBoot是来整合其它所有东西的一个框架，可以将我们未来所有的配置都可以抽取在一个配置文件当中。这个配置文件固定名字叫`application.properties`。

在这个配置文件中可以修改Tomcat的一些设置包括SpringBoot的一些设置，都可以在这修改。

例如修改Tomcat启动端口

```properties
# 修改tomcat端口号
server.port=8888
```

这些设置，你可以在SpringBoot的[官方文档](https://docs.spring.io/spring-boot/docs/2.4.3/reference/html/appendix-application-properties.html#common-application-properties)中找到



## 依赖管理

父项目的主要做用是用来做依赖管理，这不需要我们人为的去干涉，在父项目的父项目中，也就是`spring-boot-dependencies`，声明了当前使用的这个版本中，几乎声明了我们开发的所有常用依赖

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.4.3</version>
</parent>
```

- 父项目引入后，子项目以后引入依赖，就不需要太过注意版本号了。这也叫自动版本仲裁...

- SpringBoot默认引入的版本号都是当前SpringBoot所支持的最高版本，因为每个人的开发环境不一样，所以我们可以根据自己的需求去进行修改。

- 除了在`dependency`中添加版本号以外，还可以在`properties`中添加，这样可以更方便管理我门自定义的版本号。例如需要修改mysql的版本号

  ```xml
  <properties>
  	<mysql.version>5.1.49</mysql.version>
  </properties>
  ```

  > 修改版本号之前，需要去[maven repertory](https://mvnrepository.com)中查看这个依赖的版本号列表



### Starters场景

以后我们经常会见到这种引入方式`spring-boot-starter-*`，其中`*`就是代表某种场景，只要引入starter，那么这个场景的所有常规的依赖SpringBoot都会自动帮我们引入

例如`spring-boot-starter-web`这个依赖，我们引入之后，他就会自动帮我们把`web`场景下的所有常规依赖引入，其实也没那么神奇，看看`spring-boot-starter-web`的`pom.xml`就知道了

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.4.3</version>
    <scope>compile</scope>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
    <version>2.4.3</version>
    <scope>compile</scope>
  </dependency>
  .....
</dependencies>
```

SpringBoot所有场景都可以在官方文档找到：[传送门](https://docs.spring.io/spring-boot/docs/2.4.3/reference/html/using-spring-boot.html#using-boot-starter)

在未来如果见到：`*-spring-boot-starter`它并不是SpringBoot为我们提供的，而是第三方为我们提供的场景启动器。 

![分析依赖树](http://qiniu-note-image.ctong.top/note/images/202112271053699.png)



> 在SpringBoot中，引入依赖推荐不写版本号，**但这不是绝对的**。



## 自动配置

- 自动配好Tomcat

  1. SpringBoot自动帮我们引入Tomcat依赖
  2. 配置Tomcat

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <version>2.4.3</version>
    <scope>compile</scope>
  </dependency>
  ```

  

- 自动配置好SpringMVC

  1. 引入SpringMVC全套组件
  2. 自动配置好了Spring MVC的常用组件「功能」

- 自动配好Web常见功能

  1. SpringBoot帮我们配置好了所有**web**开发的场景

- 默认的包结构

  1. 主程序所在的包及其下面的所有子包里面的组件都会被扫描到，不需要我们手动的去配置扫描路径。

  2. 如果需要自定义扫描路径，可以使用`@SpringBootApplication`注解提供的`scanBasePackages`属性进行定义

     ```java
     @SpringBootApplication(scanBasePackages = "java.com.xxx")
     public class Example {}
     ```

- 在SpringBoot提供的默认配置中，每一个配置都有默认值，这些默认配置都是映射到某一个类中的。例如：`spring.servlet.multipart.file-size-threshold`，这是一个现在文件上传的大小，这个属性的值会被绑定到`org.springframework.boot.autoconfigure.web.servlet.MultipartProperties`类中

- 这些自动配置项并不是全部加载，而是你当前项目中引入了哪些场景，那么这个场景的自动配置才会被加载。SpringBoot所有自动配置功能都在`spring-boot-autoconfigure`依赖中

  > 自动配置，按需加载 



### 自动配置原理

在能够扫描到的包下创建一个配置类，使用`@Configuration`注解标注它，用于告诉SpringBoot这是一个配置类，这等同于SpringMVC的配置文件

```java
package com.upyou.hall.config;

import org.springframework.context.annotation.Configuration;
import java.io.Serializable;

@Configuration
public class MyConfig implements Serializable {

    private static final long serialVersionUID = -7095527472630716540L;

    public MyConfig() { }

}
```

使用`@Bean`注解标准一个方法，这个方法将给Spring容器返回一个组件。以方法名作为主键id，方法返回值类型作为组件类型，返回值就是组件在容器中的实例。如果不需要将方法名用作组件名，也可以使用`@Bean`提供的`value`属性进行自定义

```java
@Bean
public User user() {
  return new User("UpYou", 18);
}
```

在主程序中验证我们的组件是否成功添加到Spring容器，通过我们的组件名称可以获取

```java
@SpringBootApplication()
public class Example {

    public Example() { }

    public static void main(String[] args) {
        ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
				User user = app.getBean("user", User.class);
        System.out.print(user);
    }
}
```

![获取自定义组件](http://qiniu-note-image.ctong.top/note/images/202112271053702.png)

组件注册时默认时注册为单实例组件，无论获取多少次，都是同一个组件。

```java
@SpringBootApplication()
public class Example {

    public Example() { }

    public static void main(String[] args) {
        ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
        User user = app.getBean("user", User.class);
        User user2 = app.getBean("user", User.class);
        System.out.print(user == user2);
    }
}
```

![单实例组件验证结果](http://qiniu-note-image.ctong.top/note/images/202112271053766.png)

在我们配置类中用`@Bean`标注的方法是一个组件，但被`@Configuration`标注的类也是一个组件

```java
@SpringBootApplication()
public class Example {

    public Example() { }

    public static void main(String[] args) {
        ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
        System.out.println(app.getBean(MyConfig.class));
    }
}
```

输出

`com.upyou.hall.config.MyConfig$$EnhancerBySpringCGLIB$$1a6812ad@2375b321`

---

## 容器

### 组件

#### Configuration代理

在`@Configuration`中，有一个属性`proxyBeanMethods`，它是指定当前被标注的类中的`@Bean`是否使用代理，默认`true`，如果这个类中的`@Bean`被代理，那么调用这个类中的方法的时候是通过代理去调用的「单例」。如果这个类中`@Bean`被代理，那么SpringBoot总会去检查容器中是否存在这个组件，如果不存在则创建，保持组件的单实例。

测试不代理：

```java
@Configuration(proxyBeanMethods = false)
public class MyConfig implements Serializable {...}
```

再去调用方法，检查内存地址是否还是相同

```java
public static void main(String[] args) {
  ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
  MyConfig user = app.getBean(MyConfig.class);
  System.out.print(user.user() == user.user());
}
```

![对象不被代理测试](http://qiniu-note-image.ctong.top/note/images/202112271053771.png)



- 配置类中使用@Bean标注在方法上给容器注册组件，默认是单实例
- 配置类本身也是组件
- `proxyBeanMethods` 代理`@Bend`的方法
  Full（proxyBeanMethods=true）
  Lite（proxyBeanMethods=false）



> 如果没有组件依赖我们的组件，那么就不需要使用代理，不使用代理还可以提升Spring启动速度，如果被依赖，那么就使用代理，可以保证使用的就是这个容器中的组件。



#### @Import

通过`@Import`注解可以给当前被标注的容器导入组件

```java
import org.springframework.context.annotation.Import;
@Import({User.class})
public class MyConfig implements Serializable {...}
```

它可以给容器自动创建指定类型的组件，这些被`Import`创建的组件名字默认为完整类名`com.upyou.hall.bean.User`。`@Import`接受一个`Class<?>[]`类型

#### @Conditional

这是一个条件装配器，当满足`@Conditional`制定的条件的时候才会给容器注入相关的组件。 

使用idea的`control + h`查看`Conditional`类继承树，`conditional`有很多派生注解，每一个注解都代表着一个功能。

![Conditional派生注解](http://qiniu-note-image.ctong.top/note/images/202112271053962.png)



##### @ConditionalOnBean

当容器中有某个组件时才注册指定组件，例如以下示例代码，并没有注册`pet`组件

```java
@Configuration()
public class MyConfig implements Serializable {

    private static final long serialVersionUID = -7095527472630716540L;

    public MyConfig() { }

    @ConditionalOnBean(name = "pet")
    @Bean
    public User user() {
        return new User("UpYou", 18);
    }

    public Pet pet() {
        return new Pet("Tom");
    }

}

```

测试是否成功注册`user`组件

```java
ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
boolean user = app.containsBean("user");
System.out.println(user);
```

输出`false`，将`pet`注册时输出为`true`

> 注意：以上示例在注册了`pet`组件时，`app.containsBean("user");`结果仍然为`false`，这是因为组件注册时存在先后问题，让`pet`组件比`user`组件先注册就可以了。

**这里只做一个例子，因为这些派生注解用法都一样**



### 引入原生配置文件

#### @ImportResource

可以使用`@ImportResource`注解将xml中注册的组件注册到容器中，例如SpringMVC就使用了非常多的xml文件配置。

例如在`resources`文件夹中有一个`beans.xml`文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userForXml" class="com.upyou.hall.bean.User">
        <property name="age" value="18"/>
        <property name="name" value="upyou"/>
    </bean>
</beans>
```

这是一个老的组件注册方式，我们使用SpringBoot时这里配置的组件肯定无法注册到容器中。

测试在容器中是否有`xml`中注册的`userForXml`组件

```java
ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
boolean userForXml = app.containsBean("userForXml");
System.out.println(userForXml);
```

结果为`false`

顺便在某个配置类下使用`@ImportResource`导入我们的`xml`配置文件

```java
@Configuration()
@ImportResource("classpath:beans.xml")
public class MyConfig implements Serializable {...}
```

现在再测试是否有我们使用`xml`注册的组件

```java
ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
boolean userForXml = app.containsBean("userForXml");
System.out.println(userForXml);
```

结果输出为`true`



### 配置绑定

我们连接数据库，需要用到账号和密码，我们一般将连接信息写到一个`properties`配置文件中，但这样我们读取这个信息的时候就变的非常麻烦。可以使用SpringBoot提供的配置绑定来解决这些问题。

#### @ConfigurationProperties

这个注解有两个参数`prefix`、`value`，`prefix`是配置文件中的配置名的前缀，例如配置文件中有

```properties
mysql.bean.user=root
mysql.bean.pass=123
```

那么他的前缀就是`mysql.bean`，他会根据被标注的类中的属性名进行绑定，例如

```java
@Component
@ConfigurationProperties(prefix = "mysql.bean")
@Data
public class ConnMysql implements Serializable {

  private static final long serialVersionUID = -8865269478790296187L;

  // 将mysql.bean.user的值绑定到此处
  private String user;

  // 将mysql.bean.pass的值绑定到此处
  private String pass;

  public ConnMysql() { }

```

测试一下这个两个字段是否已经将配置文件中的值绑定。

```java
ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
ConnMysql bean = app.getBean(ConnMysql.class);
System.out.println(bean.getUser());
System.out.println(bean.getPass());
```

分别输出`root,123`



> 只有在容器中的组件，才能拥有SpringBoot提供的`ConfigurationProperties`



#### @EnableConfigurationProperties

这个注解是与`@Configuration`一起使用，它可以开启指定类的属性配置绑定功能，并且将这个组件注册到此容器

以下代码是没有使用`@Component`注解的，此时idea可能会警告，忽略即可。

```java
@ConfigurationProperties(prefix = "mysql.bean")
@Data
public class ConnMysql implements Serializable {

  private static final long serialVersionUID = -8865269478790296187L;

  // 将mysql.bean.user的值绑定到此处
  private String user;

  // 将mysql.bean.pass的值绑定到此处
  private String pass;

  public ConnMysql() { }


```

再在我们的配置文件中使用`@EnableConfigurationProperties`使用这个组件

```java
@Configuration()
@EnableConfigurationProperties(ConnMysql.class)
public class MyConfig implements Serializable {

    private static final long serialVersionUID = -7095527472630716540L;

    public MyConfig() { }
  
}

```

此时再测试是否成功导入

```java
ConfigurableApplicationContext app = SpringApplication.run(Example.class, args);
ConnMysql bean = app.getBean(ConnMysql.class);
System.out.println(bean.getUser());
System.out.println(bean.getPass());
```

分别输出`root,123`



## 自动配置原理

### 引导加载自动配置类

```java

@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {...}
```



#### @SpringBootConfiguration

`@SpringBootConfiguration`就是`@Configuration`，它代表当前是一个配置类。



#### @ComponentScan

指定包扫描路径



#### @EnableAutoConfiguration

```java
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {...}

```



##### @AutoConfigurationPackage

```java

@Import({Registrar.class})
public @interface AutoConfigurationPackage {}
```

`@import`导入的并不只是一个组件，`Registrar`给容器批量导入了非常多的组件

```java
public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
  AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
}
```

以上代码将指定包下的所有组件导入进来「被`SpringBootConfiguration`标注的类所在的包」



##### @Import

核心代码

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
  AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
}
```

利用`getAutoConfigurationEntry(annotationMetadata)`给容器中批量导入组件

调用`List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);`获取到所有需要导入到容器的配置类。

![待导入的组件](http://qiniu-note-image.ctong.top/note/images/202112271053455.png)

通过工厂加载器`Map<String, List<String>> loadSpringFactories(ClassLoader classLoader) `加载得到所有组件

从`META-INF/spring.factories`位置来加载一个文件「默认扫描我们当前系统里面所有`META-INF/spring.factories`位置的文件」`spring-boot-autoconfigure-2.4.3.jar`中就有`META-INF/spring.factories`，这里面配置了需要系统启动就需要加载的东西，这些配置文件都是写死的。



### 按需开启自动配置项

虽然在`spring-boot-autoconfigure-2.4.3.jar`中定义了非常多的场景，并且这些场景的所有自动配置启动的时候默认全部加载，按照条件装配规则，最终都会被按需配置



随便拿个包，例如`org.springframework.boot.autoconfigure.aop`

这个场景我们并没有使用，索引并不会给我门开启，因为这个配置类中使用了`@ConditionalOnClass`

```java
public class AopAutoConfiguration {

	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass(Advice.class)
	static class AspectJAutoProxyingConfiguration {...}
}
```

指定`Advice.class`类存在的时候才开启这个配置类，因为我们没有`Advice`相关的包，所以并不会给我们导入`AspectJAutoProxyingConfiguration`这个配置类

> Spring Boot默认会在底层配好所有的组件，但是如果用户自己配置了那么以用户的优先。



- SpringBoot 先加载所有的自动配置类
- 每个自动配置类都按照条件进行生效，默认都会绑定配置文件指定的值。
- 生效的配置类都会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就都有了
- 只要用户有自己配置的，就以用户优先 
- 定制化配置
  - 自己`@Bean`替换底层的组件
  - 去看这个组件是获取的配置文件有什么值，最后按照规定进行修改即可。