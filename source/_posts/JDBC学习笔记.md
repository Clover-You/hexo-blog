title: JDBC 学习笔记
categories: MySQL
tags: [JAVA,MySQL,SQL]
date: 2022-01-02 17:55:23

---

# JDBC 是什么

JDBC 全称（Java DataBase Connectivity）

JDBC 是 SUN 公司制定的一套接口（interface）在 ** java.sql.\* ** 包下，接口都有调用者和实现者。面向接口调用、面向接口写实现类，这都属于面向接口编程。面向接口编程是为了降低程序的耦合度，提高程序的拓展力。多态机制就是非常典型的：面向接口编程。（不要面向具体编程）

那么为什么 SUN 要去制定这么一套 JDBC 接口呢？因为每个数据库的底层实现都不一样，每一个数据库产品都有自己独特的海鲜原理。如果没有这套接口，那么我们连接数据库时意味着需要多些几套连接代码。

# JDBC 开发前的准备工作

若想使用 JDBC，需要先从官网下载对应的驱动 jar 包，然后将其配置到环境变量 classpath 当中

我将它放在了`/Library/Java/Extensions/jdbc`文件夹中

确定位置之后配置环境变量

```
╰>>> open -e ~/.bash_profile
```

配置环境变量，前面必须以`.;`开始，不然以后包全部找不着了~~

```
export CLASSPATH=.:/Library/Java/Extensions/jdbc/mysql-connector-java-8.0.21/mysql-connector-java-8.0.21.jar
```

保存之后刷新

```
╰>>> source ~/.zshrc
```

# JDBC 编程六步

### 第一步 注册驱动

告诉 java 程序，即将连接什么数据库。

```java
Driver mysqlDriver = new com.mysql.jdbc.Driver();
DriverManager.registerDriver(mysqlDriver);
```

### 第二步 获取连接

JVM 的进程和数据库进程之间的通道打开了，这属于进程之间的通信，重量级的，使用完之后一定要关闭。

```java
String url = "jdbc:mysql://127.0.0.1:3306/upyou_database";
String user = "root";
String pass = "123456";
Connection conn = DriverManager.getConnection(url, user, pass);
```

- url：统一资源定位符（网络中某个资源的绝对路径）
  URL 包括：协议、IP、PORT、资源名

### 第三步 获取数据库操作对象（专门执行 sql 语句的对象）

专门执行 sql 语句的对象

```java
Statement stmt = conn.createStatement();
```

以下代码专门执行 DML 语句，返回值是“影响数据库中的记录条数”

```java
stmt.executeUpdate(sql);
```

### 第四步 执行 SQL 语句

执行 SQL 语句主要执行 DQL、DML 等...

```java
Strign sql = "INSERT INTO DEPT(DEPTNO, DNAME, LOC) VALUES(413, 'UpYouDEPT', 'CHICAGO');";
```

### 第五步 处理查询结果集

只有当第四步执行的是 select 语句的时候，才有这第五步处理查询结果集。

### 第六步 释放资源

使用完资源之后一定要关闭资源，java 和数据库属于进程间的通信，开启之后一定要关闭。并且要遵循从小到大依次关闭，分别对其 try...catch

```java
stmt.close();
conn.close();
```

# JDBC 执行删除与更新

### delete

```java
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Driver;
import java.sql.DriverManager;
import java.sql.Statement;

public class LearnJdbc {
  public static void main(String[] args) {
    Connection conn = null;
    Statement stmt = null;
    try {
      // 注册驱动
      Driver driver = new com.mysql.cj.jdbc.Driver();
      DriverManager.registerDriver(driver);
      // 连接数据库
      String url = "jdbc:mysql://localhost:3306/upyou_database";
      String user = "root";
      String pass = "123456";
      conn = DriverManager.getConnection(url, user, pass);
      // 获取数据库操作对象
      stmt = conn.createStatement();
      // 执行DQL语句
      String sql = "delete from dept where deptno = 413";
      int count = stmt.executeUpdate(sql);
      if (count > 0) {
        System.out.println("删除成功");
      } else {
        System.out.println("删除失败");
      }
    } catch(SQLException e) {
      e.printStackTrace();
    } finally {
      // 释放资源
      if (stmt != null) {
        try {
          stmt.close();
        } catch(SQLException e) {
          e.printStackTrace();
        }
      }
      if (conn != null) {
        try {
          conn.close();
        } catch(SQLException e) {
          e.printStackTrace();
        }
      }
    }
  }
```

