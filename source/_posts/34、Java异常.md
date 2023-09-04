title: 34、Java异常
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 17:06:26
---
## 引入


1. 什么是异常

   - 以下程序执行过程中发生了不正常的情况，而这种不正常的情况叫做异常。

     ```java
     int result = 10 / 0;
     ```

     该程序执行时出现了：

     Exception in thread "main" java.lang.ArithmeticException: / by zero
             at com.upyou .learns.ExceptionTest01.main(ExceptionTest01.java:5)

     这个信息被称为异常，这个信息是`JVM`打印的。

2. 异常机制有什么用

   - Java语言是很完善的语言，提供了异常的处理方式，程序在执行过程中出现了不正常的情况，Java把该异常信息打印输出到控制台，供开发人员参考。开发人员看到异常信息后，可以对程序进行修改，让程序更加健壮。

3. Java语言中，异常是以什么形式存在的？

   -  异常在Java中以类的形式存在，每一个异常类都可以创建异常对象。
   - 异常对应现实生活中是怎样的？
     - 火灾（异常类）
       1. xxx年xx月xx日 小明家着火了（异常对象）
       2. xxx年xx月xx日 小刚家着火了（异常对象）

4. 程序在执行到某一行发生异常时，`JVM`会在这一行`new`一个相应的异常对象并抛出这个异常。

## 理论

1. 不管是错误还是异常，都是可抛出的。
2. 所有的错误只要发生，Java程序只有一个结果那就是终止程序的执行，退出`JVM`,错误是不能处理的。  
3. 所有`Exception`的直接子类，都叫做编译时异常。编译时异常是在编译阶段发生的吗？不是。编译时异常是表示必须在编写程序的时候预先对这种异常进行处理，如果不处理编译器报错。编译时异常又被称为受检异常、受控异常。
4. 所有`RuntimeException`及子类都属于运行时异常。运行时异常在编写程序阶段，你可以选择处理，也可以不处理。
5. 编译时异常和运行时异常，都是发生在运行阶段。编译阶段异常是不会发生的。
   - 编译时异常必须在编译（编写）阶段预先处理，如果不处理编译器报错
   - 所有异常都是在运行阶段发生的，因为所有程序只有运行阶段才可以`new`对象。因为异常的发生就是**new异常对象**
6. 编译时异常和运行时异常的区别
   - 编译时异常一般发生的概率比较高。
   - 运行时异常一般发生的概率比较低。
   - 对于一些发生概率较高的异常，需要在运行之前对其进行预处理。
   - 假设Java中没有对异常进行划分，没有分为：编译时异常和运行时异常，所有的异常都需要在编写程序阶段对其进行预处理，将是怎么样的效果呢？
     - 如果这样的话，程序肯定是绝对安全，但是编写程序太累，到处都是处理异常的代码。
     - 假设出门之前，把能够发生的异常都预先处理，那么会更加安全，但活得很累。



## 异常处理

1. Java中对异常的处理包括两种方式
   - 在方法声明的位置上使用`throws`关键字,抛给上一级。
   - 使用`try...catch`语句进行异常捕捉。
2. Java 中异常发生之后，如果一直上抛，最终抛给了`main`方法，`main`方法继续往上抛，抛给了调用者`JVM`，`JVM`知道这个异常发生，只有一个结果**「终止Java程序的执行」**

**例子：**

假设我是某公司的销售员，因为我的失误，导致公司损失了1000块钱，“损失1000块钱”这可以看作是一个异常发生了。我有两种处理方式

1. 我把这件事告诉我的领导「异常上抛」
2. 我自己掏腰包把钱补上「异常捕捉」



以下程序为什么没有输出`Hello World`？

程序执行到`10 / 0发生`了`ArithmeticException`异常，底层`new`了一个`ArithmeticException`异常对象后抛出，由于是`main`方法调用了`10 / 0`，所以这个异常`ArithmeticException`抛给了`main`方法，`main`方法没有处理，将这个异常自动抛给了`JVM`，`JVM`最终终止程序的执行



`ArithmeticException`继承于`RuntimeException`，属于运行时异常，在编写程序阶段不需要对这种异常进行预先处理。


```java
public static void main(String[] args) {
  System.out.println(10 / 0);
  System.out.println("Hello World");
}
```



`doSome`方法在声明的位置上使用了：`throws ClassNotFoundException`，这个代码表示`doSome`方法执行过程中，有可能会出现`ClassNotFoundException`异常。叫做类没找到异常，这个异常直接父类是：`Exception`，所以`ClassNotFoundException`属于编译时异常。

```java
public static void doSome() throws ClassNotFoundException {
  System.out.println("hiahiahiahiahiahia!");
}
```

以下代码报错原因：

由于`doSome`方法声明位置上有 `throws ClassNotFoundException`，在调用`doSome`方法的时候必须对这种异常预先处理，如果不处理，编译器就报错

