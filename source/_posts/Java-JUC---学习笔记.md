title: Java JUC - 学习笔记
categories: 其它
tags: [JAVA,学习笔记]
date: 2022-01-23 23:55:00
---
# JUC

## 什么是 JUC

在 JAVA 中，线程部分是一个重点，JUC 是 `java.util.concurrent` 工具包的简称，他是一个处理线程的工具包（JDK1.5推出）

## 进程和线程的概念

### 进程和线程

进程（Process）是计算机中的程序关于某数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位，是操作系统结构的基础。在当代面向线程设计的计算机结构中，进程是线程的容器，程序是指令、数据以其组织形式的描述，进程是程序的实体。

线程（Thread）是操作系统能够进行运算调度的最小单位，它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程可以并行执行不同的任务。

> - 进程：指在系统中正在运行的一个应用程序，程序一旦运行那么它就是一个进程。它是资源分配的最小单位。
> - 线程：系统分配处理器时间资源的基本单元，或者说进程之内独立执行的一个单元执行流。它是程序执行的最小单位。轻量级的进程依附于某一个进程上，共享这个进程所攫取到的内存资源

### 线程的状态

新建状态 ==> 准备状态 ==> 运行状态 ==> 阻塞状态 ==> 死亡状态

### wait 和 sleep

1. sleep 是 Thread 的静态方法，wait 是 Object 的方法，任何对象实例都能够调用
2. sleep 不会释放锁，它也不需要占用锁。wait 会释放锁，但调用它的前提是当前线程占有锁（即代码需要在 `synchronized` 中）
3. 他们都可以被 `interrupted` 方法中断

### 并发和并行

#### 串行模式

串行表示所有任务都按先后顺序进行。

>  串行意味着必须先装完一车货才能运送这车货，只有送到了，才能卸下这车货，并且只有完成了这整个步骤，才能再次进行。「串行是一次只能领取一个任务，并执行这个任务。」

#### 并行模式

并行意味着可以同时取得多个任务，并同时去执行所取得的这些任务。并行模式相当于一条长长的队列被划分成了多条短的队列，所以并行缩短了任务队列的长度，并行的效率从代码层次上强依赖于多进程/多线程代码，从硬件的角度上则依赖多核CPU

#### 并发 

并发（concurrent）指的是多个进程可以同时运行的现象，更细化的是多进程可以同时运行或者多指令可以同时运行。并发是一种现象，它描述的是多线程同时运行的现象。但实际上，对于单核CPU来说，同一时刻只能运行一个线程。所以，这里的**“同时运行”**表示的不是真的可以同一时刻有多个线程运行，这是并行的概念，而是提供一种功能让用户看起来可以多个线程同时运行起来，但实际上这些程序中的进程不是一直霸占CPU的，而是执行一会停一会。

要解决大并发问题，通常是将一个大任务分解成多个小任务。由于操作系统对进程的调度是随机的，所以切分成多个小任务后，会任意执行一个小任务。这会出现一些现象：

- 可能出现一个任务执行了多次，还没开始下个任务的情况。这种情况一般会采用队列或类似的数据结构来存放各个任务的成果。
- 可能出现还没准备好第一步就开始执行第二步的问题。这种情况一般采用多路复用或异步的方式，比如只有准备好生产了事件通知才执行某个任务。
- 可以多进程/多线程的方式并行执行这些小任务，也可以单进程/单线程执行这些小任务，这时可能要配合多路复用才能达到较高的效率。



> 并发：线程频繁切换，给人一种同时执行的错觉
>
> 并行：多个线程可以同时运行

### 管程

监视器（Monitor），我们常见的“锁”其实就是监视器，他是一种同步机制，能够保证在同一时间只有一个线程能够访问被保护的数据或代码。管程对象会对临界区进行加锁，在进入的时候进行加锁，在退出时解锁。执行线程操作时，首先要持有管程对象后才能执行方法。当在方法执行过程中，因为当前线程持有这个管程对象，所以别的线程就不能获取同一个管程对象。当你方法执行完成之后，最终才释放拿到的管程对象，别的线程才能再拿到这个管程对象

### 用户线程和守护线程

- 在平时使用线程的时候， 使用的基本都是用户线程。以下结果为：`false`

  ```java
  public static void main(String[] args) {
      Thread t = new Thread(() -> {
        String tn = Thread.currentThread().getName();
        boolean isDaemon = Thread.currentThread().isDaemon();
        System.out.println("当前线程的名称：" + tn + ":是守护线程:" + isDaemon);
      });
      t.run();
    }
  ```

  > 主线程结束后，如果用户线程还在执行，那么jvm还存活。

- 守护线程是用在后台中的一种特殊的线程（垃圾回收）

  > 主线程结束后，如果没有用户线程，那么jvm结束

## Lock 接口

已实现的类：

- `java.util.concurrent.locks.ReentrantLock` 可重入锁
- `java.util.concurrent.locks.ReentrantReadWriteLock.ReadLock` 可读锁
- `java.util.concurrent.locks.ReentrantReadWriteLock.WriteLock` 可写锁

Lock 实现提供了比使用 `synchronized` 方法和语句可获得的更广泛的锁定操作。此实现允许更灵活的结构，可以具有差别很大的属性，可以支持多个相关的 **`Condition`** 对象

