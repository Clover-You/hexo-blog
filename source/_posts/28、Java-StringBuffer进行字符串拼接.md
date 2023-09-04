title: 28、Java StringBuffer进行字符串拼接
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 16:51:04
---
在实际开发中，如果需要进行字符串的频繁拼接，会有什么问题？

- 因为Java中的字符串是不可变的。每一次拼接都会产生新的 字符串。这样会占用大量的方法区内存，造成内存空间的浪费。

  ```java
  String str = "Hello" + "World";
  ```

  以上代码就创建了三个`String`对象在方法区当中。`Hello`、`World`、`HelloWorld`

- 使用`StringBuffer`

  ```java
  StringBuffer stringBuffer = new StringBuffer();
  ```
  
- 字符串拼接，调用`append`方法

  ```java
  stringBuffer.append("Hello");
  stringBuffer.append("World");
  ```

- 在`String`底层使用的是`final`修饰的`byte[]`数组存储字符串，而在`StringBuffer`底层，也是使用`byte[]`，区别是它并没有使用`final`修饰。`byte[]`默认长度是**16**,在`append`方法中，如果追加的时候`byte[]`满了，会自动扩容

  **底层代码**

  ```java
  public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
  }
  ```

  ```java
  public AbstractStringBuilder append(String str) {
    if (str == null)
      return appendNull();
    int len = str.length();
    ensureCapacityInternal(count + len);
    str.getChars(0, len, value, count);
    count += len;
    return this;
  }
  ```

   检查`byte[]`是否需要扩容
  ```java
  private void ensureCapacityInternal(int minimumCapacity) {
    // overflow-conscious code
    if (minimumCapacity - value.length > 0) {
      value = Arrays.copyOf(value,
                            newCapacity(minimumCapacity));
    }
  }
  ```

  对数组进行扩容 `System.arraycopy`

  ```java
  public static char[] copyOf(char[] original, int newLength) {
    char[] copy = new char[newLength];
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
  }
  ```

- `如何优化StringBuffer的性能？`

  1. 在创建StringBuffer的时候尽可能给定一个初始化容量。减少底层数组的扩容次数。

- `StringBuffer`和`StringBuilder`区别

  - `StringBuffer`中的方法都有`synchronized`关键字修饰，表示`Stringbuffer`在多线程环境下运行是安全的。
  - `StringBuilder`中的方法都没有`synchronized`关键字修饰，表示`StringBuffer`在多线程环境下是不安全的。