title: SpringMVC学习笔记
categories: Spring
tags: [Spring,SpringMvc,JAVA,学习笔记]
date: 2022-01-02 18:04:00
---
# SpringMVC学习笔记

- 使用Spring来实现Web模块，简化Web开发
- Spring为展现层提供的基于MVC设计理念的优秀的Web框架，是目前最主流的MVC框架之一
- Spring3.0后全面超越Struts2，成为最优秀的MVC框架
- SpringMVC通过一套MVC注解，让POJO(Plain Old Java Object/普通的Java对象)成为处理请求的控制器，而无需实现任何接口
- 支持REST风格的URL请求
- 采用了松散耦合可插拔组件结构，比其他MVC框架更具扩展性和灵活性



![MVC流程图](http://qiniu-note-image.ctong.top/note/images/R-C.4008554c694ec2cc55d912974aea6dd8.png)



## 导包

- SpringMVC是Spring的Web模块。所有模块的运行都是依赖核心模块（IOC）

  1. 核心容器模块

     ```
     commons-logging-1.1.3.jar
     spring-aop-4.0.0.RELEASE.jar
     spring-beans-4.0.0.RELEASE.jar
     spring-context-4.0.0.RELEASE.jar
     spring-core-4.0.0.RELEASE.jar
     spring-expression-4.0.0.RELEASE.jar
     ```

     

  2. Web模块

     ```
     spring-web-4.0.0.RELEASE.jar
     spring-webmvc-4.0.0.RELEASE.jar
     ```



## 配置

SpringMVC思想是有一个前端控制器能拦截所有请求并智能派发。这个前端控制器是一个**Servlet**。所以它应该在web.xml中配置这个servlet拦截所有请求。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
  <!-- 配置SpringMVC DispatcherServlet -->
  <servlet>
    <servlet-name>springDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <!--   contextConfigLocation: 指定配置文件路径     -->
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>springDispatcherServlet</servlet-name>
    <!-- /*和/都是拦截所有请求。但/*的范围更大，还会拦截*.jsp -->
    <url-pattern>/</url-pattern>
  </servlet-mapping>
</web-app>
```



## HelloWorld

创建一个MVC项目



### 编写处理器

新建一个类`top.ctong.controller.MyFirstController`，需要使用注解的方式告诉Spring这个是一个处理器，可以处理请求。

可以使用注解的方式，这需要`context`名称空间和进行包扫描

```java
@Controller
public class MyFirstController { ... }
```

在SpringMVC中，`@Controller`不能随便加，它会被区分为处理器。当然，你也不能使用其它注解代替`@Controller`注解。

接下在写一个方法来处理请求

```java
public void myFirstRequest() {
  System.out.println("Hello World！");
}
```

昨晚这些后还不够，你还要告诉SpringMVC你这个方法处理哪个请求，这时可以使用`@RequestMapping`。这个注解有一个`value`属性，这个属性用来指定你需要处理的路径

```java
@RequestMapping("/hello")
public void myFirstRequest() { ... }
```

用浏览器发送这个请求后后端得到打印结果`localhost:[your port]/hello`

```
Hello World！
```



### 转发

你的方法直接返回一个字符串，SpringMVC会自动进行转发，Servlet会根据这个路径尝试去资源中找对应的资源。

```java
/**
 * 转发到欢迎页
 * @return /welcome.jsp
 */
@RequestMapping("/welcome")
public String welcomePage() {
  return "/WEB-INF/pages/welcome.jsp";
}
```

![welcome页转发结果](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-07-29%2021.03.11.png)



### 视图解析器

如果我们要转发视图，需要返回视图的具体路径，但我们页面都是在同一个目录下`/WEB-INF/pages`。可以让视图解析器自动帮我们加上，而且页面都是`jsp`，也可以通过视图解析器配置。

```xml
<!--  配置视图解析器  -->
<bean class="org.springf ramework.web.servlet.view.InternalResourceViewResolver">
	<!--  前缀  -->
  <property name="prefix" value="/WEB-INF/pages/"/>
	<!--  后缀  -->
  <property name="suffix" value=".jsp"/>
</bean>
```

```java
/**
 * 转发到欢迎页
 * @return /welcome.jsp
 */
@RequestMapping("/welcome")
public String welcomePage() {
  return "welcome";
}
```



### 流程

1. 客户端发送请求
2. 来到tomcat服务器
3. SpringMVC的前端控制器收到所有请求
4. 来看请求地址和`@RequestMapping`标注的哪个匹配，来找到使用哪个类的哪个方法
5. 前端控制器找到了目标处理器和目标方法，直接利用反射执行目标方法
6. 方法执行完成之后会有一个返回值，SpringMVC认为这个返回值就是要去的页面地址
7. 拿到方法返回值以后，用视图解析器进行拼接字符串得到完整的页面地址
8. 拿到页面地址，前端控制器帮我们转发到页面



## @RequestMapping

`@RequestMapping`注解其实就是告诉SpringMVC这个方法用来处理什么请求。在控制器的类定义及方法定义处都可标注

- 类定义处
  提供初步的请求映射信息，相对于WEB应用的根目录
- 方法处
  提供进一步的细分映射信息，相对于类定义处的URL，若类定义处未标注`@RequestMapping`，则方法处标记的URL相对于WEB应用的根目录



> `DispatcherServlet`截获请求后，就通过控制器上的`@RequestMapping`提供的映射信息确定请求所对应的处理方法。



### 标注在类上

```java
@Controller
@RequestMapping("/request_mapping")
public class RequestMappingTestController implements Serializable {

  /**
	 * 测试RequestMapping
	 * @return welcome.jsp
	 */
  @RequestMapping("/test")
  public String testRequestMapping() {
    System.out.println("testRequestMapping输出了···");
    return "welcome";
  }

}
```

![标注在类上的RequestMapping测试结果](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-07-30%2011.06.59.png)



### 规定请求方式

`@RequestMapping`默认可以处理所有方式的请求，若需要限定请求方式，可以通过`method`属性指定。

它支持这么多种请求方式`GET` `HEAD` `POST` `PUT` `PATCH` `DELETE` `OPTIONS` `TRACE`

```java
/**
 * 测试RequestMapping的method属性
 */
@RequestMapping(value = "/testRequestMappingMethod", method = RequestMethod.POST)
public String testRequestMappingMethod() {
    return "welcome";
}
```

此时发送GET请求时，发生405错误

```http
GET {{url}}/{{rquest_mapping_api}}/test_request_mapping_method HTTP/1.1
```

```http
HTTP/1.1 405 Method Not Allowed
Server: Apache-Coyote/1.1
Allow: POST
Content-Type: text/html;charset=utf-8
Content-Language: zh-CN
Content-Length: 689
Date: Sat, 31 Jul 2021 04:18:44 GMT
Connection: close
```

使用对应请求方式请求通过

```http
POST {{url}}/{{rquest_mapping_api}}/test_request_mapping_method HTTP/1.1
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=FD4CF51D92D3A2105958A5AE4DC6DF54; Path=/SpringMvcHelloWorld_war_exploded; HttpOnly
Content-Type: text/html;charset=UTF-8
Content-Language: zh-CN
Content-Length: 1222
Date: Sat, 31 Jul 2021 04:21:09 GMT
Connection: close
```





### 规定请求参数

在`@RequestMapping`中，有个`params`参数，它用于规定请求中必须存在指定参数。它还支持简单表达式

- `param1`表示请求必须包含名为`param1`的参数
- `!param1`表示请求必须不携带`param1`参数
- `param1=xxx`表示请求必须包含`param1`参数，并且值为`xxx`

```java
/**
 * 测试RequestMapping的method属性
 */
@RequestMapping(value = "/test_request_mapping_params", method = RequestMethod.POST,
                params = {"userName"})
public String testRequestMappingParam() {
    return "welcome";
}
```

如果当前请求没有携带指定参数，那么返回响应404

```http
POST {{url}}/{{rquest _mapping_api}}/test_request_mapping_params HTTP/1.1
```

```http
HTTP/1.1 404 Not Found
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=utf-8
Content-Language: zh-CN
Content-Length: 649
Date: Sat, 31 Jul 2021 04:34:58 GMT
Connection: close
```

携带参数后正常访问

```http
POST {{url}}/{{rquest_mapping_api}}/test_request_mapping_params?userName=CLOVER HTTP/1.1
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=UTF-8
Content-Language: zh-CN
Content-Length: 1222
Date: Sat, 31 Jul 2021 04:38:43 GMT
Connection: close
```

使用`非`表达式后，这个请求直接404

```http
HTTP/1.1 404 Not Found
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=utf-8
Content-Language: zh-CN
Content-Length: 649
Date: Sat, 31 Jul 2021 04:42:16 GMT
Connection: close
```



### 规定请求头

根据你的规则定制请求头参数，如果不满足你的规则，那么返回404。用法与[规定请求参数](###规定请求参数)一致。

```java
/**
 * 规定请求头
 */
@RequestMapping(value = "/test_request_mapping_headers", headers = {"author"})
public String testRequestMappingHeaders() {
  return "welcome";
}
```

```http
GET {{url}}/{{rquest_mapping_api}}/test_request_mapping_headers HTTP/1.1
author: clover
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=UTF-8
Content-Language: zh-CN
Content-Length: 1222
Date: Sat, 31 Jul 2021 04:59:04 GMT
Connection: close
```



### Ant风格的URL

Ant风格资源地址支持3中匹配符

- `?` 匹配文件名中的一个字符
- `*` 匹配文件名中的任意字符
- `**` 匹配多层路径

`@RequestMapping`还支持Ant风格的URL

- `/user/*/createUser`匹配`/user/aaa/createUser`、`/user/bbb/createUser`等URL
- `/user/**/createUser`匹配`/user/createUser`、`/user/aaa/bbb/createUser`等URL
- `/user/createUser??`匹配`/user/createUseraaa`、`/user/createUser/bb`等URL



## @PathVariable

- 带占位符的URL是Spring3.0新增的功能，该功能在SpringMVC向REST目标挺进发中过程中具有里程碑的意义
- 通过`@PathVariable`可以将URL中占位符参数绑定到控制器处理方法的入参中。URL中的`{xxx}`占位符可以通过`@PathVariable(“xxx”)`绑定到操作方法的入参中。



```java
/**
 * 测试
 * @return
 */
@RequestMapping("/delete/{id}")
@ResponseBody
public String testPathVariable(@PathVariable("id") String id) {
    return "id="+id;
}
```

```http
GET {{url}}/{{path_variable}}/delete/2020-13 HTTP/1.1
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 10
Date: Sat, 31 Jul 2021 05:49:34 GMT
Connection: close

id=2020-13
```

*路径上的占位符只能占一层路径*

```http
GET {{url}}/{{path_variable}}/delete/hia/2020-13 HTTP/1.1
```

```http
HTTP/1.1 404 Not Found
Server: Apache-Coyote/1.1
Content-Type: text/html;charset=utf-8
Content-Language: zh-CN
Content-Length: 649
Date: Sat, 31 Jul 2021 05:56:05 GMT
Connection: close
```



## REST

- REST 即Representational State Transfer。（资源）表现层状态转化。**是目前最流行的一种互联网软件架构。**它结构清晰、符合标准、易于理解、扩展方便，所以才能得到越来越多网站的采用
- 资源（Resources）网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌、一种服务，总之就是一个具体的存在。可以用一个URL（统一资源定位符）指向它，每种资源对应一个特定的URL。需要获取这个资源，访问它的URL就可以。因此URL即为每一个资源独一无二的识别符。
- 表现层（Representational）把具体资源呈现出来的形式，叫做它的表现层。比如，文本可以用txt格式表现，也可以用HTML格式、XML格式、JSON格式表现，甚至可以采用二进制格式
- 状态转化（State Transfer）每发出一个请求，就代表客户端和服务器的一次交互过程。HTTp协议，是一个无状态协议，即所有的状态都保存在服务器端。因此，如果客户端想要操作服务器，必须通过某种手段，让服务器端发生“状态转化”。而这种转化是建立在表现层之上的。所以就是“表现层状态转化”。具体说就是**HTTP协议里面，四哥表示操作方式的动词：GET、POST、PUT、DELETE。他们分别对应四种基本操作：GET用来获取资源，POST用来新建资源，PUT用来更新资源，DELETE用来删除资源。**



> REST风格URL：`/资源名/资源标识符`



```java
@Controller
@RequestMapping("/rest")
public class RestStyleController {
    /**
     * 获取用户信息
     * @param userId 用户id
     */
    @RequestMapping(value = "/user/{userId}", method = RequestMethod.GET)
    @ResponseBody
    public String getUser(@PathVariable String userId) {
        return "get user: " + userId;
    }

    /**
     * 修改用户信息
     * @param userId 用户id
     */
    @RequestMapping(value = "/user/{userId}", method = RequestMethod.PUT)
    @ResponseBody
    public String updateUser(@PathVariable String userId) {
        return "update user: " + userId;
    }

    /**
     * 新增用户
     * @param userName 用户id
     */
    @RequestMapping(value = "/user/{userName}", method = RequestMethod.POST)
    @ResponseBody
    public String addUser(@PathVariable String userName, String pass) {
        return "add user: " + userName + "; pass: " + pass;
    }


    /**
     * 删除用户信息
     * @param userName 用户id
     */
    @RequestMapping(value = "/user/{userName}", method = RequestMethod.DELETE)
    @ResponseBody
    public String deleteUser(@PathVariable String userName) {
        return "delete user: " + userName;
    }

}
```

GET方式

```http
GET {{url}}/rest/user/8848 HTTP/1.1
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 14
Date: Sat, 31 Jul 2021 06:37:58 GMT
Connection: close

get user: 8848
```



POST方式

```http
POST {{url}}/rest/user/clover?pass=123 HTTP/1.1
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 25
Date: Sat, 31 Jul 2021 06:38:38 GMT
Connection: close

add user: clover; pass: 123
```



PUT方式

```http
PUT {{url}}/rest/user/8848 HTTP/1.1
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 17
Date: Sat, 31 Jul 2021 06:38:49 GMT
Connection: close

update user: 8848
```



DELETE方式

```http
DELETE {{url}}/rest/user/8848 HTTP/1.1
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 17
Date: Sat, 31 Jul 2021 06:39:11 GMT
Connection: close

delete user: 8848
```



> 高版本tomcat（8.0+）jsp页面只允许GTE、POST、HEAD请求访问



## 参数处理

由于每一种请求方式、请求类型的不同，需要从不同的地方获取参数，此时可以使用SpringMVC提供的几个注解从指定位置获取参数。



### @RequestParam

默认方式获取请求参数，可以在方法上写一个和请求参数名相同的变量，SpringMVC会根据这个变量的变量名获取到对应的请求参数，**如果获取不到这个参数值，那么默认null。**

```java
@RequestMapping(value = "/not_request_param")
public String noRequestParam(String userName) {
    System.out.println("userName="+userName);
    return "welcome";
}
```

```http
GET {{url}}/param/not_request_param?userName=Clover HTTP/1.1
```

```
userName=Clover
```

通过`@RequestParam`注解，可以指定获取请求参数的值。如果不指定`value`那么默认`value`就是被标注的参数的参数名。

```java
@RequestMapping("/request_param")
public String requestParam(@RequestParam("userName") String userName) {
    System.out.println("userName=" + userName);
    return "welcome";
}
```

```http
GET {{url}}/param/request_param?userName=Clover HTTP/1.1
```

```
userName=Clover
```

如果指定参数不存在，那么会出现400的请求错误，被当前注解标注的参数，表示该参数是必须的。可以通过`required`属性声明当前参数并不是必须的。

```java
public String requestParam(@RequestParam(value = "...", required = false) ...) {...}
```

当`required`属性为`false`时，可以通过`defaultValue`来为当前变量设置一个默认值。如果请求没有携带指定参数，那么就会使用`defaultValue`中定义的值作为参数值。

```java
public String requestParam(@RequestParam(value = "...", required = false, defaultValue = "Clover") ...) {...}
```





### @RequestHeader

用于获取请求头中的某个key

```java
/**
 * RequestHeader的使用
 * @param name 用户名
 */
@RequestMapping("/request_header")
public String requestHeader(@RequestHeader("author") String name) {
    System.out.println("author=" + name);
    return "welcome";
}
```

```http
GET {{url}}/param/request_header HTTP/1.1
author: Clover
```

```
author=Clover
```

用法与`@RequestParam`一致



### @CookieValue

获取指定cookie值

```java
/**
 * RequestHeader的使用
 * @param name 用户名
 */
@RequestMapping("/cookie_value")
public String cookieValue(@CookieValue("cookieName") String name) {
    System.out.println("cookieName=" + name);
    return "welcome";
}
```

```http
GET {{url}}/param/cookie_value HTTP/1.1
Cookie: cookieName="Clover cookie"
```

```
cookieName=Clover cookie
```

用法与`@RequestParam`一致



### POJO参数

如果我们的请求参数是一个POJO，那么SpringMVC会自动为这个POJO赋值

- 将POJO中的每一个属性，从request中获取出来并封装。
- 支持集联封装
- 请求参数的参数名要和对象中的属性名一一对应 

pojo类

```java
public class Book {

    /**
     * 书籍名称
     */
    private String bookName;

    /**
     * 作者
     */
    private String author;

    /**
     * 价格
     */
    private Double price;

    /**
     * 库存
     */
    private Short stock;

    /**
     * 销量
     */
    private Short sales;

    /**
     * 地址
     */
    private Address address;

}

public class Address {

    /**
     * 省
     */
    private String province;

    /**
     * 市
     */
    private String city;

    /**
     * 街道
     */
    private String street;
}

```

```java
@RequestMapping(value = "/pojo")
public String usePOJO(Book book) {
    System.out.println(book);
    return "welcome";
}
```

```http
POST {{url}}/param/pojo HTTP/1.1
content-type: application/x-www-form-urlencoded

bookName=SpringMVC&author=Clover&price=9.99&stock=2048&sales=1024&address.province=广东省&address.city=广州市&address.street=xxx街道
```

```
{"bookName":"SpringMVC","author":"Clover","price":9.99,"stock":2048,"sales":1024,"address":{"province":"广东省","city":"广州市","street":"xxx街道"}}
```



### 原生API

SpringMVC支持原生API参数例如：`HttpServletRequest`、`HttpSession`等…

```java
@RequestMapping("/primitiveAPI")
public String primitiveAPI(HttpSession session, HttpServletRequest request) {
    session.setAttribute("session","hi!");
    request.setAttribute("req", "hello world!");
    return "welcome";
}
```

```http
GET {{url}}/param/primitiveAPI HTTP/1.1
```

```jsp
<body>
  <h1>Welcome~~~</h1>
  <h1>Request: ${requestScope.req}</h1><br/>
  <h1>Session: ${sessionScope.session}</h1>
</body>
```

![原生API写入结果](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-08-01%2015.08.20.png)

原生API也只能传个别的，并不是全部都能传：

- `HttpServletRequest`
- `HttpServletResponse`
- `HttpSession`
- `java.security.Principal`
- `Locale`
- `InputStream`
- `OutputStream`
- `Reader`
- `Writer`



## SpringMVC解决中文乱码

如果遇到`POST`中文乱码问题，可以使用SpringMVC中提供的`org.springframework.web.filter.CharacterEncodingFilter`过滤器。过滤字符编码。

```xml
<!-- 配置SpringMVC提供的字符编码过滤器 设置字符编码 解决中文乱码问题 -->
<filter>
  <filter-name>characterEncodingFilter</filter-name>
  <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
  <!--    解决请求乱码    -->
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
  <!--    解决响应乱码    -->
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





## 数据输出

SpringMVC除了在方法上传入原生的request和session外，还能在方法上传入Map、Model、或者ModelMap。给这些参数里面保存的所有数据都会被放在请求域中。可以通过JSP页面获取



### Map

```java
@RequestMapping(value = "/byMap")
public String byMap(Map<String, Object> map) {
    map.put("msg", "使用了Map");
    return "output/displayData";
}
```

Map会被放在request域对象中

```jsp
<h1>pageContext域对象: ${pageContext.msg}</h1>
```



### Model

```java
@RequestMapping(value = "/byModel")
public String byModel(Model model) {
    model.addAttribute("msg", "使用了Model");
    return "output/displayData";
}
```

Model会被放在request域中

```jsp
<h1>request域对象: ${requestScope.msg}</h1><br/>
```



### ModelMap

```java
@RequestMapping(value = "/byModelMap")
public String byModel(ModelMap model) {
    model.put("msg", "使用了ModelMap");
    return "output/displayData";
}
```

同样会放在域对象

```jsp
<h1>request域对象: ${requestScope.msg}</h1><br/>
```



### ModelAndView

除了使用以上三种类型作为方法参数外，还可以将方法返回值设置为`ModelAndView`。它既包含视图信息也包含模型数据。而且数据是放在请求域中的。

```java
@RequestMapping("/byModelAndView")
public ModelAndView byModelAndView() {
    ModelAndView mv = new ModelAndView("output/displayData");
    mv.addObject("msg", "byModelAndView");
    return mv;
}
```

```jsp
<h1>request域对象: ${requestScope.msg}</h1><br/>
```



### @SessionAttributes

给BindingAwareModelMap中保存的数据，或者ModelAndView中保存的数据，同时也给session中保存一份。

它只能写在类上，并且它有两个属性

1. `value` 保存数据时要给session中放的数据的key
2. `types` 保存数据时，如果这个数据的类型与`types`中的类型一致才保存。

```java
@SessionAttributes("msg")
@Controller
@RequestMapping("/output")
public class OutputDataController {
    @RequestMapping(value = "/byMap")
    public String byMap(Map<String, Object> map) {
        map.put("msg", "使用了Map");
        return "output/displayData";
    }
}
```

```jsp
<h1>request域对象: ${requestScope.msg}</h1><br/>
<h1>session域对象: ${sessionScope.msg}</h1><br/>
```



### @ModelAttribute

这个注解可以标注在两处位置：方法上、方法参数上

- 参数：取出model中保存的数据
- 方法上：每次有请求过来时，被标注的方法都会先于目标方法运行。在此可以保存一些必要信息到request域对象中。



```java
/**
 * 模拟查询数据库数据
 */
@ModelAttribute
public void selectUser(Model model) {
    User user = new User();
    user.setAge(19);
    user.setSex("男");
    user.setName("Clover");
    model.addAttribute("user", user);
}

@RequestMapping("/byModelAttribute")
public String byModelAttribute(@ModelAttribute("user") User user) {
    System.out.println("user ==========>>>>> "+user);
    return "output/displayData";
}

```

```
user ==========>>>>> {"name":"Clover","sex":"男","age":19}
```

`@ModelAttribute`还有一个功能，就是，如果前端传了这个参数，那么就使用前端的参数将对应属性进行覆盖。

例如这个请求，我传了一个`name`参数

```
xxx/output/byModelAttribute?name=Ali总裁
```

后台打印结果

```
user ==========>>>>> {"name":"Ali总裁","sex":"男","age":19}
```



## DispatcherServlet 分析

### 继承图

![DispatcherServlet继承图](http://qiniu-note-image.ctong.top/note/images/DispatcherServlet.png)





### 请求大致处理流程

发送请求时，这个请求会被`org.springframework.web.servlet.DispatcherServlet#doDispatch`方法处理。

- 检查当前请求是否为文件上传请求，如果是文件上传请求，重新包装`request`

  ```java
  processedRequest = checkMultipart(request);
  multipartRequestParsed = processedRequest != request;
  ```

- 根据请求地址到所有Controller中找，看哪个Controller能处理当前请求

  ```java
  // Determine handler for the current request.
  mappedHandler = getHandler(processedRequest);
  ```

  - `handlerMappings`保存了所有的handler映射信息。在ioc容器创建Controller对象的时候，扫描每个处理器都能处理什么请求，然后将其保存在HandlerMapping的handlerMap中

    ```java
    for (HandlerMapping hm : this.handlerMappings) { ... }
    ```

  - 检查当前handler是否支持处理这个`request`

    ```java
    HandlerExecutionChain handler = hm.getHandler(request);
    ```

  - 如果支持处理当前请求那么就将匹配到的handler直接返回

    ```java
    if (handler != null) {
       return handler;
    }
    ```

- 如果没有找到或者说没有handler能处理这个请求，那么就抛出异常或者转发到404页面

  ```java
  if (mappedHandler == null || mappedHandler.getHandler() == null) {
     noHandlerFound(processedRequest, response);
     return;
  }
  ```

- 拿到这个handler的适配器

  ```java
  // Determine handler adapter for the current request.
  HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
  ```

  - 循环便利所有的`handlerAdapters`，默认情况下只有三个adapter

    ```java
    for (HandlerAdapter ha : this.handlerAdapters){...}
    ```

  - 通过调用每个Adapter的`supports`方法确定当前adpter是否支持处理这个**handler**

- 适配器执行目标方法。将目标方法执行完成后的返回结果作为视图名，设置保存到`ModelAndView`中。目标方法无论怎么写，最终适配器执行完成后都会将执行结果封装成`ModelAndView`

  ```java
  // Actually invoke the handler.
  mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
  ```

- 根据方法最终执行完成后封装的ModelAndView转发到对应页面，并且ModelAndView中的数据可以从请求域中获取

  ```java
  processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
  ```

  

### SpringMVC九大组件

`org.springframework.web.servlet.DispatcherServlet`中有九大组件。SpringMVC在工作的时候都是由这些组件来完成。

```java
/** 文件上传解析器 */
private MultipartResolver multipartResolver;

/** 区域信息解析器「国际化」 */
private LocaleResolver localeResolver;

/** 主题解析器 */
private ThemeResolver themeResolver;

/** List of HandlerMappings used by this servlet */
private List<HandlerMapping> handlerMappings;

/** handler适配器 */
private List<HandlerAdapter> handlerAdapters;

/** SpringMVC强大的异常解析器 */
private List<HandlerExceptionResolver> handlerExceptionResolvers;

/** RequestToViewNameTranslator used by this servlet */
private RequestToViewNameTranslator viewNameTranslator;

/** SpringMVC中允许重定向携带数据的功能*/
private FlashMapManager flashMapManager;

/** 视图解析器 */
private List<ViewResolver> viewResolvers
  
```

他们都有一个共同点：他们都是接口



> 接口就是规范

SpringMVC九大组件在`org.springframework.web.servlet.DispatcherServlet#onRefresh`方法中进行初始化。在初始化过程中，全部组件都是从ioc容器中获取，但如果ioc容器中没有这个组件，就使用默认策略。这个默认策略都在`spring-webmvc-4.0.0.RELEASE.jar!/org/springframework/web/servlet/DispatcherServlet.properties`文件中定义好了



### 执行目标方法源码分析

目标方法`org.springframework.web.servlet.DispatcherServlet#doDispatch`。

- 跳过其它，**Step Into**进去

  ```java
  mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
  ```

- 检查当前这个类(controller)是否已经被缓存在`org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter#sessionAnnotatedClassesCache`中

  ```java
  Boolean annotatedWithSessionAttributes = this.sessionAnnotatedClassesCache.get(clazz);
  ```

  

- 如果这个类的指定注解状态没有被缓存，那么就去检查这个类是否存在这个注解`@SessionAttributes`。最后将检查结果缓存。

  ```java
  if (annotatedWithSessionAttributes == null) {
    annotatedWithSessionAttributes = (AnnotationUtils.findAnnotation(clazz, SessionAttributes.class) != null);
    this.sessionAnnotatedClassesCache.put(clazz, annotatedWithSessionAttributes);
  }
  ```
  
- 其它代码不管，直接跳到最后一行，**Step Into**进去

  ```java
  return invokeHandlerMethod(request, response, handler);
  ```

- 获取方法解析器

  ```java
  ServletHandlerMethodResolver methodResolver = getMethodResolver(handler);
  ```

- 根据解析器解析当前请求，检查哪个方法能够处理处理当前请求，最后拿到一个`Method`

  ```java
  Method handlerMethod = methodResolver.resolveHandlerMethod(request);
  ```

- 根据方法解析器创建一个方法执行器

  ```java
  ServletHandlerMethodInvoker methodInvoker = new ServletHandlerMethodInvoker(methodResolver);
  ```

- 创建一个`org.springframework.validation.support.BindingAwareModelMap`，之前使用`ModelAttribute`获取的什么Map、Model、ModelMap就是使用的是它。貌似它**非常重要、非常重要、非常重要**

  ```java
  ExtendedModelMap implicitModel = new BindingAwareModelMap();
  ```

- 执行目标方法，利用反射执行期间确定参数值、提前执行`ModelAttribute`等所有操作都在这个方法中完成  **Step Into**进去

  ```java
  Object result = methodInvoker.invokeHandlerMethod(handlerMethod, handler, webRequest, implicitModel);
  ```

- 检查调用的方法是否为桥接方法，如果是，那么尽可能找到原方法(不清楚是不是这个逻辑)。[可以到这里了解桥接方法是什么](https://www.cnblogs.com/heihaozi/p/14142671.html)

  ```java
  Method handlerMethodToInvoke = BridgeMethodResolver.findBridgedMethod(handlerMethod);
  ```

- 获取SessionAttribute指定的名字。如果有，那么将它的值保存到`implicitModel`。看到这这貌似是一个域对象？

  ```java
  for (String attrName : this.methodResolver.getActualSessionAttributeNames()) {
     Object attrValue = this.sessionAttributeStore.retrieveAttribute(webRequest, attrName);
     if (attrValue != null) {
        implicitModel.addAttribute(attrName, attrValue);
     }
  }
  ```

- 获取这个controller中是否存在被`@ModelAttribute`标注的类，并循环它。

  ```java
  for (Method attributeMethod : this.methodResolver.getModelAttributeMethods()) {...}
  ```

  - 获取到这个被标注的方法`@ModelAttribute`

    ```java
    Method attributeMethodToInvoke = BridgeMethodResolver.findBridgedMethod(attributeMethod);
    ```

  - 确定方法执行是所需要的所有参数的值

    ```java
    Object[] args = resolveHandlerArguments(attributeMethodToInvoke, handler, webRequest, implicitModel);
    ```

  - 获取当前方法中`@ModelAttribute`的值

    ```java
    String attrName = AnnotationUtils.findAnnotation(attributeMethod, ModelAttribute.class).value();
    ```

  - 通过`@ModelAttribute`的值判断如果当前域对象中存在这个属性，那么就跳出当前循环。在这里证明了，被`@ModelAttribute`标注的方法无法覆盖域对象中存在的属性。

    ```java
    if (!"".equals(attrName) && implicitModel.containsAttribute(attrName)) {
    	continue;
    }
    ```

  - 开始执行这个方法

    ```java
    Object attrValue = attributeMethodToInvoke.invoke(handler, args);
    ```

  - 如果`@ModelAttribute`注解没有制定value，那么就使用返回值类型名称的小驼峰作为value

    ```java
    if ("".equals(attrName)) {
       Class<?> resolvedType = GenericTypeResolver.resolveReturnType(attributeMethodToInvoke, handler.getClass());
       attrName = Conventions.getVariableNameForReturnType(attributeMethodToInvoke, resolvedType, attrValue);
    }
    ```

  - 最后把目标方法的返回值添加到`BindingAwareModelMap`中，以attrName作为key。到此`ModelAttribute`执行结束

    ```java
    if (!implicitModel.containsAttribute(attrName)) {
       implicitModel.addAttribute(attrName, attrValue);
    }
    ```

    

- 确定目标方法执行时所需要的参数值 **Step Into**进去看看怎么为POJO赋值的

  ```java
  Object[] args = resolveHandlerArguments(handlerMethodToInvoke, handler, webRequest, implicitModel);
  ```

  - 获取到所有参数。并且创建一个`Object[]`，用于接收对应的参数值

    ```java
    Class<?>[] paramTypes = handlerMethod.getParameterTypes();
    Object[] args = new Object[paramTypes.length];
    ```

  - 循环处理所有参数

    ```java
    for (int i = 0; i < args.length; i++) { ... }
    ```

  - 获取到当前参数

    ```java
    MethodParameter methodParam = new MethodParameter(handlerMethod, i);
    ```

  - 初始化`parameterNameDiscoverer`，为了后面获取参数名，这是Spring提供的工具

    ```java
    methodParam.initParameterNameDiscovery(this.parameterNameDiscoverer);
    ```

  - 其它不认识，跳过…

  - 获取到参数中所有的注解

    ```java
    Annotation[] paramAnns = methodParam.getParameterAnnotations();
    ```

  - 循环处理所有的注解

    ```java
    for (Annotation paramAnn : paramAnns) { ... }
    ```

  - 根据判断找到对应的注解，我当前这个方法有一个参数所POJO，它有一个注解`@ModelAttribute("user")`。它会根据对应注解进行对应操作

    ```java
    if (RequestParam.class.isInstance(paramAnn)) {...}
    else if (RequestHeader.class.isInstance(paramAnn)) {...}
    else if (RequestBody.class.isInstance(paramAnn)) {...}
    else if (CookieValue.class.isInstance(paramAnn)) {...}
    else if (PathVariable.class.isInstance(paramAnn)) {...}
    else if (ModelAttribute.class.isInstance(paramAnn)) {...}
    else if (Value.class.isInstance(paramAnn)) {...}
    else if (paramAnn.annotationType().getSimpleName().startsWith("Valid")) {...}
    ```

  - 判断是不是有重复注解，如果有那么抛出异常。在这里证明，在SpringMVC中注解只能使用一个，并且只能使用上一步标出来的那些注解。

    ```java
    if (annotationsFound > 1) {
       throw new IllegalStateException("Handler parameter annotations are exclusive choices - " +
             "do not specify more than one such annotation on the same parameter: " + handlerMethod);
    }
    ```

  - 如果自定义类型没有注解的情况下处理流程

    ```java
    if (annotationsFound == 0) {...}
    ```

    - 检查是否为原生API，如果是就根据API赋值

      ```java
      Object argValue = resolveCommonArgument(methodParam, webRequest);
      ```

    - 检查参数是否为Model类型，如果是就直接吧`implicitModel`丢过去

      ```java
      Class<?> paramType = methodParam.getParameterType();
      if (Model.class.isAssignableFrom(paramType) || Map.class.isAssignableFrom(paramType)) {
         if (!paramType.isAssignableFrom(implicitModel.getClass())) {
            throw new IllegalStateException("Argument [" + paramType.getSimpleName() + "] is of type " +
                  "Model or Map but is not assignable from the actual model. You may need to switch " +
                  "newer MVC infrastructure classes to use this argument.");
         }
         args[i] = implicitModel;
      }
      ```

    - 其它类型不认识，不管…

    - 判断如果是基本数据类型

      ```java
      if (BeanUtils.isSimpleProperty(paramType)) {
        paramName = "";
      }
      ```

    - 以上其它都不是，就把`attrName`初始化为空字符串

  - 这些不管

    ```java
    if (paramName != null) {
       args[i] = resolveRequestParam(paramName, required, defaultValue, methodParam, webRequest, handler);
    }
    else if (headerName != null) {
       args[i] = resolveRequestHeader(headerName, required, defaultValue, methodParam, webRequest, handler);
    }
    else if (requestBodyFound) {
       args[i] = resolveRequestBody(methodParam, webRequest, handler);
    }
    else if (cookieName != null) {
       args[i] = resolveCookieValue(cookieName, required, defaultValue, methodParam, webRequest, handler);
    }
    else if (pathVarName != null) {
       args[i] = resolvePathVariable(pathVarName, methodParam, webRequest, handler);
    }
    ```
  
  - 解析POJO参数
  
    ```java
    if (attrName != null) {...}
    ```
  
    - 确定自定义类型参数的值
  
      ```java
      private WebDataBinder resolveModelAttribute(String attrName, MethodParameter methodParam,
            ExtendedModelMap implicitModel, NativeWebRequest webRequest, Object handler) throws Exception {
      
         // Bind request parameter onto object...
         String name = attrName;
        // 如果指定注解的value值为空，那么就获取参数名给到attrName
         if ("".equals(name)) {
            name = Conventions.getVariableNameForParameter(methodParam);
         }
        // 获取参数类型
         Class<?> paramType = methodParam.getParameterType();
        // 目标对象的值
         Object bindObject;
        // 判断implicitModel中是否有这个属性，如果有就直接赋值
         if (implicitModel.containsKey(name)) {
            bindObject = implicitModel.get(name);
         }
        //在session中获取对应的value，并且value类型要一致
         else if (this.methodResolver.isSessionAttribute(name, paramType)) {
            bindObject = this.sessionAttributeStore.retrieveAttribute(webRequest, name);
            if (bindObject == null) {
               raiseSessionRequiredException("Session attribute '" + name + "' required - not found in session");
            }
         }
         else {
           // 如果以上几步都无法确定值，那么就使用反射创建一个对象
            bindObject = BeanUtils.instantiateClass(paramType);
         }
         WebDataBinder binder = createBinder(webRequest, bindObject, name);
         initBinder(handler, name, binder, webRequest);
         return binder;
      }
      ```
    
    
    
  - 开始通过反射调用目标方法
    
    ```java
    handlerMethodToInvoke.invoke(handler, args);
    ```
    
    
    
    
  
  > 如果是自定义类型，那么赋值时只有两种结果，要么根据指定注解的value找，要么是空串



## 视图和视图解析器



### 视图解析器

- 请求处理方法执行后，最终返回一个`ModelAndView`对象。对于那些放回String、View或者ModelMap等类型的处理方法，SpringMVC也会在内部将他们装配成一个`ModelAndView`对象，它包含了逻辑名和模型对象的视图。
- SpringMVC借助视图解析器（ViewResolver）得到最终的视图对象（View），最终的视图可以是JSP，也可能是Excel、JFreeChart等各种表现形式的视图
- 对于最终究竟采用哪种视图对象对模型数据进行渲染，处理器并不关心。处理器工作重点聚焦在生产模型数据的工作上，从而实现MVC的充分解耦。



### 视图

- 视图的作用是渲染模型数据，将模型里的数据以某种形式呈现给客户。
- 为了实现视图模型和具体实现技术的解耦，Spring定义了一个高度抽象的View接口`org.springframework.web.servlet.View`
- 视图对象由视图解析器负责实例化。由于视图是无状态的，所以他们不会有线程安全问题。

![常用的视图实现类](http://qiniu-note-image.ctong.top/note/images/%E5%B8%B8%E7%94%A8%E7%9A%84%E8%A7%86%E5%9B%BE%E5%AE%9E%E7%8E%B0%E7%B1%BB.png)



### 转发

使用`forward`将当前请求转发到一个指定的页面。

```java
@Controller
@RequestMapping("/view_resolver")
public class ViewResolverController {
    @RequestMapping
    public String view() {
        return "forward:/WEB-INF/pages/view_resolver/index.jsp";
    }
}

```

```http
GET {{url}}/view_resolver HTTP/1.1
```

```http
HTTP/1.1 200 
Set-Cookie: JSESSIONID=839F1BA4F614AEE5B2758EE57FCF0283; Path=/; HttpOnly
Content-Type: text/html;charset=UTF-8
Content-Language: zh-Hans-CN
Content-Length: 1210
Date: Tue, 03 Aug 2021 07:55:28 GMT
Connection: close
```

使用`forward`前缀的转发不会由我们配置的视图解析器拼接字符串

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/pages/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

### 重定向

使用`redirect`将一个请求重定向到另一个页面

```java
@Controller
@RequestMapping("/view_resolver")
public class ViewResolverController {

    @RequestMapping("/redirect_page")
    public String redirectPage() {
        LogUtils.info("当前是/view_resolver/redirect_page");
        return "view_resolver/redirectPage";
    }

    @RequestMapping("/redirect")
    public String testRedirect() {
        LogUtils.info("当前是/view_resolver/redirect， 准备重定向...");
        return "redirect:/view_resolver/redirect_page";
    }

}
```

```http
GET {{url}}/view_resolver/redirect HTTP/1.1
```

```http
HTTP/1.1 200 
Content-Type: text/html;charset=UTF-8
Content-Language: zh-Hans-CN
Content-Length: 1214
Date: Tue, 03 Aug 2021 08:49:00 GMT
Connection: close
```

```
当前是/view_resolver/redirect， 准备重定向...
当前是/view_resolver/redirect_page
```

`/`代表从当前项目下开始





> 有前缀的转发和重定向操作视图解析器就不会拼接配置的字符串





### 视图解析流程

1. 方法执行后的返回值会作为页面地址参考，转发或重定向到页面
2. 在配置了视图解析器的情况下可能会进行页面地址的拼接

略过其他，直接定位到`org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter#invokeHandlerMethod`。

- 在目标方法执行完后拿到返回值`redirect:/view_resolver/redirect_page`

  ```java
  Object result = methodInvoker.invokeHandlerMethod(handlerMethod, handler, webRequest, implicitModel);
  ```

- 无论返回值是什么，都会包装成为一个`ModelAndView`

  ```java
  ModelAndView mav = methodInvoker.getModelAndView(handlerMethod, handler.getClass(), result, implicitModel, webRequest);
  ```

`invokeHandlerMethod`这个方法执行完后直接回到`org.springframework.web.servlet.DispatcherServlet#doDispatch`

- 拿到包装好的ModelAndView

  ```java
  mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
  ```

- 处理`ModelAndView`。**Step Into**进去

  ```java
  processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
  ```

- 如果之前有什么错误，这里可以处理。

  ```java
  if (exception != null) {
     if (exception instanceof ModelAndViewDefiningException) {
        logger.debug("ModelAndViewDefiningException encountered", exception);
        mv = ((ModelAndViewDefiningException) exception).getModelAndView();
     }
     else {
        Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
        mv = processHandlerException(request, response, handler, exception);
        errorView = (mv != null);
     }
  }
  ```

- 如果没有什么错误就可以渲染页面，就看这个方法，其它不看，**Step Into**

  ```java
  render(mv, request, response);
  ```

- 声明一个空的`View`对象

  ```java
  View view;
  ```

  `View`与`ViewResolver`的关系： `ViewResolver`是一个接口，它里面只有一个方法 ==> 

  `View resolveViewName(String viewName, Locale locale) throws Exception;`
  
- 如果还没有解析视图，那么就通过视图名称去解析它。**Step Into**进去

  ```java
  view = resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
  ```

- 我在配置文件中有配置到一个视图解析器，所以SpringMVC在启动的时候不会帮我们创建视图解析器。所以这里只获取到一个视图解析器。拿到视图解析器后，通过视图解析器来解析视图，解析失败时返回null。**Step Out**出去

  ```java
  for (ViewResolver viewResolver : this.viewResolvers) {
    View view = viewResolver.resolveViewName(viewName, locale);
    if (view != null) {
      return view;
    }
  }
  ```

- 如果成功拿到一个视图解析器，那么就调用这个视图解析器的`render`方法.**Step Into**进去

  ```java
  view.render(mv.getModelInternal(), request, response);
  ```

- 不认识忽略….在`render`方法的最后一步进行渲染**Step Into**进去

  ```java
  renderMergedOutputModel(mergedModel, request, response);
  ```

- 不认识….忽略

- 将Model中的所有属性暴露到request。实际上就是将所有的Map中的项目设置到request。`request.sertAttributre(Map.Key, Map.Value)`

  ```java
  // Expose the model object as request attributes.
  exposeModelAsRequestAttributes(model, requestToExpose);
  ```

- 确定转发的路径，因为它是根据forward前缀给我们new了`InternalResourceView`。这是一个**转发器**？

  ```java
  String dispatcherPath = prepareForRendering(requestToExpose, response);
  ```

- 转发

  ```java
  rd.forward(requestToExpose, response);
  ```

- 完了…



> 视图解析器只是为了得到视图对象。视图对象才能真正的转发或重定向到页面视图对象才能真正的渲染视图



### 自定义视图和视图解析器

[源码到我GitHub上下载](https://github.com/YouChuantong/spring_learn_demo/tree/main/CustomViewAndViewResolver)

在SpringMVC中，是通过视图解析器找到对应视图，最后是视图将数据输出。无论是jsp页面还是json数据，都是如此。所以第一步需要先写一个数据我们自己的视图解析器。

既然有`forward`前缀，那么我搞一个能`clover:`前缀的视图解析器。解析器需要实现`org.springframework.web.servlet.ViewResolver`接口并把它注册到容器中

```java
@Component
public class MyCustomViewResolver implements ViewResolver {
  
    /* 视图名称前缀 */
    public static final String VIEW_NAME_PREFIX = "clover:";

    /* 空字符 */
    private static final String EMPTY_STRING = "";

    /* 日志记录 */
    private final transient Logger log = LoggerFactory.getLogger(MyCustomViewResolver.class);

    @Override
    public View resolveViewName(String viewName, Locale locale) throws Exception {
        log.info("resolveViewName执行了...");
        if (viewName.startsWith(VIEW_NAME_PREFIX)) {
            String uri = viewName.replaceFirst(VIEW_NAME_PREFIX, EMPTY_STRING);
            return new MyCustomView(uri);
        }
        return null;
    }
  
}
```

`VIEW_NAME_PREFIX`属性就是我们的规则，通过判断控制器返回的字符串前缀是否是`clover:`

。如果不是那么返回null，springmvc会继续找对应的视图解析器。解析成功之后将前缀截掉就行了。

```java
  String uri = viewName.replaceFirst(VIEW_NAME_PREFIX, EMPTY_STRING);
```

符合我们定的规则，直接`new` 一个自定义的视图回去。接下来需要定义我们的视图。视图不需要让容器管理。视图需要实现`org.springframework.web.servlet.View`接口

```java
public class MyCustomView implements Serializable, View {

    private static final long serialVersionUID = 8278828333781661446L;

    /* 视图名称 */
    private final String VIEW_NAME;

    private transient HttpServletRequest request;

    private transient HttpServletResponse response;

    public MyCustomView(String viewName) {
        this.VIEW_NAME = viewName;
    }

    /**
     * 获取 Content-Type
     */
    @Override
    public String getContentType() {
        String contentType = request.getContentType();
        System.out.println(contentType);
        return contentType == null ? MediaType.APPLICATION_JSON_VALUE + ";charset=utf-8;" : contentType;
    }

    /**
     * 渲染视图
     * @param model BindingAwareModelMap对象「域对象」
     * @param request request 信息
     * @param response response 信息
     */
    @Override
    public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
        this.request = request;
        this.response = response;
        try (PrintWriter writer = this.response.getWriter()) {
            this.response.setCharacterEncoding("utf-8");
            this.response.setContentType(getContentType());
            writer.println(User.init());
        } catch (IOException e) {
            throw new IOException(e.getMessage(), e.getCause());
        }
    }

}
```

这个无参构造用来接收、保存视图名称

```java
public MyCustomView(String viewName) {
  this.VIEW_NAME = viewName;
}
```

输出就行了，不管这么多

```java
writer.println(User.init());
```



## 数据绑定

页面提交的所有数据都是字符串，页面提交过来的数据必然牵扯到以下操作：
1. 数据绑定期间的数据类型转换
    `String --> Integer 或者String --> Boolean`
2. 数据绑定期间的数据格式化问题，比如日期转换
    `String=2017-12-15 ---->> Date=2017/12/15｜2017.12.15`
3. 数据校验
    前端提交的数据必须是合法的。所以前端不但需要校验，重要数据后端校验也是必须的

```java
@Data
public class User implements Serializable {

    public static final User data;

    static {
        User user = new User();
        user.setAge(19);
        user.setName("Clover");
        user.setSex("男");
        user.setBirth(new Date());
        data = user;
    }

    private static final long serialVersionUID = 7826062904698679590L;

    public User() {
        // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
    }

    private String name;

    private String sex;

    private Integer age;

    private Date birth;

    @Override
    public String toString() {
        final StringBuilder sb = new StringBuilder("{");
        sb.append("\"name\":\"").append(name).append('\"');
        sb.append(",\"sex\":\"").append(sex).append('\"');
        sb.append(",\"age\":").append(age);
        sb.append(",\"birth\":\"").append(birth).append('\"');
        sb.append('}');
        return sb.toString();
    }

    public void setBirth(Date birth) {
        this.birth = birth;
    }

}
```

```http
GET {{url}}/user/getUser?age=19&name=clover&birth=2020-08-05 11:03
```



```java
@Controller
@RequestMapping("/user")
public class VerifyDataController {

    @RequestMapping("/getUser")
    @ResponseBody
    public String getUser(User user) {
        System.out.println(user);
        return user.toString();
    }

}
```

debug到这个方法中`org.springframework.web.method.annotation.ModelAttributeMethodProcessor#resolveArgument`。

- 创建数据绑定器，绑定参数、类型转换、数据、数据格式化、校验等操作都是这个绑定器提供的服务。

  ```java
  WebDataBinder binder = binderFactory.createBinder(request, attribute, name);
  ```

  - `ConversionService`：负责数据类型的转换以及格式化功能，处理不同类型，使用的是不同的Converter。默认有以下这么多个converter

    ```
    "java.lang.Number -> java.lang.Number : org.springframework.core.convert.support.NumberToNumberConverterFactory@6fc796e8"
    "java.lang.String -> java.lang.Number : org.springframework.core.convert.support.StringToNumberConverterFactory@61006560"
    "java.lang.Number -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@653ffd98"
    "java.lang.String -> java.lang.Character : org.springframework.core.convert.support.StringToCharacterConverter@203d6f5b"
    "java.lang.Character -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@731edd62"
    "java.lang.Number -> java.lang.Character : org.springframework.core.convert.support.NumberToCharacterConverter@11d78d04"
    "java.lang.Character -> java.lang.Number : org.springframework.core.convert.support.CharacterToNumberFactory@b1e619b"
    "java.lang.String -> java.lang.Boolean : org.springframework.core.convert.support.StringToBooleanConverter@40129014"
    "java.lang.Boolean -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@45e2a0b9"
    "java.lang.String -> java.lang.Enum : org.springframework.core.convert.support.StringToEnumConverterFactory@68f0baac"
    "java.lang.Enum -> java.lang.String : org.springframework.core.convert.support.EnumToStringConverter@101a0c90"
    "java.lang.String -> java.util.Locale : org.springframework.core.convert.support.StringToLocaleConverter@445cacb7"
    "java.util.Locale -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@14eb3bed"
    "java.lang.String -> java.util.Properties : org.springframework.core.convert.support.StringToPropertiesConverter@79a3d606"
    "java.util.Properties -> java.lang.String : org.springframework.core.convert.support.PropertiesToStringConverter@391d3b2e"
    "java.lang.String -> java.util.UUID : org.springframework.core.convert.support.StringToUUIDConverter@180511ae"
    "java.util.UUID -> java.lang.String : org.springframework.core.convert.support.ObjectToStringConverter@55fc7510"
    "org.springframework.core.convert.support.ArrayToCollectionConverter@32089a4f"
    "org.springframework.core.convert.support.CollectionToArrayConverter@621c136b"
    "org.springframework.core.convert.support.ArrayToArrayConverter@52a4e6e7"
    "org.springframework.core.convert.support.CollectionToCollectionConverter@64c65d56"
    "org.springframework.core.convert.support.MapToMapConverter@126a7963"
    "org.springframework.core.convert.support.ArrayToStringConverter@10efa70e"
    "org.springframework.core.convert.support.StringToArrayConverter@6589f82a"
    "org.springframework.core.convert.support.ArrayToObjectConverter@33733520"
    "org.springframework.core.convert.support.ObjectToArrayConverter@7213e34c"
    "org.springframework.core.convert.support.CollectionToStringConverter@1f29edd7"
    "org.springframework.core.convert.support.StringToCollectionConverter@354b1eb0"
    "org.springframework.core.convert.support.CollectionToObjectConverter@1fbbe70c"
    "org.springframework.core.convert.support.ObjectToCollectionConverter@725f7e34"
    "org.springframework.core.convert.support.ByteBufferConverter@22c862fd"
    "org.springframework.core.convert.support.ByteBufferConverter@22c862fd"
    "java.util.TimeZone -> java.time.ZoneId : org.springframework.core.convert.support.TimeZoneToZoneIdConverter@4f9b568a"
    "java.time.ZoneId -> java.util.TimeZone : org.springframework.core.convert.support.ZoneIdToTimeZoneConverter@f585bf2"
    "org.springframework.core.convert.support.IdToEntityConverter@67f2af3,org.springframework.core.convert.support.ObjectToObjectConverter@67416f4b"
    "org.springframework.core.convert.support.FallbackObjectToStringConverter@72fb4a2f"
    "@org.springframework.format.annotation.NumberFormat java.lang.Double -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Double: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "@org.springframework.format.annotation.NumberFormat java.lang.Float -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Float: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "@org.springframework.format.annotation.NumberFormat java.lang.Short -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Short: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "@org.springframework.format.annotation.NumberFormat java.math.BigDecimal -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "java.lang.String -> @org.springframework.format.annotation.NumberFormat java.math.BigDecimal: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "@org.springframework.format.annotation.NumberFormat java.math.BigInteger -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "java.lang.String -> @org.springframework.format.annotation.NumberFormat java.math.BigInteger: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "@org.springframework.format.annotation.NumberFormat java.lang.Integer -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Integer: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "@org.springframework.format.annotation.DateTimeFormat java.lang.Long -> java.lang.String: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@40e685fe,@org.springframework.format.annotation.NumberFormat java.lang.Long -> java.lang.String: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.lang.Long: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@40e685fe,java.lang.String -> @org.springframework.format.annotation.NumberFormat java.lang.Long: org.springframework.format.number.NumberFormatAnnotationFormatterFactory@3fb11849"
    "@org.springframework.format.annotation.DateTimeFormat java.time.LocalDate -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.time.LocalDate -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@4331c15"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.LocalDate: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.lang.String -> java.time.LocalDate: org.springframework.format.datetime.standard.TemporalAccessorParser@33f73f8a"
    "@org.springframework.format.annotation.DateTimeFormat java.time.LocalTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.time.LocalTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@474a957f"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.LocalTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.lang.String -> java.time.LocalTime: org.springframework.format.datetime.standard.TemporalAccessorParser@67574179"
    "@org.springframework.format.annotation.DateTimeFormat java.time.LocalDateTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.time.LocalDateTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@7b182a32"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.LocalDateTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.lang.String -> java.time.LocalDateTime: org.springframework.format.datetime.standard.TemporalAccessorParser@711951b4"
    "@org.springframework.format.annotation.DateTimeFormat java.time.ZonedDateTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.time.ZonedDateTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@626bc385"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.ZonedDateTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.lang.String -> java.time.ZonedDateTime: org.springframework.format.datetime.standard.TemporalAccessorParser@3fea1a38"
    "@org.springframework.format.annotation.DateTimeFormat java.time.OffsetDateTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.time.OffsetDateTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@599fe4d"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.OffsetDateTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.lang.String -> java.time.OffsetDateTime: org.springframework.format.datetime.standard.TemporalAccessorParser@3e368bbb"
    "@org.springframework.format.annotation.DateTimeFormat java.time.OffsetTime -> java.lang.String: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.time.OffsetTime -> java.lang.String : org.springframework.format.datetime.standard.TemporalAccessorPrinter@27d0013"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.time.OffsetTime: org.springframework.format.datetime.standard.Jsr310DateTimeFormatAnnotationFormatterFactory@2cb1819e,java.lang.String -> java.time.OffsetTime: org.springframework.format.datetime.standard.TemporalAccessorParser@6da9f1be"
    "java.time.Instant -> java.lang.String : org.springframework.format.datetime.standard.InstantFormatter@278801d0"
    "java.lang.String -> java.time.Instant: org.springframework.format.datetime.standard.InstantFormatter@278801d0"
    "java.util.Date -> java.lang.Long : org.springframework.format.datetime.DateFormatterRegistrar$DateToLongConverter@2de93dc9"
    "java.util.Date -> java.util.Calendar : org.springframework.format.datetime.DateFormatterRegistrar$DateToCalendarConverter@47feddff"
    "java.util.Calendar -> java.util.Date : org.springframework.format.datetime.DateFormatterRegistrar$CalendarToDateConverter@65f8f0bf"
    "java.util.Calendar -> java.lang.Long : org.springframework.format.datetime.DateFormatterRegistrar$CalendarToLongConverter@219fb51"
    "java.lang.Long -> java.util.Date : org.springframework.format.datetime.DateFormatterRegistrar$LongToDateConverter@667a89ae"
    "java.lang.Long -> java.util.Calendar : org.springframework.format.datetime.DateFormatterRegistrar$LongToCalendarConverter@57dab1bb"
    "@org.springframework.format.annotation.DateTimeFormat java.util.Date -> java.lang.String: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@40e685fe"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.util.Date: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@40e685fe"
    "@org.springframework.format.annotation.DateTimeFormat java.util.Calendar -> java.lang.String: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@40e685fe"
    "java.lang.String -> @org.springframework.format.annotation.DateTimeFormat java.util.Calendar: org.springframework.format.datetime.DateTimeFormatAnnotationFormatterFactory@40e685fe"
    ```

    

  - `validators`：是数据校验器默认情况下是不会有数据校验功能，所以当前这里是0

  - `bindingResult`：负责保存以及解析数据绑定期间数据校验产生的错误

- 使用绑定器对目标对象就行处理「入参」

  ```java
  bindRequestParameters(binder, request);
  ```

  



---

- SpringMVC将ServletRequest对象及目标方法的入参实例传给`WebDataBinderFactory`实例以创建DataBinder实例对象
- DataBinder调用装配在SpringMVC上下文中的`ConversionService`组件进行数据类型转换、数据格式化工作。将Servlet中的请求信息填充到入参对象中。
- 调用`Validator`组件对已经绑定了请求消息的入参对象进行数据合法性校验，并最终生成数据绑定结果`BindingData`对象
- SpringMVC抽取`BindingResult`中的入参对象和校验错误对象，将它们赋给处理方法的响应入参。



### 自定义类型转换

[源码下载地址](https://github.com/YouChuantong/spring_learn_demo/tree/main/CustomConverter)

正常情况下，SpringMVC的默认入参是根据参数名到请求中去匹配，匹配上了就赋值。那么如果前端发了自定义格式`user=clover-19-sex-2020/08/05`，SpringMVC是无法解析的。这时需要自定义一个这种数据的类型转换器。

```java
@Controller
@RequestMapping("/user")
public class VerifyDataController {

    @RequestMapping("/getUser")
    @ResponseBody
    public String getUser(@RequestParam("user") User user) {
        System.out.println(user);
        return user.toString();
    }
}
```



#### 数据格式化

- 对属性对象的输入输出进行格式化，从其本质上讲依然属于“类型转换”的范畴。
- Spring在格式化模块中定义了一个实现`ConventionService`接口的`FormattingConvertionService`实现类。该实现类扩展了`GenericConvertionService`，因此它既具有类型转换的功能，又具有格式化的功能。
- `FormattingConversionService`拥有一个`FormattingConversionServiceFactoryBean`工厂类，后者用于在Spring上下文中构造前者。



### 实现Converter

需要实现`org.springframework.core.convert.converter.Converter`接口

```java

@Component
public class StringToUserConverter implements Serializable, Converter<String, User> {

    private static final long serialVersionUID = 5119628925082740825L;

    /**
     * 自定义类型转换器，String =====>> User。数据格式：clover-19-男-2021/08/05
     * @param source 参数
     */
    @Override
    public User convert(String source) {
        try {
            if (!StringUtils.hasText(source)) return null;
            if (!source.contains("-")) return null;
            // 通过"-"分割
            String[] split = source.split("-");
            User user = new User();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
            Short age = Short.parseShort(split[1]);
            Character sex = split[2].toCharArray()[0];
            Date da = sdf.parse(split[3]);

            user.setUserName(split[0]);
            user.setAge(age);
            user.setBirthday(da);
            user.setSex(sex);
            return user;
        } catch (Exception e) {
            return null;
        }
    }
}
```

Converter是ConvertionService中的组件，Converter写好后，还需要将它注册到ConventionService。

那么如何注册到`ConventionService`呢

- `ConversionService`是Spring类型转换体系的核心接口
- 可以利用`ConversionServiceFactoryBean`在Spring的IOC容器中定义一个`ConversionService`。Spring将自动识别出IOC容器中的`ConversionService`。并在Bean属性配置及SpringMVC处理方法入参绑定等场合使用它进行数据的转换。
- 可通过`ConversionServiceFactoryBean`的`converters`属性注册自定义的类型转换器。

```xml
<!--  conversion-service告诉SpringMVC使用我们自己配置的类型转换组件  -->
<mvc:annotation-driven conversion-service="conversionService"/>
<bean class="org.springframework.context.support.ConversionServiceFactoryBean" id="conversionService">
  <property name="converters">
    <set>
      <bean class="top.ctong.bean.component.converter.StringToUserConverter"/>
    </set>
  </property>
</bean>
```

```http
GET {{url}}/user/getUser?user=clover-19-男-2021/08/05 HTTP/1.1
```

```json
{"name":"clover","sex":"男","age":19}
```





## 关于mvc:annotation-driven

- `<mvc:annotation-driven>`会自动注册`RequestMappingHandlerMapping`、`RequestMappingHandlerAdapter`和`ExceptionHandlerExceptionResolver`三个bean
- 还将提供以下支持
  1. 支持使用`ConversionService`实例对表单参数进行类型转换
  2. 支持使用`@NumberFormatAnnotation`、`@DateTimeFormat`注解完成数据类型的格式化。
  3. 支持使用`@Valid`注解对JavaBean实例进行JSR303验证
  4. 支持使用`@RequestBody`和`@ResponseBody`注解



在Spring中，有一个`org.springframework.beans.factory.xml.BeanDefinitionParser`接口，它的作用就是用来解析Spring中各种标签。而`org.springframework.web.servlet.config.AnnotationDrivenBeanDefinitionParser` 是它众多实现类中的其中一个，专门用来解析`<mvc:annotation-driven>`标签。

这个类解析时给容器中添加了好多好多好多组件...

### 静态资源无法访问原理

在加上`<mvc:annotation-driven>`后，发现静态资源无法访问，动态资源没问题。这时再加上一个`<mvc:default-servlet-handler/>`就都可以访问。

原因是默认情况下都没配置的时候动态资源可以访问，静态资源不能访问。在`HandlerMappings`组件中，保存了所有的请求信息，通过请求路径可以找到谁能处理当前请求，所以动态资源可以访问。
在加上`<mvc:default-servlet-handler/>`后，静态资源可以访问，而动态资源不能访问。其根本原因是，默认情况下，SpringMVC使用的是`RequestMappingHandlerMapping`，它里面保存了所有的映射信息。而加上`<mvc:default-servlet-handler/>`后，在`HandlerMapping`组件注册了一个`SimpleUrlHandlerMapping`。在这个HandlerMapping中，映射路径为`/**`，并且当前请求全部交给Tomcat处理。**但是**，`RequestMappingHandlerMapping`不见了。

`<mvc:default-servlet-handler/>`标签是由`org.springframework.web.servlet.config.DefaultServletHandlerBeanDefinitionParser`进行解析。在里面给容器中添加了`SimpleUrlHandlerMapping.class`。在`DispatcherServlet` 的`initHandlerMappings` 方法中，如果找到一个`HandlerMpping.class`类型的Bean，那么就不使用默认策略。由此可得出其原因。

刚刚说了，`<mvc:annotation-driven>`由`org.springframework.web.servlet.config.AnnotationDrivenBeanDefinitionParser` 解析，里面创建了好多好多好多组件...这好多组件中，`RequestMappingHandlerMapping`就是其中之一。



## 数据校验

只做前端的校验是不安全的，重要的数据必须在后端进行校验：

1. 可以将用户传过来的数据进行校验，如果校验失败可以转发到相应页面进行处理或返回错误代码。

2. SpringMVC可以根据JSR303来做数据校验
   
    > JSR303是java为Bean数据合法性校验提供的标准框架，它已经包含在	JavaEE6.0中。
    > JSR303通过在Bean属性上标注类似于`@NotNull`、`@Max`等标准的注解执行校验规则，并通过标准的验证接口对Bean进行验证
    >
    > 
    
    | 注解                       | 功能说明                                                 |
    | -------------------------- | -------------------------------------------------------- |
    | @Nnull                     | 被标注的元素必须为null                                   |
    | @NotNull                   | 被标注的元素必须不为null                                 |
    | @AssertTrue                | 被标注的元素必须为true                                   |
    | @AssertFalse               | 被标注的元素必须为false                                  |
    | @Min(value)                | 被标注的元素必须是一个数字，其值必须大于等于指定的最小值 |
    | @Max(value)                | 被标注的元素必须是一个数字，其值必须小于等于指定的最大值 |
    | @DecimaiMin(value)         | 被标注的元素必须是一个数字，其值必须大于等于指定的最小值 |
    | @DecimaiMax(value)         | 被标注的元素必须是一个数字，其值必须小于等于指定的最大值 |
    | @Size(max, min)            | 被标注的元素大小必须在指定的范围内                       |
    | @Digits(integer, fraction) | 被标注的元素必须是一个数字，其值必须在可接受的范围内     |
    | @Past                      | 被标注的元素必须是一个过去的日期                         |
    | @Future                    | 被标注的元素必须是在一个未来的日期                       |
    | @Pattern(value)            | 被标注的元素必须符合指定的正则表达式                     |
    
    

### hibernate-validator

使用hibernate实现的数据校验api。[下载地址：传送门](http://hibernate.org/validator/releases/)

导入这些包

```
hibernate-validator-annotation-processor-5.0.3.Final.jar
hibernate-validator-cdi-5.0.3.Final.jar
hibernate-validator-5.0.3.Final.jar
classmate-1.0.0.jar
jboss-logging-3.1.1.GA.jar
validation-api-1.1.0.Final.jar
```

导入这些包之后就可以使用上面列出的注解，如果使用的tomcat是7.0以上，不需要导入`lib/required`文件夹中带有`el` 的jar包。



### 使用

只需要给javaBean的属性加上校验注解

```java
@Data
public class Employee implements Serializable {

    private static final long serialVersionUID = -2000200405058799837L;

    /**
     * 员工姓名
     */
    @NotEmpty
    private String empName;

    /**
     * 邮箱
     */
    @Email
    private String email;

    /**
     * 年龄
     */
    @Min(18)
    private Integer age;

    /**
     * 生日
     */
    @Past
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date birth;

}
```

在SpringMC封装对象的时候，使用`@Valid`告诉SpringMVC这个javaBean需要校验:

```java
public String testValidator(@Valid Employee employee) { ... }
```

如果需要获取校验结果，那么在需要校验的参数后面**紧跟**一个`BindingResult`参数，这个参数封装了校验结果。

```java
public String testValidator(@Valid Employee employee, BindingResult result) { ... }
```

`BindingResult`有几个比较常用的方法：

- `hasErrors()` 检查验证结果是否有错误，如果有错误则返回`true`反之`false`
- `getAllErrors()` 获取全部异常
- `getFieldErrors()`获取字段错误信息



## SpringMVC响应JSON

导入这三个包

```
jackson-annotations-2.1.5.jar
jackson-core-2.1.5.jar
jackson-databind-2.1.5.jar
```

完事之后在你需要返回JSON的控制器中加上`@ResponseBody`，表示返回值不使用视图解析器处理，而是使用返回值解析器进行处理。如果返回的是一个对象，那么由Jackson包将返回值自动转为json格式。

```java
@RequestMapping("/hibernate-validator")
@ResponseBody
public Employee testValidator(@Valid Employee employee, BindingResult result){}
```



### JsonFormat

在jackson中，除了能返回json外，他还提供了非常多的注解，jackson工作时使用对应处理器处理被这些注解标注的属性。而`JsonFormat`就是其中之一，具体可查看`com.fasterxml.jackson.annotation`包下的注解。

```java
@JsonFormat(pattern = "yyyy-MM-dd")
private Date birth;
```

 返回结果

```json
{"empName":"你好","email":"21312","age":1232,"birth":"2021-08-15"}
```



## ResponseBody

`@RequestBody`可以标注在类上，也可以标注在方法上。

- 在类上：表示当前类中所有`RequestMapping`都使用返回值解析器的方式解析而不再使用视图解析器。
- 在方法上：表示当前`RequestMapping`使用返回值解析器的方式解析而不再使用视图解析器。



它会把解析器处理好的数据放到响应体中：

```java
/**
 * ResponseBody响应数据
 */
@RequestMapping("/response-body")
@ResponseBody
public String testResponseBody() {
    return "hello world";
}
```

```http
GET http://localhost:8080/springmvc_helloworld_war_exploded/validator/response-body HTTP/1.1
```

```http
HTTP/1.1 200 
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 11
Date: Sun, 15 Aug 2021 01:11:44 GMT
Connection: close

hello world
```





## RequestBody

`@RequestBody`用于获取一个请求的请求体（只有POST有请求体）

发送一个POST请求

```http
POST http://localhost:8080/springmvc_helloworld_war_exploded/validator/request-body HTTP/1.1
Content-Type: application/json

{
    "empName":"你好",
    "email":"21312",
    "age":1232,
    "birth":"2021-08-15"
}
```

```java
/**
 * 通过ResponseBody获取请求体
 * @param employee
 * @return
 */
@RequestMapping(value = "/request-body", method = RequestMethod.POST)
@ResponseBody
public Employee testRequestBody(@RequestBody Employee employee) {
    log.info("employee: {}", employee);
    return employee;
}
```

```
INFO  08-15 08:57:45 employee: Employee(empName=你好, email=21312, age=1232, birth=Sun Aug 15 08:00:00 CST 2021)  (EmployeeController.java:70)
```

```http
HTTP/1.1 200 
Content-Type: application/json;charset=UTF-8
Transfer-Encoding: chunked
Date: Sun, 15 Aug 2021 00:57:45 GMT
Connection: close

{
  "empName": "你好",
  "email": "21312",
  "age": 1232,
  "birth": "2021-08-15"
}
```

这种方式非常依赖`Content-Type`，如果你没告诉服务器你发送的是什么数据类型，那么服务器可能会出现异常。因为SpringMVC是根据请求头来匹配对应的解析器。







## HttpEntity

可以获取请求中的大部分信息

```java
@RequestMapping(value = "/test-http-entity", method = RequestMethod.POST)
@ResponseBody
public String testHttpEntity(HttpEntity<String> httpEntity) {
    return httpEntity.toString();
}
```

```http
POST http://localhost:8080/springmvc_helloworld_war_exploded/validator/test-http-entity HTTP/1.1
```

```http
HTTP/1.1 200 
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 185
Date: Sun, 15 Aug 2021 01:20:15 GMT
Connection: close

<,{user-agent=[vscode-restclient], accept-encoding=[gzip, deflate], cookie=[JSESSIONID=7938CB3D5740911ECD4697D79118757B], host=[localhost:8080], connection=[close], content-length=[0]}>

```



## ResponseEntity

 `ResponseEntity`可以将返回数据放在响应体中，并且支持自定义响应头和状态码。如果返回值是`ResponseEntity`，那就不要加`@ResponseBody`。

```java
@RequestMapping("/test-response-entity")
public ResponseEntity<String> testResponseEntity(){
    MultiValueMap<String, String> headers = new HttpHeaders();
    String body = "hello man!";
    return new ResponseEntity<>(body, headers, HttpStatus.OK);
}
```

```http
POST http://localhost:8080/springmvc_helloworld_war_exploded/validator/test-response-entity HTTP/1.1
```

```http
HTTP/1.1 200 
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 10
Date: Sun, 15 Aug 2021 01:32:48 GMT
Connection: close

hello man!
```



## SpringMVC文件下载

主要是使用`ResponseEntity`定制响应头

```java
@RequestMapping("/")
public ResponseEntity<byte[]> downloadFile(HttpServletRequest request) throws IOException {

    // 找到项目真实路径
    ServletContext servletContext = request.getServletContext();
    // 找到下载文件
    String realPath = servletContext.getRealPath("/WEB-INF/classes/a8154754ca508cf399952de76c60693f.jpg");
    FileInputStream fileInputStream = new FileInputStream(realPath);

    byte[] bytes = new byte[fileInputStream.available()];
    fileInputStream.read(bytes);
    fileInputStream.close();
    MultiValueMap<String, String> headers = new HttpHeaders();
    headers.set("Content-Disposition", "attachment;filename=a8154754ca508cf399952de76c60693f.jpg");

    // 将文件流返回
    return new ResponseEntity<>(bytes, headers, HttpStatus.OK);
}
```



## 文件上传

导入依赖包：

```
commons-io-2.0.jar
commons-fileupload-1.2.1.jar
```

导入依赖包后在配置文件中配置文件上传解析器（文件上传解析器是SpringMVC九大组件之一）

```xml
<!--  文件上传解析器  -->
<bean id="multipartResolver"
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!--    配置文件上传最大大小    -->
    <property name="maxUploadSize" value="#{1024 * 1024 * 20}"/>
    <!--    配置默认字符编码    -->
    <property name="defaultEncoding" value="utf-8"/>
</bean>
```

在SpringMVC中，只需要有一个`MultipartFile`类型的参数，并且`Content-Type`是`multipart/form-data`就可以得到上传的图片。

```java
@RequestMapping("/upload")
public void upload(
        @RequestParam(value = "userName", required = false) String userName,
        @RequestParam(value = "file", required = false) MultipartFile file) {
    log.info("userName: ====>>> {}", userName);
    if (file != null) {
        log.info("fileName: =====>>> {}", file.getName());
    }
}
```

```http
POST http://localhost:8080/springmvc_helloworld_war_exploded/file/upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="userName"

userName
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="美少女.jpg"
Content-Type: image/jpeg

< /Users/clover/Desktop/美少女.jpg
------WebKitFormBoundary7MA4YWxkTrZu0gW--

```

控制台打印：

```
INFO  08-15 17:02:34 userName: ====>>> userName  (FileController.java:71) 
INFO  08-15 17:02:58 fileName: =====>>> file  (FileController.java:73) 
```

如果需要保存上传的文件，只需要使用`MultipartFile`中提供的`transferTo()`方法就能快速保存文件：

```java
file.transferTo(new File("/Users/clover/Desktop/" + System.currentTimeMillis() + ".jpg"));
```



### 多文件上传

只需要将`MultipartFile`参数声明为数组或者`List`即可

```java
@RequestMapping("/upload")
public void upload(@RequestParam(value = "userName", required = false) String userName,
        @RequestParam(value = "file", required = false) MultipartFile[] files) throws IOException {
    log.info("userName: ====>>> {}", userName);
    if (files != null) {
        for (MultipartFile file : files) {
            log.info("fileName: =====>>> {}", file.getName());
            file.transferTo(new File("/Users/clover/Desktop/" + System.currentTimeMillis() + ".jpg"));
        }

    }
}
```

```http
POST http://localhost:8080/springmvc_helloworld_war_exploded/file/upload HTTP/1.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="userName"

userName
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="0.jpg"
Content-Type: image/jpeg

< /Users/clover/Downloads/f2215fb6a14e1c6742770c18b5166872.jpg
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="1.jpg"
Content-Type: image/jpeg

< /Users/clover/Downloads/a8154754ca508cf399952de76c60693f(1).jpg
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="2.jpg"
Content-Type: image/jpeg

< /Users/clover/Downloads/wallpaper (1).jpg
------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="3.jpg"
Content-Type: image/jpeg

< /Users/clover/Downloads/wallpaper.jpg
------WebKitFormBoundary7MA4YWxkTrZu0gW--

```







## HttpMessageConverter(内容协商)

> 更详细的原理、用法在我博客SpringBoot学习笔记中有记录

![HttpMessageConverter工作原理](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-08-15%2015.52.22.png)

- `HttpMessageConverter<T>`： 是Spring3.0新添加的一个接口，**负责将请求信息转换为一个对象（类型为T），将对象输出为响应信息**
- `HttpMessageConverter<T>`接口定义的方法：
  - `boolean canRead(Class<?> clazz, MediaType mediaType);`：指定转换器可以读取的对象类型，即转换器是否可将请求信息转换为clazz类型的对象，同时指定支持MIME类型（text/html，application/json等）
  - `boolean canWrite(Class<?> clazz, MediaType mediaType);`：指定转换器是否可将clazz类型的对象写到响应流中，响应流支持的媒体类型在MediaType中定义。
  - `List<MediaType> getSupportedMediaTypes();`：该转换器支持的媒体类型。
  - `T read(Class<? extends T> clazz, HttpInputMessage inputMessage)throws IOException,HttpMessageNotReadableException;`：将请求信息流转换为T类型的对象。
  - `void write(T t, MediaType contentType, HttpOutputMessage outputMessage)throws IOException,HttpMessageNotWritableException;`：将T类型的对象写到响应流中，同时指定相应的媒体类型为`contentType`
- 在加入Jackson 包后，有个叫`MappingJackson2HttpMessageConverter`的转换器被加入了`RequestMappingHandlerAdapter`的`HttpMessageConverter`



这东西其实就是用来做请求/响应数据的格式转换的。



## 拦截器

SpringMVC提供了拦截器，它允许运行目标方法之前进行一些拦截工作，或者目标方法运行之后进行一些其他处理。

SpringMVC的拦截器是一个接口：`org.springframework.web.servlet.HandlerInterceptor`，它有三个方法

- `preHandle` 在目标方法运行之前调用。它需要返回一个Boolean，他类似javaWeb过滤器的`chain.doFilter()`。
  - `true` 表示通过
  - `false` 表示不通过
- `postHandle` 在目标方法执行之后被调用。
- `afterCompletion`在请求全部执行完成之后被调用。



### 自定义拦截器

实现`HandlerInterceptor`接口

```java
@Component
public class LoginInterceptor implements Serializable, HandlerInterceptor {

    private static final long serialVersionUID = -3000412083847294171L;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object o) {
        HttpSession session = request.getSession();
        return session.getAttribute("user") != null;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o,
            ModelAndView modelAndView) throws Exception {
        // 无需实现
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
            Object o, Exception e) throws Exception {
        // 无需实现
    }

}
```

在SpringMVC配置文件中注册这个拦截器，并且配置这个拦截器的拦截规则。

```xml
<!--  配置拦截器  -->
<mvc:interceptors>
  <!--    配置某个拦截器，默认拦截所有请求    -->
  <bean id="loginInterceptor" class="top.ctong.springmvc.interceptor.LoginInterceptor"/>
</mvc:interceptors>
```

如果你需要定义拦截规则，那么还需要借助`<mvc:interceptor>`标签和`<mvc:mapping>`标签

- `path` 指定拦截的路径

```xml
<mvc:interceptors>
  <mvc:interceptor>
    <!--     定义拦截规则       -->
    <mvc:mapping path="/api"/>
    <bean id="loginInterceptor" class="top.ctong.springmvc.interceptor.LoginInterceptor"/>
  </mvc:interceptor>
</mvc:interceptors>
```

若需要**排除**哪些路径，可以通过`<mvc:exclude-mapping/>`标签指定排除规则

```xml
<mvc:interceptors>
  <mvc:interceptor>
    <mvc:exclude-mapping path="/static"/>
    <bean id="loginInterceptor" class="top.ctong.springmvc.interceptor.LoginInterceptor"/>
  </mvc:interceptor>
</mvc:interceptors>
```



## 异常处理

- SpringMVC通过`HandlerExceptionResolver` 处理程序的异常，包括Handler映射、数据绑定以及目标方法执行时发生的异常。
- SpringMVC默认提供的`HandlerExceptionResolver`的实现类
  - `ExceptionHandlerExceptionResolver`
  - `ResponseStatusExceptionResolver`
  - `DefaultHandlerExceptionResolver`

> 在SpringMVC处理异常源码中，它默认提供的那三个异常基本只能解决Spring/SpringMVC出现的异常。解决不了的异常都抛给Tomcat

### @ExceptionHandler

这个注解标注在`Controller`的某一个异常处理方法上，用于告诉SpringMVC这个方法专门处理**这个类**的异常。

```java
@Controller
public class EmployeeController {

    @ExceptionHandler
    public void handlerException() {...}

}
```

这个注解有一个属性

- `Class<? extends Throwable>[] value()` 用于指定这个方法处理哪些异常信息。默认全部异常都能处理

若需要获取异常信息，可以在写一个`Exception`类型的参数，SpringMVC会自动把异常信息传过来。
注意：`@ExceptionHandler`标注的方法参数可以随便写，但SpringMVC只认识`Exception`

```java
@ExceptionHandler({NullPointerException.class})
public String handlerException(Exception e) {
  return "error.jsp";
}
```

如果要将异常信息返回到页面，可以返回一个`ModelAndView`.

```java
@ExceptionHandler({NullPointerException.class})
public ModelAndView handlerException(Exception e) {
  ModelAndView mv = new ModelAndView("/error.jsp");
  mv.addObject("errmsg",e.getMessage());
  return mv;
}
```



### @ControllerAdvice全局处理异常

由于`@ExceptionHandler`在`Controller`中只能处理当前`Controller`中发生的异常，这种方式会导致代码无法复用，所以SpringMVC写了一个`ControllerAdvice`注解，专门处理异常的类。

标注了`ControllerAdvice`的类中所有标注了`@ExceptionHandler`的方法就能处理全局异常。



### @ResponseStatus

`@ResponseStatus`这个注解用于标注在自定义异常上，SpringMVC可以使用`ResponseStatusExceptionResolver`异常解析器解析它，它有两个属性：

- `HttpStatus value();` 响应状态码，解析器会将这个状态码响应出去
- `String reason() default "";`  错误原因



### SimpleMappingExceptionHandle

映射异常信息

```xml
<bean id="simpleMappingExceptionHandle"
      class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
  <!--    exceptionMappings配置哪些异常去哪些页面，它是一个Properties    -->
  <property name="exceptionMappings">
    <props>
      <!-- key：异常全类名，value：要去的页面 -->
      <prop key="java.lang.ArithmeticException">/error.jsp</prop>
    </props>
  </property>
</bean>
```

在错误页面如果想要取错误信息，需要使用`${ex}`，SimpleMappingExceptionResolver也提供了一个属性可以修改封装错误时异常信息用的Key。

```xml
<bean id="simpleMappingExceptionHandle"
      class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
  <property name="exceptionAttribute" value="errmsg"/>
</bean>
```



## SpringMVC运行流程

- 前端控制器(DispatcherServlet)收到请求后会调用`doDispath`进行处理。
- 根据HandlerMapping中保存的请求映射信息找到处理器当前处理器的执行链，执行链中包含拦截器。
- 根据当前处理器找到他的HandlerAdapter(适配器)
- 执行拦截器`preHandle`方法
- 使用适配器执行目标方法并返回`ModelAndView`
  - `ModelAttribute` 注解标注的方法先于目标方法执行
  - 确定执行目标方法参数
    1. 有注解
    2. 没有注解
       1. 看是否是Model、Map等
       2. 如果是自定义类型
          1. 从隐含模型中看有没有，如果有就从模型中取值
          2. 再看是否是`SessionAttribute`标注的属性，如果是就从Sessiong中拿，如果拿不到就抛异常
          3. 如果都不是，就利用反射创建对象
- 执行拦截器的`postHandle`方法
- 处理结果；（页面渲染流程）
  - 如果有异常就使用异常解析器处理异常。处理完了还会返回`ModelAndView`
  - 调用render进行页面渲染。
    1. 视图解析器根据视图名得到试图对象
    2. 视图对象调用render方法完成渲染
  - 执行拦截器的`afterCompletion`方法