title: 42、java反射机制
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 17:21:00
---
# java反射机制

## 反射机制有什么用

通过java语言中的反射机制可以操作字节码文件。通过反射机制可以操作代码片段。（class文件）

## 反射机制相关类

### java反射机制的相关类在哪个包下？

java的反射机制在 java.lang.reflect.*这个包下



### java反射机制相关的重要的类有哪些？

- java.lang.Class
  代表字节码文件，代表一个类型、代表整个类

- java.lang.reflect.Method
  代表字节码中的方法，代表类中的方法

- java.lang.reflect.Constructor
  代表字节码中的构造方法字节码，代表类中的构造方法

- java.lang.reflect.Field

  代表字节码中的属性字节码。代表类中的成员变量（静态变量和实例变量）



## 获取Class的三种方式

需要操作一个类的字节码，首先需要获取到这个类的字节码，怎么获取`java.lang.Class`实例?

### 第一种

`Class.forName()`

1. 静态方法
2. 方法的参数是一个字符串
3. 字符串需要的是一个完整的类名。
4. 完整类名必须带有包名。即使是在`java.lang`包下的也不能省略

> Class.forName方式，会导致这个类被加载（加载到方法区）

通过类名获取到这个类

```java
Class clazz = Class.forName("learnClass.TestClass");
```

clazz代表的是`TestClass`类型

TestClass

```java
package learnClass;
public class TestClass {}
```

> 注意，需要处理`ClassNotFoundException`异常，这是一个编译时异常。



### 第二种

java中任何一个对象都有一个方法：getClass();这个方法是在`java.lang.Object`包下的。

```java
TestClass tc = new TestClass();
Class clazz = tc.getClass();
```

如果使用`==`来比较两个相同类型的`Class`，那么它们的结果为`true`，因为它们保存的内存地址是一样的，都指向方法区中的节码文件。

```java
try {
  Class clazz = Class.forName("learnClass.TestClass");
  TestClass tc = new TestClass();
  Class clazz2 = tc.getClass();
  System.out.println(clazz == clazz2);
} catch (ClassNotFoundException e) {
	e.printStackTrace();
}
```

-- `true`



### 第三种

java语言中任何一种数据类型，包括基本数据类型，它都有`.class`属性。

```java
Class c1 = TestClass.class;
Class c2 = int.class;
Class c3 = void.class;
```



## 通过反射实例化对象

通过反射机制获取Class，通过Class来实例化对象

```java
public class TestClass {
  public TestClass() {
    System.out.println("构造方法已执行！");
  }
}

```

```java
Class clazz = TestClass.class;
Object obj = clazz.newInstance();
```

newInstance这个方法返回的是一个范型，这个方法会调用TestClass这个类的无参数构造方法，完成对象的创建。

-- `构造方法已执行！`

如果需要实例化的对象需要一个参数才能完成实例化，那么会出现一个实例化异常

```java
java.lang.InstantiationException: learnClass.TestClass
  at java.lang.Class.newInstance(Class.java:427)
  at learnClass.ClassForName.main(ClassForName.java:13)
  Caused by: java.lang.NoSuchMethodException: learnClass.TestClass.<init>()
    at java.lang.Class.getConstructor0(Class.java:3082)
    at java.lang.Class.newInstance(Class.java:412)
    ... 1 more
```

> newInstance()方法内部实际上调用了无参数构造方法，必须保证无参数构造存在才可以。



## Class.forName发生了什么？

例如用forName方式获取TestClass

```java
public class TestClass {
  static {
    System.out.println(TestClass.class.getName() + "开始加载！！");
  }
}

```

```java
try {
  Class clazz = Class.forName("learnClass.TestClass");
} catch (ClassNotFoundException e) {
  e.printStackTrace();
}
```

-- `learnClass.TestClass开始加载！！`

经过以上测试，**forName导致类加载**。

如果你只是希望一个类的静态代码块执行，其它代码一律不执行，可以采用：`Class.forName();`。



## Field

### 获取Field

Field翻译为字段，其实就是属性、成员

4个Field，分别采用了不同的访问控制权限

```java
public class Student {
  public int no;
  private String name;
  protected int age;
  boolean sex;
}
```

### 获取类名

获取类名`getSimpleName`获取的是简类名，而`getName`获取的是完整类名（带包名）

```java
Class clazz = Class.forName("reflect.Student");
System.out.println(clazz.getSimpleName());
SYstem.out.pritnln(clazz.getName());
```

-- `Student`

-- `reflect.Student`

