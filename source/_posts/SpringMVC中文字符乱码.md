title: SpringMVC中文字符乱码
categories: 随笔
tags: [随笔,SpringMvc]
date: 2022-01-02 18:29:11
---
## 问题

使用SpringMVC在返回一个字符串时发生了中文乱码问题。`produces`属性无效

```java
@RequestMapping(value = "/nihao", produces = "text/plain;charset=UTF-8")
@ResponseBody
public String hello(HttpServletResponse response) throws UnsupportedEncodingException {
    User user = new User();
    user.setSex("男");
    user.setName("Clover");
    user.setAge(19);
    return user.toString();
}
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 36
Date: Sun, 01 Aug 2021 12:20:21 GMT
Connection: close

{
  "name": "Clover",
  "sex": "?",
  "age": 19
}

```

添加常用的过滤器`org.springframework.web.filter.CharacterEncodingFilter`依然无法解决

```xml
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

## 问题根源

最后查看源码时发现问题出现在处理内容协商的时候，SpringMVC使用了一个叫做`org.springframework.http.converter.StringHttpMessageConverter`的转换器进行处理`java.lang.String`。在这个处理器中，有个一默认的编码格式，它甚至使用了`final`修饰…..

```java
public static final Charset DEFAULT_CHARSET = Charset.forName("ISO-8859-1");
```

并且，通过Postman或者REST Client发送请求时，`Accept`默认是`*/*`。

## 解决方案



### 方案一

注册一个`StringHttpMessageConverter`，注册之后不再使用SpringMVC默认的。它可以将`produces`设置为`Content-Type`。也就是说`@RequestMapping`的`produces`属性生效了

```xml
<mvc:annotation-driven>
    <mvc:message-converters>
        <bean class="org.springframework.http.converter.StringHttpMessageConverter"/>
    </mvc:message-converters>
</mvc:annotation-driven>
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Accept-Charset: ...
Content-Type: text/plain;charset=utf-8
Content-Length: 37
Date: Sun, 01 Aug 2021 13:09:35 GMT
Connection: close

{
  "name": "Clover",
  "sex": "男",
  "age": 19
}

```



### 方案二

Accept问题，SpringMVC的默认`StringHttpMessageConverter`处理的是`*/*`，那手动设置一个Accept尽可能避开它…..

```http
POST {{url}}/nihao HTTP/1.1
Accept: text/plain;charset=utf-8
```

```http
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Content-Type: text/plain;charset=utf-8
Content-Length: 38
Date: Sun, 01 Aug 2021 13:20:16 GMT
Connection: close

{
  "name": "Clover",
  "sex": "男",
  "age": 19
}
```