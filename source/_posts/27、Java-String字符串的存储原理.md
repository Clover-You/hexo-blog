title: 27、Java String字符串的存储原理
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 16:50:10
---
  

1. String 表示字符串类型，属于引用数据类型，不属于基本数据类型。

2. 在Java中随便使用双引号括起来的都是String对象。例如：`"abc"`,`"Hello Wold!"`,这是两个String对象。

3. Java中规定，双引号括起来的字符串是不可变的，也就是说`"abc"`出生到最终死亡都是`"abc"`,不能变成例如`"abcd"`

4. 在JDK中双引号括起来的字符串，例如`"abc"`都是直接存储在方法区的字符串常量池当中的。

   ```java
   public class StringTest {
     public static void main(String[] args) {
       String str = "abc";
     }
   }
   ```

   以上代码内存图方式表示

   ![image.png](http://qiniu-note-image.ctong.top/note/images/image-4243eeb3a17f4ba4b8e901a42f43f3e0.png)
   如果有字符串拼接，那么在内存图中该怎么表示？例如：

   ```java
   String str2 = "abc" + "def";
   ```

   `"abc"`在字符串常量池中存在，那么拿到该字符串在常量池中的内存地址，加上`"def"`,而`"def"`在常量池中并不存在，所以需要在常量池中开辟一块空间存储`"def"`,最后拼接成一个新的字符串`"abcdef"`,如果常量池中没有保存到这个字符串，那么就会重新开辟一块空间存储，否则使用已创建的这个字符串的内存地址。而此时，如果没有另外一个引用指向`"def"`,那么`"def"`此时是一个空引用

   ![image.png](http://qiniu-note-image.ctong.top/note/images/image-f12dcfe13cfb41b58ce73a04e112cf16.png)
   
   ```java
   String str3 = new String("def");
   ```
   
     以上程序内存图应该这样表示
   
   ![image.png](http://qiniu-note-image.ctong.top/note/images/image-9cfb3a3b8a214bc6a5c823c82749ae3e.png)
   
5. 垃圾回收器不会回收常量池里的数据。

6. 由于双等号"=="比较的是内存地址，所以两个字符串可以使用双等号进行比较，例如

   ```java
   "Hello" == "Hello"// true
   ```

   那么，`new String("Hello"); `可以和`new String("Hello");`使用双等号比较吗？答案是不可以，因为双等号比较的是对象的内存地址,而使用`new`创建的对象必然是一个新的对象。

   ![image.png](http://qiniu-note-image.ctong.top/note/images/image-b5b17a5a1b0d4e418cd5aee9d402afb7.png)



### String构造方法

1. `String str = new String("");`
2. `String str = "";`
3. `String str = new String(byte[]);`
4. `String str = new Strinh(byte[], offset, count);`
5. `String str = new String(char[]);`
6. `String str = new String(char[], offset, count);`