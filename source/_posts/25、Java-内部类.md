title: 25、Java 内部类
categories: Java 基础
tags: [学习笔记,JAVA]
date: 2022-01-02 16:47:00
---
匿名内部类：

1. 什么是内部类？

   - 内部类：在类的内部又定义了一个新的类。被称为内部类。

2. 内部类的分类：

   - 静态内部类：类似于静态变量

     ```java
     class test {
       static class Inner{}
     }
     ```

     

   - 实例内部类：类似于实例变量

     ```java
     class test {
       class Inner{}
     }
     ```

     

   - 局部内部类：类似于局部变量

     ```java
     class test {
       public void doSome(){
         class Inner{}
       }
     }
     ```

     

3. 使用匿名内部类编写的代码，可读性差，能不用就不用，例如：

   **使用匿名内部类**

   ```java
   class Test {
     public static void main(String[] args){
       MyMath m = new MyMath();
       // 打印 200 + 300 = 500
       m.mySum(new Compute(){
         public int sum(int a, int b){
           return a + b;
         }
       }, 200, 300);
     }
   }
   
   interface Compute {
     int sum(int a, int b);
   }
   
   class MyMath {
     public void mySum(Compute c, int x, int y){
       System.out.println(x + "+" + y + "=" + c.sum(x, y));
     }
   }
   ```

   **使用`lambada`语法**

   ```java
   class Test {
     public static void main(String[] args){
       MyMath m = new MyMath();
       // 打印 200 + 300 = 500
       m.mySum((int a, int b) -> {
         return a + b;
       }, 200, 100);
     }
   }
   ```

4. 匿名内部类是局部内部类的一种。因为这个类没有名字而得名，叫做匿名内部类。