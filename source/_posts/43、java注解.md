title: 43、java注解
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 17:23:33
---
# java注解



1. 注解，或者叫注释，英文单词是：Annotation
2. 注解是一种引用数据类型。编译之后也是生成xxx.class文件
3. 怎么自定义一个注解呢？语法格式？
   [修饰符列表] @interface 注解类型名 {
   }
4. 注解怎么用，用在什么地方？
   1. 注解使用时的语法格式
      @注解类型名
   2. 注解可以出现在类上、属性上、方法上、变量上等...
      注解还可以出现在注解类型上。



## JDK内置注解

- Deprecated和@Deprecated注解的程序元素，不推荐使用这样的元素，通常因为它很危险或存在更好的选择。
- Override 表示一个方法声明打算重写超类中的另一个方法声明
- SuppressWarnings 指示应该在注解元素（以及包含在该注解元素中的所有程序元素）中取消显示指定的编译器警告。

> 注解的注解叫做元注解



### 元注解

- 什么是元注解
  用来标注“注解类型”的注解叫做元注解

- 常见的元注解有哪些？

  1. Target
     这个注解可以用来指定，被标注的注解可以出现在哪些位置上。
     `@Target(ElementType.METHOD)`表示被标注的注解只能出现在方法上。

  2. Retention

     用来标注被标注的注解最终保存在哪里。（注解保存策略）
     `@Retention(RetentionPolicy.SOURCE)`表示该注解只被保留在java源文件中。
     `@Retention(RetentionPolicy.CLASS)`表示该注解被保留在Class文件当中

     `@Retention(RetentionPolicy.RUNTIME)`表示该注解被保存在class文件中，并且可以被反射机制所读取。

  



### Override

`@Override` 只能出现在方法上.`@Override`这个注解是给编译器参考的，也运行阶段没有关系。凡是java中的方法带有这个注解的，编译器都会进行检查，如果这个方法不是重写父类的方法，编译器报错。但这不是必须的。

源码

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override { }
```



### Deprecated

表示被标注的元素已过时，这个注解主要是向其他程序员传达一个信息，告知已过时，有更好的解决方案存在。

```java
@Deprecated
public @interface MyAnnotation {}
```

已过时的方法或类在不同的编译器可能有不同的展示效果，例如vscode：
![vscode](http://qiniu-note-image.ctong.top/note/images/image-8be00261e20d45288ba53b85794d7c41.png)

idea

![idea](http://qiniu-note-image.ctong.top/note/images/image-12ed2c3c434f4bc5977851c75eb0a819.png)

## 自动定义Annotation

语法

```java
[修饰符列表] @interface 注解类型名 {...}
```

定义

```java
public @interface MyAnnotation {}
```

我们通常在注解当中可以定义属性，以下这个是MyAnnotation的name属性。一个注解如果有定义了一个不能为空的属性，那么使用该注解的时候必须给属性赋值

```java
String name();
```

给指定的属性名赋值

```java
@MyAnnotation(name = "UpYou")
public static void sun(){
  return;
}
```

让它使用自己的默认值

在Annotation中这样定义一个属性，这个属性有个默认值18

```java
int age() default 18;
```

在有默认值的情况下，我们可以设置它的属性值，也可以使用默认

```java
@MyAnnotation(name = "UpYou")
public static void sun(){}
```

不使用默认值

```java
@MyAnnotation(name = "UpYou", age = 18)
public static void sun(){}
```

> 如果一个注解的属性名字是value，并且只有一个属性的话，在使用的时候，该属性名可以省略。

---

注解当中的属性可以是哪一种类型？

- byte
- short
- int
- long
- float
- double
- boolean
- char
- String
- Class
- 枚举类型

以及以上每一种类型的数组形式

### 数组属性

定一个数组类型的属性

```java
public @interface MyAnnotation {
    int[] age();
}
```

如果只有一个值的情况下，可以这样用`name = value`，如果你有多个值时需要使用大括号`{}`而不是中括号`[]`包裹，例如：

```java
@MyAnnotation(age = 18)
public static void sun(){}