- Lock 不是 Java 语言内置的，synchronized 是 Java 语言的关键字，因此是内置特性。Lock 是一个类，通过这个类可以实现同步访问。
- Locke 和 synchronized 有一点非常大的不同，采用 synchronized 不需要用户手动去释放锁，当 synchronized 方法或代码块执行完成之后，系统会自动让线程释放对锁的占用。而 Lock 则必须要手动去释放锁，如果没有主动释放锁，那么就有可能出现死锁现象。

> 使用 Lock 锁会话费更少的时间来调度线程，性能更好并具有更好的拓展性，所以我们应优先考虑使用 Lock 锁

### Synchronized

`synchronized` 是 Java 中的关键字，是一种同步锁，它修饰的对象有以下几种：

1. 修饰一个代码块，被修饰的代码块称为同步代码块，其作用的范围是大括号括起来的代码，作用的对象是调用这个代码块的对象

   ```java
   synchronized(this) {
     // 同步代码块
   }
   ```

2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象。

   ```java
   // 同步方法
   synchronized public void test() {
       // 同步代码块
   }
   ```

3. 修饰一个静态方法，其作用的范围是整个静态方法

   ```java
   // 同步方法
   synchronized public static void test() {
     // 同步代码块
   }
   ```

4. 修饰一个类，其作用范围是被修饰的这整个类，也就是说被修饰的类中，所有的方法都是同步的。

### 可重入锁 ReentrantLock

> 什么是 “可重入”，可重入就是说某个线程已经获得某个锁，可以再次获取锁而不会出现死锁，其他线程失效。

```java
try {
  rl.lock();
  //...
} finally {
  rl.unlock();
}
```



> `new Thread().start()` 执行后并不是立刻启动线程，而是进入就绪状态等待操作系统/CPU的底层调度通知。



## 生产者和消费者模式

现在两个线程，可以操作初始为零的一个变量， 实现一个线程对该变量加1，一个线程对该变量减1，实现交替，来10轮，变量初始值为零。

思路：

- 加法：判断一个变量是否为0，如果该变量为0，那么让该变量加1，如果为1，那么当前线程无期限睡眠，等待消费线程通知。
- 减法：判断一个变量是否为1，如果该变量为1，那么让该变量减1后通知生产线程，如果为0，那么当前线程无期限睡眠，等待生产线程通知。

> 这需要使用 `wait` 睡眠，因为它会放弃当前线程占有的锁。

```java
public class ThreadWaitNotify {
    public static void main(String[] args) {
        AirConditioner airConditioner = new AirConditioner();
      // 消费线程
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    airConditioner.decrement();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "A").start();
      // 生产线程
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    airConditioner.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();
    }
}

class AirConditioner {

    private int number = 0;

    public synchronized void increment() throws InterruptedException {
        if (number != 0) {
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + "::" + number);
        // 生产好了，通知消费
        this.notifyAll();
    }

    public synchronized void decrement() throws InterruptedException {
        if (number != 1) {
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + "::" + number);
        // 消费完了，通知生产
        this.notifyAll();
    }

}
```

### 防止虚假唤醒

在以上程序中存在虚假唤醒的严重问题，如果存在多个生产或消费线程，例如：A、B是生产者线程C、D是消费者线程，当A拿到执行权时，此时发现需要生产，进行生产后唤醒所有线程进行消费，这些线程中包括了一个此时不该唤醒的生产者线程B。而此时并没有人进行消费，所以B在执行了 `wait()` 后沉睡（因为它需要等待消费），并**放弃了已占有的锁**，此时一个消费者拿到了执行权进行消费后再唤醒所有线程，让其进行生产，而此时拿到执行权的又是A，A进行生产后唤醒所有线程。此时被B拿到了执行权，B又**继续上一次** `wait()` 沉睡的位置往下执行，对正确结果进行加1，因为 `if` 执行之后就往下执行了，这就造成了虚假唤醒的问题。

如果是使用 `while` ，当B又**继续上一次** `wait()` 沉睡的位置往下执行时，`while` **会重新判断当前结果是否为 `true`** （因为循环体执行完后会再次判断下一次是否需要循环），如果是 `true` 那么继续执行循环体中的内容，如果不正确才往下执行。

所以使用 `while` 改造一下判断即可

```java
class AirConditioner {

    private int number = 0;

    public synchronized void increment() throws InterruptedException {
        if (number != 0) {
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + "::" + number);
        // 生产好了，通知消费
        this.notifyAll();
    }

    public synchronized void decrement() throws InterruptedException {
        if (number != 1) {
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + "::" + number);
        // 消费完了，通知生产
        this.notifyAll();
    }

}
```

### ReentrantLock 实现

逻辑不变，只是加锁方式变了，使用 `Lock` 中的可重入锁进行实现。

