title: 17、Java多态
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:47:11
---
## 多态基础

关于多态中涉及到的几个概念：

1. 向上转型(upcasting)
   - 子类型 --> 父类型
     又被称为：自动类型转换。
2. 向下转型(downcasting)
   - 父类型 --> 子类型
     又被称为：强制类型转换。「需要加强制类型转换符」
3. 无论是向上转型还是向下转型，两种类型之间必须要有继承关系，没有继承关系，程序是无法编译通过的。



**多态语法机制**

1. `Animal`和`Cat`之间存在继承关系，`Animal`是父类，`Cat`是子类
2. Cat is an Animal.
3. `new Cat()`创建的对象的类型是`Cat`，`cat`这个引用的数据类型是`Animal`，可见他们进行了类型转换。子类型转成父类型，称为向上转型
4. Java支持父类型引用指向子类型对象。

```java
Animal a2 = new Cat();
```

```java
public class Cat extends Animal {
  @Override
  public void move(){
    System.out.println("猫在地上走!");
  }
}
```


```java
public class Animal {
  public void move(){
    System.out.println("动物在飞行!");
  }
}
```

1. Java程序永远都分为编译阶段和运行阶段。
2. 先分析编译阶段，再分析运行阶段，编译无法通过，程序是无法运行的。
3. 编译阶段编译期检查`cat`这个引用的数据类型为`Animal`,由于`Animal.class`字节码文件中有`move()`方法，所以编译通过了。这个过程我们称为静态绑定，编译阶段绑定。只有静态绑定成功后才有后续的运行。
4. 在程序运行阶段，`JVM`堆内存当中真实存在的对象是`Cat`对象，那么以下程序在运行阶段一定会调用`Cat`对象的`move()`方法，此时发生了程序的动态绑定，运行阶段绑定。
5. 无论`Cat`类有没有重写`move`方法，运行阶段调用的是`Cat`对象的`move`方法，因为底层真实对象是`Cat`对象
6. 父类型引用指向子类型对象这种机制导致程序在编译阶段绑定和运行阶段绑定两种不同的状态、形态，这种机制可以成为一种多态语法机制。



输出结果`猫在地上走!`

```java
a2.move();
```



假设`Cat里有一个catchMouse`方法，以下程序为什么不能调用？

因为编译阶段编译器检查到`a2`的类型是`Animal`类型，从`Animal.class`字节码文件中查找`catchMouse()`方法，最终没有找到该方法，导致静态绑定失败，没有绑定成功，也就是说编译失败了。

```java
a2.carchMouse();
```



- 若想要调用`catchMouse()`方法，怎么办？

  1. `a2`是无法直接调用的，因为`a2`的类型是`Animal`，`Animal`中没有`catchMouse()`方法。我们可以将`a2`强制类型转换为`Cat`类型

  2. `a2`类型是`Animal`（父类），转换成`Cat`类型（子类），被称为向下转型/downcasting/强制类型转换。

- 什么时候需要使用向下转型呢？

  1. 当调用的方法是子类型中特有的，在父类型中不存在。必须向下转型。

**向下转型也需要两种类型之间必须有继承关系。不然编译报错。强制类型转换需要加强制类型转换符。**

```java
((Cat)a2).catchMouse();
```



**`Animal a2 = new Cat();`内存图**

