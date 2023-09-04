title: 9、Java类的定义
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:35:35
---
1. 语法结构：
   - [修饰符列表] class 类名 {}`
   - 类体里面可以有：
     - 属性
     - 方法

2. 学生类，描述所有学生对象的共同特征：
   - 学生对象有哪些状态信息：
     - 学号
     - 名字
     - 性别
     - 年龄
     - 住址
     - ...
   - 学生对象有哪些动作信息：
     - 吃饭
     - 睡觉
     - 学习
     - 玩
     - 唱歌
     - 跳舞
     - ...
3. 属性通常是采用一个变量的方式来完成定义的。
   - int no;
   - int age;
   - String name;
   - String address;
   - boolean sex;
4. 在类体当中，方法体之外定义的变量被称为**“成员变量”**
   - 成员变量没有赋值时，系统会赋默认值：一切向0看齐。
5. 方法描述的是对象的动作信息
6. 属性描述的是对象的状态信息
7. Java语言中，所有的class都是引用类型。



---

`Student`是一个类，代表了所有学生对象。是一个学生模板。

以下程序表述的是一个对象的属性。

```java
public class Student{
  public int age;
  public int no;
  public String name;
  public String address;
}
```