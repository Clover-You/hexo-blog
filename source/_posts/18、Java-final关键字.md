title: 18、Java final关键字
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:48:48
---
关于Java语言当中final关键字：

1. `final` 是一个关键字，表示最终的，不可变的。
2. `final` 修饰的类无法被继承
3. `final` 修饰的方法无法被覆盖
4. `final` 修饰的变量一旦赋值之后，不可重新赋值「不可二次赋值」
5. `final` 修饰的实例变量，必须手动赋值，不能采用系统默认值
6. `final` 修饰的引用不能再指向其它对象，那么被指向的对象无法被垃圾回收器回收。
7. `final` 修饰的实例变量一般和`static`联合使用，被称为**常量**



以下程序编译出现错误，因为成员变量有默认值，这个默认值可能并不是该变量的最终结果，而`final`一旦赋值就不能重新赋值，所以出现错误，需要指定变量值。

```java
public class test{
  final int age;
}
```

以上程序有两种解决方案，第一种，声明时手动赋默认值

```java
final int age = 18;
```

第二种是在构造方法中赋值。因为程序执行时，构造方法执行完成之后才会运算

```java
public class MainApplication{
  final int age;
  public MainApplication(int age){
    this.age = age;
  }
}
```

---



`final`修饰的引用，一旦指向某个对象之后，不能再指向其它对象，那么被指向的对象无法被垃圾回收器回收。



```java
final User user = new User();
```



`final` 修饰的引用虽然指向某个对象之后不能指向其它对象，但是所指向的对象内部的内存是可以修改的。例如`User`类成员变量`name`是`ZhangSan`，因为`user`保存的是`User`对象的内存地址，修改的时候修改的是`User`对象内部的`name`变量

```java
final User user = new User();
user.name = "UpYou";
```



每一个中国人的国籍都是中国，而国籍是不会改变的，为了防止国籍被修改，可以添加`final`关键字修饰。`final`修饰的实例变量是不可变的，这种变量一般和`static`联合使用，被称为**常量**。

常量的定义语法格式：`public static final 类型 常量名 = 值;`

Java规范中要求所有长量的名字全部大写，每个单词之间使用下划线连接。

```java
class Chinese{
  static final String COUNTRY = "中国";
}
```





---