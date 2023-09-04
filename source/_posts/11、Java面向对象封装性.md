title: 11、Java面向对象封装性
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:39:04
---
` Student`类中的`age	`属性在外部程序中可以随意访问，导致`age`属性的不安全。一个`Student`对象表示一个用户，年龄不可能等于负数，以下程序当中年龄值为负数，程序运行的时候并没有报错，这是当前程序存在的缺陷。

```java
public class MainApplication {
    /**
     * 程序的主入口
     * @param args 主方法默认参数
     **/
    public static void main(String[] args){
        Student St = new Student();
        St.address = "湛江";
        St.age = -17;
        St.name = "张美美";
    }
}
```

`Student.class`

```java
public class Student {
    public String name;
    public int age;
    public String address;
}
```

面向对象包括三大特征：

- 封装
  1. 封装之后，对于那个事物来说，看不到这个事物比较复杂的那一面，只能看到该事物简单的那一面。复杂性封装，对外提供简单的操作入口。
  2. 封装之后才会形成真正的**“对象**”，真正的“**独立体**”
  3. 封装之后以后的程序可以重复使用。并且这个事物适应性比较强，在任何场合都可以使用。
  4. 封装之后，对于事物本身提高了安全性。「**安全级别高**」
- 继承
- 多态



### 如何封装？

1. 所有属性私有化，使用`private`关键字进行修饰，表示私有的，修饰的所有数据只能在本类访问。
2. 对外提供简单的操作入口，也就是说以后外部程序想要访问某一属性，必须通过这些简单的入口进行访问。
   - 对外提供两个公开的方法，分别是`set`方法和`get`方法
   - 想修改某一属性，调用`set`方法
   - 想读取某一属性，调用`get`方法
3. 接口的命名规范
   - Set 
     - `public void set + 属性名首字母大写(形参){}`

   - Get 
     - `public 返回值 get + 属性名首字母大写(){}`



对于以下程序来说，属性安全性太高了，外部无法访问。

```java
public class Student {
    private String name;
    private int age;
    private String address;
}
```

一个属性通常访问的时候包括几种访问形式？

1. 想读取这个属性的值
   - Get
2. 想修改这个属性的值
   - Set





以下程序若想安全控制，可直接在`getter`、`setter`方法体中编写逻辑代码，如`setAge`方法，若在`intelliJ IDEA`编辑器中，按下`control + enter`组合快捷键，选择`Getter and Setter`选项，选择你需要生成的属性，点击选择`OK`即可自动生成。

```java
public class Student {
    private String name;
    private int age;
    private String address;
//-------- Set --------//
    public void setAge(int ae){
        if(ae < 0 || ae > 100)
            System.out.printf("提供了不合法的age:%d\n", ae);
        else
            age = ae;
    };

    public void setAddress(String as){
        address = as;
    }

    public void setName(String ne){
        name = ne;
    }

 //-------- Get --------//
    public int getAge(){
        return age;
    }
    public String getName(){
        return name;
    }
    public String getAddress(){
        return address;
    }

}
```

封装完后，应该这样使用

```java
public static void main(String[] args){
  Student St = new Student();
  St.setAge(17);// 发生错误，输出 提供了不合法的age:-17
  St.setName("UpYou");
  St.setAddress("广东湛江");
  
  System.out.println(ZF.getName());// 输出 UpYou
}
```