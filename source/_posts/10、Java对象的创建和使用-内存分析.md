title: 10、Java对象的创建和使用-内存分析
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:37:10
---
### 创建

学生类是一个模板，描述了学生的特征【状态 + 行为】

当前类只描述学生的状态信息【属性】

当使用`public`修饰这个类时，这个类的类名必须与源文件名一致。

```java
public class Student{
  public int age;
  public int no;
  public String name;
  public String address;
}
```



类体是有属性和方法组成

`Student`是类，属于引用数据类型

```java
public class Student{类体}
```



由于变量定义在类体当中，方法体之外，这种变量称为成员变量。

所有学生都有学号信息，但是每个学生的学号都是不同的。

所以，要访问这个学号必须先创建对象，通过对象去访问学号信息

学号信息不能直接通过“类”去访问，所以这种成员变量又被叫做 **“实例变量”**

对象又被称为实例，实例变量又被称为对象变量。「对象级别的变量」

不创建对象，这个`no`变量的内存空间是不存在的，只有创建了对象，这个`no`变量的内存空间才会创建。

```java
public class Student{
  public int no;
}
```

### 使用

通过一个类，可以实例化N个对象

实例化对象的语法

```java
new className();
```

`new`是Java语言中的一个运算符

`new`运算符的作用是创建对象，在`JVM`堆内存当中开辟新的内存空间



## 理论

方法区内存：在类加载的时候，class字节码代码片段被加载到该内存空间当中。

栈内存（局部变量）：方法代码片段执行的时候，会给该方法分配内存空间，在栈内存中压栈

堆内存：`new`的对象在堆内存中存储

成员变量没有手动赋值的话，系统赋默认值

|        数据类型        | 默认值 |
| :--------------------: | :----: |
| byte, short, int, long |   0    |
|     float, double      |  0.0   |
|        boolean         | False  |
|          char          | \u0000 |
|      引用数据类型      |  null  |

---



```java
public class testRecursion {
    public static void main(String[] args){
      int i = 1;
      Student students = new Student();
    }
}

class Student{
   public int age;
   public String name;4
   public String addRess;
}
```

以上代码如何用内存图表示？



### 使用new时，JVM内存图如何表示
![image.png](http://qiniu-note-image.ctong.top/note/images/image-61a9c4947f2845ae8cdec72559d8c6df.png)


`i`变量保存的一个字面值`1`

`students`保存的是一个对象的内存地址，我们称`students`为**引用**

堆内存开辟的空间叫对象，对象的内存地址保存在`students`中,由于`students`保存了内存地址指向了堆内存中的对象，所以我们叫`students`为引用



> 1. 什么是对象？new 运算符在堆内存中开辟的内存空间称为对象。
>
> 2. 什么是引用？引用是一个变量，由于这个变量保存了另一个Java对象的内存地址，所以我们称之为引用
>
> 3. 在Java中，不能直接操作堆内存，Java中没有指针，如果想要访问堆内存中的数据，必须使用引用的方式去访问。
>
> 4. 局部变量在栈内存中存储。
>
> 成员变量中的实例变量在堆内存的Java对象内部存储。

### 空指针异常

例如有如下程序：

```java
public class test1{
  public static void main(String[] args){
    test2 t = new test2();
    System.out.println(t.name);
    t = null;
    System.out.println(t.name);
  }
}

class test2{
  public String name = "UpYou";
}
```

以上程序编译可以通过，运行出现异常，对象索引断裂，发生错误：`java.lang.NUllPointerException`

代码第三行时，拿到了`test2`对象的内存地址。
![image.png](http://qiniu-note-image.ctong.top/note/images/image-ffc3769ad6a244c5871c589db2ebcd1a.png)

当代码走到第5行时，`t`的值不再是`test2`的内存地址。当值改变后，意味着索引断裂。断裂之后`test2`对象会被垃圾回收，这时无论如何都无法访问到该对象。空引用无法访问实例相关的数据
![image.png](http://qiniu-note-image.ctong.top/note/images/image-baf35d428fc24a4d8c99f3feab7e7191.png)


**“实例”相关的数据：这个数据访问的时候必须有对象的参与。这种数据就是实例相关的数据**



### 总结

1. JVM（Java虚拟机）主要包括三块内存空间，分别是：栈内存、堆内存、方法区内存
2. 堆内存和方法区内存个有一个，一个线程一个栈内存
3. 方法调用的时候，该方法所需要的内存空间在栈内存中分配，称为**压栈**。方法执行结束之后，该方法所属的内存空间释放，称为弹栈。
4. 栈中主要存储的是方法体当中的局部变量。
5. 方法的代码片段以及整个类的代码片段都被存储到方法区当中，在类加载的时候这些代码片段会载入。
6. 在程序执行过程中使用`new`运算符创建的Java对象，存储在堆内存当中。对象内部有实例变量，所以实例变量存储在堆内存当中。
7. 变量分类：
   - 局部变量「方法体中声明」
   - 成员变量「方法体外声明」
     - 实例变量「前边修饰符没有`static`」
     - 静态变量「前边修饰符中有`static`」
8. 静态变量存储在方法区内存当中
9. 三块内存当中，变化最频繁的是栈内存，最先有数据的是方法区内存，垃圾回收器主要针对的是堆内存。
10. 垃圾回收器「自动垃圾回收、GC机制」什么时候会考虑将某个Java对象的内存回收呢？
    - 当随内存当中的Java对象成为垃圾数据的时候，会被垃圾回收器回收。
    - 当堆内存中的Java对象没有更多的引用指向它的时候就会被当成垃圾。当它被回收的时候，这个对象无法被访问，因为访问对象只能通过引用的方式访问。


![image.png](http://qiniu-note-image.ctong.top/note/images/image-5a05a8584f2142aab0e67d1f7c00b022.png)