@MyAnnotation(age = {17, 18, 19, 20})
public static void doT(){}
```



## 反射注解

被反射的类

```java
@MyAnnotation(age = {14, 15, 16, 17, 18})
public class AnnotationTest {
  public void doT() {}
}
```

获取一个类

```java
Class clazz = Class.forName("annotation.AnnotationTest");
```

判断类上面是否有注解某个注解

```java
boolean isMyAnnotationPresent = clazz.isAnnotationPresent(MyAnnotation.class);
```

-- `true`

获取到这个注解对象

```java
if (isMyAnnotationPresent) {
  MyAnnotation clazzMyAnotation = (MyAnnotation) clazz.getAnnotation(MyAnnotation.class);
}
```

指定获取这个注解的属性

```java
int[] clazzMyAnotationAge = clazzMyAnotation.age();
for (int i = 0; i < clazzMyAnotationAge.length; i++) {
  System.out.println(clazzMyAnotationAge[i]);
}
```

-- `14`
-- `15`
-- `16`
-- `17`
-- `18`

### 获取注解对象的属性值

获取doT方法上的注解信息

```java
// 读取do方法是否带有MethodAnnotation
Method clazzMethods = clazz.getDeclaredMethod("doT");
boolean clazzMethodsIsAnnotationPresent = clazzMethods.isAnnotationPresent(MethodAnnotation.class);
if (clazzMethodsIsAnnotationPresent) {
  MethodAnnotation methodAnnotation = clazzMethods.getAnnotation(MethodAnnotation.class);
  System.out.println(methodAnnotation.name());
  System.out.println(methodAnnotation.pass());
}
```

-- `admin`
-- `123`



## 注解在开发中有什么用

假设有这样一个注解，叫做：@Id

这个注解只能出现在类上面，当这个类上有这个注解的时候，要求这个类中必须有一个int类型的id属性。如果没有这个属性就报异常。如果有这个属性则正常执行。

```java
@Id(fiel = "id", type = int.class)
public class User {}
```



### Id注解的定义

这个注解@Id用来标注类，被标注的类中必须有一个对应类型的属性属性，没有就报异常。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Id {
  // 必须出现的字段
  String fiel();
  // 字段类型
  Class type();
}
```



### 反射

 拿到User类

 ```java
Class userClass = Class.forName("User");
 ```

判断这个类是否存在这个注解

```java
boolean userClassIsAnnotationPresent = userClass.isAnnotationPresent(Id.class);
// 存在这个注解时执行对应操作
if (userClassIsAnnotationPresent) {
  ...在这里执行操作...
}
```

获取到这个类上的@Id注解

```java
Annotation idAnnotation = userClass.getAnnotation(Id.class)
```

拿到指定的属性跟属性类型

```java
String idFieldName = idAnnotation.fiel();
Class idFieldType = idAnnotation.type();
```

判断这个属性是否存在

```java
try {
  Field idField = userClass.getDeclaredField(idFieldName);
} catch (NoSuchFieldException | SecurityException e) {
  System.out.println("错误，不存在属性: " + idFieldName);
}
```

判断这个属性类型是否对应

```java
if (idField.getType() != idFieldType) {
  System.out.println("期望的属性类型为" + idFieldType.getSimpleName() + "; 可得到的类型为" + idField.getType().getSimpleName());
}
```



#### 测试

当`User.java`文件内容如下时

```java
@Id(fiel = "id", type = int.class)
public class User {}
```

-- `错误，不存在属性: id`

```java
@Id(fiel = "id", type = int.class)
public class User {
  public String id;
}
```

-- `期望的属性类型为int; 可得到的类型为String`

```java
@Id(fiel = "id", type = int.class)
public class User {
  public int id;
}
```

没有错误

---

注解在程序当中等同于一种标记，如果这个元素上有这个注解怎么办，没有这个注解怎么办。

[注解+反射练习](https://github.com/Clover-You/my-blog-file/blob/main/%E6%B3%A8%E8%A7%A3%2B%E5%8F%8D%E5%B0%84%E7%BB%83%E4%B9%A0%E4%BB%A3%E7%A0%81.zip)