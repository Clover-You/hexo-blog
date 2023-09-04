title: 16、Java 方法覆盖
categories: Java 基础
tags: [学习笔记,JAVA]
date: 2022-01-02 14:45:27
---
1. 方法覆盖又被称为方法重写`override`
2. 什么时候使用方法重写？
   - 当父类中的方法无法满足子类的业务需求，子有必要见父类中继承过来的方法进行重新编写，这个重新编写的过程称为方法重写、覆盖。
3. 什么条件满足之后方法会发生重写？
   - 方法重写发生在具有关系的父子类之间。
   - 方法名相同、返回值类型相同、参数列表相同。
   - 访问权限不能更低，可以更高。
   - 抛出异常不能更多，可以更少。
4. 注意：
   - 私有方法不能继承，所以不能覆盖。
   - 构造方法不能继承，所以不能覆盖。
   - 静态方法不存在覆盖。
   - 覆盖只针对方法，不针对属性。



有一个`Animal`类，里面有一个`move`方法

```java
public class Animal {
  public void move(){
    System.out.println("动物在飞行!");
  }
}
```

`Cat`类继承了`Animal`，那么`Cat`可以使用`move`方法

```java
public class Cat extends Animal {}
```

输出`动物在飞行!`


```java
Cat cat = new Cat();
cat.move();
```

而猫会飞吗？它不会，`Animal`中的`move`方法已经无法满足`Cat`，这时需要重写`Animal`中的 ``move`方法

根据上面第3条第2项规则：

输出`猫在地上走!`

```java
public class Cat extends Animal {
    public void move(){
        System.out.println("猫在地上走!");
    }
}
```

这时有一个`Bird`类也继承了`Animal`

```java
public class Bird extends Animal {}
```

输出:

`猫在地上走!`
`动物在飞行!`

```java
Cat cat = new Cat();
cat.move();
Bird bird = new Bird();
bird.move();
```