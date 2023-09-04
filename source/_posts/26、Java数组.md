title: 26、Java数组
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 16:48:52
---
## 数组概述

1. java 语言中的数组，是一种引用数据类型，不属于基本数据类型。数组的父类是Object。

2. 数组实际上是一个容器，可以同时容纳多个元素。数组是一个数据的集合。

3. 数组当中可以存储**基本数据类型**的数据，也可以存储引用数据类型的数据。

4. 数组因为是引用类型，所以数组对象是在堆内存当中。

5. 数组在内存中是怎样的一个表示？
   ![截屏20200606 17.49.49.png](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2020-06-06%2017.49.49-2938dc19f5fb40a080037548eb1d1c40.png)   
6. 数组当中如果存储的是“**Java对象**”的话，实际上存储的是对象的“**引用**”

7. 数组一旦创建，在Java中规定，长度不可变。

8. 数组的分类

   - 一维数组
   - 二维数组
   - 三维数组
   - 多维数组

9. 所有的数组对象，都有length属性，用来获取数组中元素的个数。

10. Java中的数组要求数组中元素的类型统一。

11. 数组在内存方面存储的时候，数组中的元素内存地址是连续的（存储的每一个元素都是有规则的挨着排序的）。内存地址连续这是数组存储元素的特点。数组实际上是一种简单的数据结构。

12. 数组中，每一个元素都是有下标的，下标从0开始，以1递增。最后一个元素的下标是：length - 1
    **下标非常重要，因为我们对数组中元素进行“存取”的时候，都需要通过下标来进行。**

13. 数组中首元素的内存地址作为整个数组对象的内存地址。

14. 数组这种数据结构的优点和缺点是什么？

    - 优点：查询、查找、检索某个下标上的元素时效率极高，可以说是查询效率最高的一个数据结构。

      1. 每一个元素的内存地址在空间存储上是连续的。
      2. 每一个元素类型相同，所以占用空间大小一样。
      3. 知道第一个元素内存地址，知道每一个元素占  用空间大小，又知道下标，所以通过一个数学表达式就可以计算出某个下标上元素的内存地址。直接通过内存地址定位元素，所以数组的检索效率是最高的。
         **数组中存储100个元素和100万个元素，在元素查询、检索方面，效率是相同的，因为数组中元素查找的时候不会一个一个找，是通过数学表达式计算出来的。算出一个内存地址直接定位的。**

    - 缺点

      1. 由于为了保证数组中每个元素的内存地址连续，所以在数组上随机删除或增加元素的时候，效率较低。因为随机增删元素会涉及到后面元素统一向前或向后位移的操作。
      2. 数组不能存储大数据量，因为很难在内存空间上找到一块特别大的连续的内存空间。

      

      > 对于数组中最后一个元素的增删，是没有效率影响的。







## 一维数组

### 定义


```java
int[] arrayInt;
String[] arrayString;
boolean[] arrayBoolean;
double[] arrayDouble;
Object[] arrayObject;
```

### 初始化

静态初始化一维数组：

```java
int[] arrayInt = {100, 200, 300, 400};
```



动态初始化一维数组：

```java
int[] arrayInt = new int[5];
```

`5`表示数组的元素个数。

初始化一个长度为5的int类型一维数组，每个元素默认值为0

```java
String[] arrayString = new int[2];
```

初始化一个长度为2的String类型一维数组，每个元素默认值为null

### 读写

```java
int[] arrayInt = {100, 200, 300, 400};
arratInt[0] = 520;
arrayInt[arrayInt.length - 1] = 1314;
System.out.println("第一个元素：" + arrayInt[0]);
System.out.println("最后一个元素：" + arrayInt[attayInt.length - 1])
```

将第一个元素改为520，将最后一个元素改成1314,输出第一个元素的值和第最后一个元素的值：
\-- `第一个元素：520`

\-- `最后一个元素：1314`



### 遍历

```java
int[] arrayInt = {100, 200, 300, 400};
for (int i = 0; i < arrayInt.length; i++) {
  System.out.println(arrayInt[i]);
}
```

分别输出：

\-- `100`

\-- `200`

\-- `300`

\-- `400`

