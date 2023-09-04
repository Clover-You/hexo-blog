title: 19、Java关于package和import
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 14:49:00
---
## java package 机制

关于Java语言中的包机制：

1. 包又称为package，Java中引入package这种语法机制主要是为了方便程序的管理。不同功能的类被分门别类放到不同的软件包当中，查找比较方便，管理比较方便，易维护。

2. 怎么定义package呢？

   - 在Java源程序的第一行上编写package语句。
   - package只能编写一个语句。
   - 语法结构：
     `package 包名;`

3. 包名的命名规范：
   公司域名倒序 + 项目名 + 模块名 + 功能名;
   采用这种方式重名的机率较低。因为公司域名具有全球唯一性。

4. 包名要求全部小写，包名也是标识符，必须遵守标识符的命名规则。

5. 一个包对应的是一个目录

   例如：

   ```java
   package com.upyou.blog.service;// 4个目录
   ```

6. 使用了`package`机制之后，应该怎么编译？怎么运行呢？

   ```java
   package com.upyou.blog.service;
   public class TestPackage{}
   ```

   - 如上项程序，使用了 `package` 机制之后，使用java命令运行时类名不再是 `testPackage` 了，而是`com.upyou.blog.service.TestPackage`
   
   - 编译：`javac Java源文件路径`（在硬盘上生成对应class文件）
   
   - 手动创建目录，将`TestPackage.class`字节码文件放到指定的目录下
   
   - 运行 `java com.upyou.blog.service.TestPackage`
   
   - 另一种编译方式：`javac -d .*.java`，会将全部java文件编译后自动放到对应的包目录下，没有该目录时自动创建。如果没有指定编译后的路径那么编译后默认在当前源文件位置生成（包括所有包）。`javac -d ~/plugins ./TestPackage.java` 命令编译后在`~/plugins`目录下生成（包括所有包），例如:
    
![image.png](http://qiniu-note-image.ctong.top/note/images/image-b93efa74f598422da58f1cd4f2758e34.png)    




注意，该类所在位置需要和包名一致，而不是说cd到对应位置后使用java命令执行`com.upyou.blog.service.TestPackage`

正确目录：

- com

  1. upyou
     1. blog
        1. service
           1. TestPackage.java
  

当程序处于以上目录后，在项目根目录执行 `java com.upyou.blog.service.TestPackage`



假如有一个程序：

```java
package com.upyou.blog.service.controller;
public class User(){}
```

以下程序使用以上程序时为什么会报错？

1. 当省略包名后，会在当前包下找User,实际上编译器去找`com.upyou.blog.service.User`了，这个类不存在。

   

```java
package com.upyou.blog.service;
public class TestPackage{
  public static void main(String[] args){
    User user = new User();
  }
}
```

当程序不再同一个包下时，应该使用包名加类名去使用，例如

```java
com.upyou.blog.service.controller.User user = new com.upyou.blog.service.controller.User();
```

当`TestPackage`和`User`在同一个包下时不需要加包名。

例如：

```java
package com.upyou.blog.service;
public class User(){}
```

```java
package com.upyou.blog.service;
public class TestPackage{
  public static void main(String[] args){
    User user = new User();
  }
}
```



---

### Java import 机制

import 语句用来完成导入其它类，当你需要使用的类不在同一个包下面时，访问通常使用包名的方式来进行访问，而import则可以让你使用外部的类时，像使用同一个包下的类一样轻松愉快。

**java.lang.*;不需要手动引入，系统自动引入。**

**lang：language语言包，是Java语言的核心类，不需要手动引入**



- `import` 语法格式：
  1. `import 类名;`
  2. `import 包名.*`
- `import`需要编写到`package`语句之下，`class`语句之上。
- 什么时候需要import？
  - 不是java.lang包下，并且不在同一个包下的时候，需要使用import进行引入





例如：

```java
package com.upyou.blog.service.controller;
public class User(){}
```

```java
package com.upyou.blog.service;
import com.upyou.blog.service.controller.*;
//或者使用 import com.upyou.blog.service.controller.User;
public class TestPackage{
  public static void main(String[] args){
    User user = new User();
  }
}
```



**import 包名.*;是导入这个包下所有的类**