```java
public class ThreadWaitNotify {
    public static void main(String[] args) {
        AirConditioner airConditioner = new AirConditioner();
        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    airConditioner.decrement();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "A").start();

        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    airConditioner.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "B").start();

        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    airConditioner.decrement();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "C").start();

        new Thread(() -> {
            try {
                for (int i = 0; i < 10; i++) {
                    airConditioner.increment();
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "D").start();
    }
}

class AirConditioner {

    private int number = 0;

    private final Lock lock = new ReentrantLock();

    private final Condition condition = lock.newCondition();

    public void increment() throws InterruptedException {
        lock.lock();
       try {
           while (number != 0) {
               condition.await();
           }
           number++;
           System.out.println(Thread.currentThread().getName() + "::" + number);
           // 生产好了，通知消费
           condition.signalAll();
       } finally {
           lock.unlock();
       }
    }

    public void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number != 1) {
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName() + "::" + number);
            // 消费完了，通知生产
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

}
```

> \*在多线程交互中，要防止多线程的虚假唤醒。

## List不安全

单线程执行以下代码，结果正常

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            String str = UUID.randomUUID().toString().substring(0, 8);
            list.add(str);
            System.out.println(list);
        }
    }
}
```

输出

```
[673be649]
[673be649, 87b8596a]
[673be649, 87b8596a, 97b5d7cf]
```

现在使用多线程对这个 List 操作

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                list.add(str);
                System.out.println(list);
            }).start();
        }
    }
}
```

输出发现数据错乱了，如果线程数过多，甚至还发生 `ConcurrenModificationException` 异常。

```
[null, 9f999a5c, 2e938025]
[null, 9f999a5c, 2e938025]
[null, 9f999a5c, 2e938025]
```

这是因为 `ArrayList` 是线程不安全的。而导致的原因是，有一个线程在写的时候，还没写完就有一个线程突然把对象抢走修改掉了，此时这个 `ArrayList` 可能就坏了。相当于你拿笔在纸上写，还没写完突然有人把纸抢走，那你的笔是不是可能会把纸**划**坏？

### 解决方案

#### Vector

要解决这个问题除了加锁，还可以使用 `Vector` 集合来代替 `ArrayList`

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        List<String> list = new Vector<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                list.add(str);
                System.out.println(list);
            }).start();
        }
    }
}
```

输出

```
[95374cd5]
[95374cd5, 2b7d7c02, a0e85ef2]
[95374cd5, 2b7d7c02]
```

但是通过锁的方式来处理，性能会比较低，如果在乎稳定性不太在乎性能，那么可以选择 `Vector`  ，因为他是线程安全的



#### Collections 工具类

`Collections` 工具类中，又一个静态方法`synchronizedList()` 可以将一个线程不安全的 `List` 转为线程安全的。

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        List<String> list = Collections.synchronizedList(new ArrayList<>());
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                list.add(str);
                System.out.println(list);
            }).start();
        }
    }
}
```

输出

```
[1bdf1ac9, 39e8acc8, cdbd2100]
[1bdf1ac9, 39e8acc8, cdbd2100]
[1bdf1ac9, 39e8acc8, cdbd2100]
```

`Collections` 不仅有可以将线程不安全的 List 转为线程安全的 `List` ，他还提供相应的方法 `synchronizedSet()、synchronizedMap`将线程不安全的 `Set、Map` 转为线程安全的。

#### JUC CopyOnWriteArrayList

既要保证写时数据的一致性，也要保证读的时候多个人来读，那么就可以使用 JUC 提供的 `CopyOnWriteArrayList` 

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        List<String> list = new CopyOnWriteArrayList<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                list.add(str);
                System.out.println(list);
            }).start();
        }
    }
}
```

输出

```
[f15d5d7c, ecdf8157, ff947b0d]
[f15d5d7c, ecdf8157, ff947b0d]
[f15d5d7c, ecdf8157, ff947b0d]
```

他的底层是使用的 `ReentrantLock` 锁实现（jdk1.8）的， `CopyOnWriteArrayList` 即写时复制的容器，也是读写分离思想的一种变种。在往一个容器添加元素的时候，不直接往当前容器 `Object[]` 中添加，而是先将当前容器 `Object[]` 进行拷贝，复制出一个新的容器 `Object[] newElements` ，然后往新的容器中添加元素，添加完后再将原容器的引用指向新的容器 `setArray(newElements)`。这样做对的好处是可以对 `CopyOnWrite` 容器进行并发读而不需要加锁，因为当前容器不会添加任何元素，所以 CopyOnWrite 容器也是一种**读写分离**的思想。

```java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
  final ReentrantLock lock = this.lock;
  lock.lock();
  try {
    Object[] elements = getArray();
    int len = elements.length;
    Object[] newElements = Arrays.copyOf(elements, len + 1);
    newElements[len] = e;
    setArray(newElements);
    return true;
  } finally {
    lock.unlock();
  }
}
```

## Set不安全

`HashSet` 和 `ArrayList` 一样都是线程不安全的，在多线程操作下可能会出现数据不一致甚至抛出 `ConcurrentModificationException`异常

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        Set<String> list = new HashSet<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                list.add(str);
                System.out.println(list);
            }).start();
        }
    }
}
```

输出

```
[8320c0e3, 668c6a05]
[8320c0e3, 668c6a05]
[8320c0e3, 668c6a05]
```

### 解决方案

#### Collections 工具类

