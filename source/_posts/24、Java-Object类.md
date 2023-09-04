title: 24、Java Object类
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 16:45:38
---
### 常用方法

负责对象克隆的。

`Protected Object clone();`

获取对象哈希值的一个方法

`int hashCode();`

判断两个对象是否相等

`Boolean equals(Object obj);`

将对象转换成字符串形式

`String toString();`

垃圾回收器负责调用的方法

`protrcted void finalize();`



### toString

关于`Object`中的`toString`方法

1. 源代码
   ```java
   public String toString() {
     return getClass().getName() + "@" + Integer.toHexString(hashCode());
   }
   ```
   
   源码上`toString`方法的默认实现是：
   类名@对象的内存地址转换为十六进制的形式
   
2. SUN公司设计`toString`方法的目的是什么？作用是什么？

   - `toString`方法的设计目的是：通过调用这个方法可以将一个**Java对象**转换成**字符串表示形式**

3. SUN公司开发Java语言的时候，建议所有的子类都去重写`toString`方法。`toString`方法应该是一个简洁的、详实的、易阅读的。

4. 输出一个引用时，会自动调用该引用的`toString`方法
   *打印引用时,例如有个引用a

   ```java
   System.out.println(a); 
   ```

   由于每个类都默认继承了`Object`对象，`println`底层使用了多态: PrintStream.class

   ```java
   public void println(Object x) {
     String s = String.valueOf(x);
     ...
   }
   ```

   可见`println`源码调用了`String.valueOf()`: String.class

   ```java
   public static String valueOf(Object obj) {
     return (obj == null) ? "null" : obj.toString();
   }
   ```

   ` obj.toString()`由此可见，只要obj是一个`Object`类型，那么就一定有`toString()`方法，而所有`class`，都默认继承了`Object`





### equals

 关于`Object`中的`equals`方法

1. `equals` 源码

   ```java
   public boolean equals(Object obj) {
     return (this == obj);
   }
   ```

   以上是`equals`方法的默认实现

2. SUN公司设计`equals`方法的目的是什么？

   - 以后编程当中，都要通过`equals`方法来判断两个对象是否相等。`equals`方法是判断两个对象是否相等的。

   - 判断两个java对象是否相等，不能使用“==”，因为这比较的是两个对象的内存地址。例如：

     ```java
     B b = new B(2, 3);
     B b2 = new B(2, 3);
     if (b == b2)// false
     ```

     在`Object`中的`equals`方法采用的是“\==”来判断两个对象是否相等。需要判断的是两个对象的内容是否相等，所以java默认的`equals`无法实现需求，所以需要重写`equals`。如何去重写这个`equals`需要按照你当前的需求来进行重写，例如比较`id`是否相等：

     ```java
     class B {
       public int id;
       public B(int id){
         this.id = id;
       }
       public boolean equals(Object obj){
         if (this instanceof obj && obj != null){
           return (this.id == ((B)obj).id);
         }
         return false;
       }
     }
     ```

     ```java
     B b = new B(2);
     B b2 = new B(2);
     b.equals(b2); // true
     ```

     



> Java中基本数据类型可以使用"**\==**"来进行判断
>
> Java中引用数据类型统一使用`equals`方法来进行判断
>
> String 类中已重写`equals`方法



### finalize

关于Object类中的finalize方法。

1. 在Object类中的源代码

   ```java
   protected void finalize() throws Throwable { }
   ```

2. `finalize`方法只有一个方法体，里面没有任何代码，而且这个方法是protected修饰的。

3. 这个方法不需要手动调用，JVM的垃圾回收器负责调用这个方法。

4. finalize方法执行时：
   当一个Java对象即将被垃圾回收器回收的时候，垃圾回收器负责调用finalize方法。

5. finalize方法实际上是一SUN公司为java程序员准备的一个时机，垃圾销毁机。如果希望在对象销毁时执行一段代码的话，这段代码要写到finalize方法。

6. Java中的垃圾回收器是不轻易启动的，垃圾太少或者时间没到，种种条件下，有可能启动，也有可能不启动。



> 可使用`System.gc`();方法建议垃圾回收器启动