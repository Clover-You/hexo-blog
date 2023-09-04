title: 7、Java方法重载
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:32:23
---
## 使用

以下程序有哪些缺点？

```java
public class overLoad{
  public static void main(String[] args){
    int a = sumInt(1, 2);
    double b = sumBouble(1.0, 2.0);
    long c = sumLong(1L, 2L);
  }
  public static int sumInt(int a, int b){
	  return a + b;
	}
  public static double sumDouble(double a, double b){
    return a + b;
  }
  public static long sumLong(long a, long b){
    return a + b;
  }
}
```

1. `sumInt` `sumLong` `sumDouble` 方法虽然不同，但是功能相似，都是求和
2. 以上程序当中功能相似的方法，分别起了三个不同的名字，这对于程序员来说，调用方法的时候不方便，程序员需要记忆更多的方法。

有没有这样一种机制：

		这些功能虽然不同，但是**功能相似**的时候，可以让程序员使用这些方法的时候就像在使用同一个方法一样，这		样程序员以后编写代码比较方便，也不许要记忆更多的方法名。

有这种机制：

- Java提供了方法重载机制`overload`
- 不是所有语言都支持重载机制，例如: `JavaScript` 就不支持

#### 改写

```java
public class overLoad{
  public static void main(String[] args){
    int a = sum(1, 2);
    double b = sum(1.0, 2.0);
    long c = sum(1L, 2L);
  }
  public static int sum(int a, int b){
	  return a + b;
	}
  public static double sum(double a, double b){
    return a + b;
  }
  public static long sum(long a, long b){
    return a + b;
  }
}
```

重载：

- 使用时就像在使用一个方法，但实际上是使用了 3 个方法。
- 调用时会根据传递的：参数类型，对应调用的方法不同。



## 理论



方法重载：

1. 方法重载又被称为：`overload`
2. 什么时候考虑使用方法重载？
   - 功能相似的时候，尽可能选择重载**「功能不同、不相似的时候，尽可能让方法名不同」**
3. 什么条件满足之后构成了方法重载？
   - 同级
   - 方法名相同
   - 参数列表
     - 参数长度
     - 类型不同
     - 顺序不同
4. 方法重载和什么有关系，和什么没有关系？
   - 和方法名 + 参数列表有关
   - 和返回值类型无关
   - 修饰符列表无关