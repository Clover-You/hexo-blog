title: 39、IO和Properties的联合使用
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 17:16:48
---
# IO和Properties的联合使用

1. Properties是一个Map集合，key和value都是String类型。想将`userinfo`文件中的数据加载到Properties对象当中。



userinfo文件内容

```java
 userName=UpYou
 password=123
```

创建一个输入流和一个Map集合

```java
private final static String FILE_PATH = "/Users/yct/Desktop/userinfo.txt";
```



```java
try(FileReader fr = new FileReader(FILE_PATH)) {
  Properties userInfo = new Properties();
}catch(IOException e){
  e.getStackTrace();
}
```

调用Properties对象的load方法将文件中的数据加载到Map集合中。文件中的数据顺着管道加载到Map集合中，其中等号左边做key，右边做value。

```java
userInfo.load(fr);
```

通过key来获取value

```java
String userName = userInfo.getProperty("userName");
System.out.println(userName);
```

-- `UpYou`

> IO+Properties的联合使用
>
> 类似以上机制的这种文件被称为配置文件。并且当配置文件中的内容格式是：
> 	key1=value
>
> 	key2=value
>
> 的时候，我们把这种配置文件叫做属性配置文件。Java规范中有要求：属性配置文件建议以.properties结尾，但这不是必须的。
>
> 这种以.properties结尾的文件在java中被称为属性配置文件。
>
> 其中Properties是专门存放属性配置文件内容的一个类。