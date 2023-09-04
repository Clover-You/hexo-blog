title: 33、Java枚举类型
categories: Java 基础
tags: [学习笔记,JAVA]
date: 2022-01-02 16:58:30
---
## 引入

以下案例没有使用java中的枚举，分析以下程序在设计方面有什么缺陷？

```java
// 计算两个int类型的商，计算成功返回1，计算失败返回0
public static int divide(int a, int b) {
  try {
    int c = a / b;
    return 1;
  } catch(Exception e) {
    return 0;
  }
}
```

以上程序设计缺陷在返回值类型上。返回一个`int`不恰当，既然最后的结果只是成功和失败，最好使用布尔类型。 因为布尔类型`true`和`false`正好可以表示两种不同的状态。



以上的这个设计没问题，返回`true`和`false`表示两种情况，但是在以后的开发中，有可能遇到一个方法的执行结果返回多种情况，但是每一个都是可以数清楚的，一枚一枚都是可以列举出来的。这个布尔类型就无法满足需求了。此时需要使用java语言中的枚举类型。其实`boolean`也可以看作一个枚举，`true`和`false`是一个枚举的值。



## 枚举的使用

- 一枚一枚可以列举出来的，才建议使用枚举类型。
- 枚举编译之后也是生成**class**文件。
- 枚举也是一种引用数据类型。
- 枚举中的每一个值都可以看作是常量。



```java
public static DivideEnum divide(int a, int b) {
  try {
    int c = a / b;
    return DivideEnum.SUCCESS;
  } catch(Exception e) {
    return DivideEnum.FAIL;
  }
}
```

枚举声明

```java
enum DivideEnum {
  SUCCESS, FAIL
}
```

使用

```java
System.out.println(divide(10, 0));
```

\-- `FAIL`