可以使用 `Collections` 提供的 `synchronizedSet()` 将一个线程不安全的 `Set` 转为线程安全的

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        Set<String> list = Collections.synchronizedSet(new HashSet<>());
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                list.add(str);
                System.out.println(list);
            }).start();
        }
    }
}
```

输出

```
[3d8c21f0, 6280c2c9, 2912b50d]
[3d8c21f0, 6280c2c9, 2912b50d]
[3d8c21f0, 6280c2c9, 2912b50d]
```

#### CopyOnWriteArraySet

也可以使用 JUC 提供的 `CopyOnWriteArraySet` ，他的实现方式与 `CopOnWriteArrayList` 一样

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        Set<String> list = new CopyOnWriteArraySet<>();
        for (int i = 0; i < 300; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                list.add(str);
                System.out.println(list);
            }).start();
        }
    }
}
```

## Map不安全

`HashMap` 是线程不安全的，在多线程下同样会发生数据撕裂和 `ConcurrentModificationException` 异常的问题

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        Map<String, String> map = new HashMap<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                map.put(str, str);
                System.out.println(map);
            }).start();
        }
    }
}
```

输出

```
{38cf87b2=38cf87b2, f4765eb9=f4765eb9}
{38cf87b2=38cf87b2, 764f0313=764f0313, f4765eb9=f4765eb9}
{38cf87b2=38cf87b2, f4765eb9=f4765eb9}
```

### 解决方案

#### Collections 工具类

同样可以使用 `Collections` 提供的 `synchronizedMap` 将一个线程不安全的 Map 转为线程安全的。

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        Map<String, String> map = Collections.synchronizedMap(new HashMap<>());
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                map.put(str, str);
                System.out.println(map);
            }).start();
        }
    }
}
```

输出

```
{b11cc6c6=b11cc6c6, 69e2856c=69e2856c, 72771c76=72771c76}
{b11cc6c6=b11cc6c6, 69e2856c=69e2856c, 72771c76=72771c76}
{b11cc6c6=b11cc6c6, 69e2856c=69e2856c, 72771c76=72771c76}
```

#### ConcurrentHashMap

使用JUC提供的 `ConcurrentHashMap`

```java
public class NotSafeDemo {
    public static void main(String[] args) {
        Map<Object, Object> map = new ConcurrentHashMap<>();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> {
                String str = UUID.randomUUID().toString().substring(0, 8);
                map.put(str, str);
                System.out.println(map);
            }).start();
        }
    }
}
```

输出

```
{2e96c45b=2e96c45b, 8e1cf37c=8e1cf37c, 8509b40b=8509b40b}
{2e96c45b=2e96c45b, 8e1cf37c=8e1cf37c, 8509b40b=8509b40b}
{2e96c45b=2e96c45b, 8e1cf37c=8e1cf37c, 8509b40b=8509b40b}
```

## Callable

对比老的创建线程的方式，这种方式创建的线程可以拿到线程的执行结果。`Callable` 提供了一个 `get()` 方法，这个方法会阻塞当前线程，知道目标线程完成并返回结果为止。所以 `get()` 方法一般放在最后一行

```java
public class CallableThread {
    public static void main(String[] args) throws Exception {
        MyCallableThread myCallableThread = new MyCallableThread();
        FutureTask task = new FutureTask(myCallableThread);
        new Thread(task).start();
        System.out.println(task.get());
        System.out.println("结束");
    }
}

class MyCallableThread implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("come in here..");
      // 线程睡3秒
        TimeUnit.SECONDS.sleep(3);
        return "Hello world";
    }
}
come in here..
Hello world
结束
```

如果有多个线程调用同一个 `FutureTask` 对象，那么只会有一个线程调用 `call`，因为这个方法的返回结果是一样的，等执行完后，其他线程直接拿到结果缓存就行

```java
public class CallableThread {
    public static void main(String[] args) throws Exception {
        MyCallableThread myCallableThread = new MyCallableThread();
        FutureTask task = new FutureTask(myCallableThread);
        new Thread(task, "A").start();
        new Thread(task, "B").start();
        System.out.println(task.get());
        System.out.println("结束");
    }
}


class MyCallableThread implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println(Thread.currentThread().getName() + " come in here..");
        TimeUnit.SECONDS.sleep(3);
        return "Hello world";
    }
}
```

输出

```
A come in here..
Hello world
结束
```

## CountDownLatch

`CountDownLatch` 是一个闭锁，它主要有两个方法，当一个或多个线程调用 `await` 方法时，这些线程会阻塞。其他线程调用 `countDown` 方法会将计数器减1(调用 countDown 方法的线程不会阻塞)，当计数器的值变为0时，被 `await` 阻塞的线程会被自动唤醒，继续执行。

```java
public class CountDownlatchLockTest {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(5);
        for (int i = 0; i < 5; i++) {
            final int y = i;
            new Thread(() -> {
                System.out.println(y);
                countDownLatch.countDown();
            }).start();
        }
        countDownLatch.await();
        System.out.println("结束...");
    }
}

```

输出

```
0
2
1
4
3
结束...
```

## CyclicBarrier

CyclicNarrier 从字面上看，叫做循环栅栏/循环屏障，它与`CountDownLatch` 相同，只不过 `CountDownLatch` 是将指定数值减到0结束，而 `CyclicBarrier` 是从0开始计数到指定数值出发，计数时阻塞所有调用 `await` 方法的线程。当计数完成后如果有，那么开启指定线程

