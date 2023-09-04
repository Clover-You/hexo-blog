title: 21、Java 抽象类的概述
categories: Java 基础
tags: [学习笔记,JAVA]
date: 2022-01-02 16:36:00
---
- 抽象类：

  1. 什么是抽象类？

     ![image.png](http://qiniu-note-image.ctong.top/note/images/image-2ab2f44e314946b3b284e28b0e4e21d1.png)    

     - 抽象类无法实例化，无法创建对象。类和类之间有共同特征，将这些具有共同特征的类再进一步抽象形成了抽象类。由于类本身是不存在的，所以抽象类无法创建对象。
     - 抽象类和抽象类实际上可能还会有共同特征，还可以进一步再抽象
     - 类到对象是实例化，对象到类是抽象。

  2. 抽象类属于什么类型？

     - 抽象类也属于引用数据类型。

  3. 抽象类怎么定义？

     ```java
     [修饰符列表] abstract class 类名{}
     ```

  4. `final` 和 `abstract` 不能联合使用，这两个关键字是对立的。

     - `final class` 无法被继承
     - `abstract` 要被继承使用

  5. 抽象类的子类可以是抽象类，也可以是非抽象类

     ```java
       abstract class CreditCard extends BankCard{}
       abstract class BankCard{}
     ```

  6. 抽象类虽然无法实例化，但是抽象类有构造方法，这个构造方法是提供子类使用的。
  
       使用`super`给父类构造传参
  
     ```java
     public class CreditCard extends BankCard{
       public CreditCard(){
         super("UpYou");
       }
     }
     ```

     ```java
     abstract class BankCard{
       public BankCard(String name){
         this.name = name;
       }
       public String name;
     }
     ```
  
  7. 抽象类关联到一个概念：抽象方法。
     - 抽象方法: 没有实现的方法，没有方法体的方法
     `public abstract void doSome();`
     - 抽象方法特点
     1. 没有方法体，以分号结尾
       2. 前面修饰符列表中有`abstract`关键字
     - 抽象类中不一定有抽象方法
     
     - 抽象方法只能在抽象类中使用
     
     - 如果子类不是抽象类，那么子类必须重写父类的所有抽象方法。
     
       ```java
       class CreditCard extends BankCard{
         @Override
         public String getName(){
           return "Bank";
         }
       }
       abstract class BankCard{
         public abstract String getName();
       }
       ```
     
       

---

### 非抽象类继承抽象类必须将抽象方法实现

以下程序有没有问题？有什么问题？

```java
abstract class Animal{
  public abstract void move();
}
class Bird extends Animal{}
```

- 抽象类中不一定有抽象方法，抽象方法必须出现在抽象类中
- 分析
  1. `Animal`是父类，并且是抽象类
  2. `Animal`这个抽象类中有一个抽象方法`move`
  3. `Bird` 是子类，并且是非抽象的
  4. `Bird`继承`Animal`之后，会将抽象方法继承过来。「**抽象方法的概述**第7条第4项」


以上程序需要将从父类中继承过来的抽象方法进行覆盖/重写，或者也可以叫做**实现**

#### 实现

当一个非抽象类继承抽象类的时候，需要将抽象方法**重写/实现**

```java
abstract class Animal{
  public abstract void move();
}
class Bird extends Animal{
  @Override
  public void move(){
    System.out.println("我在移动！");
  }
}
```

以上程序支持多态机制使用，例如

```java
Animal bird = new Bird();// 向上转型
bird.move();
```

输出：`我在移动！`





> 1. 一个非抽象类继承抽象类，必须将抽象类中的抽象方法实现了
>
> 2. 面向抽象编程，不要面向具体编程，降低程序的耦合度，提高程序的扩展力。这种编程思想符合OCP原则。



---



  

  

  