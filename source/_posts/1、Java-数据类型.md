title: 1、Java 数据类型
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 13:27:00
---
### 数据类型转换规则

1. 八种数据类型当中,除布尔类型之外剩下的七种数据类型可以互相转换。
2. 小容量向大容量转换,称之为自动类型转换。容量从小到大排序:
    - byte < short < int < long < float < double < char
3. 大容量转换成小容量,叫做强制类型转换,需要加强制类型转换符 `(int)`,程序才能编译通过,但是有可能会[损失精度](#float转short损失精度)。
4. 当整数字面值没有超出`byte short char`的取值范围,可以直接赋值给`byte short char`类型的变量。
5. `byte short char`混合运算时,各自先转成`int`类型再做运算。
6. 多种数据类型混合运算时,先转换成容量最大的类型再做运算。


### 关于Java编译问题

**注意:** 可以编译通过, 3 没有超出`byte`类型取值范围
```Java
byte b = 3;
```
编译错误,因为编译器只检查语法,不会"运算" `i / 3`,只有涉及到`JVM`才会运算,编译期不执行
```Java
int i= 10;
byte b = i / 3;
```
**注意:** 任何浮点类型不管占用多少字节,都比整数型容量大。`char`和`short`可表示种类数量相同, 但是`char`可以取更大的正整数。

`javac`检查语法与编译,并不会执行代码,只有经过`JVM`才可能执行

### float转short损失精度

当`float`转为`short`时,编译成功但损失精度,最终结果为`20`
```Java  
  public static void main(String[] args){
    float a = 20.12f;
    short s = (short)a;
    out.println(s);
  }
```

### 类型转换
在Java中从左到右先算小括号,以下代码编译报错是因为第五条规则**byte short char混合运算时,各自先转成int类型再做运算**,而下面先给`s`转为`byte`,但是运算时依然会将`s`转回`int`,所以语法错误
```Java
short s = 10;
byte b = 3;
byte i = (byte)s + b;
```
编译错误
```cmd
Exception in thread "main" java.lang.Error: Unresolved compilation problem:
        Type mismatch: cannot convert from int to byte

        at DataType.main(DataType.java:8)
```

而若是将`s`与`b`先运算得出结果,再转为`byte`编译就不会报错,语法通过
```Java
    short s = 10;
    byte b = 3;
    byte i = (byte)(s + b);
    out.println(i);
```
**编译器(javac)只检查语法与编译,并无任何代码执行**

### 自动类型转换

`10 / 3`正确结果等于3.333333...,但以下输出结果是`3.0`,这是因为`d`变量的字面值是整数运算完成后的结果,而计算时`10`与`3`都是`int`类型,上面第六条提到: **多种数据类型混合运算时,先转换成容量最大的类型再做运算**,所以最终结果计算为`3`并赋值给`d`

```java
  public static void main(String[] args){
    double d = 10 / 3;
    out.println(d);
  }
```

而当`10`与`3`都为`double`类型时,结果为`3.3333333333333335`

```Java
  public static void main(String[] args){
    double a = 10.0d;
    double b = 3.0d;
    double d = a / b;
    out.println(d);
  }
```

到此可以得出一个结论,当字面值运算时,它的结果不一定与等号左侧数据类型相等,而我们看到的数据,是它们转换后的结果


### 总结

1. 除`Boolean`外,其它七种**基本数据类型**可以互相转换
2. 小容量转大容量叫做**自动类型转换**
3. `char` 和 `short`平级
4. 大容量 转换成小容量叫做**强制类型转换**,强制类型转换需要加强制类型转换符`(...)`如`(int)`,编译通过但是程序运行期可能会导致**精度损失**

> 所有学习资料均来自互联网,本人各种摘抄实验的结果