```java
public class CyclicBarrierTest {
    public static void main(String[] args) throws BrokenBarrierException, InterruptedException {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(5, () -> {
            System.out.println("计数完成");
        });
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                try {
                    System.out.println("准备计数");
                    cyclicBarrier.await();
                    System.out.println("计数减一");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }).start();
        }
    }
} 
```

输出

```
准备计数
准备计数
准备计数
准备计数
准备计数
计数完成
计数减一
计数减一
计数减一
计数减一
计数减一
```

## Semaphore

Semaphore通常称之为信号量或信号灯，他可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。

其实就是指定一个值，如果有线程调用 `acquire` 方法，那么该值减1，如果该值为0，那么其它线程将无法获取到信号量，需要等到有线程释放信号量才继续执行。

```java
public class SemaphoreTest {
    public static void main(String[] args) throws InterruptedException {
        Semaphore semaphore = new Semaphore(3);
        semaphore.tryAcquire();
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                String name = Thread.currentThread().getName();
                try {
                    semaphore.acquire();
                    System.out.println(name + "抢到了车位");
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println(name+"离开了车位");
                    semaphore.release();
                }

            }, i + "").start();
        }
    }
}
```

输出

```
0抢到了车位
1抢到了车位
1离开了车位
0离开了车位
2抢到了车位
3抢到了车位
2离开了车位
3离开了车位
4抢到了车位
4离开了车位
```

- `acquire` 当一个线程调用时，它要么通过成功获取信号量，要么一直等下去，直到有线程释放信号量或超时。
- `release` 释放一个信号量，实际上是将信号量的值加1，然后唤醒等待的线程。
- `tryAquire` 他的作用与 `acquire` 一样，只不过没获取到信号量时不会等待其他线程释放，而是直接返回一个 `false`，如果拿到信号量，那么返回 `true`。

## ReadWriteLock

读写锁（ReadWriteLock），多个 线程同时读一个资源类时没有问题，所以为了满足并发量，读取共享资源应该可以同时运行。但是如果有一个线程想去写共享资源，就不应该再有其他线程对该资源进行读或写。

有些操作是需要保证原子性的，如果不能保证原子性，那么会造成数据错乱的问题。

```java
class MyCache {
  private volatile Map<String, Object> map = new HashMap<>();

  public void put(String key, Object val) {
    String name = Thread.currentThread().getName();
    System.out.println(name + "\t 写入数据" + key);
    try {
      TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    map.put(key, val);
    System.out.println(name + "\t 写入完成");
  }

  public Object get(String key) {
    String name = Thread.currentThread().getName();
    System.out.println(name + "\t 读取数据" + key);
    Object o = map.get(key);
    System.out.println(name + "\t 读取完成" + o);
    return o;
  }
}
public class ReadWriteLockTest {
  public static void main(String[] args) {
    MyCache myCache = new MyCache();
    for (int i = 0; i < 5; i++) {
      final int tempInt = i;
      new Thread(() -> {
        myCache.put(tempInt + "", tempInt);
      },"A").start();
    }

    for (int i = 0; i < 5; i++) {
      final int tempInt = i;
      new Thread(() -> {
        myCache.get(tempInt + "");
      },"B").start();
    }
  }
}
```

输出

```
A    写入数据0
A    写入数据2
A    写入数据1
A    写入数据3
A    写入数据4
B    读取数据0
B    读取完成null
B    读取数据2
B    读取完成null
B    读取数据1
B    读取完成null
B    读取数据4
B    读取完成null
B    读取数据3
B    读取完成null
A    写入完成
A    写入完成
A    写入完成
A    写入完成
A    写入完成
```

要保证原子操作，并且需要高性能，可以使用读写锁来操作

```java
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();

    private final ReadWriteLock rwl = new ReentrantReadWriteLock();

    public void put(String key, Object val) {
        Lock lock = rwl.writeLock();
        lock.lock();
       try {
           String name = Thread.currentThread().getName();
           System.out.println(name + "\t 写入数据" + key);
           try {
               TimeUnit.SECONDS.sleep(3);
           } catch (InterruptedException e) {
               e.printStackTrace();
           }
           map.put(key, val);
           System.out.println(name + "\t 写入完成");
       } finally {
           lock.unlock();
       }
    }

    public Object get(String key) {
        Lock lock = rwl.readLock();
        lock.lock();
        try {
            String name = Thread.currentThread().getName();
            System.out.println(name + "\t 读取数据" + key);
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Object o = map.get(key);
            System.out.println(name + "\t 读取完成" + o);
            return o;
        } finally {
            lock.unlock();
        }
    }
}
public class ReadWriteLockTest {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        for (int i = 0; i < 5; i++) {
            final int tempInt = i;
            new Thread(() -> {
                myCache.put(tempInt + "", tempInt);
            },"A").start();
        }

        for (int i = 0; i < 5; i++) {
            final int tempInt = i;
            new Thread(() -> {
                myCache.get(tempInt + "");
            },"B").start();
        }
    }
}
A    写入数据0
A    写入完成
A    写入数据1
A    写入完成
A    写入数据2
A    写入完成
A    写入数据3
A    写入完成
A    写入数据4
A    写入完成
B    读取数据0
B    读取数据1
B    读取数据2
B    读取数据3
B    读取数据4
B    读取完成2
B    读取完成4
B    读取完成3
B    读取完成0
B    读取完成1
```

