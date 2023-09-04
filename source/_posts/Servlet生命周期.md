title: Servlet生命周期
categories: 随笔
tags: [JAVA,随笔]
date: 2022-01-02 18:25:08
---
# Servlet生命周期

1. 什么是生命周期？
   生命周期表示一个Java对象从最初创建到被销毁的过程

2. Servlet的生命周期是谁来管理的？程序员可以干涉吗？

   Servlet对象的生命周期javaweb程序员是无权干涉的，包括Servlet对象的相关方法的调用，javaweb程序员也是无权干涉的。Servlet对象从最初的创建开始，方法的调用、以及最后Servlet对象的销毁，整个过程都是由WEB容器来管理的。
   Web Container管理Servlet对象的生命周期。

3. **默认情况下**，Servlet对象在WEB服务器启动阶段不会被实例化。若希望在web服务器启动阶段实例化Servlet对象，需要使用特殊设置。



---



# 简单描述Servlet生命周期

1. 用户在浏览器地址栏上输入URL：http://localhost:8080/xxx/xxx
2. web容器窃取请求路径：/xxx/xxx
3. web容器在容器上下文中找请求路径`/xxx/xxx`对应的Servlet对象
4. 若没有找到对应的Servlet对象
   1. 通过web.xml文件中相关的配置信息，得到请求路径`/xxx/xxx`对应的Servlet完整类名
   2. 通过反射机制，调用Servlet类的无参数构造方法完成Servlet对象的实例化
   3. web容器调用Servlet对象的`init`方法完成初始化操作。
   4. web容器调用Servlet对象的service方法提供服务
5. 若找到对应的Servlet对象
   1. web容器直接调用Servlet对象的service方法提供服务
6. web容器关闭的时候/webapp重新部署的时候/该Servlet对象长时间没有用户再次访问的时候，web容器会将该Servlet对象销毁，在销毁该对象之前，web容器会调用Servlet对象的destroy方法，完成销毁之前的准备。







> init方法执行的时候，Servlet对象已经创建好了。destroy方法执行的时候，Servlet对象还没有被销毁，即将被销毁。
>
> Servlet对象是单例，但是不符合单例模式，只能称为伪单例。真单例的构造方法是私有化的，Tomcat服务器是支持多线程的。所以Servlet对象在单实例多线程的环境下运行的，那么Servlet对象中若有实例变量，并且实例变量涉及到修改操作，那么这个Servlet对象一定会存在线程安全问题。