数组可以通过下标来指定数组的元素，可以将`i`当作数组的下标





```java
int[] arrayInt = {100, 200, 300, 400};
System.out.println(arrayInt[4]);
```

下标为4表示第五个元素，第五个元素不存在，下标越界抛出异常：

`ArrayIndexOutOfBoundsException`



### 多态

数组中支持使用动态，如果调用的方法是父类型中存在的方法不需要向下转型。直接使用父类型引用调用即可。

例如：

```java
public class ArrayTest01 {
    public static void main(String[] args) {
        Animal[] animals = {new Cat(), new Bear()};
        for (int i = 0; i< animals.length; i++){
            animals[i].move();
        }
    }
}
class Animal {
    public void move(){
        System.out.println("动物在走路!");
    }
}
class Cat extends Animal{
    @Override
    public void move(){
        System.out.println("猫在走路!");
    }
}
class Bear extends Animal{
    @Override
    public void move(){
        System.out.println("熊在走路!");
    }
}
```

输出：

\-- `猫在走路!`

\-- `熊在走路!`



> 对于数组来说，实际上只能存储java对象的“**内存地址**”。数组中存储的每个元素是“**引用**”



### 扩容

关于一维数组的扩容：

1. 在java开发中，数组长度一旦确定就不可变，数组满了需要扩容

   - Java中对数组的扩容是先新建一个大容量的数组，然后将小容量数组中的数据一个一个拷贝到大数组当中。
     ![截屏20200608 10.36.59.png](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2020-06-08%2010.36.59-14e053cd335a490f9bb0c21cf97d5a43.png)

2. 使用`System.arraycopy`方法拷贝数组

   ```java
   Object[] objs = {new Object(), new Object(), new Object()};
   Object[] newObjs = new int[5];
   System.arraycopy(arrayInt, 0, destArrayInt, 0, arrayInt.length);
   ```

   结果：

   `1 2 3 4 0 0 0 0`
   `System.arraycopy`方法有5个参数，第一个参数为拷贝源，第二个参数为源的拷贝起始位置，第三个参数为拷贝到的目标数组，第四个参数为目标数组的起始位置，第五个参数为需要拷贝源的对象数量。
   ![截屏20200608 11.22.59.png](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2020-06-08%2011.22.59-51c5494102174b6dbc22c12f54032be7.png)



> 数组扩容效率较低，因为涉及到拷贝的问题。



## 二维数组

关于Java中的二维数组：

1. 二维数组其实是一个特殊的一维数组，特殊在每一个一维数组当中的每一个元素是一个一维数组。

### 定义

```java
int[][] arrayInt;
```



### 初始化

#### 静态初始化

一维数组

```java
int[] arrayInt = {...};
```

二维数组

```java
int[][] arrayInt = {{1, 2, 3, 4}, {5, 6, 7}}; 
```

#### 动态初始化

初始化一个3行4列的二维数组，每一个一维数组当中有4个元素。

```java
int[][] arrayInt = new int[3][4];
```

### 取值



```java
int[][] arrayInt = {{1, 2, 3, 4}, {5, 6, 7}};
System.out.println(arrayInt[0][2]);
```

第一个括号表示的是二维数组的下标，第二个括号表示的是二维 数组中的一维数组的下标。第一个一维数组中的第三个元素。 

输出：

\-- `3`





### 遍历

```java
int[][] newArrayInt = {{1, 2, 3, 4}, {5, 6, 7}};
for (int i = 0; i < newArrayInt.length; i++) { 
  for (int y = 0; y < newArrayInt[i].length; y++) {
    System.out.printf("第%d一维数组的第%d个值为：%d\n", i + 1, y + 1, newArrayInt[i][y]);
  }
}
```

\-- `第1一维数组的第1个值为：1`
\-- `第1一维数组的第2个值为：2`
\-- `第1一维数组的第3个值为：3`
\-- `第1一维数组的第4个值为：4`
\-- `第2一维数组的第1个值为：5`
\-- `第2一维数组的第2个值为：6`
\-- `第2一维数组的第3个值为：7`



> 二维数组与一维数组的读写方式一样。
>
> 多维数组与二维数组雷同。