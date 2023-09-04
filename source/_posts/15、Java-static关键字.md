title: 15、Java static关键字
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:44:29
---
### 静态变量

以下程序中，如果创建100份,每个对象都有一个国籍，而中国人的国籍都是一样的，这样很浪费内存空间。

```java
public class Chinese {
  // 每个人身份证不同
  String id;
  // 每个人名字不同
  String name;
  //每个中国人国籍一样
  String country;
}
```

![image.png](http://qiniu-note-image.ctong.top/note/images/image-7342af6a1cac447496bfa1e54456d31b.png)


那么该如何解决这个问题？

所有对象的`couuntry`一样，这种特征属于类级别的特征，可以提升为整个模板的特征，可以在变量前添加`static`关键字修饰。如果某一个方法、变量使用了`static`修饰符，那么这个变量、方法也叫**静态**变量、方法。

`static String country;`



**静态变量在类加载的时候初始化，不需要创建对象，内存就开辟了**

**例如：**

```java
public class Chinese {
  // 每个人身份证不同
  String id;
  // 每个人名字不同
  String name;
  //每个中国人国籍一样
  static String country = "中国";
}
```

**使用:**

```java

public static void main(String[] args) {

  Chinese zhangSan = new Chinese();
  zhangSan.setName("张三");
  zhangSan.setId("1");
  Chinese liSi = new Chinese();
  liSi.setName("李四");
  liSi.setId("2");
  Chinese wangWu = new Chinese();
  wangWu.setName("王五");
  wangWu.setId("3");

  System.out.printf("我是：%s,id:%s，我在：%s\n", zhangSan.getName(), zhangSan.getId(), Chinese.country);
  System.out.printf("我是：%s,id:%s，我在：%s\n", liSi.getName(), liSi.getId(), Chinese.country);
  System.out.printf("我是：%s,id:%s，我在：%s\n", wangWu.getName(), wangWu.getId(), Chinese.country);
}
```



![image.png](http://qiniu-note-image.ctong.top/note/images/image-da349751030a4a6191be6146a89110dc.png)



![image.png](http://qiniu-note-image.ctong.top/note/images/image-b6d0e49fbe1841c1af55f062a85c2beb.png)


### 静态代码块

- 可以使用`static`关键字来定义“静态代码块":
  1. 语法格式：
     `static{java 语句}`
  2. 静态代码块在类加载时执行，并且只执行一次。
  3. 静态代码块在一个类中可以编写多个，并且遵循自上而下的顺序依次执行。
  4. 静态代码块的作用是什么？怎么用？用在哪儿？什么时候用？
     - 类发生装载动作时记录日志时使用。
     - 静态代码块是Java为程序员准备的一个特殊的时刻，这个特殊的时刻被称为类加载时刻。若希望在此时刻执行一段特殊的程序，这段程序可以直接放到静态代码块当中。

```java
public class staticClass {
  static {
    Systrm.out.println("类加载--->");
  }
  public static void main(String[] args){ ... }
}
```





### 结论



- 什么时候成员变量声明为实例变量呢？
  - 所有对象都有这个属性，但是这个属性的值会随着对象的变化而变化「不同对象的这个属性具体的值不同」
- 什么时候声成员变量声明为静态变量呢？
  - 所有对象都有这个属性，并且所有对象的这个属性的值是一样的，建议定义为静态变量，节省内存的开销。
- 静态变量在类加载的时候初始化，内存在方法区中开辟。访问的时候不需要创建对象，直接使用**类名.静态变量名**的方式访问。
- 关于Java中的`static`关键字：
  1. `static`翻译为静态的
  2. `static`修饰的方法叫静态方法
  3. `static`修饰的变量叫静态变量
  4. 所有`static`修饰的元素都称为静态的，都可以使用“类名.xxx”的方式访问
  5. `static`修饰的所有元素都是类级别的特征，和具体的对象无关。