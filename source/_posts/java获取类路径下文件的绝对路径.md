title: java获取类路径下文件的绝对路径
categories: 随笔
tags: [随笔,JAVA]
date: 2022-01-02 18:21:39
---
## 获取文件绝对路径

在idea中，默认的当前路径是project的根路径，如果你使用idea的默认路径，只要离开idea换到其他位置，可能当前路径就不是project的根路径了。

**使用以下通用方式的前提是：这个文件必须在类路径下。在项目的src目录下的都是类路径下，src是类的根路径**



```java
String currentPath = Thread.currentThread().getContextClassLoader().getResource("fileName").getPath();
```

`Thread.currentThread()`是当前线程对象

`getContextClassLoader()`是线程对象的方法，可以获取到当前线程类加载器对象。

`getResource()`类加载器的方法，当前线程的类加载器默认从类的根路径下加载资源。

采用以上方法可以拿到一个文件的绝对路径

---
这种方式无法获取一个`.java`文件，因为java程序是需要编译后才能执行，而` .java`会被编译成` .class`。
而java编译时可能会被编译在别的路径下，例如IDEA，是在`target`文件下，这才是真正的类路径
![image.png](http://qiniu-note-image.ctong.top/note/images/202112271115453.png)

## 资源绑定器

java.util包下提供了一个资源绑定器，便于获取属性配置文件中的内容。使用以下这种方式的时候，属性配置文件xxx.properties必须放到类路径下。

例如我类路径下有一个`properties.properties`文件：

```java
public static String get(String key) {
  ResourceBundle rb = ResourceBundle.getBundle("properties");
  return rb.getString(key);
}
```