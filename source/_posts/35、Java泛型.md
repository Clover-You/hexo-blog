title: 35、Java泛型
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 17:07:25
---
### 泛型的使用

1. 泛型是**jdk5.0**之后推出的新特性
2. 泛型这种语法机制只在程序编译阶段起作用，这中语法只是用来骗编译器的。
3. 使用泛型好处
   1. 集合中存储的元素类型统一了
   2. 从集合中取出的元素类型是泛型指定的类型，不需要大量的向下转型。
4. 使用了泛型之后，导致集合中存储的元素缺乏多样性！
5. 如果调用子类型中特有的方法，还是需要向下转型！

不使用泛型，分析程序存在哪些缺点

```java
public class GenericityTest {
  public static void main(String[] args) {
    List animalList = new ArrayList();
    Cat catObj = new Cat();
    Bird birdObj = new Bird();
    animalList.add(catObj);
    animalList.add(birdObj);

    // 遍历集合，让动物走路
    Iterator it = animalList.iterator();
    while ( it.hasNext() ) {
      Object o = it.next();
      if ( o instanceof Cat ) {
        Cat cat = ( Cat )o;
        cat.move();
      } else if ( o instanceof Bird ) {
        Bird bird = ( Bird )o;
        bird.move();
      }
    }
  }
}
// 动物类
class Animal { }
// 猫科类
class Cat extends Animal {
  public void move() {
    System.out.println("猫抓老鼠!");
  }
}
// 鸟类
class Bird extends Animal {
  public void move() {
    System.out.println("鸟在飞！");
  }
}
```

迭代器返回的是`Object`类型，`Object`类型没有`move`方法，无法调用，需要向下转型。

使用泛型，`ArrayList`支持泛型

```java
public class ArrayList<E> {...}
```

使用泛型`List<Animal>`之后，表示`List`集合中只允许存储`Animal`类型的数据。用泛型来指定集合中存储的数据类型，集合中的数据更统一了。

```java
List<Animal> list = new ArrayList<Animal>();
```

```java
List<Animal> animalList = new ArrayList<Animal>();
Cat catObj = new Cat();
Bird birdObj = new Bird();
animalList.add(catObj);
animalList.add(birdObj);

// 遍历集合，让猫抓老鼠，让鸟飞
Iterator<Animal> it = animalList.iterator();
while ( it.hasNext() ) {
  Animal animal = it.next();
  animal.move();
}
```

`Iterator<Animal> it`指明迭代器返回的是一个`Animal`类型。

---

### 自动类型推断

JDK8之后的新特性：自动类型推断，又称钻石表达式

使用类型推断之前泛型需要这样写：

```java
List<Animal> list = new ArrayList<Animal>();
```

使用类型推断后：

```java
List<Animal> list = new ArrayList<>();
```

### 自定义泛型

自定义泛型可以吗？可以

自定义泛型的时候`<>`尖括号里面是一个标识符

```java
public class GenericityTest {
  public static void main(String[] args) {
    Genericity< String > s = new Genericity<>();
    System.out.println(s.see("看猴子呢！"));
  }
}

class Genericity< 你看什么 > {
  public 你看什么 see(你看什么 s) {
    return s;
  }
}

```

如果调用时不使用泛型，那么泛型默认是`Object`

Java源代码中经常出现的是:  <E>和<T>

- E是Element
- T是Type 