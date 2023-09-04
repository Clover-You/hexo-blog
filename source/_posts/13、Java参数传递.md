title: 13、Java参数传递
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:41:29
---
对象和引用的概念：

- 对象：目前在使用`new`运算符在堆内存中开辟的内存空间称为对象。
- 引用：是一个变量，不一定是局部变量，这可能是成员变量。引用保存了内存地址，指向了堆内存当中的对象。所有访问实例相关的数据，都需要通过“引用”的方式访问，因为只有通过引用才能找到对象。如果有一个空的引用，访问对象的实例相关的数据会出现空指针异常。





Java语言当中方法调用的时候涉及到参数传递的问题，参数传递实际上传递的是变量中保存的具体值。

```java
public static void main(String[] args){
  int i = 10;
  add(i);
  System.out.println("main --> " + i);
}
public static void add(int i){
  i++;
  System.out.println("add --->" + i);
}
```

`main` 方法中`i`变量是局部变量，`i`变量本身占有内存空间，所以`i`变量本身也有内存地址，`i`变量中保持的是 10 这个字面值。

`add(i);`方法在调用的时候，实际上给`add`方法传递的是`main` 方法中`i`这个变量保存的值。与`i`本身无关。



**以下程序如何执行？**
```java
public class MainApplication {
  public static void main(String[] args){
    User user = new User();
    user.setName("Rename");// 第一次做修改
    setName(user);
    System.out.println(user.getName());
  }

  public static void setName(User user){
    user.setName("UpYou");// 第二次做修改
    System.out.println("当前 name 值为：" + user.getName());
  }
}
```

上面程序分别打印出什么结果？为什么呢？

- 分别打印出
  - 当前 name 值为：UpYou
  - UpYou
  - 因为`main`中的`user`拿到的是`User`对象的内存地址，当第一次修改时，将`user`对象中的`name`字段修改为`Rename`,后又调用`setName`方法，将`User`对象的内存地址给到`setName`方法的`user`变量，调用这个对象中的`setName`方法修改`name`字段。打印输出：`当前 name 值为：UpYou`后弹栈，继续执行`main`方法，`main`使用`println`方法打印，因为堆内存中的`User`对象中的`name`字段已经被`setName`方法修改，而`user`的内存地址也是指向`User`对象，所以拿到的是已经修改后的值。







**`user`本身也有内存地址，是局部变量，`user`变量中保存的那个值，这个值是另一个Java对象在堆内存中的内存地址。**



> 在Java中，方法在调用的时候，参数在传递的时候，传的永远都是变量中保存的那个值。