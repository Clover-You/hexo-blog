title: Spring与SpringMVC整合
categories: 随笔
tags: [SpringMvc,JAVA,随笔,Spring]
date: 2022-01-02 18:28:20
---
SpringMVC和Spring整合的目的是为了分工明确。例如：SpringMVC的配置文件就来配置和网站转发逻辑以及网站功能有关的配置，如：视图解析器、文件上传解析器、支持ajax....

而Spring的配置文件用来配置和业务有关的，如：事物控制、数据源....

# import

可以在`resources`文件夹下创建三个配置文件:`include-config.xml`、`spring-config`、`springmvc-confg`

在`include-config.xml`配置文件中使用spring提供的`import`标签引入并合并另外两个配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  <import resource="spring-config.xml"/>
  <import resource="springmvc-config.xml"/>
</beans>
```

这样就搞定了。。。



# Spring和SpringMVC分容器

Spring管理业务逻辑组件，SpringMVC管理控制器组件

在`SpringMVC`配置文件中指定扫描对象并且禁用默认规则

- `type` 指定排除方式

```xml
<context:component-scan base-package="top.ctong.springmvc" use-default-filters="false">
  <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

在`Spring` 配置文件中指定接管所有业务逻辑组件

```xml
<context:component-scan base-package="top.ctong.springmvc">
  <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

最后需要在`web.xml`文件中启动两个容器

```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:spring-config.xml</param-value>
</context-param>
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<servlet>
  <servlet-name>dispatcherServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc-config.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>dispatcherServlet</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```


