title: 31、Java BigDecimal 数字类
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 16:56:27
---
### DecimalFormat

数字格式有：

- `#` 代表任意数字
- `,`代表千分位
- `.`代表小数点
- `0`不够时补0



`DecimalFormat`专门用于数字的一个格式化

```java
DecimalFormat df = new DecimalFormat();
```

加入千分位并且保留两位小数

```java
DecimalFormat df = new DecimalFormat("###,###.##");
String money = df.format(2000.213);
```

\-- `2,000.21`

---

保留4位小数，不够时补上0

```java
DecimalFormat df = new DecimalFormat("###,###.0000");
String money = df.format(123456789.2);
```

\-- `123,456,789.2000`



### BigDecimal

1. `BigDecimal` 属于大数据，精度极高。不属于基本数据类型，属于java对象（引用数据类型），频繁使用在财务软件当中。
2. 财务软件当中的double是不够用的。



这个一百不是普通的一百，而是一个**精度**极高的100 

```java
BigDecimal bd = new BigDecimal(100.87987);
```

如果需要做运算，需要使用`BigDecimal`提供的方法进行运算

#### 加法

```java
BigDecimal bd = new BigDecimal(100.87987);
BigDecimal db1 = new BigDecimal(200);
System.out.println(bd.add(db1));
```

\-- `300.8798699999999968213160173036158084869384765625`



#### 最大值

```java
BigDecimal bd = new BigDecimal(100.87987);
BigDecimal db1 = new BigDecimal(200);
System.out.println(bd.max(db1));
```

\-- `200`



#### 除法

```java
BigDecimal bd = new BigDecimal(100.87987);
BigDecimal db1 = new BigDecimal(200);
System.out.println(bd.divide(db1));
```

\-- `0.5043993499999999841065800865180790424346923828125`