以上的例子可能看不出效果，看下图，当有写锁时，后面的无论是读锁还是写锁，都得等着，如果是写，那么相当于无所状态。

![](https://qiniu-note-image.ctong.top/note/images/202201290010378.gif)

## BlockingQueue

当队列是空的，从队列中获取元素的操作将会被阻塞，当队列是满的，从队列中添加元素的操作将会被阻塞。

试图从空的队列中获取元素的线程将会被阻塞，直到其他线程往空的队列插入新的元素。

试图向已满的队列中添加新元素的线程将会被阻塞，直到其他线程从队列中移除一个或多个元素或者完全清空，使队列变得空闲起来并后续新增。

在多线程领域，所谓的阻塞，在某些情况下会挂起线程（即堵塞），一旦条件满足，被挂起的线程又会自动被唤醒。而 `BlockingQueue` 的好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切 `BlockingQueue` 都给你做好了，在 concurrent 包发布以前，在多线程环境下，我们每个人都必须去自己控制这些细节，尤其还需要兼顾效率和线程安全，而这会给我们的程序带来不小的复杂度。

BlockingQueue的实现类有：

- `ArrayBlockingQueue` 他是由数组结构组成的有界阻塞队列
- `LinkedBlockingQueue` 由链表结构组成的有界（但大小默认值为`integer.MAX_VALUE`）阻塞队列
- `PriorityBlockingQueue` 支持优先级排序的无界阻塞队列
- `DelayQueue` 是有优先级队列实现的延迟无界阻塞队列
- `SynchronousQueue` 不存储元素的阻塞队列，也即单个元素的队列
- `LinkedTransferQueue` 由链表组成的无界阻塞队列
- `LinkedBlockingDeque` 由链表组成的双向阻塞队列

![Collection](https://qiniu-note-image.ctong.top/note/images/202201291749094.png)

### 方法

- 插入
  1. `add(e)` 插入一个值到队列中，如果队列已满，那么抛出异常
  2. `offer(e)` 插入一个值到队列中，如果成功返回 `true`，如果队列已满则返回 `false` 
  3. `put(e)` 插入一个值到队列中，如果队列已满，那么阻塞当前线程，直到队列空出位置后将值插入。
  4. `offer(e, time, unit)`插入一个值到队列中，如果队列已满，则阻塞当前线程一定时间再插入，如果成功返回 `true` ，失败则返回 `false`，可被 `interrupted()` 方法中断。
- 移除
  1. `remove()` 移除队列中的第一个值，如果队列为空那么异常
  2. `poll()` 移除队列中第一个值，成功则返回当前被移除的元素，如果队列没有元素那么返回 `null`
  3. `take()` 移除队列中的第一个值，如果队列为空，那么阻塞当前线程直到队列插入数据。
  4. `poll(time, unit)` 移除队列中的第一个值，如果队列为空则阻塞当前线程一定时间后再移除，成功则返回当前被移除的元素，失败返回 `null`。阻塞过程中可被 `interrupted` 方法打断
- 检查
  1. `element()` 查看队列中的值，如果队列没有值，那么抛出异常
  2. `peek` 查看队列中的值，如果队列为空则返回`null`，特殊值



- 抛出异常：
  当阻塞队列满时，如果再往队列中 `add` 插入元素就会抛出 `IllegalStateException` 异常 。
  当阻塞队列为空时，如果再 `remove` 移除元素会抛出 `NoSuchElementException`
- 特殊值：
  插入方法，成功为 `true` 失败 `false`。
  移除方法，成功返回出队列的元素，队列里没有就返回 null
- 阻塞：
  当阻塞队列满时，生产者线程继续往队列里 `put` 元素，队列会一直阻塞生产者线程直到 `put` 数据 or 响音中断退出。
  当阻塞队列为空时，消费者线程试图从队列里 `take` 元素，队列会一直阻塞消费者线程，直到队列可用。
- 超时退出：
  当阻塞队列满时，队列会阻塞生产者线程一定时间，超时限时后生产者线程会退出

> 队列先入先出

## 线程池

线程池做的工作是只要控制运行的线程数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量，超出数量的线程排队等候，等其他线程执行完毕，再从队列中取出任务来执行。

他的主要特点为：线程复用，控制最大并发数，管理线程。

Java 中的线程池是通过 `Executor` 框架实现的，该框架中用到了`Executor`，`Executors`，`ExecutorService`，`ThreadPoolExecutor` 这几个类。

![MyThreadPoolDemo](https://qiniu-note-image.ctong.top/note/images/202201292219375.png)

### Executors 工具类

线程池一般都使用 `Executors` 工具类来进行创建

```java
public class MyThreadPoolDemo {
    public static void main(String[] args) {
         // 最大5个线程
        ExecutorService executorService = Executors.newFixedThreadPool(5);
    }
}
```

- `Executors.newFixedThreadPool(int)` 执行长期任务性能好，创建一个线程池，一个线程池有多个固定的线程。
- `Executors.newSingleThreadExecutor()` 创建一个只有一个线程的线程池
- `Executors.newCachedThreadPool()` 执行很多短期异步任务，线程池根据需要创建新的线程，但在先前构建的线程可用时将重用他们，如果没有线程可用，则创建新线程。

### ForkJoinPool

采用分治算法将一个大任务分成诺干个小任务，之后再将这些若干个小任务并行计算，最终再汇总这些任务的结果。

![分治算法](https://qiniu-note-image.ctong.top/note/images/202201310429115.png)

下列例子：给定一个区间进行计算，如果计算量大于临界值，那么采用分治算法将任务切分成若干个小任务分配给其它线程并行计算。

```java
class ForkJoinDemoMyTask extends RecursiveTask<Integer> {
    private static final int ADJUST_VALUE = 10;

    private int begin;

    private int end;

    private int result;

    public ForkJoinDemoMyTask(int begin, int end) {
        this.begin = begin;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        if ((end - begin) > ADJUST_VALUE) {
            for (int i = begin; i < end; i++) {
                result = result + i;
            }
        } else {
            int middle = (end + begin) / 2;
            ForkJoinDemoMyTask myTask1 = new ForkJoinDemoMyTask(begin, middle);
            ForkJoinDemoMyTask myTask2 = new ForkJoinDemoMyTask(middle + 1, end);
            ForkJoinTask<Integer> fork = myTask1.fork();
            ForkJoinTask<Integer> fork1 = myTask2.fork();
            result = fork.join() + fork1.join();
        }
        return result;
    }
}

public class ForkJoinDemo {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinDemoMyTask myTask = new ForkJoinDemoMyTask(0, 100);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinTask<Integer> submit = forkJoinPool.submit(myTask);
        System.out.println(submit.get());

        forkJoinPool.shutdown();
    }
}
```

```
4950
```



###   ThreadPoolExecutor

使用 `Executors` 工具类初始化一个具有固定线程的线程池

```java
public class MyThreadPoolDemo {
    public static void main(String[] args) {
         // 最大5个线程
        ExecutorService executorService = Executors.newFixedThreadPool(5);
    }
}
```

使用 `execute` 方法将执行任务分配给线程池中的线程

```java
public class MyThreadPoolDemo {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        try {
            for (int i = 0; i < 10; i++) {
                executorService.execute(() -> {
                    System.out.println(Thread.currentThread().getName());
                });
            }
        } finally {
            executorService.shutdown();
        }
    }
}
```

如果使用 `shutdown` 方法关闭线程池后，再次调用将会抛出 `RejectedExecutionException` 异常

#### 线程池的7大参数

1. `corePoolSize`: 线程池中的常驻核心线程数

2. `maxinumPoolSize`: 线程池中能够容纳同时执行的最大线程数，此值必须大于等于1
3. `keepAliveTime`: 多余的空闲线程存活时间当前池中线程数量超过`corePoolSize` 时，当前空闲时间达到 `keepAliveTime` 时，多余线程会被销毁直到剩下 `corePoolSize` 个线程为止
4. `unit`：`keepAliveTime` 的单位
5. `workQueue` 任务队列，被提交但尚未被执行的任务
6. `threadFactory`：表示生成线程池中工作线程的线程工厂，用于创建线程
7. `handler`：拒绝策略，表示当前队列满了，并且工作线程大于等于线程池的最大线程数（`maxinumPoolSize`）时如何拒绝请求执行的 runnable 的策略



### 线程池原理

1. 在创建了 线程后，开始等待请求
2. 当调用`executor()` 方法添加一个请求任务时，线程池会做出相应的判断：
   1. 如果正在运行的线程数小于 `corePoolSize` ，那么马上创建线程来运行这个任务。
   2. 如果正在运行的线程数量大于或等于 `corePoolSize` ，那么将这个任务放入队列。
   3. 如果这个时候队列满了且正在运行的线程数量还小于 `maximumPoolSize` 那么还是要创建非核心线程立刻运行这个任务。
   4. 如果队列满了且正在运行的线程数量大于或等于 `maximumPoolSize` ，那么线程池会启动饱和拒绝策略来执行。
3. 当一个线程完成任务时，他会从队列中取下一个任务来执行。
4. 当一个线程无事可做超过一定时间（`keepAliveTime`）时，线程会判断：如果当前运行的线程数大于 `corePoolSize` ，那么这个线程就会被停掉。所以线程池的所有任务完成后，它最终会收缩到 `corePoolSize` 的大小。



> 在实际开发中，严禁使用 `Executors` 来创建线程池，因为 `Executors` 中创建的`newFixedThreadPool(int)` 、`newCachedThreadPool` 中的 `maximumPoolSize` 参数为 `Integer.MAX_VALUE` ，该参数会导致 OOM 风险。

### 线程池的拒绝策略

在线程池中的等待队列已经满了，并且线程池中的最大线程也已经达到了，无法继续为新的任务提供服务的时候就需要拒绝策略机制合理的处理这个问题。所有内置策略都实现了 `RejectedExecutionHandle` 接口。

#### AbortPolicy

该策略是一个默认策略，在线程池无法处理任务时，将抛出一个 `RejectedExecutioException` 异常。

```java
ExecutorService executorService = new ThreadPoolExecutor(
        2,
        5,
        10L,
        TimeUnit.SECONDS,
        new LinkedBlockingQueue<>(3),
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.AbortPolicy()
);
try {
    for (int i = 0; i < 9; i++) {
        executorService.execute(() -> {
            try {
                TimeUnit.SECONDS.sleep(3l);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName());
        });
    }
}catch (Exception e) {
    System.out.println(e);
}finally {
    executorService.shutdown();
}
```

```
java.util.concurrent.RejectedExecutionException: Task top.ctong.lock.MyThreadPoolDemo$$Lambda$1/381259350@3feba861 rejected from java.util.concurrent.ThreadPoolExecutor@5b480cf9[Running, pool size = 5, active threads = 5, queued tasks = 3, completed tasks = 0]
pool-1-thread-2
pool-1-thread-3
pool-1-thread-1
pool-1-thread-4
pool-1-thread-5
pool-1-thread-2
pool-1-thread-3
pool-1-thread-1
```

#### CallerRunsPolicy

“调用者运行”是一种调节机制，该策略即不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量。

也就是哪个线程调用的线程池就把任务交给那个调用者去运行。

```java
ExecutorService executorService = new ThreadPoolExecutor(
        2,
        5,
        10L,
        TimeUnit.SECONDS,
        new LinkedBlockingQueue<>(3),
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.CallerRunsPolicy()
);
try {
    for (int i = 0; i < 10; i++) {
        executorService.execute(() -> {
            try {
                TimeUnit.SECONDS.sleep(3l);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName());
        });
    }
}catch (Exception e) {
    System.out.println(e);
}finally {
    executorService.shutdown();
}
```

```
main
pool-1-thread-4
pool-1-thread-2
pool-1-thread-5
pool-1-thread-3
pool-1-thread-1
main
pool-1-thread-5
pool-1-thread-2
pool-1-thread-4
```

#### DiscardPolicy

该策略会丢弃无法处理的任务，不予任何处理也不抛出异常，如果允许任务丢失，这是或许最好的一种策略

```java
ExecutorService executorService = new ThreadPoolExecutor(
        2,
        5,
        10L,
        TimeUnit.SECONDS,
        new LinkedBlockingQueue<>(3),
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.DiscardPolicy()
);
try {
    for (int i = 0; i < 10; i++) {
        executorService.execute(() -> {
            try {
                TimeUnit.SECONDS.sleep(3l);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName());
        });
    }
}catch (Exception e) {
    System.out.println(e);
}finally {
    executorService.shutdown();
}
```

```
pool-1-thread-4
pool-1-thread-3
pool-1-thread-5
pool-1-thread-2
pool-1-thread-1
pool-1-thread-3
pool-1-thread-4
pool-1-thread-5
```



#### DiscardOldestPolicy

抛弃队列中等待最久的任务，然后把当前任务加入队列中尝试再次提交当前任务。

```java
ExecutorService executorService = new ThreadPoolExecutor(
        2,
        5,
        10L,
        TimeUnit.SECONDS,
        new LinkedBlockingQueue<>(3),
        Executors.defaultThreadFactory(),
        new ThreadPoolExecutor.DiscardOldestPolicy()
);
try {
    for (int i = 0; i < 10; i++) {
        int y = i;
        executorService.execute(() -> {
            try {
                TimeUnit.SECONDS.sleep(3l);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "::" + y);
        });
    }
}catch (Exception e) {
    System.out.println(e);
}finally {
    executorService.shutdown();
}
```

````
pool-1-thread-4::6
pool-1-thread-2::1
pool-1-thread-3::5
pool-1-thread-1::0
pool-1-thread-5::7
pool-1-thread-2::8
pool-1-thread-3::9
pool-1-thread-4::4
````

#### 优化

- 如果业务是要求CPU密集型就以CPU核心数多1到2，也即（CPU核数 + 1）

- IO密集型使用CPU核数除阻塞系数。
  - 阻塞系数：线程花在系统IO上的时间与CPU密集任务所耗的时间比值

## 异步回调

`CompletableFuture` 结合 `Future` 类的优点，提供了非常强大的拓展功能，可以帮我们简化异步编程的复杂性，提供了函数式编程的能力，可以通过回调的方式处理计算结果， 并提供了转换和组合 `CompletableFuture` 的方法。

- `CompletableFuture.runAsync(Runnable r)` 该方法没有返回值

```java
CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
    System.out.println("没有返回值");
});
System.out.println(future.get());
```

- `CompletableFuture.supplyAsync(Supplier s)` 该方法可以返回一个指定值。

```java
CompletableFuture<String> stringCompletableFuture = CompletableFuture.supplyAsync(() -> {
    return "返回";
});
System.out.println(stringCompletableFuture.get());
```

```
返回
```

还可以监听 Future 任务的执行状态

监听执行成功，参数 `t` 是执行成功后的返回结果，是如果有异常，那么 `u` 参数就是异常信息，否则为 `null`

```java
stringCompletableFuture.whenComplete((t, u) -> {
  System.out.println("t::" + t);
  System.out.println("u::" + u);
})
```

监听执行失败，例如 Future 有个除零错误，`e` 参数是错误信息，错误后将异步方法的返回值作为结果返回。

```java
stringCompletableFuture.exceptionally((e) -> {
  System.out.println("e::" + e);
  return "错误";
});
```