### 获取类中所有的Field

```java
Class clazz = Class.forName("reflect.Student");
// 获取类中所有 的Filed
Field[] fields = clazz.getFields();
System.out.println(fields.length);
```

-- `1`

### 取出这个Field的名字

```java
String fieldFirstName = fields[0].getName();
System.out.println(fieldFirstName);
```

-- `no`

使用`getDeclaredFields`获取Field

```java
Field[] fs = clazz.getDeclaredFields();
System.out.println(fs.length);
```

-- `4`

> getFields()只能获取被`public`修饰的Field，而`getDeclaredFields();`可以获取任意修饰符的Field

### 查看获取到的Field的名字

```java
Class clazz = Class.forName("reflect.Student");
// 获取类中所有 的Filed
Field[] fs = clazz.getDeclaredFields();
for (Field field : fs) {
  System.out.println(field.getName());
}
```

-- `no`
-- `name`
-- `age`
-- `sex`

### 获取FIeld类型

```java
Class clazz = Class.forName("reflect.Student");
Field[] fs = clazz.getDeclaredFields();
for (Field field : fs) {
  Class fieldType = field.getType();
  System.out.println(fieldType.getSimpleName());
}
```

-- `int`

-- `String`

-- `int`

-- `boolean`



### 获取修饰符列表

返回的修饰符是一个数字，每个数字都是修饰符的代号！！！

```java
Class clazz = Class.forName("reflect.Student");
Field[] fs = clazz.getDeclaredFields();
for (Field field : fs) {
  System.out.println(field.getModifiers());
}
```

-- `1`
-- `2`
-- `4`
-- `0`

可以使用`Modifier`这个类将这个代号转为对应修饰符的名称

```java
Class clazz = Class.forName("reflect.Student");
System.out.println(clazz.getSimpleName());
Field[] fs = clazz.getDeclaredFields();
for (Field field : fs) {
  int modifier = field.getModifiers();
  String modifierName = Modifier.toString(modifier);
  System.out.println(modifierName);
}
```

-- `public`
-- `private`
-- `protected`

-- ` `



### 反编译

通过反射机制来反编译一个类的Field

```java
public class Student {
  public int no;
  private String name;
  protected int age;
  boolean sex;
}
```

将类结构大致模拟出来

```java
StringBuilder sb = new StringBuilder();
Class clazz = Class.forName("java.lang.String");
// 获取到修饰符名称
String classModifierName = Modifier.toString(clazz.getModifiers());
sb.append(classModifierName + " class ");
sb.append(clazz.getSimpleName());
sb.append(" {");

sb.append(" }");
System.out.println(sb);
```

-- `public class Student { }`

设置类体

```java
StringBuilder sb = new StringBuilder();
Class clazz = Class.forName("java.lang.String");
// 获取到修饰符名称
String classModifierName = Modifier.toString(clazz.getModifiers());
sb.append(classModifierName + " class ");
sb.append(clazz.getSimpleName());
sb.append(" {");

Field[] fields = clazz.getDeclaredFields();
for (Field field : fields) {
  sb.append("\n\t");
  String fieldModifierName = Modifier.toString(field.getModifiers());
  String fieldName = field.getName();
  sb.append(fieldModifierName + " ");
  sb.append(field.getType().getSimpleName() + " ");
  sb.append(fieldName);
  sb.append(";\n");
}
sb.append(" }");
System.out.println(sb);
```

-- `public final String {`

-- `private final char[] value;`

-- `private int hash;`

-- `private static final long serialVersionUID;`

-- `private static final ObjectStreamField[] serialPersistentFields;`

-- `public static final Comparator CASE_INSENSITIVE_ORDER;`

-- `}`



### 通过反射机制访问对象的属性

```java
public class Student {
  public int no;
  private String name;
  protected int age;
  boolean sex;
}
```

**以下代码的obj对象就是上面这个实体**

怎么通过反射机制访问一个java对象的属性？

- 给属性赋值set
- 获取属性的值get

获取到需要访问的对象，让后将它实例化（以下代码obj就是Student对象，底层调用无参数构造方法）

```java
Class clazz = Class.forName("reflect.Student");
Object obj = null;
try {
  obj = clazz.newInstance();
} catch(Exception e) {
  e.printStackTrace();
}
```

获取某个特定的属性，返回Field对象

```java
Field noField = clazz.getDeclaredField("no");
```

给obj对象（Student对象）的对应属性赋值

```java
Field noField = clazz.getDeclaredField("no");
noField.set(obj, 555555);
```

