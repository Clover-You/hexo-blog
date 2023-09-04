title: 40、Java 多线程
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 17:19:00
---
## 什么是进程？什么是线程？

1. 进程是一个应用程序。

2. 线程是一个进程中的执行场景、执行单元。一个进程可以启动多个线程

3. 对于Java程序来说，当在DOS命令窗口输入：java Hello World回车之后，会先启动JVM，而JVM就是一个进程。JVM再启动一个主线程调用main方法。同时再启动一个垃圾回收线程负责看护，回收垃圾。

4. 使用了多线程机制之后，main方法结束，是不是有可能程序也不会结束。main方法结束只是主线程结束了，主栈空了，其它的栈（线程）可能还在压栈弹栈。

    > 进程A和进程B的内存独立不共享
    >
    > 在java语言中：线程A和线程B，堆内存和方法区内存共享。但是栈内存独立，一个线程一个栈。
    
    ![多线程内存结构图](http://qiniu-note-image.ctong.top/note/images/image-fcc7160104824cb5b20facbca02b58b5.png)

5. 对于单核的CPU来说，真的可以做到真正的多线程并发吗？对于多核的CPU来说，真正的多线程并发是没问题的，4核CPU表示同一个时间点上，可以真正的有4个进程并发执行。

   **什么是多线程并发？**

   t1线程执行t1的，t2线程执行t2的，它们不会相互影响，这叫做真正的多线程并发。对于单核CPU来说，在某一个时间点上实际上只能处理一件事情，但是由于CPU的处理速度极快，多个线程之间频繁切换执行，给人的感觉是多个事情同时在做。

   **单核CPU表示只有一个大脑：**

   不能够做到真正的多线程并发，但是可以做到给人一种“多线程并发”的感觉。

   > 电影院采用交卷播放电影，一个交卷一个交卷播放速度达到一定程度之后，人类的眼睛产生了错觉，感觉是动画的，这说明人类的反应速度很慢。

## 实现方式

java语言中，实现线程有两种方式

### 第一种方式

 编写一个类，直接继承**java.lang.Thread**，重写`run`方法

```java
public class TestThread {
  public static void main(String[] args) {
    MyThread myThread = new MyThread();
    // 启动线程
    myThread.start();
    int forCount = 100;
    for (int i = 0; i < forCount; i++) {
      System.out.println("主线程打印=======》》》》" + i);
    }
  }
}

public class MyThread extends Thread {
  @Override
  public void run() {
    int forCount = 100;
    for (int i = 0; i < forCount; i++) {
      System.out.println("分支线程打印=======》》》》" + i);
    }
  }
}
```

start() 方法的作用是：启动一个分支线程，在JVM中开辟一个新的栈空间，这段代码任务完成之后瞬间就结束了。这段代码的任务只是为了开辟一个新的栈空间，只要新的栈空间开出来，start() 方法就结束了，启动成功的线程会自动调用run方法，并且run方法在分支栈的栈底部（压栈）。run方法在 分支栈的栈底部，main方法在主栈的栈底部，它们两是平级的！
    
![执行结果](http://qiniu-note-image.ctong.top/note/images/image-428c0fc525a94f668b2a9c194cfeba81.png)

### 第二种方式

编写一个类实现java.lang.Runable接口
 这还只是一个可运行的类，它还不是一个线程。

```java
public class MyRunnable implements Runnable {
  @Override
  public void run() {}
}

```

 需要使用`Thread`封装成一个线程对象

```java
MyRunnable myRunnable = new MyRunnable();
Thread t = new Thread(myRunnable);
```

启动多线程

```java
t.start();
```

> 第二种方式实现接口比较常用，因为一个类实现了接口，它还可以去继承其它的类，更灵活。



## 线程生命周期

1. 新建状态
2. 准备状态
3. 运行状态
4. 阻塞状态
5. 死亡状态



![线程生命周期](http://qiniu-note-image.ctong.top/note/images/image-faad6930b5514002a557978eae51425e.png)


## 获取线程对象名字

现在有一个线程对象

```java
MyThread mt = new MyThread();
Thread t = new Thread(mt);
```

设置它的名字

```java
t.setName("我的小线程！！");
```

打印输出这个线程对象的名字

```java
t.getName()
```

--  `我的小线程！！`

如果不使用`setName`方法设置线程的名字，那么它也会有一个默认名字

**Thread-0**

这个默认线程的名字以**Thread-**与当前进程的线程数组成，这个线程数以0开始，例如：

```
Thread-0、Thread-1、Thread-2、Thread-3
```



## 获取当前线程对象

可以使用`Thread`对象中的`currentThread`方法来获取当前线程对象，这是一个静态方法，返回一个`Thread`对象

```java
Thread thread = Thread.currentThread();
for (int i = 0; i < forCount; i++) {
  System.out.println(thread.getName() + "=======》》》》" + i);
}
```

-- `我的小线程！！=======》》》》0`

## 线程的sleep方法

关于线程的sleep方法：

static void sleep(long illis)

1. 静态方法
2. 参数是毫秒
3. 作用：让当前线程进入休眠，进入“阻塞状态”，放弃占有CPU时间片，让给其他线程使用。

```java
int millis = 1000 * 5;
System.out.println("准备开始睡眠！");
try {
  Thread.sleep(millis);
} catch (Exception e) {
  e.printStackTrace();
}
System.out.println("线程休眠结束！！！");
```

---

`t.sleep`会让线程**t**进入休眠状态吗？

```java
public class TestThread {
  public static void main(String[] args) {
    int millis = 1000 * 5;
    System.out.println("准备开始睡眠！");
    MyThread myThread = new MyThread();
    Thread t = new Thread(myThread);
    t.setName("t");
    t.start();
    try {
      t.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}

public class MyThread extends Thread {
  @Override
  public void run() {
    int forCount = 100;
    Thread thread = Thread.currentThread();
    for (int i = 0; i < forCount; i++) {
      System.out.println(thread.getName() + "=======》》》》" + i);
    }
  }
}
```

答案是不会，因为`sleep`是**静态方法**，它跟对象没有半毛钱关系，在执行的时候还是会转换成 `Thread.sleep(millis)`，这行代码的作用是：让当前线程进入休眠状态，也就是说main线程会进入休眠状态。



## 终止线程的睡眠

sleep睡眠太久了，如何唤醒一个正在睡眠的线程？

> run()当中的异常不能throws，只能try...catch。因为子类不能比父类抛出更多的异常。或者说重写之后的方法不能比重写之前的方法抛出更多（更宽泛）的异常，可以更少。run()方法在父类中没有抛出任何异常

```java
public class TestThread {
  public static void main(String[] args) {
    TestSleep testSleep = new TestSleep();
    Thread thread = new Thread(testSleep);
    thread.setName("沉睡中的线程");
    thread.start();
    System.out.println("5秒后叫醒沉睡的线程！！！");
    int millis = 1000 * 5;
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    // 在这里唤醒线程
  }
}

class TestSleep extends Thread {
  @Override
  public void run() {
    System.out.println("进入沉睡！！");
    int millis = 1000 * 60 * 60 * 24 * 365;
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.println("我被叫醒起来干活了！！");
  }
}
```

### interrupt

可以使用`interrupt`方法来终断线程，这种方式是依靠了java的异常处理机制来唤醒线程，相当于你在睡觉，我直接给你泼一盆冷水过去。

```java
thread.interrupt();
```

这个代码执行之后，`TestSleep`中的`Thread.sleep(millis);`受到了干扰出现了异常，这个异常被`try`抓住之后就被结束掉了，catch执行过后try...catch结束开始继续执行下边的代码。

![interrupt执行结果](http://qiniu-note-image.ctong.top/note/images/image-17bec6a0476d4ec8ac96e6a1c300d90d.png)



### stop

如果需要强行终止一个线程的执行，可以使用`stop`方法强行终止该线程（此方法已过时，不建议使用），这个stop不是终止线程睡眠，而是强制干掉指定线程。如果使用stop强行终止线程，则极大概率造成数据丢失、损坏等严重后果。

```java
public class TestThread {
  public static void main(String[] args) {
    TestSleep testSleep = new TestSleep();
    Thread thread = new Thread(testSleep);
    thread.setName("执行中的线程");
    thread.start();
    System.out.println("3秒后叫杀死线程！！！");
    int millis = 1000 * 3;
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    thread.stop();
  }
}

class TestSleep extends Thread {
  @Override
  public void run() {
    int millis = 1000 ;
    try {
      for (int i = 0; i< 10; i++) {
        System.out.println("执行中的线程-----> " + i);
        Thread.sleep(millis);
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

如何合理的终止一个线程？

可以使用标记的方式结束一个正在执行的线程。

```java

public class TestThread {
  public static void main(String[] args) {
    TestSleep testSleep = new TestSleep();
    Thread thread = new Thread(testSleep);
    thread.start();
    System.out.println("3秒后杀死线程！！！");
    int millis = 1000 * 3;
    try {
      Thread.sleep(millis);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    testSleep.run = false;
    System.out.println("thread kill");
  }
}

class TestSleep extends Thread {
  /**
     * 定义一个标记，用来结束这个线程
     */
  boolean run = true;

  @Override
  public void run() {
    for (int i = 0; i < 10; i++) {
      if (this.run) {
        try {
          int millis = 1000;
          System.out.println("执行----->" + i);
          Thread.sleep(millis);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      } else {
        // 在这里处理结束线程之前需要执行的某些操作、例如保存数据等
        return;
      }
    }
  }
}
```



## 线程调度概述

1. 常见的线程调度模型有哪些？

   - 抢占式调度模型：哪个线程的优先级比较高，抢到的CPU时间片的概率就高一些/多一些。java采用的就是抢占式调度模型。
   - 均分式调度模型：平均分配CPU时间片。每个线程占有的CPU时间片时间长短一样。平均分配，一切平等。

2. Java中提供了哪些方法是和线程调度有关系的呢？

   - 实例方法：

     1. `void setPrioriry(int new Prioriry)`设置线程的优先级

     2. `int getPriority()`获取线程优先级,最低优先级1，默认优先级5，最高优先级是10，优先级比较高的获得CPU时间片可能比较多一些（但也不完全是，大概率是多的）。

     3. `void join()`合并线程，使当前线程进入阻塞状态，t线程执行，直到t线程结束，当前线程才可以继续执行，例如代码是这样的情况下：

        ```java
        public class TestJoin {
          public static void main(String[] args) {
            MyThread1 myThread1 = new MyThread1();
            Thread t = new Thread(myThread1);
            try {
              t.start();
              t.join(); // 使当前线程进入阻塞，t开始执行
            } catch (InterruptedException e) {
              e.printStackTrace();
            }
            for (int i = 0; i < 10; i++) {
              System.out.println("main ----> " + i);
            }
          }
        }
        
        class MyThread1 extends Thread {
          @Override
          public void run() {
        
            for (int i = 0; i < 10; i++) {
              System.out.println("run ----> " + i);
            }
          }
        }
        ```

        

   - 静态方法

     1. `static void yield()` 暂停当前正在执行的线程对象，并执行其他线程（让位方法）。yield方法不是阻塞方法，让当前线程让位，让给其他线程使用。yield方法的执行会让当前线程从**“运行状态”**回到**“就绪状态”**

        > 回到就绪状态之后有可能还会再抢到时间片

        ![线程yield](http://qiniu-note-image.ctong.top/note/images/image-ac178fc547e84b62a87a0930a88cd6b0.png)



### 线程优先级

关于线程的优先级

```java
System.out.println("线程最大优先级：" + Thread.MAX_PRIORITY);
System.out.println("线程默认优先级：" + Thread.NORM_PRIORITY);
System.out.println("线程最小优先级：" + Thread.MIN_PRIORITY);
```

-- `线程最大优先级：10`
-- `线程默认优先级：5`
-- `线程最小优先级：1`

设置进程优先级，优先级高的CPU时间片多一些，是指线程处于运行状态的时间多。

```java
public class ThreadPriority {
  public static void main(String[] args){
    Thread thisThread = Thread.currentThread();
    System.out.println("设置之前：" + thisThread.getPriority());
    thisThread.setPriority(10);
    thisThread.getPriority();// 当前线程优先级
    System.out.println("设置之后：" + thisThread.getPriority());

  }
}

```

-- `设置之前：5`
-- `设置之后：10`



### 线程让位

让位，当前线程暂停，回到就绪状态，让给其它线程。

```java
Thread.yield();
```

让位回到就绪状态之后，有可能还会抢到CPU时间片

```java
public class TestYield {
  public static void main(String[] args) {
    YieldThread yieldThread = new YieldThread();
    Thread t = new Thread(yieldThread);
    t.start();
    for (int i = 0; i < 10000; i++) {
      if (i % 100) {
        Thread.yield();
      }
      System.out.println("main ----> " + i);
    }
  }
}

class YieldThread extends Thread {
  @Override
  public void run() {
    for (int i = 0; i < 10000; i++) {
      System.out.println("run ----> " + i);
    }
  }
}

```

![线程生命周期-yield](http://qiniu-note-image.ctong.top/note/images/image-ac178fc547e84b62a87a0930a88cd6b0-20211227102835587.png)


### 合并线程

join方法可以将`t`线程合并到当前线程中，当前线程受阻塞，`t`线程执行直到结束。 

```java
public class TestJoin {
  public static void main(String[] args) {
    MyThread1 t = new MyThread1();
    System.out.println("main thread start!");
    t.start();
    try {
      t.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.println("main thread over!");
  }
}

class MyThread1 extends Thread {
  @Override
  public void run() {

    for (int i = 0; i < 10; i++) {
      System.out.println("run ----> " + i);
    }
  }
}
```

-- `main thread start!`
-- `run ----> 0`
-- `run ----> 1`
-- `run ----> 2`
-- `main thread over!`

> 线程合并不是意味着两个线程合并成了一个线程，而是两个线程之前发生了等待关系。



## 线程安全

1. 为什么这个是重点？
   以后在开发中，我们的项目都是运行在服务器当中，而服务器已经将线程对象的创建、线程的启动等，都已经实现完成了。这些代码都不需要编写。最重要的是我们要知道，我们编写的程序需要放到一个多线程的环境下运行，你更需要关注的是这些数据在多线程并发的环境下是否是安全的。

2. 什么时候数据在多线程并发的环境下会存在安全问题？

   1. 多线程并发
   2. 有共享数据
   3. 共享数据有修改的行为

   满足以上三个条件之后就会存在线程安全问题。

   ![造成线程安全原理图](http://qiniu-note-image.ctong.top/note/images/image-ffd3735a179643adb0831bdf779b1f11.png)

   

   <iframe src="//player.bilibili.com/player.html?aid=93347594&bvid=BV1mE411x7Wt&cid=163966574&page=305" scrolling="no" width="100%" style='height: -webkit-fill-available' border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

3. 怎么解决线程安全问题呢？

   当多线程并发的环境下，有共享数据，并且这个数据还会被修改，此时就存在线程安全问题，怎么解决这个问题？

   - 线程排队执行。（不能并发）。用排队执行解决线程安全问题，这种机制被称为：线程同步机制。

   - 专业术语叫做：线程同步，实际上就是线程不能并发了，线程必须排队执行。
   - 使用**线程线程同步机制**解决这个问题，线程同步就是线程排队了，线程排队了就会牺牲一部分效率

4. 说到线程同步这块，涉及到这两个专业术语：
   
   - 异步编程模型：
     线程t1和线程t2，各自执行各自的，t1不管t2，t2不管t1，谁也不需要等谁，这种编程模型叫做：异步编程模型。其实就是：多线程并发（效率较高。）
   - 同步编程模型：
     线程t1和线程t2，在线程t1执行的时候，必须等待t2执行结束，或者说在t2线程执行的时候，必须等待t1线程执行结束，两个线程之间发生了等待关系，这就是同步编程模型，效率较低。线程排队执行



### synchronized 关键字 -- 线程同步机制

线程同步的语法是：

```java
synchronized () {
  // 线程同步代码块
}
```

synchronized后面小括号中传的这个数据是相当关键的。这个数据必须是多线程共享的数据。才能达到多线程排队。小括号中写什么那主要是看你想要哪些线程同步了。

假有：t1、t2、t3、t4、t5，有五个线程，你只希望t1、t2、t3排队，t4、t5不需要排队怎么办？

- 只需要在小括号中写一个：t1、t2、t3共享的对象。而这个对象对t4、t5来说不是共享的。

  例如有一个账户类，类中的金额这个数据是共享的，这个类还有一个取款方法（`withbraw`中加`sleep`是为了模拟网络延迟）：

  ```java
  
  class Account {
    /**
       * 金额
       */
    private double balance = 10000;
  
    private String Name;
  
    public String getName() {
      return Name;
    }
  
    public void setName(String name) {
      Name = name;
    }
  
    public double getBalance() {
      return balance;
    }
  
    public void setBalance(double balance) {
      this.balance = balance;
    }
  
    /**
       * 取款
       *
       * @param money
       */
    public void withbraw(double money) {
        double before = getBalance();
        double after = before - money;
        try {
          Thread.sleep(1000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        setBalance(after);
      }
  }
  ```

  对于以上程序来说，这是相当危险的操作，因为`sleep`模拟网络延迟之后，无法调用`setBalance`方法去修改金额，第二个线程进来的时候金额还是`1000`，导致两个人对同一个账户取款5000时还剩下5000，如果加上线程同步就不会发生这种问题，因为某个线程在执行的时候其他线程在后面排队

  ```java
  public void withbraw(double money) {
    synchronized (this) {
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      double before = getBalance();
      double after = before - money;
      setBalance(after);
    }
  }
  ```

- 在java语言中，任何一个对象都有**"一把锁"**，其实这把锁就是一个标记。(只是把它叫做锁)，100个对象100把锁，一个对象一把锁。

- 以上代码的执行原理？

  1. 假设t1和t2线程并发，开始执行以上代码的时候，肯定有一个先一个后。
  2. 假设t1先执行了，遇到了`synchronized`的时候自动找“后面共享对象”的对象锁，找到之后，并占有这把锁，然后执行同步代码块中的程序，在程序执行过程中占有这把锁的。直到同步代码块执行结束，这把锁才会释放。
  3. 假设t1已经占有这把锁，此时t2也遇到了`synchronized`关键字，也会去占有后面共享对象的这把锁，结果这把锁被t1占有，t2只能在同步代码块外面等待t1的结束，直到t1把同步代码块执行结束，t1会归还这把锁，此时t2终于等到这把锁，然后t2占有这把锁之后，进入同步代码执行程序。

  > 这样就达到了线程排队执行
  >
  > 这里需要注意的是：这个共享对象一定要选好。这个共享对象一定是你需要排队执行的这些线程对象共享的。

  ![线程生命周期-synchronized](http://qiniu-note-image.ctong.top/note/images/image-1b76c2ef7b0a43c5bb7e1e814cb23ec1.png)
  
  4. Java中有三大变量？
  
     - 实例变量：在堆中
     - 静态变量：在方法区
     - 局部变量：在栈中
  
     > 以上三大变量中，局部变量永远都不会存在线程安全问题。因为局部变量不共享。一个线程一个栈，局部变量在栈中。
     >
     > 实例变量在堆中，堆只有一个。
     >
     > 静态变量在方法区中，方法区只有一个。
     >
     > 堆和方法区都是多线程共享的，所以可能存在线程安全问题。
  
  5. 同步代码块越小效率越高。
  
  6. 在实例方法上可以使用`synchronized`
     synchronized出现在实例方法上，一定锁的是this，也只能是this。不能再是其他对象了，所以这种方式不灵活。
  
     另外还有一个缺点：synchronized出现在实例方法上，表示整个方法体都需要同步，可能会无故扩大同步的范围，导致程序的执行效率降低。所以这种方式不常用
  
  7. synchronized有三种写法：
  
     - 同步代码块，比较灵活
  
       ```java
       synchronized(线程共享对象) {
         同步代码块;
       }
       ```
  
     - 在实例方法上使用synchronized
  
       表示共享对象一定是this，并且同步代码块是整个方法体。
  
     - 在静态方法上使用synchronized
       表示找类锁
       类锁永远只有一把，就算创建了100个对象，那类锁也只有一把。



## 线程死锁

死锁会导致程序不出异常，也不会出现错误，程序一直僵持在那里，这种错误最难调试。



### 死锁简单代码

```java

public class DeadLock {
  public static void main(String[] args) {
    Object o1 = new Object();
    Object o2 = new Object();
    MyThread2 t1 = new MyThread2(o1, o2);
    MyThread3 t2 = new MyThread3(o1, o2);
    t1.start();
    t2.start();
  }
}

class MyThread2 extends Thread {
  Object o1;
  Object o2;
  public MyThread2(Object o1, Object o2) {
    this.o1 = o1;
    this.o2  = o2;
  }
  @Override
  public void run() {
    synchronized (o1) {
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      synchronized (o2) {
        System.out.println("MyThread2");
      }
    }
  }
}
class MyThread3 extends Thread {
  Object o1;
  Object o2;
  public MyThread3(Object o1, Object o2) {
    this.o1 = o1;
    this.o2  = o2;
  }
  @Override
  public void run() {
    synchronized (o2) {
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      synchronized (o1) {
        System.out.println("MyThread3");
      }
    }
  }
}

```

> synchronized在开发中最好不要嵌套使用，一不小心就可能导致死锁现象发生。

### 解决方案

在开发中是一上来就选择线程同步吗？synchronized，不是，synchronized会让程序执行效率降低，用户体验不好。 

1. 尽量使用局部变量来代替“实例变量和静态变量”
2. 如果必须是实例变量，那么可以考虑创建多个对象，这样实例变量的内存就不共享里。（一个线程对应一个对象，100个线程对应100个对象，对象不共享，就没有数据安全问题了。）
3. 如果不能使用局部变量，对象也不能创建多个，这个时候就只能选择synchronized了，线程同步机制。



## 线程相关内容

1. 守护线程

   Java语言中线程分为两大类：

   1. 用户线程

      主线程`main`方法是一个用户线程。

   2. 守护线程（后台线程）
      其中具有代表性的就是：gc垃圾回收线程。

      #### 守护线程的特点

      一般守护线程是一个死循环，所有的用户线程只要结束，守护线程自动结束。

      #### 守护线程用在什么地方呢？

      守护线程的目的是守护，比如每天00:00的时候系统数据自动备份，这个需要使用到定时器，并且我们可以将定时器设置为守护线程。一直在那里看着，每到00:00的时候开始备份一次。

      #### 设置守护线程

      在启动之前将线程设置为守护线程

      ```java
      Thread t = new Thread(...);
      t.setDaemon(true);
      t.start();
      ```

      即使是死循环，但由于该线程是守护者，当用户线程结束，守护线程自动终止。

      

2. 定时器
   间隔特定的时间去做特定的事。

   比如每周要进行银行账户的总帐操作。每天要进行数据的备份操作。在实际开发中，每隔多久执行一段特定的程序，这种需求是很常见的，那么在java中其实可以采用很多种方法实现

   1. 可以使用sleep方法，睡眠，设置睡眠时间，每到这个点醒来，执行任务，这种方式是最原始的。
   2. 在java的类库中已经写好了一个定时器：java.util.Timer，不过这种方式在目前的开发中也很少用，因为现在有很多高级框架都是支持定时任务的。

   #### 使用

   ```java
   public class TestTimer {
     public static void main(String[] args) {
       logTimerTask logTimerTask = new logTimerTask();
       Timer timer = new Timer("记录日志");
       SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-d HH:mm:ss");
       try {
         Date da = sdf.parse("2020-11-03 23:52:00");
         timer.schedule(logTimerTask,da, 1000 * 10);
       } catch (ParseException e) {
         e.printStackTrace();
       }
     }
   }
   
   class logTimerTask extends TimerTask {
   
     @Override
     public void run() {
       System.out.println("开始记录！");
     }
   }
   
   ```

   

3. 实现线程的第三种方式：FutrueTask方式，实现Callable接口。（JDK8新特性）
   这种方式实现的线程可以获取线程的返回值，之前写的那几种方式是无法获取线程的返回值，因为run方法返回void

   #### 实现线程的第三种方式

   创建一个未来任务类对象

   `java.util.concurrent.FutureTask`JUC包下的，属于java的并发包，老JDK中没有这个包。

   ```java
   FutureTask ft = new FutureTask(new Callable() {
     @Override
     public Object call() throws Exception {
       // call 方法就相当于run方法
       return null;
     }
   });
   ```

   使用这个未来任务类对象

   ```java
   FutureTask ft = new FutureTask(...);
   Thread t = new Thread(ft);
   t.start();
   ```

   使用FutureTask中的get方法来获取线程的返回值。

   ```java
   FutureTask ft = new FutureTask(...);
   Thread t = new Thread(ft);
   t.start();
   try {
     ft.get();
   } catch (InterruptedException e) {
     e.printStackTrace();
   } catch (ExecutionException e) {
     e.printStackTrace();
   }
   ```

   **使用get方法之后会阻塞当前线程，而get方法可能需要很久。因为get方法是为了拿另一个线程的执行结果，另一个线程执行是需要时间的。**

   

4. 关于Object类中的wait和notify方法（生产者和消费者模式）

   - wait和notify不是线程对象的方法，是java中任何一个对象都有的方法，因为这两个方法是Object类中自带的。

   - wait方法的作用

     ```java
     Object o = new Obect();
     o.wait();
     ```

     表示让正在o对象上活动的线程进入等待状态，无期限等待，直到被唤醒为止。（当前线程）

     **o.wait();方法会让正在o对象上活动的当前线程对象进入等待状态，并且释放之前占有的o对象的锁**

   - notify方法的作用

     T线程在o线程上活动，T线程是当前线程对象。当调用o.wait方法之后。T线程进入无期限等待。当前线程进入等待状态，直到最终调用notify方法。o.notify方法的调用可以让正在o对象上等待的线程唤醒。
     `o.notify();`

     **表示唤醒正在o对象上等待的线程。**

   ### 生产者和消费者

   生产者和消费者模式是为了专门解决某个特定需求的。

   一个线程负责生产，一个线程负责消费，而且还要达到生产和消费必须均衡。例如：

   生产满了，就不能再继续生产了，必须让消费线程进行消费。消费完了，就不能消费了，必须让生产线程进行生产

   

   > wait方法和notify方法建立在synchronized线程同步的基础之上。

 [点击下载「生产者和消费者模式」的学习代码](https://github.com/Clover-You/my-blog-file/blob/main/%E7%94%9F%E4%BA%A7%E8%80%85%E5%92%8C%E6%B6%88%E8%B4%B9%E8%80%85%E6%A8%A1%E5%BC%8F%E5%AD%A6%E4%B9%A0%E4%BB%A3%E7%A0%81.zip)

