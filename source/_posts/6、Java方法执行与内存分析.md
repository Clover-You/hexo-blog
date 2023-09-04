title: 6、Java方法执行与内存分析
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:30:07
---
## 理论

* 方法在执行过程中，在JVM中的内存是如何分配的呢，内存是如何变化的？
  1. 方法只定义，不调用，是不会执行的，并且在`JVM`也不会给该方法分配**运行所属**的内存空间。
     只有在调用这个方法的时候，才会动态的给这个方法分配所属的内存空间。
  2. 在`JVM`内存划分上有这样三块主要的内存空间（还有其它的内存空间）：
     * 方法区内存
     * 栈内存
     * 堆内存
  3. 关于**栈**数据结构：
     * 栈： stack，是一种数据结构
     * 数据结构反应的是数据的存储形态。
     * 数据结构是独立的学科，不属于任何编程语言的范畴，只不过在大多数编程语言当中要使用数结构。
     * 作为程序员需要提前精通：数据结构 + 算法[计算机必修]
  4. 方法执行的时候代码片段存在哪里？方法执行的时候执行过程的内存在哪里分配？
     * 方法代码片段属于`.class`文件的一部分，字节码文件在类载（classLoader、类加载器）加载的时候，将其放到了方法区内存当中。所以`JVM`中的三块主要的内存空间中方法区内存最先有数据。
     * 代码片段虽然在方法区内存当中只有一份，但是可以被重复调用。每一次调用这个方法的时候，需要给该方法分配独立的活动场所，在栈内存中分配。【栈内存中分配方法运行的所属内存空间】
  5. 方法在调用瞬间，在**栈内存**中会给该方法分配独立的内存空间，此时发生**压栈**动作(push)，方法执行结束之后，给该方法分配的内存空间全部释放，此时发生**弹栈**动作(pop)。
     * 压栈： 给该方法分配内存
     * 弹栈： 释放该方法的内存空间
  6. 局部变量在**方法体**中声明，局部变量在运行阶段内存在栈中分配。
     * 局部变量有形参也有在方法体定义的局部变量。
     * 局部变量生命周期最短，因为只要出了这个方法的大括号，那么就意味着这个方法结束了，方法结束了那么就再也没有机会去访问这个内存空间因为内存空间释放掉了，没有了。





#### **`JVM`执行原理图**

![image.png](http://qiniu-note-image.ctong.top/note/images/image-69e9c7753b6c44a2a08575d961f749e8.png)


### 栈数据结构

![image.png](http://qiniu-note-image.ctong.top/note/images/image-239a3582d5964387bf9cd767fd53e6b9.png)




1. 栈帧永远指向栈顶元素。
2. 栈顶元素处于活跃状态，其它元素静止。
3. 术语：
   ![image.png](http://qiniu-note-image.ctong.top/note/images/image-85540f1ac0e84fe09c802e350cec1ad1-20211227095638630.png)

   * 压栈/入栈/push
   * 弹栈/出栈/pop
4. 栈数据结构的特点是：
   * 先进后出
   * 后进先出

## 内存分析



以下程序在`JVM`中如何执行？

```java
public class testMethod(){
  public static void main(String[] args){
    int a = 10;
    int b = 20;
    int retValue = sumInt(a, b);
    System.out.println("retValue = " + retValue);
  }
  
  public static int sumInt(int i, int j){
    int result = i + j;
    int num = 3;
    int retValue = divide(result, num);
    return retValue;
  }
  
  public static int divide(int x, int y){
    int z = x / y;
    return z;
  }
}
```



`.java`文件通过编译后,**Class Loader**(类加载器)将`testMethod.class`放到了方法区内存中。


![image.png](http://qiniu-note-image.ctong.top/note/images/image-b99899b9d5114573a60cdb8a37893285.png)



`JVM`会默认执行入口函数`main`，代码执行时`JVM`在栈内存开辟一块空间供`main`执行。`JVM`将`main`放到了栈内存当中执行，期间发生了**压栈**(push)动作。**栈帧**永远指向栈顶元素，栈顶元素是**活跃**的。

![image.png](http://qiniu-note-image.ctong.top/note/images/image-349613633bbb4048907d50adaa027daa-20211227095132369.png)


代码一步一步执行后，逐渐为局部变量开辟内存空间。为`main`栈开辟了两个名为`a、b`的内存空间

![image.png](http://qiniu-note-image.ctong.top//note/images/202201021410746.png)


当执行到第 5 行的时候也就是`int retValue = sumInt(a, b);`,调用了`sumInt`方法。调用的这一瞬间`JVM`又给`sumInt`开辟内存。而这时栈帧发生改变，指向了`sumInt`栈。由于栈帧改变，所以`main`已暂停执行、被阻塞，现在执行的是栈顶元素，由于栈帧永远指向的是栈顶元素，所以栈顶元素永远处于**活跃**状态。

![image.png](http://qiniu-note-image.ctong.top/note/images/image-c7d507c6a7d14b93b2758bd98eaf8848.png)


在`main`调用`sumInt`的时候，在参数传递的时候，实际上传递的是变量中保存的值。将`a`和`b`变量的值给到`sumInt`而不是`a`和`b`。所以`sumInt`无法操作`main`里面的局部变量**(无法得到内存地址)**。参数传递的时候是按顺序传递。


![image.png](http://qiniu-note-image.ctong.top/note/images/image-0d36dbb8e9174cac98f4680c393c5691.png)

执行到第10行的时候` int result = i + j;`,这时候需要计算 `i + j` 的结果给`result`栈储存。而计算是由中央处理器也就是CPU来执行，CPU将处理后的结果给到`result`

![image.png](http://qiniu-note-image.ctong.top/note/images/image-fa449887fdd24dcb9fcd8aa9b11eb365.png)


程序继续执行，当执行到第12行的时候`int retValue = divide(result, num);`,又调用 `divide`,`JVM`给`divide`开辟了一块内存空间，发生了**压栈**动作。逐步开辟`x、y、z`栈，而`z`的结果需要通过CPU计算得到。

![image.png](http://qiniu-note-image.ctong.top/note/images/image-37e46b5564f34327b6be5911b7d3e339.png)

而程序遇到`return`语句后就会**强制弹栈**（释放内存空间），继续栈顶元素的执行，这时候栈顶元素以及变成`sumInt`，**弹栈**后代表第12行`int retValue = divide(result, num);`结束。

![image.png](http://qiniu-note-image.ctong.top/note/images/image-289106c2b78743afb6b89966d09a4844.png)

将`divide` 的结果给到`retValue`


![image.png](http://qiniu-note-image.ctong.top/note/images/image-06e3f54419e041bea5a6eb0fb5a4ad27.png)


而往下走遇到`return`强制弹栈`return retValue;`。`sumInt`弹栈之后代表着第5行` int retValue = sumInt(a, b);`结束。

![image.png](http://qiniu-note-image.ctong.top/note/images/image-7ad9bae6dacd4042bc0aab3567d0b1c9.png)


往下走遇到了`System`类,这个类实际与其它class(包含`testMethod.class`)一起被加载到代码区。调用`System`类里面的`println`方法后又压栈，执行完成后弹栈，最后`main`执行完成弹栈。资源全部释放。



> 代码是逐行执行，从上倒下。
>
> 代码编译期不会执行任何计算，JVM执行时计算
>
> 栈结构遵循 先进后出，后进先出 的规则
>
> 栈内存主要存储的是局部变量
>
> 方法调用的时候，在传参的时候，实际上传递的是变量保存的**值**
>
> 栈帧永远在栈顶，栈顶元素永远处于**活跃**状态