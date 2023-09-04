title: ssm整合
categories: 随笔
tags: [Spring,SpringMvc,JAVA,随笔]
date: 2022-01-02 18:30:03
---
[源码GitHub地址](https://github.com/YouChuantong/ssm_Integration)



# 导包

整的的第一步...先导包

- Spring

  ```
  AOP增强
  com.springsource.net.sf.cglib-2.2.0.jar
  com.springsource.org.aopalliance-1.0.0.jar
  com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar
  
  AOP核心
  spring-aspects-4.0.0.RELEASE.jar
  
  IOC核心包
  commons-logging-1.1.3.jar
  spring-aop-4.0.0.RELEASE.jar
  spring-beans-4.0.0.RELEASE.jar
  spring-context-4.0.0.RELEASE.jar
  spring-core-4.0.0.RELEASE.jar
  spring-expression-4.0.0.RELEASE.jar
  spring-tx-4.0.0.RELEASE.jar
  
  测试核心包
  spring-test-4.0.0.RELEASE.jar
  
  JDBC核心包
  spring-jdbc-4.0.0.RELEASE.jar
  spring-orm-4.0.0.RELEASE.jar
  ```

  

- SpringMVC

  ```
  MVC核心包
  spring-web-4.0.0.RELEASE.jar
  spring-webmvc-4.0.0.RELEASE.jar
  
  文件上传下载 核心包
  commons-io-2.0.jar
  commons-fileupload-1.2.1.jar
  
  JSR303校验核心包
  hibernate-validator-annotation-processor-5.0.3.Final.jar
  hibernate-validator-cdi-5.0.3.Final.jar
  hibernate-validator-5.0.3.Final.jar
  classmate-1.0.0.jar
  jboss-logging-3.1.1.GA.jar
  validation-api-1.1.0.Final.jar
  
  jstl-jsp标准标签库
  jstl.jar
  standard.jar
  
  ajax/JSON支持
  jackson-databind-2.1.5.jar
  jackson-core-2.1.5.jar
  jackson-annotations-2.1.5.jar
  ```

  

- MyBatis

  ```
  MyBatis核心包
  mybatis-3.4.1.jar
  
  日志框架
  log4j-1.2.17.jar
  slf4j-log4j12-1.7.30.jar
  slf4j-api-1.7.30.jar
  
  MyBatis与Spring整合包
  mybatis-spring-1.3.0.jar
  ```

  

- 其他

  ```
  数据库驱动
  mysql-connector-java-8.0.21.jar
  
  数据源
  c3p0-0.9.1.2.jar
  ```

  

# 配置



## Spring



### web.xml

启动Spring容器，在web.xml文件中配置Spring容器：

```xml
<!--  启动Spring 容器 -->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <!--  指定配置文件位置  -->
  <param-value>classpath:spring-config.xml</param-value>
</context-param>
<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```

### 自动包扫描

在`spring-config.xml` 开启自动包扫描，Spring不需要接管`Controller`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
  <!--  自动包扫描  -->
  <context:component-scan base-package="top.ctong.ssm">
    <!--  排除Controller注解标注的类 -->
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
  </context:component-scan>
</beans>
```



### 配置数据源

```xml
<!--  导入jdbc配置文件  -->
<context:property-placeholder location="jdbc.properties"/>

<!--  配置c3p0数据源  -->
<bean id="comboPooledDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
  <property name="user" value="${jdbc.user}"/>
  <property name="password" value="${jdbc.pass}"/>
  <property name="jdbcUrl" value="${jdbc.url}"/>
  <property name="driverClass" value="${jdbc.driver}"/>
</bean>
```

`jdbc.properties` 配置文件

```properties
jdbc.user=root
jdbc.pass=123
jdbc.database=mybatis
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mybatis
```



### 事务控制

配置事务管理器，让他控制住数据源里面的连接的关闭和提交

```xml
<!--  配置事务控制  -->
<bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="comboPooledDataSource"/>
</bean>
```

配置事务切面

```xml
<!--  事务增强、事务属性、事务建议
  - transaction-manager：指定事务管理器
  -->
<tx:advice id="myTx" transaction-manager="dataSourceTransactionManager">
  <!--  配置事务属性  -->
  <tx:attributes>
    <!--  任何方法发生异常就回滚
	   - rollback-for: 触发指定异常时回滚
  	 -->
    <tx:method name="*" rollback-for="java.lang.Exception"/>
    <!--  get开头的方法设置只读，可优化性能  -->
    <tx:method name="get*" read-only="true"/>
  </tx:attributes>
</tx:advice>

<!--  基于xml配置，配置事务  -->
<aop:config>
  <aop:pointcut id="txPoint" expression="execution(* top.ctong.ssm.service..*(..))" />
  <aop:advisor advice-ref="myTx" pointcut-ref="txPoint" />
</aop:config>
```





## SpringMVC



### web.xml

在`web.xml` 中配置SpringMVC前端控制器

```xml
<!--  配置SpringMVC前端控制器  -->
<servlet>
  <servlet-name>dispatcherServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc-config.xml</param-value>
  </init-param>
</servlet>
<servlet-mapping>
  <servlet-name>dispatcherServlet</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>
```

在`web.xml` 中配置字符编码

```xml
<!--  字符编码  -->
<filter>
  <filter-name>characterEncodingFilter</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>utf-8</param-value>
  </init-param>
  <init-param>
    <param-name>forceEncoding</param-name>
    <param-value>true</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>characterEncodingFilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

配置REST风格过滤器

```xml
<!--  REST风格  -->
<filter>
  <filter-name>hiddenHttpMethodFilter</filter-name>
  <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>hiddenHttpMethodFilter</filter-name>
  <url-pattern>/</url-pattern>
</filter-mapping>
```

### 自动包扫描

在`springmvc-config.xml`开启自动包扫描，需要禁用默认行为，因为默认行为是全包扫描，SpringMVC只需要接管Controller类

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
  <!--  开启自动包扫描，禁用默认行为  -->
  <context:component-scan base-package="top.ctong.ssm" use-default-filters="false">
    <!--  接管被Controller接管的类  -->
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
  </context:component-scan>
</beans>
```

### 资源访问配置

防止静态或动态资源失效。加上这两个配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
  <!--  静态资源映射  -->
  <mvc:default-servlet-handler/>
  <!--  动态资源映射  -->
  <mvc:annotation-driven/>
</beans>
```

### 文件上传解析器

```xml
<!--  文件上传解析器  -->
<bean id="multipartResolver"
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
  <!--  最大内存大小  -->
  <property name="maxUploadSize" value="#{1024 * 1024 * 100}"/>
  <!--  单文件最大内存大小-->
  <property name="maxInMemorySize" value="#{1024 * 1024 * 10}"/>
  <!-- 默认编码格式-->
  <property name="defaultEncoding" value="utf-8"/>
</bean>
```



## MyBatis

在`spring-config.xml`配置文件中整合Mybatis

```xml
<!--  整合MyBatis, SqlSessionFactoryBean可以根据配置文件得到SqlSessionFactory  -->
<bean id="sqlSessionFactoryBean" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!--  配置文件位置  -->
  <property name="configLocation" value="classpath:mybatis-config.xml"/>
  <!--  数据源  -->
  <property name="dataSource" ref="comboPooledDataSource"/>
  <!--  xml映射文件路径  -->
  <property name="mapperLocations" value="classpath:mapper/*.xml"/>
</bean>

<!--  把每个Dao接口的"实例"加入到IOC  -->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <!--  指定Dao接口所在的包  -->
  <property name="basePackage" value="top.ctong.ssm.dao"/>
</bean>
```




