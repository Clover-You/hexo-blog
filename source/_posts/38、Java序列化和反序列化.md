title: 38、Java序列化和反序列化
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 17:15:59
---
# Java序列化和反序列化

1. 序列化：Serialize
   把Java对象存储到文件中，将java对象的状态保存下来的过程
2. 反序列化：DeSerialize
   将硬盘上的数据重新恢复到内存当中，恢复成java对象

## 序列化



```java
public class ObjectInputStreamTest {
  public static void main(String[] args) {
    Test t = new Test();
    t.setName("UpYou");
    try (ObjectOutputStream ois = new ObjectOutputStream(new FileOutputStream("/Users/yct/Desktop/test"))) {
      ois.writeObject(t);
      ois.flush();
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}

class Test {
  private int sex;
  private String name;

  public void setSex(int sex) {
    this.sex = sex;
  }

  public void setName(String name) {
    this.name = name;
  }
  
  @Override
  public String toString() {
    return "Test{" +
            "sex=" + sex +
            ", name='" + name + '\'' +
            '}';
  }
}
```

以上程序出现错误

java.io.NotSerializableException: io.Test
	at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1184)
	at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:348)
	at io.ObjectInputStreamTest.main(ObjectInputStreamTest.java:13)



原因是Test类没有实现可序列化接口`Serializable`，这是一个空的接口，里面没有任何方法和属性，这是一个标识

```java
class Test implements Serializable {
 ...
}
```

1. 参与序列化和反序列化的对象，必须实现`Serializable`接口。

2. 通过源码发现，`Serializable`接口只是一个标志接口：

   ```java
   public interface Serializable {
   }
   ```

   这个接口中什么代码都没有。那么它起到一个什么作用呢？
   	起到标识的作用、标志的作用，java虚拟机检查到实现了这个接口，可能会对这个类进行特殊待遇。



## 反序列化

反序列化回来是一个Test对象

```java
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("/Users/yct/Desktop/test"))) {
  Object t = ois.readObject();
  System.out.println(t);
} catch (IOException e) {
  e.printStackTrace();
} catch (ClassNotFoundException e) {
  e.printStackTrace();
}
```

-- `Test{sex=0, name='UpYou'}`



## transient 关键字

如果序列化时，不希望某个成员参与序列化，可以使用`transient`关键字。`transient`表示游离的，不参与序列化。

```java
Test t = new Test();
t.setSex(18);
t.setName("UpYou");
try (ObjectOutputStream ois = new ObjectOutputStream(new FileOutputStream("/Users/yct/Desktop/test"))) {
  ois.writeObject(t);
  ois.flush();
} catch (IOException e) {
  e.printStackTrace();
}
class Test implements Serializable {
  private transient int sex;
  ...
}
```



再读取这个test文件

```java
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("/Users/yct/Desktop/test"))) {
  Object t = ois.readObject();
  System.out.println(t);
} catch (IOException e) {
  e.printStackTrace();
} catch (ClassNotFoundException e) {
  e.printStackTrace();
}
```

-- `Test{sex=0, name='UpYou'}`



## 序列号版本号

1. 序列号版本号不对应出现的错误：
   java.io.InvalidClassException: io.Test; local class incompatible: stream classdesc serialVersionUID = 3728061353932211421, local class serialVersionUID = -2284980033278438903
2. java语言中是采用什么机制来区分类的？
   1. 首先通过类名进行比对，如果类名不一样，肯定不是同一个类。
   2. 如果类名一样，再通过序列号版本号进行区分。
      假如有两个同名的类，对于java虚拟机来说，只要两个类实现了`Serializable`接口，就算这两个名字一样，java虚拟机也是可以区分开的，因为它们都有默认的序列化版本号，它们的序列化版本号不一样，所以可以区分开（这是自动生成序列化版本号的好处）
3. 这种自动生成的序列化号的缺陷：
   一旦代码确定后，不能进行后续的修改，因为只要修改，必然会重新编译，此时会自动生成全新的序列化版本号，这个时候java虚拟机就会认为这是一个全新的类。
4. 凡是一个类实现了`Serializable`接口，建议给该类提供一个固定不变的序列化版本号



```java
class Test implements Serializable {
  private static final long serialVersionUID = 3728061353932211421L;
...
}
```