### update

```java
import java.sql.Driver;
import java.sql.DriverManager;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;

public class LearnJdbc {
  public static void main(String[] args) {

    Connection conn = null;
    Statement stmt = null;
    try {
      // 注册驱动
      Driver driver = new com.mysql.cj.jdbc.Driver();
      DriverManager.registerDriver(driver);
      String url = "jdbc:mysql://localhost:3306/upyou_database";
      String user = "root";
      String pass = "123456";
      // 获取数据库连接
      conn = DriverManager.getConnection(url, user, pass);
      // 获取数据库操作对象
      stmt = conn.createStatement();
      // 执行DQL语句
      String sql = "update dept set dname = 'UpYou那并不存在的妹妹！', deptno = 1 where deptno = 301";
      int count = stmt.executeUpdate(sql);
      if (count > 0) {
        System.out.println("修改成功！");
      }else {
        System.out.println("修改失败！");
      }

    }catch(SQLException e) {
      e.printStackTrace();
    } finally {
      if (stmt != null) {
        try {
          stmt.close();
        } catch(SQLException e) {
          e.printStackTrace();
        }
      }
      if (conn != null) {
        try {
          conn.close();
        } catch(SQLException e) {
          e.printStackTrace();
        }
      }
    }
  }
```

# 类加载的方式注册驱动

### 第一种注册方式

```java
DriverManager.registerDriver(new com.mysql.cj.jdbc.Driver());
```

### 第二种注册方式

在 mysql 的 jdbc 驱动中已尽帮我们实现了驱动的注册

```java
public class Driver extends NonRegisteringDriver implements java.sql.Driver {
    //
    // Register ourselves with the DriverManager
    //
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
            throw new RuntimeException("Can't register driver!");
        }
    }
}
```

我们只需要让这个类实现类加载就好，这种方式是常用的，因为参数是一个字符串，字符串可以写到 xxx.properties 文件中。

```java
Class.froName("com.mysql.cj.jdbc.Driver");
```

# 处理查询结果集

executeQuery(String sql); 返回一个 ResultSet 对象

```java
String sql = "select * from emp;";
ResultSet result = stmt.executeQuery(sql);
```

ResultSet 的用法类似于 Map 集合的迭代器，next();方法变动迭代器指针位置，有数据返回 true，没有数据返回 false

```java
boolean rn = result.next();// true
```

### getString

getString 方法的特点是：不管数据库中的数据类型是什么，都以 String 的形式取出。其中参数可以是列的下标，也可以是列名，在 jdbc 中，下标是从 1 开始

```java
if (rn) {
  System.out.println(result.getString("ename"));
}
```

-- `SMITH`

```java
if (rn) {
  System.out.println(result.getString(2));
}
```

-- `SMITH`

> 列名不是表中的列名，而是查询结果集的列名。

# 解决 SQL 注入问题

### 导致 SQL 注入的根本原因是什么？

用户输入的信息中含有 SQL 语句的关键字，并且这些关键字参与了 SQL 语句编译的过程。导致 SQL 语句的原意被扭曲，进而达到 SQL 注入

### 解决 SQL 注入

只要用户的输入信息不参与 SQL 语句的编译过程，问题就解决了。即使用户输入的信息含有 SQL 语句的关键字，但是没有参与编译所以不起作用。