![image.png](http://qiniu-note-image.ctong.top/note/images/image-8db0c4e196ff4d25a0569a7529e31ed2.png)

---



```java
public class Bird extends Animal{
  public void move(){
    System.out.println("小鸟在天上飞!");
  }
}
```

```java
Animal a3 = new Bird();
```

以下程序会不会报错？为什么？？

1. 以下程序编译时没有问题的，因为编译器检查到`a3`的数据类型是`Animal`，`Animal`和`Cat`之间存在继承关系，并且`Animal`是父类型，`Cat`是子类型，父类型转换子类型叫做向下转型，语法通过。
2. 程序在运行阶段出现异常，因为`JVM`堆内存中真实存在的对象是`Bird`类型，`Bird`对象无法转换成`Cat`对象，因为两种类型之间不存在任何继承关系，此时出现著名的异常：
   `java.lang.ClassCastException`
   类型转换异常，这种异常总是在**向下转型**的时候发生。

```java
Cat c3 = (Cat)a3;
```

1. 以上异常只有在强制类型转换的时候会发生，也就是说“向下转型”存在安全隐患（编译过了，但运行错了！）
2. 向上转型只要编译过了，运行一定不会出问题。`Animal a2 = new Cat();`
3. 向下转型编译通过，运行可能发生错误
   `Animal a3 = new Bird();`
   `Cat c3 = (Cat)a3;`
4. 怎么避免向下转型出现的异常`ClassCastExcption`？
   - 使用`instanceof`运算符可以避免出现以上的异常。
5. `instanceof`
   1. 语法
      `(引用 instanceof 数据类型名称)`
   2. `instanceof`运算符的执行结果是布尔类型，结果是`true`/`false`
      - 关于运算结果`true`/`false`
          假设(`a instanceof Animal`)
        1. `true`表示`a`这个引用指向的对象是一个`Animal`类型
        2. `false`表示`a`这个引用指向的对象不是一个`Animal`类型
6. Java规范中要求：在进行强制类型转换之前，建议采用`instanceof`运算符进行判断，避免`ClassCastException`异常的发生



### instanceof运算符

```java
public class Cat extends Animal {
  @Override
  public void move(){
    System.out.println("猫在地上走!");
  }
  public void catchMouse(){
    System.out.println("我开始抓老鼠了!");
  }
}
```

使用`instanceof`

```java
Animal cat = new Cat();
if (cat instanceof Bird)
  ((Bird)cat).move();
else if (cat instanceof Cat)
  ((Cat)cat).catchMouse();
```

输出`我开始抓老鼠了!`

```java
Animal cat = new Bird();
if (cat instanceof Bird)
  ((Bird)cat).move();
else if (cat instanceof Cat)
  ((Cat)cat).catchMouse();
```

输出`小鸟在天上飞!`





## 多态的作用

多态在实际开发中的作用，以下以主人喂养宠物为例说明多态：

1. 主人喂养宠物这个场景要实现需要进行的抽象：

   - 主人「类」

     1. 主人可以喂养宠物，所以主人有喂养的这个动作

   - 宠物「类」

     1. 宠物可以吃东西，所以宠物有吃东西这个动作
2. 定义好类，后将类实例化为对象，给一个环境驱使一下，让各个对象之间协作起来形成一个系统。
3. 多态的作用是什么？
   - 降低程序的耦合度，提高程序的扩展力
   - 能使用多态尽量使用多态
   - 父类型引用向子类型对象。
4. 面向抽象编程，尽量不要面向具体编程

### 创建类：

```java
//主人类
public class Hoster{
  public void fedd(Cat cat){
    cat.eat();
  }
}

//猫类
public class Cat {
  public void eat(){
    System.out.printf("猫在吃小鱼干!\n");
  }
}
```



### 实例

```java
Hoster hoster = new Hoster();
Cat tom = new Cat();
tom.petName = "tom";
hoster.fedd(tom);
```

运行输出:
`tom在吃小鱼干!`

如果主人想养狗，那么以上程序不使用多态需要这样改动：

```java
public class Hoster{
  public void fedd(Cat cat){
    cat.eat();
  }
  public String petName;
  public void fedd(Dog dog){
    dog.eat();
  }
}
//狗类
public class Dog {
  String catType;
  public void eat(){
    System.out.printf("狗在吃骨头!\n");
  }
}
```

以上程序因为耦合度太高，无法灵活拓展。如果需要程序拓展能力强，需要降低程序的耦合度「解耦合」，提高扩展能力



### 使用多态

添加一个宠物类，将所有宠物都继承这个宠物类。

```java
public class HousePet{
  public void eat(){
    System.out.printf("主人不给东西吃!\n");
  }
}
// 将所有宠物类继承宠物类
public class Dog extends HousePet{...}
public class Cat extends HousePet{...}
```

主人类使用多态

`Hoster`主人类面向的是一个抽象的`HousePet`，不再是具体的宠物.

```java
public class Hoster{
  public void fedd(HousePet HP){
    HP.eat();
  }
}
```

使用时只需要将继承了`HousePet`的宠物传给`Hoster`下的`fedd`方法

```java
Hoster hoster = new Hoster();
hoster.fedd(new Cat());
hoster.fedd(new Dog());
```

输出：

`猫在吃小鱼干!`

`狗在吃骨头!`