`Unhandled exception: java.lang.ClassNotFoundException`、`Unhandled exception type ClassNotFoundException`

```java
public static void main(String[] args) {
  doSome();
}
```



### 第一种处理方式

在方法声明的位置上继续使用`throws`来完成异常的继续上抛。抛给调用者。

```java
public static void main(String[] args) throws ClassNotFoundException {
  doSome();
}
```

\-- `hiahiahiahiahiahia!`



### 第二种处理方式

使用`try...catch`进行捕捉

```java
public static void main(String[] args) {
  try {
    doSome();
    // 以上代码如果出现异常，这里是无法执行的.
  } catch(ClassNotFoundException e) {
    e.printStackTrace();
  }
}
```

\-- `hiahiahiahiahiahia!`



### 注意

只要异常没有捕捉，采用上报的方式，此方式的后续代码不会执行。
另外需要注意，`try`语句快中的某一行出现异常，该行后面的代码不会执行，直接进入`catch`语句快。



**在以后的开发中，处理编译时异常，应该上报还是捕捉？怎么选？**

- 如果希望调用者来处理，选择`throws`上报。
- 其它情况使用捕捉的方式。   



## 深入try...catch

1. `catch`后面的小括号中的类型可以是具体的异常类型，也可以是该异常类型的父类型
2. `carch`可以写多个，建议`catch`的时候，精确的一个一个处理，这样有利于程序调试。
3. `catch`写多个的时候，从上到下，必须遵循从小到大。

 

## 异常对象的常用方法

### getMessage

`getMessage();`方法获取异常发生后的简单描述信息，这个信息实际上是构造方法上`String`参数。

```java
try {
  System.out.println(10 / 0);
} catch(ArithmeticException e) {
  String errorMessage = e.getMessage();
  System.out.println(errorMessage);
}
```

\-- `/ by zero`

```java
ArithmeticException e = new ArithmeticException("哟！傻子又在找BUG呢？");
System.out.println(e.getMessage());
```

\-- `哟！傻子又在找BUG呢？`



### printStackTrace

`printStackTrace` 用于打印异常堆栈信息。java 后台打印异常堆栈追踪信息的时候，采用了异步线程的方式打印的。

```java
try {
  System.out.println(10 / 0);
} catch(ArithmeticException e) {
  e.printStackTrace();
}
```

\-- `java.lang.ArithmeticException: / by zero
        at com.upyou.learns.ExceptionTest05.main(ExceptionTest05.java:6)`

```java
ArithmeticException e = new ArithmeticException("哟！傻子又在找BUG呢？");
e.printStackTrace();
```

\-- `java.lang.ArithmeticException: 哟！傻子又在找BUG呢？
        at com.upyou.learns.ExceptionTest05.main(ExceptionTest05.java:6)`

## JDK8新特性

jdk8的特性，可以使用`｜`的方式捕捉多个异常

```java
public static void main(String[] args) {
  try {
    readFile("***");
    System.out.println(10 / 0);
  }catch(FileNotFoundException | ArithmeticException e){
    e.printStackTrace();
    System.out.println("哦豁！又是一个找不到异常的傻子!");
  }
}

public static void readFile(String filePath) throws FileNotFoundException {
  FileInputStream file = new FileInputStream(filePath);
}
```



## 查找异常信息

找一个并不存在的文件，肯定会报错，那么我们假装不知道这个文件存不存在!!!如何查看异常的追踪信息呢！

```java
public static void main(String[] args) {
  try {
    M1();
  } catch (FileNotFoundException e) {
    e.printStackTrace();
  }
}

private static void M1() throws FileNotFoundException {
  FileInputStream FIS = new FileInputStream("../哈哈哈哈懵了吧！你找不到我.dog");
}
```

以上程序报错信息如下：

java.io.FileNotFoundException: ../哈哈哈哈懵了吧！你找不到我.dog (No such file or directory)
        at java.io.FileInputStream.open0(Native Method)
        at java.io.FileInputStream.open(FileInputStream.java:195)
        at java.io.FileInputStream.<init>(FileInputStream.java:138)
        at java.io.FileInputStream.<init>(FileInputStream.java:93)
        at com.upyou.learns.ExceptionTest05.M1(ExceptionTest05.java:20)
        at com.upyou.learns.ExceptionTest05.main(ExceptionTest05.java:9)

1. 异常追踪信息应该从上往下看，首先看异常描述信息

