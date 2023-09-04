title: 29、Java包装类
categories: Java 基础
tags: [学习笔记,JAVA]
date: 2022-01-02 16:53:23
---
1. Java中为8种基本数据类型又对应准备了**8种包装类**。**8种包装类**属于引用数据类型，父类是`Object`。

2. 如果有这么一种需求：调用`doSome`方法的时候要传一个数字，但是数字属于基本数据类型，而`doSome`方法参数的类型是`Object`。可见`doSome`方法无法接受基本数据类型的数字。

   ```java
   public static void doSome(Object obj){
     System.out.println(obj);
   }
   ```

   虽然`doSome`方法不能直接传`int`类型，但是可以传一个对应的包装类型

   ```java
   public static void main(String[] args){
     Integer myInt = new Integer(100);
     doSome(myInt);
   }
   ```

3. 8种基本数据类型对应的包转类：

   | 基本数据类型 |      包装类型       |
   | :----------: | :-----------------: |
   |     byte     |   java.lang.Byte    |
   |    short     |   java.lang.Short   |
   |     int      |  java.lang.Integer  |
   |     long     |   Java.lang.Long    |
   |    float     |   Java.lang.Float   |
   |    double    |  java.lang.Double   |
   |   boolean    |  Java.lang.Boolean  |
   |     char     | Java.lang.Character |

4. 拆装箱

   - 装箱
     将基本数据类型 --(转换)--> 引用数据类型（装箱）

     ```java
     Integer i = new Integer(100);
     ```

   - 拆箱
     将引用数据类型 --(转换)--> 基本数据类型（拆箱）

     ```java
     i.intValue();// 100
     ```

5. `Integer`的构造方法

   `Integer`有两个构造方法

   ```java
   Integer(int value)
   Integer(String s)
   ```

   接受一个`int`或者`String`类型的数据

   ```java
   new Integer(100);// 100
   new Integer("100");// 100
   ```

6. 通过访问包装类的常量，来获取最大值和最小值

   ```java
   Integer.MAX_VALUE;
   Integer.MIN_VALUE;
   ```

7. 自动装拆箱

   - 自动装箱

     ```java
     Integer x = 100;
     ```

   - 自动拆箱

     ```java
     int y = x;
     ```

     虽然支持自动拆箱，当是底层实际上还是
     
     ```java
     Integer x = new Integer(100);
     ```
     
     由于底层使用的是new的方式创建，所以无法使用双等号"=="进行判断，双等号无法自动拆箱
     
     ```java
     Integer a = 1000;
     Integer b = 1000;
     System.out.println(a == b);// false
     ```
     ![image.png](http://qiniu-note-image.ctong.top/note/images/image-e6d15767a5624c79bc5cca5a4b36245c.png)
   
8. 整数型常量池

   java中为了提高程序的执行效率，将**「-128 ～ 127」**之间所有的包装对象提前创建好，放到了一个方法区的**“整数型常量池”**当中，目的是只要用这个区间的数据不需要再`new`了，直接从**整数型常量池**中取出来。

   ```java
   Integer a = 128;
   Integer b = 128;
   System.out.println(a == b);// false
   Integer x = 127;
   Integer y = 127;
   System.out.println(x == y);// true
   ```

   以上程序`127`并不是拆箱了，而是`x,y`这两个变量中保存的内存地址是同一个。双等号`==`比较的是**对象的内存地址**。

   ![image.png](http://qiniu-note-image.ctong.top/note/images/image-1b2abab7b1d144fc9fb882bc2912d3f6.png)

   源代码中，在**类加载**的时候创建了对应的`Integer`对象。

   

   ![image.png](http://qiniu-note-image.ctong.top/note/images/image-417b9f7ad8b44c2298afcad2df86ad3a.png)

9. NumberFormatException 异常

   这个异常是出现在将`String`类型转`int`类型的时候

   ```java
   Integer.parseInt("中文");
   ```

   



> 在**JDK 1.5**之后，支持自动拆箱和自动装箱。
>
> 其它包装类和`Integer`使用方法一样。
>
> 在进行类加载的时候，整数型常量池已经初始化好了。