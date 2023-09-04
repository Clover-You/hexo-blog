title: idea创建普通Web项目lib目录无法输出，tomcat不报错问题
categories: 随笔
tags: [JAVA,随笔]
date: 2022-01-02 18:20:09
---
# idea创建普通Web项目lib目录无法输出，tomcat不报错问题

idea版本：2021.2

tomcat版本：9.0.50



## 项目结构

创建一个普普通通的web项目，目录结构大概就是这样

```
.
├── .idea
│   ├── artifacts
│   ├── inspectionProfiles
│   ├── libraries
│   └── sonarlint
│       └── issuestore
├── conf
├── lib
├── out
│   ├── artifacts
│   │   └── test03_war_exploded
│   │       └── WEB-INF
│   │           ├── classes
│   │           │   └── top
│   │           │       └── ctong
│   │           │           └── controller
│   │           └── lib
│   └── production
│       └── test03
│           ├── generated
│           └── top
│               └── ctong
│                   └── controller
├── src
│   └── top
│       └── ctong
│           └── controller
└── web
    └── WEB-INF
```

`lib`文件夹已经设置为`Library`文件夹。

`controller`搞了个`hello`请求

```java
@Controller
public class TestController implements Serializable {

    private static final long serialVersionUID = 6290967590153117614L;

    @RequestMapping("/hello")
    public void test01() {
        System.out.println("Hello World!");
    }

}
```

配置了自动包扫描

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="top.ctong"/>
</beans>
```

加载了`DispatcherServlet`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```



## 问题

点启动，访问`/hello`成功荣获404….干掉`out`目录重启….404…

重新建一个项目，启动…..`hello world`…..

再创建新项目，启动…`404`…

打开`FileMerge`文件对比工具…嗯？差不多啊…

…打开`out`文件夹，嗯？怎么lib目录没加进来…

…反反复复2天后…

…

…

…



## 解决

尝试将`lib`目录拖进`web/WEB-INF`目录…`hello world`…

不信邪，再把`lib`拖到项目根目录…`404`…

……**MMP**……我之前一直这样都没问题…

不知道第几次打开 idea === >> Proiect Structure…

无意间点了 Proiect Settings  ===>> Artifacts…嗯？怎么有警告…..

![截屏2021-08-06 10.41.23](http://qiniu-note-image.ctong.top/note/images/202112271548244.png)

`Library lib ' required for module test03 is missing from the artifact` 嗯？莫名其妙双击了右边的`lib`。警告消失了…

启动…`hello world`…

什么妖魔鬼怪…