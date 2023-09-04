title: 37、Java IO流
categories: Java 基础
tags: [学习笔记,JAVA]
date: 2022-01-02 17:14:11
---
# Java IO 流

把文件里存的东西放到内存里的过程叫：**输入（Input）**，数据的流动我门叫：**输入流**（InputStream），输入的过程又被称为：**读（Read）**。

从内存里出来到到硬盘叫：**输出（Output）**，数据流动的过程叫：**输出流（OutputStream）**，又被叫：**写（Write）**

![image.png](http://qiniu-note-image.ctong.top/note/images/image-048b5f52f35e48d49d3b97ad7b3e6f17.png)

## IO是什么？

通过IO可以完成文件的读写。IO就是Input和Output首字母。

I：Input

O：Output



## IO流的分类

IO流有多种分类方式：

1. 一种方式是按照流的方向进行分类：
   - 以内存作为参造物，往内存中去，叫做输入或者叫做读。（Input）
   - 从内存中出来叫做输出或者叫做写。（Output、Write）
2. 另一种方式是按照读取数据方式不同进行分类：
   - 有的流是按照字节的方式读取数据，一次读取1个字节byte，等同于一次读取8个二进制位。这种流是万能的，什么类型的文件都能读取。（字节流）
   - 有的流是按照字符的方式读取数据的，一次读取一个字符，这种流是为了方便读取普通文本而存在的，这种流不能读取：图片、声音、视频等文件。只能读取纯文本文件，连word文件都无法读取。



假设文件file.txt，其内容是：

```tex
a中国bc
```

#### **如果采用字节流的方式去读**：

第一次读：一个字节，正好读到 ‘a’

第二次读：一个字节，正好读到‘中’字符的一半（原因是在Windows系统中，中文占用两个字节，字节流一次只能读取一个字节）

第三次读：一个字节，正好读到‘中’的另一半。



#### **如果采用字符流来对该文件读取**：

第一次读：‘a’字符

第二次读：‘中’字符





> 输入流、输出流、字节流、字符流
>
> *‘a’英文字母，在Windows操作系统中是1个字节，但是‘a’字符在java中占用2个字节，flex.txt和java没有关系，因为这个文件只是Windows操作系统上的普通文件。Windows操作系统中文件中的‘a’字符占用的是1个字节（读文件是调用Windows操作系统，操作系统读取文件的时候，认为一个字符是一个字节，所以和JVM没有关系）*





## ‘流’应该怎么学习

Java中的IO流都已经写好了，我们学习时不需要去关心底层是如何去实现的，我们最主要的还是掌握，在java中已经提供了哪些流，每个流的特点是什么，每个流对象上的常用方法有哪些？？

Java中所有的流都是在：`java.io.*;`下。

Java中主要学习怎么new流对象，调用流对象的哪个方法是读，哪个方法是写。

## Java IO流中的四大家族

1. java.io.InputStream 字节输入流

2. java.io.OutputStream 字节输出流

3. java.io.Reader 字符输入流

4. java.io.Writer 字节输出流

   

所有的流都实现了：
   java.io.Closeable接口，都是可关闭的，都有close方法。流毕竟是一个管道，这个是内存和硬盘之间的通道，用完之后一定要关闭，不然会耗费很多资源。

> 四大家族的首领都是抽象类 abstract
>
> 在java中只要‘类名’以Stream结尾的都是字节流。以‘Reader/Writer’结尾的都是字符流。



所有的输出流都实现了：

	java.io.Flushable接口，都是可刷新的，都有flush方法，输出流在最终输出之后，一定要记得flush刷新一下。这个刷新表示将通道/管道当中剩余未输出的数据强行输出完（清空管道）。

> 如果没有flush可能会导致丢失数据。并不是所有的流都有flush方法，只有输出流实现了java.io.Flushable接口



## 需要掌握哪些流

java.io包下需要掌握的流有16个：

文件专属

1. Java.io.FileInputStream.
2. Java.io.FileOutputStream
3. java.io.FileReader
4. java.io.FileWriter

转换流，用于将字节流转换为字符流

5. java.io.InputStreamReader
6. java.io.OutputStreamWriter

缓冲流专属

7. java.io.BufferedReader
8. java.io.BufferedWriter
9. java.io.BufferedInputStream
10. java.io.BufferedOutputStream

数据流专属

11. java.io.DataInputStream
12. java.io.DataOutputStream

标准输出流

13. java.io.PrintWriter
14. java.io.PrintStream

对象专属流

15. java.io.ObjectInputStream
16. java.io.ObjectOutputStream



> 看着很多，但实际上把文件专属流前两个搞懂，其它的都差不多。

## FileInputStream



1. 文件字节输入流，万能的，任何类型的文件都可以采用这个流来读。
2. 字节的方式，完成输入的操作，完成读的操作（硬盘 --> 内存）



创建一个文件字节输入流对象,`path`为你的文件物理路径

```java
FileInputStream fis = new FileInputStream("/Users/yct/Desktop/citys.js");
```

创建流后，当流不等于空时，必须要关闭流。

```java
fis.close();
```

```java
FileInputStream fis = null;
try {
  fis = new FileInputStream(PATH);
  ....
}catch(IOException e) {
  System.out.println(e.getMessage());
}finally {
  // 关闭资源
  if (fis != null) {
    fis.close();
  }
}
```



### 使用try-with-resources

Java7的编辑器和运行环境支持新的**try-with-resources**，称为ARM块（Automatic Resource Management ）自动资源管理。可自动关闭任何可关闭的资源，这些可关闭的资源必须实现java.lang.AutoCloseable接口。

```java
try(FileInputStream fis = new FileInputStream(PATH)) {
	....
}catch (IOException e) {
	System.out.println(e.getMessage());
}
```

### 读取文件

使用`read()`方法读取文件，读取原理与集合`next()`方法一致，[next读取原理传送门](http://39.97.110.136/archives/java%E9%9B%86%E5%90%88#%E8%BF%AD%E4%BB%A3%E5%99%A8%E6%89%A7%E8%A1%8C%E5%8E%9F%E7%90%86%E5%86%85%E5%AD%98%E5%9B%BE)

`read()`返回值是读取到的**字节**本身，例如读取一个文本，文本中内容为：abcdef

当第一次调用`read()`方法时，读取a，而a的字节码为97，将读取到的字节返回 --> 97

再一次调用方法时，`read()`再向后移动一位，读取b，将读取到的字节返回 --> 98

以此类推，当读到文件末尾时，如果再次读取时，读不到任何数据`read()`会返回-1

```java
try(FileInputStream fis = new FileInputStream(PATH)) {
	System.out.println(fis.read());
}catch (IOException e) {
	System.out.println(e.getMessage());
}
```



#### 使用循环读取

使用`while`死循环读取文件，**读到-1**退出`while`循环

```java
try(
  FileInputStream fis = new FileInputStream("/Users/yct/Desktop/TEST.txt")
){
   int byteData = 0;
  while ((byteData = fis.read()) != -1) {
    System.out.println(byteData);
  }
}catch(IOException e) {
  System.out.println(e.getMessage());
}
}
```

> 一次读取一个字节byte，这样内存和硬盘交互太频繁，基本上时间/资源都消耗在交互上面流。



#### 使用int read(byte[] b)

`read()`一次只能读取一个字节，`int read(byte[] b)`可以一次最多读取`b.length`个字节，减少硬盘和内存的交互，提高程序的执行效率。

准备一个长度为4的byte数组`byte[4]`，我们读取的文件同上。`read(byte[] b)`这个方法返回值是：读取到的字节数量。（不是字节本身）

```java
try(
  FileInputStream fis = new FileInputStream("/Users/yct/Desktop/TEST.txt")
){
  byte[] fileByte = new byte[4];
  int readCount1 = fis.read(fileByte);
  System.out.println(readCount1);
}catch(IOException e) {
  System.out.println(e.getMessage());
}
```

第一次读取时读到了4个字节，当前byte数组中的数据为`[97,98,99,100]`。

当我们第二次再尝试读取到这个数组中时,读取到了2个字节，当前byte数组中的数据为`[101,102,99,100]`

```java
try(
  FileInputStream fis = new FileInputStream("/Users/yct/Desktop/TEST.txt")
){
  byte[] fileByte = new byte[4];
  int readCount1 = fis.read(fileByte);
  System.out.println(readCount1);
  int readCount2 = fis.read(fileByte);
  System.out.println(readCount2);
}catch(IOException e) {
  System.out.println(e.getMessage());
}
```

原因是` read(byte[] b)`方法在往byte数组中放数据时，是从byte数组的第0位开始放数据，文本有6个字节，而byte数组长度只有4，当读到第4个字节时停止读取（byte数组已满）。而第二次再去读取时，` read(byte[] b)`上次读到了第4个字节还剩下2个字节，将剩余的字节读到byte数组时是从0开始，所以将剩余的两个字节读到了byte数组的0、1位置中此时覆盖掉了当前索引中旧的数据。

如果第三次再尝试去读取的话将返回**-1**，因为`read(byte[] b)`已经将数据读完了，一个字节都没有读到。

---

将读取到的数据转成字符串，使用String类中的构造方法可将byte数组转成字符串，因为`read(byte[] b)`方法在往byte数组中放数据时，是从byte数组的第0位开始放数据，所以开始位置是0，而结束位置则是读取到的字节数，这样就保证了程序读到什么就输出什么。

```java
try(
  FileInputStream fis = new FileInputStream("/Users/yct/Desktop/TEST.txt")
){
  byte[] fileByte = new byte[4];
  int readCount1 = fis.read(fileByte);
  System.out.println(new String(fileByte, 0, readCount1));
  int readCount2 = fis.read(fileByte);
  System.out.println(new String(fileByte, 0, readCount2));
}catch(IOException e) {
  System.out.println(e.getMessage());
}
```

输出

-- abcd
-- ef

#### 改造

将上面的程序全部改造一下

```java
try (FileInputStream fis = new FileInputStream("/Users/yct/Desktop/TEST.txt")) {
  byte[] bytes = new byte[4];
  StringBuilder sb = new StringBuilder(8);
  int count = 0;
  while ((count = fis.read(bytes)) != -1) {
    sb.append(new String(bytes, 0, count));
  }
  System.out.println(sb);
} catch (IOException e) {
  System.out.println(e.getMessage());
}
```

输出

-- Hello World!



### FileInputStream其它常用方法

1. `int available()` 返回流当中剩余的字节数量
2. `long skip()` 跳过几个字节不读





#### int available()

读取一个字节，查看还有多少个字节未读取，文件内容为：abcdef

```java
try (FileInputStream fis = new FileInputStream("/Users/yct/Desktop/TEST.txt")) {
  byte[] bytes = new byte[4];
  StringBuilder sb = new StringBuilder(8);

  System.out.println(fis.read());
  System.out.println(fis.available());
} catch (IOException e) {
  System.out.println(e.getMessage());
}
```

输出

-- 5



available正确用法：new 一个byte数组，让read()方法读取一次添加到byte数组中，不需要循环。

```java
try (FileInputStream fis = new FileInputStream("/Users/yct/Desktop/TEST.txt")) {
  byte[] bytes = new byte[fis.available()];
  fis.read(bytes);
  System.out.println(new String(bytes));
} catch (IOException e) {
  System.out.println(e.getMessage());
}
```

这种方式不太适合读取大文件，因为byte[]数组不能太大



#### long skip()

使用`skip()`可以跳过几个字节不读取，文件内容为：abcdef

```java
try (FileInputStream fis = new FileInputStream("/Users/yct/Desktop/TEST.txt")) {
  byte[] bytes = new byte[fis.available()];
  fis.skip(3);
  fis.read(bytes);
  System.out.println(new String(bytes));
} catch (IOException e) {
  System.out.println(e.getMessage());
}
```

输出

-- def



## FileOutputStream

1. 文件字节输出流，负责写
2. 从内存到硬盘



创建一个`FileOutputStream`对象，传入一个PATH，这个PATH是文件输出的位置加文件名。使用这种方式会将原文件清空，然后重新写入。

```java
FileOutputStream fos = new FileOutputStream(PATH);
```

使用write读取一个byte数组。

```java
try(FileOutputStream fos = new FileOutputStream("/Users/yct/Desktop/test.txt");) {
  byte[] bytes = {97, 98, 99};
  fos.write(bytes);
} catch (IOException e){
  System.out.println(e.getMessage());
}
```

![image.png](http://qiniu-note-image.ctong.top/note/images/image-83b48eaa137a4ae99559fe58b6aaa220.png)

如果需要在原文件末尾追加内容时，不可使用上面的方式FileOutputStream提供了一个构造函数`FileOutputStream(String name, boolean append)`指在原文件末尾追加元素（以追加的方式在文件末尾写入，不会清空原文件内容）。例如原文件有一个**Hello**

```java
FileOutputStream fos = new FileOutputStream(PATH, true);
```



```java
try(FileOutputStream fos = new FileOutputStream("/Users/yct/Desktop/test.txt", true);) {
  byte[] bytes = {97, 98, 99};
  fos.write(bytes);
} catch (IOException e){
  System.out.println(e.getMessage());
}
```


![image.png](http://qiniu-note-image.ctong.top/note/images/image-1de69656529d45c88862a1c0ebc51853.png)

只输出2位字节,可以使用`write(byte[] b, int off, int len)`

```java
fos.write(bytes, 0, 2);
```

文件内容

-- ab

> 写完之后一定要刷新

## 文件复制

1. 需要使用FileInputStream加FileOutputStream完成文件的拷贝。
2. 拷贝的过程应该是一边读一边写。
3. 使用以上的字节流拷贝文件的时候，文件类型随意，万能的。什么样的文件都能拷贝。



![image.png](http://qiniu-note-image.ctong.top/note/images/image-457aff54a1bc487cb05be65a641fc8a4.png)





编写的源码

```java
public static void main(String[] args) {
  if (copy("/Volumes/UpYou 1/SystemOS/Win10_1909(全家桶)x64.iso", "/Users/yct/Win10_1909(全家桶)x64.iso")) {
    System.out.println("文件复制成功！");
  } else {
    System.out.println("文件复制失败！");
  }

}

/* 复制文件 */
public static boolean copy(String sourcePath, String targetPath) {
  try (FileInputStream fis = new FileInputStream(sourcePath);
       FileOutputStream fos = new FileOutputStream(targetPath);) {
    int readLong = 1024 * 1024;// 一次读取的长度
    long fisLength = fis.available();// 流总长度*2=位，/8转成byte
    long int2percentage = fisLength / 100; // 将长度转换成百分比
    byte[] bytes = new byte[readLong];
    int nowReadCount = 0;
    while ((nowReadCount = fis.read(bytes)) > -1) {
      fos.write(bytes, 0, nowReadCount);
      long r = fisLength - fis.available();// 已读取量

      System.out.println("====>>>>" + r / int2percentage + "%");
    }
    fos.flush();// 刷新管道
    return true;
  } catch (IOException e) {
    System.out.println(e.getMessage());
  }
  return false;
}
```

创建每次读取的长度，这是必须的，不可采用`fis.available();`的方式来设置数组的长度，因为如果遇到大文件，计算机找一块符合大小、连续地址的内存空间很难，可能都找不到，所以需要限定每次最大只能读取多大！（切片读取）

```java
int readLong = 1024 * 1024;
byte[] bytes = new byte[readLong];
```

最后将读取到的字节输入到byte数组中,采用读取多少字节输入多少的方式`fos.write(bytes, 0, nowReadCount);`因为要确保最后一次读取的准确性。

```java
long nowReadCount = 0;
while ((nowReadCount = fis.read(bytes)) > -1) {
  fos.write(bytes, 0, nowReadCount);
}
```

最后一步一定要刷新管道

```java
fos.flush();
```

读取一个5G的文件

![image.png](http://qiniu-note-image.ctong.top/note/images/image-572ba1501548408d8b19568ab0afb2d4.png)

原文件

![image.png](http://qiniu-note-image.ctong.top/note/images/image-3374c2a324684a50b2d82f0fc16ac7ef.png)

java程序拷贝的后的文件


![image.png](http://qiniu-note-image.ctong.top/note/images/image-5a557dc5736d42678ed1b59ca37da794.png)



## FileReader

1. 文件字符输入流，只能读取普通文本。读取文件时，比较方便快捷。



创建一个`FileReader`

```java
FileReader fr = new FileReader(PATH);
```

将文件读到char数组中

```java
char[] txt = new char[1024 * 1024];
int readerCount = 0;
while ((readerCount = fr.read(txt)) > -1) {
  System.out.println(new String(txt, 0, readerCount));
}
```

![image.png](http://qiniu-note-image.ctong.top/note/images/image-5c71355d8b8a48079c4da3e36b0a90fc.png)



## FileWriter

1. 文件字符输出流，只能输出普通文本。

用法与其它输出流一样，但记得一定要`flush()`

```java
try (FileReader fr = new FileReader("/Users/yct/Desktop/text.txt");
     FileWriter fw = new FileWriter("/Users/yct/Desktop/text2.txt");) {
  char[] txt = new char[1024 * 1024];
  int readerCount = 0;
  while ((readerCount = fr.read(txt)) > -1) {
    fw.write(txt, 0, readerCount);
  }
  fw.flush();
} catch (IOException e) {
  System.out.println(e.getMessage());
}
```



## BufferedReader

1. 带有缓冲期的字符输入流，使用这个流的时候不需要自定义char数组、byte数组，自带缓冲。

2. 当一个流的构造方法中需要一个流的时候，这个被传进来的流叫做：节点流。外部负责包装的这个流，叫做：包装流，还有一个名字叫做：处理流。

3. 对包装流来说，只需要关闭最外层就行，里面的节点流会自动关闭。

   源码:

   ```java
   public void close() throws IOException {
     synchronized (lock) {
       if (in == null)
         return;
       try {
         in.close();
       } finally {
         in = null;
         cb = null;
       }
     }
   }
   ```

   

### 构造函数

`BufferedReader(Reader in)` 创建一个使用默认大小输入缓冲区的缓冲字符输入流。

`BufferedReader(Reader in, int sz)`创建一个使用指定大小输入缓冲区的缓冲字符输入流。

```java
FileReader fr = new FileReader("/Users/yct/Desktop/text.txt");
BufferedReader br = new BufferedReader(fr);
br.close();
```



### readLine

`readLine()`可读取一行文本，不包括末尾的换行符，文本读完时返回`null`

```java
try (FileReader fr = new FileReader("/Users/yct/Desktop/text.txt"); BufferedReader br = new BufferedReader(fr);) {
  String s = null;
  while((s = br.readLine()) != null) {
    System.out.println(s);
  }
} catch(IOException e) {

}
```



## BufferedWriter

1. 带有缓冲区的字符输出流。

```java
BufferedWriter out = new BufferedWriter(new FileWriter(PATH));
```

输出一个内容为abc的普通文本

```java
final static String WRITER_PATH = "/Users/yct/Desktop/copy.txt"; // 文件输出的位置

public static void main(String[] args) {
  try (BufferedWriter bw = new BufferedWriter(new FileWriter(WRITER_PATH))) {
    byte[] bytes = {97, 98, 99};
    bw.write(new String(bytes));
    bw.flush();
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```



## DataOutputStream

java.io.DataOutputStream 数据专属流，这个流可以将数据连同数据的类型一并写入文件

> 这个文件不是普通文本文档

创建数据专属的字节输出流

```java
try (FileOutputStream fileOutputStream = new FileOutputStream(FILE_PATH);
     DataOutputStream dos = new DataOutputStream(fileOutputStream);) {
  dos.flush();
} catch (IOException e) {
  e.printStackTrace();
}
```

创建不同数据类型，一并写入文件

```java
byte b = 100;
short s = 200;
int i = 300;
long l = 400L;
float f = 3.0f;
double d = 3.14;
boolean sex = false;
char c = 'a';
```



写入数据

```java
try (FileOutputStream fileOutputStream = new FileOutputStream(FILE_PATH);
     DataOutputStream dos = new DataOutputStream(fileOutputStream);) {
  dos.writeByte(b);
  dos.writeShort(s);
  dos.writeInt(i);
  dos.writeLong(l);
  dos.writeFloat(f);
  dos.writeDouble(d);
  dos.writeBoolean(sex);
  dos.writeChar(c);
  dos.flush();
} catch (IOException e) {
  e.printStackTrace();
}
```



## DataInputStream

1. 数据字节输入流。
2. DataOytputStream写的文件，只能使用DataInputStream来读取。并且读的时候要知道写入的顺序。读的顺序需要和写的顺序一致，才可以正常取出数据。



读取上个栗子写入的数据

```java
try (FileInputStream fis = new FileInputStream(FILE_PATH);
     DataInputStream dis = new DataInputStream(fis);) {
  byte b = dis.readByte();
  short s = dis.readShort();
  int i = dis.readInt();
  long l = dis.readLong();
  float f = dis.readFloat();
  double d = dis.readDouble();
  boolean sex = dis.readBoolean();
  char c = dis.readChar();
  System.out.println(b);
  System.out.println(s);
  System.out.println(i);
  System.out.println(l);
  System.out.println(d);
  System.out.println(sex);
  System.out.println(c);
} catch (IOException e) {
  e.getStackTrace();
}
```



-- `100`
-- `200`
-- `300`
-- `400`
-- `3.0`
-- `3.14`
-- `false`
-- `a`



## PrintlnStream

1. 标准的字节输出流，默认输出到控制台

> 标准输出流不需要手动关闭

我们常用的`System.out.println`其实调用的就是`PrintStream`：

```java
public final static PrintStream out = null;
```



```java
public static void main(String[] args) {
  PrintStream ps = System.out;
  ps.print("Hello");
  ps.println(" World!");
}
```

-- `Hello World!`



### 改变标准输出流的输出方向

标准输出流不再指向控制台，指向“log”文件。

```java
PrintStream ps = new PrintStream(new FileOutputStream(FILE_PATH));
```

修改输出方向，将输出方向修改到“log ”文件。

```java
System.setOut(ps);
```

再输出

```java
public static void main(String[] args) throws FileNotFoundException {
  PrintStream ps = new PrintStream(new FileOutputStream(FILE_PATH));
  System.setOut(ps);
  System.out.print("Hello");
  System.out.println(" World!");
}
```


![image.png](http://qiniu-note-image.ctong.top/note/images/image-dbe2703edadf4fcbb0a6b8d8a3bf4b97.png)



## File类

1. File类和四大家族没有关系，所以File类不能完成文件的读和写。
2. File对象代表什么？
   File是文件和目录路径的抽象表示形式
   /Users/yct/Pictures 这是一个File对象
   /Users/yct/Pictures/IMG_0168.jpeg 这也是一个File对象。
   一个File对象对应的可能是目录，也可能是文件，File只是一个路径名的抽象表示形式。

### exists

判断File对象是否存在，可以使用`exists()`方法

例如读取桌面上一个不存在的File对象

```java
File f = new File(FILE_PATH);
System.out.println(f.exists());
```

-- `false`



### createNewFile

当一个File对象不存在时创建，可以使用`createNewFile`方法，以文件的形式新建。

```java
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  if (!f.exists()) {
    f.createNewFile()
  }
}
```

![image.png](http://qiniu-note-image.ctong.top/note/images/image-b0f8db4ee408421da2960e6ae6061d66.png)



### mkdir

当一个File对象不存在时创建，可以使用`mkdir`方法，以目录的形式新建。

```java
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  if (!f.exists()) {
    f.mkdir();
  }
}
```


![image.png](http://qiniu-note-image.ctong.top/note/images/image-0bfa31b7fcb741048d90c8fa127a9d48.png)


### mkdirs

需要新建多重目录需要使用`mkdirs`方法

```java
final static String FILE_PATH = "/Users/yct/Desktop/log/a/b/c/d/e/f"; // 文件输出的位置
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  if (!f.exists()) {
    f.mkdirs();
  }
}
```




![image.png](http://qiniu-note-image.ctong.top/note/images/image-1c7f2c73a4c74e6fb39ccc069ac34984.png)


### getParent

获取文件的父路径

```java
final static String FILE_PATH = "/Users/yct/Desktop/log"; // 文件输出的位置
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  String parent = f.getParent();
  System.out.println(parent);
}
```

-- `/Users/yct/Desktop`



### getParentFile

获取父文件的File对象

```java
final static String FILE_PATH = "/Users/yct/Desktop/log"; // 文件输出的位置
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  File parent = f.getParentFile();
  // 输出绝对路径
  System.out.println(parent.getAbsolutePath());
```

-- `/Users/yct/Desktop`





### getName

获取文件名

```java
final static String FILE_PATH = "/Users/yct/Desktop/log"; // 文件输出的位置
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  File parent = f.getParentFile();
  System.out.println(parent.getName());
}
```

-- `Desktop`

### isDirectory

判断是否是一个目录

```java
final static String FILE_PATH = "/Users/yct/Desktop/log"; // 文件输出的位置
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  File parent = f.getParentFile();
  System.out.println(parent.isDirectory());
}
```

-- `true`

### isFile

判断是否是一个标准文件



```java
final static String FILE_PATH = "/Users/yct/Desktop/log"; // 文件输出的位置
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  File parent = f.getParentFile();
  System.out.println(parent.isFile());
}
```

-- `false`

### lastModified

获取文件最后修改时间，获取出来的是毫秒，需要用`SimpleDateFormat`转成自定义格式

```java
  final static String FILE_PATH = "/Users/yct/Desktop/log"; // 文件输出的位置
  public static void main(String[] args) throws IOException {
    File f = new File(FILE_PATH);
    Date da = new Date(f.lastModified());
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    System.out.println(sdf.format(da));
  }
```

-- `2020-10-02 17:12:08`



### listFiles

获取当前目录下所有的子文件。

```java
final static String FILE_PATH = "/Users/yct/Desktop"; // 文件输出的位置
public static void main(String[] args) throws IOException {
  File f = new File(FILE_PATH);
  File[] files = f.listFiles();
  for (File file : files) {
    System.out.println(file.getAbsolutePath());
  }
}
```

交作业：[文件夹拷贝](https://github.com/Clover-You/my-blog-file/blob/main/javaIO%E6%B5%81%E5%AD%A6%E4%B9%A0%EF%BC%8C%E6%96%87%E4%BB%B6%E6%8B%B7%E8%B4%9D%E5%B0%8F%E7%BB%83%E4%B9%A0.java)