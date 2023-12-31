title: 12、Java构造方法
categories: Java 基础
tags: [学习笔记,JAVA]
date: 2022-01-02 14:39:57
---
关于Java类中的构造方法：

1. 构造方法又被称为构造函数、构造器、Constructor
2. 构造方法语法结构：
   [修饰符列表] 构造方法名 （形式参数列表）{构造方法体}
3. 对于构造方法来说，返回值类型不需要指定，并且也不能写`void`
4. 构造方法的方法名必须和类名保持一致
5. 构造方法的作用？
   - 构造方法存在的意义是，通过构造方法的调用，可以创建对象。
   - 给实例变量赋值。
   - 创建对象的同时，初始化实例变量的内存空间。
6. 构造方法应该怎么调用？
   - 构造方法必须使用`new`调用
   - `new 构造方法名(实参列表);`    

7. 构造方法执行之后，有返回值吗？
   - 每一个构造方法实际上执行结束之后都有返回值，但是这个“`return 值;`”这样的语句不需要写。Java程序自动返回值。
   - 返回值类型是构造方法所在类的类型。由于构造方法的返回值类型就是类本身，所以返回值类型不需要写。
8. 当一个类中没有定义任何构造方法的话，系统默认给该类提供一个无参数的构造方法，这个构造方法被称为**缺省构造器**
9. 当一个类显示的将构造方法定义出来了，那么系统将不再默认为这个类提供缺省构造器。建议开发中手动的为当前类提供无参数的构造法。
10. 构造方法支持重载机制，在一个类当中编写多个构造方法，这多个构造方法显然已经构成方法重载机制。


>成员变量之实例变量，属于对象级别的变量，这种变量必须先有对象才能有实例变量。实例变量没有手动赋值的时候，系统会赋默认值，那么这个系统默认值是在什么时候完成的呢？是在类加载的时候吗？不是，因为类加载的时候只加载了代码片段，还没来的及创建对象。所以此时实例变量并没有初始化。实际上，实例变量的内存空间是在构造方法执行过程中完成开辟的，完成初始化的。系统在默认赋值的时候，也是在构造方法当中完成的赋值