要想用户信息不参与 SQL 语句的编译，那么必须使用：`java.sql.PreparedStatement`，而不再是原来的：`java.sql.Statement`,PreparedStatement 接口继承了`java.sql.Statement`接口。

PreparedStatement 是属于与编译的数据库操作对象。他的原理是：预先对 SQL 语句框架进行预编译，然后再给 SQL 语句传“值”。

### 使用

PreparedStatement 的使用方式与 Statement 的使用方式不变，只是将 SQL 语句中的“值”替换成**"?"**这是一个占位符，这不是一个有效值。值需要通过 set 的方式进行设置。

```java
Connection conn = null;
PreparedStatement ps = null;
ResultSet rs
  try {
    // 注册驱动
    Driver driver = new com.mysql.cj.jdbc.Driver();
    DriverManager.registerDriver(driver);
    String url = "jdbc:mysql://localhost:3306/upyou_database";
    String user = "root";
    String pass = "123456";
    // 获取数据库连接
    conn = DriverManager.getConnection(url, user, pass);
    // 获取预编译的数据库操作对象
    String sql = "select empno, ename from emp where empno = ? and ename = ?";
    ps = conn.prepareStatement(sql);
    // 执行DQL语句
    ps.setInt(1, 7369);
    ps.setString(2, "SMITH");
    rs = ps.executeQuery();
    if(rs.next()) {
      System.out.println(rs.getString("empno"));
      System.out.println(rs.getString("ename"));
    }
  }catch(SQLException e) {
    e.printStackTrace();
  } finally {
    if (rs != null) {
      try {
        rs.close();
      } catch(SQLException e) {
        e.printStackTrace();
      }
    }
    if (ps != null) {
      try {
        ps.close();
      } catch(SQLException e) {
        e.printStackTrace();
      }
    }
    if (conn != null) {
      try {
        conn.close();
      } catch(SQLException e) {
        e.printStackTrace();
      }
    }
```

-- `7369`
-- `SMITH`

# PreparedStatement 对比 Statement

- Statement 存在 SQL 注入问题 PreparedStatement 解决了 SQL 注入问题。
- Statement 编译一次执行一次，PreparedStatement 编译一次执行 N 次。PreparedStatement 效率较高一些。
- PreparedStatement 会在编译阶段做类型的安全检查。

综上所述，PreparedStatement 使用较多，只要极少数情况下需要使用 Statement：

- 业务方面要求必须要支持 SQL 注入
- Statement 支持 SQL 注入，凡是业务方面要求是需要进行 SQL 语句拼接的，必须使用 Statement

# 事务

1. JDBC 中的事务是自动提交的。什么是自动提交？
   自动提交是子要执行任意一条 DML 语句，则自动提交一次。这是 JDBC 默认的事务行为。但是在实际的业务当中，通常都是 N 条 DML 语句共同联合才能完成的，必须保证他们这些 DML 语句在同一个事务中同时成功或同时失败。

### 禁止自动提交

void setAutoCommit(boolean autoCommit); 设置 JDBC 的自动事物提交，当 autoCommit 参数为 true 时表示启用自动提交模式，为 false 表示禁用自动提交模式。这也叫做开启事务。

```java
conn.setAutoCommit(false);
```

### 提交事务

禁用事务的自动提交之后，我们需要手动将事务提交。当程序执行成功后调用 commit 方法将事务提交上去。

```java
conn.commit();
```

### 回滚事务

如果程序执行出错，那么你需要将事务回滚而不是进行提交

```java
try {
  ...
} catch (...) {
  if (conn != null) {
    conn.rollback();
  }
}
```

## 悲观锁

如果在一个普通 SQL 语句后面加上一个`for update`，这个就是行级锁。表示在这条语句的执行过程中，`job = 'MANAGER'`的这些记录被锁住了，别的事物没有办法对这些记录进行修改。这也被称为悲观锁。

```sql
select * from emp where job = 'MANAGER' for update;
```

> 事务必须排队执行，数据被锁住了，不允许并发。
