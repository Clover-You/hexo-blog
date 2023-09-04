title: 22、Java接口
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 16:40:00
---
### 接口的基础语法

1. 接口也是一种**引用数据类型**

2. 接口是完全抽象的。「抽象类是半抽象的」或者也可以说接口是特殊的抽象类

3. 语法：
   `[修饰符列表] interface 接口名{}`

4. 接口支持多继承，一个接口可以继承多个接口

   ```java
   interface A{}
   interface B{}
   interface C extends A, B{}
   ```
   
5. 一个类可以实现多个接口

   ```java
   interface A{}
   interface B{}
   class C implements A, B{}
   ```

   

6. 接口中只包含两部分内容，一部分是：常量。一部分是：抽象方法。

7. 接口中所有元素都是`public`修饰的。「都是公开的」

8. 接口中`public abstract` 修饰符可以省略

9. 接口中的方法都是抽象方法，所以接口中的方法不能有方法体

10. 接口中的常量的`public static final`可以省略





在接口中，可以省略`public abstract`不写

``` java
interface A{
  void move();
}
```



在接口中，方法不能带有方法体：`Abstract methods do not specify a body.`

```java
interface A{
  void move(){} ;
}
```



### 接口的实现

1. 类和类之间叫做继承，类和接口之间叫做实现。
2. 当一个非抽象的类实现一个接口的话，必须将接口中的所有抽象方法全部实现



```java
interface MyMath{
  double PI = 3.1415926;
  int sum(int a, int b);
  int sub(int a, int b);
}
```

实现接口中的方法，接口中的常量自动继承，所以不用实现。

```java
class MyMathImpl implements MyMath {
  public int sum(int a, int b){
    return a + b;
  }
	public int sub(int a, int b){
    return a - b;
  }
}
```



### 接口和多态联合使用



```java
interface MyMath{
  double PI = 3.1415926;
  int sum(int a, int b);
  int sub(int a, int b);
}
class MyMathImpl implements MyMath {
  public int sum(int a, int b){
    return a + b;
  }
	public int sub(int a, int b){
    return a - b;
  }
}
```



父类型引用指向子类型对象,编译时调用的是`MyMath`的`sum`方法，运行是调用的是`MyMathImpl`的`sum`方法

```java
MyMath mm = new MyMathImpl();
mm.sum(10, 20);
```



### extends 和 implements 同时出现


```java
class Animal{
   String name;
}

interface Flyable{
   void fly();
}
```

`extends`在前`implements`在后

```java

class Pig extends Animal implements Flyable{
   public void fly(){
      System.out.printf("我是一只猪。\n虽然我是一只猪，但我是一只有梦想的猪！"+
      "\n我的梦想是能够像鸟儿一样飞起来！\n现在我实现了，我在天上飞！！！\n\t\t\t\t-- 小飞猪", this.name);
   }
}
```



```java
Flyable pig = new Pig();
if(pig instanceof Animal){
  ((Animal)pig).name = "小飞猪";
}
pig.fly();
```



### 接口在开发中的作用

**接口在开发中的作用，类似于多态在开发中的作用。**
**多态：面向抽象编程，降低程序的耦合度，提高程序的扩展力**

---

- 接口是完全抽象，而以后正要要求面向抽象编程。而面向抽象编程这句话可以改为：面向接口编程
- 有了接口就有了可插拔。可插拔表示扩展力强。
- 例如：中午去饭馆吃放
  - 接口是抽象的
  - 菜单是一个接口。
  - 谁面向接口调用。（顾客面向菜单点菜，调用接口）
  - 谁负责实现这个接口。（后台的厨师负责把菜做好，是接口的实现者）
  - 这个接口有什么用？（这个饭馆的**菜单**，让**顾客**和**后厨**解耦合了）



定义菜单接口

```java
 interface FoodMenu {
   void shiZhiChaoJiDan();
   void xiaoJiDunMoGu();
   void styleFood();
}
```

中餐师傅实现菜单上的菜品

```java
class ChinaChef implements FoodMenu {
  public void shiZhiChaoJiDan(){
    System.out.println("中餐柿子炒鸡蛋!");
  };
  public void xiaoJiDunMoGu(){
    System.out.println("中餐小鸡炖蘑菇!");
  };
  public void styleFood(){
    System.out.println("中餐自制卫龙辣条！");
  }
}
```

西餐师傅实现菜单上的菜品

```java
class AmericChef implements FoodMenu {
  public void shiZhiChaoJiDan(){
    System.out.println("西餐柿子炒鸡蛋!");
  };
  public void xiaoJiDunMoGu(){
    System.out.println("西餐小鸡炖蘑菇!");
  };
  public void styleFood(){
    System.out.println("西餐煎牛排!");
  }
}
```

无论是西餐厨师还是中餐厨师，只需要按照顾客根据菜单上点的菜来实现，当然，每个厨师每样菜都有不同的做法。

将菜单给到顾客，顾客不需要关系厨师怎么实现这个菜，他只负责点菜就好

```java
class Customer {
  private FoodMenu foodMenu;
  public Customer(FoodMenu foodMenu){
    this.foodMenu = foodMenu;
  }

  public void getStyleFood(){
    this.foodMenu.styleFood();
  }
  public void getShiZhiChaoJiDan(){
    this.foodMenu.shiZhiChaoJiDan();
  }
  public void getXiaoJiDunMoGu(){
    this.foodMenu.xiaoJiDunMoGu();
  }
}
```

顾客拿到菜单后开始点菜

```java
FoodMenu americChef = new AmericChef();
Customer customer = new Customer(americChef);
customer.getStyleFood();// 输出西餐煎牛排
```

---

如果将以上程序分割给不同的人员开发，那么他们只需要围绕着`FoodMenu`接口开发，最后模块之间使用接口衔接，降低耦合度



> 接口的使用离不开多态机制。