2. 需要注意的是，只需要看自己写的代码异常信息就可以了，SUN写的代码就别看了，肯定是你自己写的BUG。那么如何得知是否是SUN写的？

   - 大多数报错信息前几行都是Java的代码，例如：

     ![image.png](http://qiniu-note-image.ctong.top/note/images/image-327d9779fb0f482eab06779e80a8e79a.png)

     一般看到什么java.xxx.xxx的直接略过，我们写的代码一般都写到自己的工作空间内，例如**com.upyou.learns**、**com.upyou.xxx**

   - 在我们写的代码的异常中，查看第一个异常位置，通常都是一个方法的异常引起的连锁反应。
     20行出现问题导致第9行出现问题

   - 第一个异常位置` at com.upyou.learns.ExceptionTest05.M1(ExceptionTest05.java:20)`，错误在`com.upyou.learns`包下的`ExceptionTest05`类里的`M1`方法，在第20行java代码。

   - 找到啦！原来是没有这个`.dog`文件


## finally语句

`try`和`finally`，没有`catch`可以吗？可以。

`try`不能单独使用，可以和`finally`连用。

`try...finally`是先执行`try`再执行`finally`,最后执行`return`语句,无论`return`在哪「`return`只要执行，方法必然结束」。`finally`语句块一定会执行

```java
try {
  System.out.println("try...");
  return;
} finally {
  System.out.println("finally...");
}
```

\-- `try...`

\-- `finally...`



退出`JVM`后，`finally`就无法执行了

```java
try {
  System.out.println("try...");
  System.exit(0);
} finally {
  System.out.println("finally...");
}
```

\-- `try...`

有这么一个程序：

```java
public static void main(String[] args) {
  System.out.println(m());
}
public static int m() {
  int i = 100;
  try {
    return i;
  } finally {
    i++;
  }
}
```

\-- `100`

Java语法规则（有一些规则是不能破坏的，一旦这么说就不能变）：

方法体中的代码必须遵循自上而下顺序依次逐行执行

以上程序`finally`确实是先于`return`执行。

**以上程序反编译之后的结果**
![image.png](http://qiniu-note-image.ctong.top/note/images/image-749f1ce714104ece9106834e69e48a2c.png)

从反编译结果来看，Java遵循了自己定的语法规则**「return必须最后执行、自上而下顺序执行」**


## final、finally、finalize

`final`是一个关键字，表示最终的，不变的。**第一次赋值之后不可第二次赋值。**

```java
final int i = 100;
// i = 200; // 错误
```

`finally`也是一个关键字，和try联合使用，使用在异常处理机制中，在`finally`语句快中的代码是一定会执行的。 

```java
try{
  ...
} finally {
  ...
}
```

`finalize`是`Object`类中的一个方法，作为方法名出现。所以`finalize`是标识符。这个方法是由垃圾回收器负责调用。



## 自定义异常

Java中如何自定义异常？

1. 编写一个类继承`Exception`或者`RuntimeException`。
2. 提供两个构造方法，一个无参数一个带有`String`参数的构造方法。



### 自定义编译时异常

定义一个编译时异常

```java
public class NotFoundMeException
  extends Exception {
  public NotFoundMeException(){super();}

  public NotFoundMeException(String errorMessage){
    super(errorMessage);
  }
}
```

如何使用这个异常对象捏？

`new`一个异常对象并且使用`throw`抛出，`throws`上抛给调用者

```java
public static void test(String str) throws NotFoundMeException{
  if (str.isEmpty()) {
    throw new NotFoundMeException("hiahiahia！我是异常，你找不到我!!");
  }
}
```

处理这个异常

```java
public static void main(String[] args) {
  try {
    test("");
  } catch(NotFoundMeException e) {
    e.printStackTrace();
  }
}
```

com.upyou.learns.MyException.NotFoundMeException: hiahiahia！我是异常，你找不到我!!
        at com.upyou.learns.ExceptionTest07.test(ExceptionTest07.java:18)
        at com.upyou.learns.ExceptionTest07.main(ExceptionTest07.java:10)



### 自定义运行时异常

定义一个运行时异常类,继承于`RuntimeException`

```java
public class NotFoundMeException02 
  extends RuntimeException{
  public NotFoundMeException02(){super();}
  public NotFoundMeException02(String errorMessage){super(errorMessage);}
}
```

使用这个异常类

```java
public static void runTest(String str) 
  throws NotFoundMeException02{
  if (str.isEmpty()) {
    throw new NotFoundMeException02("哟！大家快看这个傻子又在写BUG！");
  }
}
```

```java
runTest("");
```

Exception in thread "main" com.upyou.learns.MyException.NotFoundMeException02: 哟！大家快看这个傻子又在写BUG！
        at com.upyou.learns.ExceptionTest07.runTest(ExceptionTest07.java:23)
        at com.upyou.learns.ExceptionTest07.main(ExceptionTest07.java:10)

[异常小练习点这里下载哦](https://github.com/Clover-You/my-blog-file/blob/main/%E5%BC%82%E5%B8%B8%E5%B0%8F%E7%BB%83%E4%B9%A0.zip)


> 不建议在`main`方法上使用`throws`，因为这个异常如果真正发生了，一定会抛给`JVM`，`JVM`只有终止。
>
> 重写之后的方法不能比重写之前的方法抛出更多（更宽泛）的异常，可以更少。