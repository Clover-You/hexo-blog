title: 14、Java this关键字
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:42:27
---
1. `this`是一个关键字，翻译为：**这个**
2. `this`是一个引用，`this`是一个变量,`this`变量中保存了内存地址指向自身，`this`存储在`JVM`堆内存Java对象内部。
3. 创建100个对象，每一个对象都有`this`，也就是说有100个不同的`this`
4. 没有`static`关键字的方法被称为**实例方法**
5. 没有`static`关键字的变量被称为**实例变量**
6. 实例变量、实例方法在堆内存中存储，所以访问该数据的时候，必须先创建对象，通过引用方法访问。
7. `this`可以出现在“**实例方法**”中，`this`指向当前正在执行的对象。(`this`代表当前对象)
8. `this`在多数情况下都可以省略不写。
   - 用来区分局部变量和实例变量的时候`this`不能省略

1. 如果一个程序执行过程中没有“**当前对象**”，因为带有`static`的方法是通过类名的方式访问的，或者说这个“**上下文**”当中没有"**当前对象**"，自然也不存在`this`(`this`代表的是当前正在执行这个动作的对象)
2. `this`不能使用在带有`static`方法中



**例如以下没有当前对象程序：**

```java
public class User{
  public String name;
  public static String getName(){
    return this.name;
  }
}
```

以上程序在编译过程中报错`connot be referenced from a static context.`

`name`是一个实例变量，以上代码的含义是：访问当前对象的`name`，`getName`没有当前对象，自然也不能访问当前对象的`name`。如果`static`要访问实例变量，必须通过**实例化**对象访问。例如`User u = new User();`



**以下程序不会出现空指针异常：**

```java
public class MainApplication{
  public static void main(String[] args){
    User u = new User();
    u = null;
    System.out.println(u.getName());
  }
}

class User{
  public static String getName(){
		return "getName";
  }
}
```



带有`static`的方法，其实既可以采用类名的方式访问，也可以采用引用的方式访问，但是即使采用引用的方式去访问，实际上执行的时候和引用指向的对象无关。



### `this` 内存结构图


![image.png](http://qiniu-note-image.ctong.top/note/images/image-6f47ab1909474ebca9f3b85ba58faa46.png)