> 反射给对象赋值与我们平时实例化后赋值步骤一样。

如果需要获取obj中某个属性的值，则使用get方法

```java
Field noField = clazz.getDeclaredField("no");
noField.set(obj, 555555);
System.out.println(noField.get(obj));
```

如果我们给私有属性设置值会发生什么呢？

```java
Field noField = clazz.getDeclaredField("name");
noField.set(obj, "UpYou");
```

```java
java.lang.IllegalAccessException: Class reflect.ReflectTest01 can not access a member of class reflect.Student with modifiers "private"
  at sun.reflect.Reflection.ensureMemberAccess(Reflection.java:102)
  at java.lang.reflect.AccessibleObject.slowCheckMemberAccess(AccessibleObject.java:296)
  at java.lang.reflect.AccessibleObject.checkAccess(AccessibleObject.java:288)
  at java.lang.reflect.Field.set(Field.java:761)
  at reflect.ReflectTest01.main(ReflectTest01.java:19)
```

因为被`private`修饰，所以即使是反射也是不允许被赋值的，那么该怎么办呢？**“打破封装”**

使用`setAccessible(true);`打破封装

```java
Field noField = clazz.getDeclaredField("name");
noField.setAccessible(true);
noField.set(obj, "UpYou");
System.out.println(noField.get(obj));
```

-- `UpYou`

> 反射机制的缺点：打破封装，可能会给不法分子留下机会！！



## Method

```java
public class UserService {
  /**
   * 登录方法
   * @param name 用户名
   * @param password 用户密码
   * @return true 表示成功，false表示失败！
   */
  public boolean login(String name, String password) {
    if ("admin".equals(name) && "123".equals(password)) {
      return true;
    }
    return false;
  }

  /**
   * 系统注销
   */
  public void logout() {
    System.out.println("系统已经安全退出！");
  }
}
```



### 获取Method

`getDeclaredMethods`获取全部方法

```java
Class userServiceClass = Class.forName("reflect.UserService");
Method[] userServiceClassMethods = userServiceClass.getDeclaredMethods();
for (Method method : userServiceClassMethods) {
  System.out.println(method.getName());
}
```

-- `logout`
-- `login`



### 获取返回值类型

`getReturnType`获取一个方法的返回值

```java
Class userServiceClass = Class.forName("reflect.UserService");
Method[] userServiceClassMethods = userServiceClass.getDeclaredMethods();
for (Method method : userServiceClassMethods) {
  String methodReturnTypeName = method.getReturnType().getSimpleName();
  System.out.println(methodReturnTypeName);
}
```

-- `boolean`

-- `void`

### 获取方法参数类型

`getParameterTypes()`可获取一个方法的参数类型列表

```java
Class userServiceClass = Class.forName("reflect.UserService");
Method[] userServiceClassMethods = userServiceClass.getDeclaredMethods();
for (Method method : userServiceClassMethods) {
  Class[] paraType = method.getParameterTypes();
  for (Class methodParameterType : paraType) {
    System.out.println(methodParameterType.getSimpleName());
  }
}
```

-- `String`

-- `String`



### 反射机制调用方法

获取一个类

```java
Class userServiceClass = Class.forName("reflect.UserService");
```

实例化这个类

```java
Object obj = userServiceClass.newInstance(); 
```

根据方法名获取需要调用的方法，获取一个方法不能只靠方法名，因为在java中有重载方法的机制（overload）。java中获取一个方法需要靠方法名和参数列表。

```java
Method login = userServiceClass.getMethod("login", String.class, String.class, int.class);
```

调用方法

```java
Object isLogin = login.invoke(obj, "admin", "123");
System.out.println(isLogin);
```

-- `true`



> 反射机制，让代码很具有通用性，可变化的内容都是些到配置文件当中，将来修改配置文件之后，创建的对象不一样了，调用的方法也不一样了，当是Java代码不需要做任何改动，这就是反射机制的魅力。



## 获取父类和父接口

获取一个类

```java
Class vipClass = Class.forName("java.lang.String");
```

获取父类

```java
Class vipSuperClass = vipClass.getSuperclass();
System.out.println(vipSuperClass.getSimpleName());
```

-- `Object`



获取实现类

```java
Class[] vipInterfaces = vipClass.getInterfaces();
for (Class vipInterface : vipInterfaces) {
  System.out.println(vipInterface.getSimpleName());
}
```

-- `Serializable`
-- `Comparable`
-- `CharSequence`