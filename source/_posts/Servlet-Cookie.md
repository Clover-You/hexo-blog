title: Servlet Cookie
categories: 随笔
tags: [随笔,JAVA]
date: 2022-01-02 18:24:00
---
1. Cookie是什么？

   - Cookie可以保存会话状态，但是这个会话状态是保留在客户端上。
   - 只要Cookie清楚或者失效，这个会话状态就没有了。
   - Cookie是保存在客户端上的
   - Cookie可以保存在浏览器的缓存中，浏览器关闭，Cookie消失。Cookie也可以保存在客户端的硬盘文件中，浏览器关闭Cookie还在，除非Cookie失效。

2. Cookie只有在javaweb中有吗？

   - Cookie不止是在javaweb中存在，只要是web开发，只要是B/S架构的系统，只要是基于HTTP协议，就有Cookie的存在。
   - Cookie这种机制是HTTP协议规定的。

3. Cookie实现的功能常见的有：

   - 保留购物车商品的状态在客户端上
   - xxx天内免登录

4. java中Cookie被当作类来处理，使用new运算符可以创建Cookie对象，而且Cookie有两部分组成，分别是Cookie的name和value，name和value都是字符串类型。

5. 在java中怎么创建Cookie，Cookie是在javax.servlet.http包下的

   ```java
   improt javax.servlet.http.Cookie;
   Cookie cookie = new Cookie(name, value);
   ```

6. 服务器可以一次向浏览器发送多个Cookie

7. 默认情况下，服务器发送Cookie给浏览器之后，浏览器将Cookie保存在缓存当中，只要不关闭浏览器，Cookie永远存在，并且有效。当浏览器关闭之后，缓存中的Cookie清除。

8. 在浏览器客户端无论是硬盘文件中还是缓存中保存的Cookie，什么时候会再次发送给服务器呢？

   - 浏览器会不会提交发送这些Cookie给服务器，和请求路径有关系。
   - 请求路径和Cookie是紧密关联的。
   - 不同的请求路径会发送提交不同的Cookie

9. 默认情况下Cookie会和哪些路径绑定在一起？

   - 只会在请求和servlet同路径的情况下才会携带cookie中存储的数据，包含同级目录和下级目录。

10. Cookie路径是可以指定的，可以通过java程序进行设置，保证Cookie和某个特定的路径绑定在一起。

    ```java
    cookie.setPath("/xxx");
    ```

11. 默认情况下，没有设置Cookie的有效时长，该Cookie被默认保存在浏览器的缓存当中，只要浏览器不关闭，Cookie就一直存在，如果浏览器关闭了，那么Cookie就会消失。这是会话级别的Cookie。我们可以通过设置Cookie的有效时长，以保证Cookie保存在硬盘文件当中，但是这个有效时长必须大于0。如果Cookie时长设置为0，则该cookie会被删除，如果Cookie被设置为负数，则不会被保存。

    ```java
    cookie.setMaxAge(10000);
    ```

    



```java
public void doGet(HttpServletRequest request, HttpServletResponse response) {
  response.setContentType("text/html;charset=UTF-8");
  createAndSendCookieToBrowser(request, response);
}

private void createAndSendCookieToBrowser(HttpServletRequest request, HttpServletResponse response) {
  try {
    PrintWriter pw = response.getWriter();
    Cookie cookie = new Cookie("name", "UpYou");
    cookie.setMaxAge(10000);
    response.addCookie(cookie);
  } catch(IOException e) {
    e.printStackTrace();
  }
}
```
![image.png](http://qiniu-note-image.ctong.top/note/images/202112271104479.png)