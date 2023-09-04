title: 30、Java日期类
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 16:55:24
---
### 获取系统当前时间

获取系统当前时间（精确到毫秒的系统当前时间）

```java
Date dt = new Date();// Thu Jun 11 11:11:23 CST 2020
```

### Date转String

获取出来的日期是这种格式：`Thu Jun 11 11:11:23 CST 2020`,那么怎么初始化？

需要使用到`java.text`包下的`SimpleDateFormat`类。专门负责日期格式化。

```java
SimpleDateFormat sdf = new SimpleDateFormat();
```

例如

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd kk:mm ss");
```

```java
Date dt = new Date();
```

```java
sdf.format(dt)// 2020-06-11 11:32 25
```

转换格式


![image.png](http://qiniu-note-image.ctong.top/note/images/image-74ce7f657ede4347852a8f72ed7270bd.png)

### String转Date

使用`SimpleDateFormat`将`String`字符串转为`Date`,使用`parse`时需要`throws  Exception`

```java
SimpleDateFormat sdf = new SimpleDateFormat("yyy-MM-dd kk:mm ss");
```

```java
String strDate = "2020-06-11 11:43 27";
```

```java
// Thu Jun 11 11:43:27 CST 2020
Date dt = sdf.parse(strDate);
```

字符串的日期格式和`SimpleDateFormat`对象指定的日期格式要一致。不一致会出现异常：`java.text.ParseException`



### 获取方法执行耗时

`currentTimeMillis`方法获取自**1970年1月1日 00:00:00 000**到系统当前时间的毫秒数。

```java
long beginMillis = System.currentTimeMillis();
forTestTime();
long endMillis = System.currentTimeMillis();
System.out.println("执行时间：" + (endMillis - beginMillis));
```

```java
public static void forTestTime() {
  for (int i = 0; i < 1000; i++) {
    System.out.println(i);
  }
}
```

\-- `执行时间：12`