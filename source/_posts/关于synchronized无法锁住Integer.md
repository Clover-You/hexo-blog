title: 关于synchronized无法锁住Integer
categories: 随笔
tags: [随笔,JAVA]
date: 2022-01-02 18:15:43
---
## 原因

在多线程的时候，为了保证数据安全，必须在修改数据时使用线程同步，java中的`synchronized`用来实现线程同步、线程列队。

学完多线程基础的我，写一个多线程交替输出`1,2,3,4,5...`，一个线程负责输出偶数，一个线程负责输出奇数，由于这个数是整数，在java中`int`又是基本数据类型，基本数据类型并不是共享的，也就是基础数据类型是在栈中申明，java提供了一个`Integer`包装类，使用的时候发现根本锁不住这个对象。

回忆之前对`Integer`的知识：为了提高程序效率，`Integer`在类加载时就已经创建了`-128到127`的对象

![底层代码](http://qiniu-note-image.ctong.top/note/images/202112271114093.png)

嗯！！！！如果我对这个对象进行运算的话！内存地址是不是也不一样？（细思极恐）

```java
System.out.println(num++;)
```



![内存地址](http://qiniu-note-image.ctong.top/note/images/202112271114940.png)

## 粗暴解决方案

建一个加锁类（人工造锁），将需要用到的运算什么的加进去，嗯简单粗暴！！！！

```java
class IntegerLock {
  private Integer num = 0;

  public Integer getNum() {
    return num;
  }

  public void setNum(Integer num) {
    this.num = num;
  }

  public Integer remainder(Integer o) {
    return num % o;
  }

  public void plus(Integer o) {
    num = num + o;
  }

  @Override
  public String toString() {
    return num.toString();
  }
}
```

![完成](http://qiniu-note-image.ctong.top/note/images/202112271114724.png)