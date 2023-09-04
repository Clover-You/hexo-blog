title: MyBatis 配置结构
categories: 随笔
tags: [随笔,JAVA]
date: 2022-01-02 18:23:27
---
`pom.xml` 配置文件添加以下代码，配置MyBatis运行时扫描的路径，否则可能会导致找不到 `Mapper`

```xml

<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ctong.crm</groupId>
    <artifactId>crm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <resources>
            <resource>
                <directory>src/main/java</directory><!--所在的目录-->
                <includes><!--包括目录下的.properties,.xml文件都会扫描到-->
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
    </build>
</project>


```

`resources` 包下放 `mybatis-config.xml` ，这是MyBatis的配置文件，该包下再放一个 `jdbc.properties` 里面配置数据库连接的参数。

`mybatis-config.xml`配置如下：

```xml   

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <!-- 配置一个资源配置文件 -->
  <properties resource="jdbc.properties"/>
  
  <typeAliases>
    <!-- 这里配置的是一个类型 -->
  	<package name="com.ctong.crm.domain"/>
  	
  </typeAliases>
  
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
         <!-- 这里根据`jdbc.properties`的配置来配置连接数据库 -->
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>
 
  <mappers>
      <!-- 这里配置的是你的dao层的包名 -->
      <package name="com.ctong.crm.dao"/>
  </mappers>
</configuration>

```

例如dao层有一个 `UserDao.java` 接口，它的内容如下：
```java
package com.ctong.crm.dao;
public interface UserDao {

    /**
     * find user in the database...
     * @param map query parameter
     * @return return target user or null
     */
    User login(Map<String, String> map);

}
```
那么需要操作数据库时你还得需要一个对应的xml，它可以在任何位置（前提是mybatis能够扫描到），最好规范存放它，我将它存放到dao层，与对应的接口 `UserDao.java` 同级

```xml

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.ctong.crm.dao.UserDao">
    <select id="login" resultType="User">
        select id, loginAct, name, loginPwd, email, expireTime, lockState, deptno, allowIps, createTime, createBy, editTime, editBy
        from tbl_user
        where loginAct=#{loginAct}
    </select>
</mapper>

```
注意，`namespace` 用来对应你的dao层接口，你代码调用这个dao层接口的时候MyBatis会对应的根据这个 `namespace` 来使用对应的接口，并且id跟接口中的方法名需对应...

