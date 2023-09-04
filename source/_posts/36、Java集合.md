title: 36、Java集合
categories: Java 基础
tags: [JAVA,学习笔记]
date: 2022-01-02 17:09:04
---
## 理论

2. 什么是集合？有什么用？

   - 数组其实就是一个集合。集合实际上就是一个容器，可以用来容纳其它类型的数据。

3. 集合为什么说在开发中用的较多？

   - 集合是一个容器，是一个载体，可以一次容纳多个对象。在实际开发中，假设连接数据库，数据库当中有10条记录，那么假设把这10条记录查询出来，在Java程序中会将10 条数据封装成10个Java对象，然后将10个Java对象放到某一个集合当中， 将集合传到前端，然后便利集合，将数据一个一个展现。

4. 集合不能直接存储基本数据类型，另外集合也**不能直接存储Java对象**，集合当中**存储的都是Java对象的内存地址**。(或者说集合中存储的是引用)

   - `list.add(100); `Java会自动装箱 --> `Integer`

     > 集合在Java中本身就是一个容器，是一个对象。
     >
     > 集合中任何时候存储的都是引用。

	![image.png](http://qiniu-note-image.ctong.top/note/images/image-4cada54d072e4274b6f5e8c59fa0fad1.png)     


4. 在Java中每一个不同的集合，底层会对应不同的数据结构。往不同的集合中存储元素，等于将数据放到了不同的数据结构当中。什么是数据结构？数据存储的结构就是数据结构。

   - 数组
   - 二叉树
   - 链表
   - 哈希表

5. 集合在Java JDK中哪个包下？

   - java.util.\*;
     所有的集合类和集合接口都在这个包下。

6. 在java中集合分为两大类

   - 单个方式存储元素
     单个方式存储元素这一类集合中超级父接口：java.util.Collection;
   - 键值对的方式存储元素
     以键值对的方式存储元素，这一类集合中超级父接口：java.util.Map;

7. 所有集合继承`Iterable`的含义是，所有集合都是可迭代的。

8. List集合存储元素特点：有序可重复，存储的元素有下标。

   - 有序实际上是说存进去是这个顺序，取出来还是这个顺序，这里的顺序不是说按照大小排序。有序是因为List集合有下标，下标从0开始，以1递增。

9. Set集合存储元素特点是无序不可重复。

   - 无序表示存进去是这个顺序，取出来就不一定是这个顺序了，另外Set集合中元素没有下标。Set集合中的元素不能重复。

10. List接口实现类

    1. ArrayList
       - `ArrayList`集合底层采用了数组这种数据结构
       - `ArrayList`是非线程安全的
    2. LinkedList
       - `LinkedList`集合底层采用了双向链表数据结构
    3. Vector
       - `Vector`集合底层采用了数组这种数据结构
       - `Vector`是线程安全的
       - `Vector`效率较低

11. Set接口实现类

    1. HashSet
       - 实际上HashSet集合在`new`的时候，底层实际上`new`了一个`HashMap`集合。向HashSet集合中存储元素，实际上是存储到了HashMap集合中了。
       - HashMap集合是一个哈希表数据结构。
    2. `TreeSet`实现的是`SortedSet`，`SortedSet`继承的是`Set`
       - `TreeSet`集合实际上`new`的时候在底层`new`了一个`TreeMap`集合。往`TreeSet`集合中放数据的时候，实际上是将数据放到`TreeMap`集合中了。
       - TreeMap集合底层采用了二叉树数据结构。

12. SortedSet集合存储元素的特点是：由于继承了Set集合，所以它的特点也是无序不可重复，但放在SortedSet集合中的元素可以自动排序。我们称为可排序集合。放到该集合中的元素是自动按照大小顺序排序的。 

![image.png](http://qiniu-note-image.ctong.top/note/images/image-616e1edf77b244b3a7da1d785d4cd934.png)



13. Map
    - Map集合和Collection集合没有关系。
    - Map集合以key和value的这种键值对的方式存储元素。
    - key和value都是存储java对象的内存地址。 
    - 所有Map集合的key特点：无序不可重复 
14. HashMap
    - HashMap集合底层是哈希表数据结构，是非线程安全的。
15. Hashtable
    - `Hashtable`集合底层也是哈希表数据结构，是线程安全的，其中所有的方法都带有`synchronized`关键字，效率较低，现在使用较少，因为控制线程安全有其它更好的方案。
16. Properties
    - 继承`Properties`，另外`Properties`存储元素的时候也是采用`key`和`value`的形式存储，并且`key`和`value`只支持`String`类型，不支持其它类型
    - `Properties`被称为属性类。


![image.png](http://qiniu-note-image.ctong.top/note/images/image-23aa5d91336141dba6b4360bad9e0000.png)





## 总结（所有实现类）

- `ArrayList`: 底层是数组。
- `LinkeList`：底层是双向链表。
- `Vector`：底层是数组，线程安全的，效率较低，使用较少。
- `HashSet`：底层是`HashMap`，放到`HashSet`集合中的元素等同于放到`HashMap`集合`key`部分。
- `TreeSet`：底层是`TreeMap`，放到了`TreeSet`集合中的元素等同于放到`TreeMap`集合`key`部分。
- `HashMap`：底层是哈希表。
- `Hashtable`：底层也是哈希表，只不过线程安全的，效率较低，使用较少。
- `Properties`：是线程安全的，并且`key`和`value`只能存储字符串`String`。
- `TreeMap`：底层是二叉树。`TreeMap`集合的`key`可以自动按照大小顺序排序。



### **List集合存储元素的特点**

- 有序可重复
- 有序
  存进去和取出的顺序相同，每一个元素都有下标。
- 可重复
  存进去一个1还可以再存一个1



### **Set（Map）集合存储元素的特点**

- 无序不可重复
- 无序
  存进去的顺序和取出的顺序不一定相同。Set集合中没有下标。
- 不可重复
  存进去一个1就不能再存一个1



### **SortedSet（SortedMap）集合存储元素的特点**

- 无序不可重复的，但是SortedSet集合中的元素是可排序的。 
- 无序
  存进去的顺序和取出的顺序不一定相同。Set集合中没有下标。
- 不可重复
  存进去一个1就不能再存一个1
- 可排序
  可以按照大小顺序排列。



**Map集合的key，就是一个Set集合。往Set集合中放数据，实际上放到了Map集合的key部分。**

`new HashSet().add()`方法源码：

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
  private transient HashMap<E,Object> map;
  public HashSet() {
    map = new HashMap<>();
  }
  public boolean add(E e) {
    return map.put(e, PRESENT)==null;
  }
}
```

### 关系图

![image.png](http://qiniu-note-image.ctong.top/note/images/image-14859f1bcd424e99b0dc2ba6daedbc8d.png)

![image.png](http://qiniu-note-image.ctong.top/note/images/image-ba90f9f67e88403e889f24533c899ab3.png)



## Collection接口中的常用方法


1. Collection中能放什么元素？

   - 没有使用**泛型**之前，`Collection`中可以存储`Object`的所有子类型。使用了**泛型**之后，`Collection`中只能存储某个具体的类型。
1. Collection中的常用方法
   - [`boolean add(Object e);`](#add)向集合中添加元素。
   - [`int size();`](#size)获取集合中元素的个数。
   - [`void clear();`](#clear)清空集合中全部元素。
   - [`boolean contains(Object o);`](#contains)集合是否包含`o`,包含返回`true`，不包含返回`false`
   - [`boolean remove(Object o);`](#remove)删除集合中的某个元素，如果存在。
   - [`boolean isEmpty();`](#isempty)判断集合是否为空，为空返回`true`，不为空返回`false`
   - [`Object[] toArray();`](#toarray)将集合转为`Object`类型数组



### add()

接口无法实例化对象，因为接口是抽象的。

```java
Collection c = new Collection();
```

实例化一个`Collection`下已实现的子类。如果不指定集合类型，那么默认为`Object`

```java
Collection c = new ArrayList();
```

添加一个值为`100`的`int`数值，**JDK5以上可以自动装箱。**实际上放进去了一个对象的内存地址`int --> Intiger`

```java
c.add(100);
```

指定类型，限制集合存储元素的类型。例如指定该集合只能存储`String`类型的数据。

```java
Collection<String> c = new ArrayList<String>();
```

在`String`类型的集合中存储`int`类型,出现错误:

`The method add(String) in the type Collection<String> is not applicable for the arguments (int)`

```java
c.add(100);
```

存储`String`类型的数据

```java
c.add("Hello World");
```

\-- `[Hello World]`


### size()

向集合中添加元素

```java
Collection<Character> c = new ArrayList<Character>();
String hiTo = "Hello World!";
for (int i = 0; i < hiTo.length(); i++){
  c.add(hiTo.charAt(i));
}
```

\-- `[H, e, l, l, o,  , W, o, r, l, d, !]`

获取集合中元素的个素，从`1`开始

```java
c.size();
```

\-- `12`


### clear()

清空[`size`](#size)示例中集合元素的个数。

```java
c.clear();
```

查看元素个素

```java
System.out.println(c.size());
```

\-- `0`


### contains()

向集合中添加元素
```java
Collection c = new ArrayList();
String hiTo = "Hello World!";
for (int i = 0; i < hiTo.length(); i++){
  c.add(hiTo.charAt(i));
}
```

查看集合是否包含`H`

```java
System.out.println(c.contains("H"));
```

\-- `false`

为什么返回`false`？因为`c.add(hiTo.charAt(i));`添加的是一个`char`类型的元素，`"H"`则是一个`String`类型的元素。**两个不同的类型**没有可比性

`Operator '==' cannot be applied to 'char'`

```java
System.out.println('H' == "H");
```

查看集合是否包含`char`类型的`H`

```java
System.out.println(c.contains('H'));
```

\-- `true`



#### 源码分析

查看集合是否包含某个元素，可以理解为某个元素是否等于某个元素。

```java
Collection<String> arrayList = new ArrayList<String>();
String str  = new String("Hi");
arrayList.add("Hi");
System.out.println(arrayList.contains("Hi"));// true
```

所有的类都继承于`Object`，`Object`有`equals`方法，也就是说，所有的类都有`equals`方法。那么都知道`String`数据是保存在方法区的常量池里`==`比较的是的内存地址,以下程序返回`true`

```java
System.out.println("Hi" == "Hi");
```

而`equals`未重写之前源码也是使用了`==`做比较，那么下面的程序返回`true`吗？答案是`false`，上面说到，`==`比较的是内存地址,`String`对象存储在方法区的常量池中，而`new`创建的对象都保存在堆内存，那么现在你觉得它们内存地址还一样吗？说到这可能就有人说：诶！可是我输出的时候确实是输出一个`Hi`啊！emmm！你还是去看一下`Object`的源码，以及`System.out.println`的源码，这里不做过多的赘述。

```java
String str = new String("Hi");
System.out.println(str == "Hi");
```

`String`通常使用`equals`方法来进行比较，“诶你刚刚不是说`equals`也是使用`==`来比较吗？”,我说的是未重写之前，`String`类已经重写了`equals`方法以及`toString`方法`String`类里面的`equals`方法比较的是字符串的内容。

"诶你叭叭叭说这么多，不是说分析`contains`方法的源码吗？"

......

经过我上面讲的你们应该猜到`contains`大致原理了吧？

- `new String` 不能使用双等号和`""`做比较
- 双等号比较的是内存地址
- 手动`new` 的`String`对象是在堆内存，`""`对象是在方法区

没错！`contains`底层就是使用了`Object`的`equals`方法，这里使用的是多态，如果对比的是`String`，那么`equals`就是`String`的!!!我再问一下，`arrayList`是否包含`str`?说包含的建议源码分析从头再看一次,答案是不包含。

##### 扒一下源码

只扒下实现源码哈！

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
  private int size;
  transient Object[] elementData;
  public boolean contains(Object o) {
    return indexOf(o) >= 0;
  }
  public int indexOf(Object o) {
    return indexOfRange(o, 0, size);
  }

  int indexOfRange(Object o, int start, int end) {
    Object[] es = elementData;
    if (o == null) {
      for (int i = start; i < end; i++) {
        if (es[i] == null) {
          return i;
        }
      }
    } else {
      for (int i = start; i < end; i++) {
        if (o.equals(es[i])) {
          return i;
        }
      }
    }
    return -1;
  }
}
```

成员变量`size`是用来记录集合长度或者说元素个数,`elementData`就是这个集合的数据。主要先看`indexOfRange`方法。

直接看`else`分支,这里是不是使用了`Object`的`equals`方法？那么`new String("Hi").equals("Hi");`是不是等于`true`？**String重写了equals方法,String重写的equals方法比较的是内容**

```java
if (o.equals(es[i]))
```

如果比较的是你自己定义的一个类，那么如果你没有重写`equals`方法，铁定是`false`，重写得看你的对比规则。

```java
public class CollectionTest {
  public static void main(String[] args) {
    Collection arrayList = new ArrayList();
    arrayList.add(new Test("UpYou"));
    Test test = new Test("UpYou");
    System.out.println(arrayList.contains(test));
  }

  public static class Test {
    public String name;

    public Test(String n) {
      this.name = n;
    }

    @Override
    public boolean equals(Object o) {
      if (this == o) return true;
      if (o == null || getClass() != o.getClass()) return false;
      Test test = (Test) o;
      return this.name.equals(test.name);
    }

  }
}
```

\-- `true`


### remove()

向集合中添加元素

```java
Collection c = new ArrayList();
String hiTo = "Hello World!";
for (int i = 0; i < hiTo.length(); i++){
  c.add(hiTo.charAt(i));
}
```

删除集合中的某个元素

```java
c.remove('H');
```

\-- `[e, l, l, o,  , W, o, r, l, d, !]`



### isEmpty()

创建一个集合

```java
Collection<String> c = new ArrayList<String>();
```

判断集合是否为空

```java
System.out.println(c.isEmpty());
```

\-- `true`

向一个空的集合添加元素后判断是否为空

```java
c.add("Hello World!");
System.out.println(c.isEmpty());
```

\-- `false`

#### 源码

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
  private int size;
  public boolean isEmpty() {
    return size == 0;
  }
}
```

### toArray()

创建一个`String`集合并添加元素

```java
Collection<String> c = new ArrayList<String>();
c.add("Hello World!");
```

将集合转成`Object`数组

```java
Object[] list = c.toArray();
```

获取这个数组的第一个元素

```java
System.out.println(list[0]);
```

\-- `Hello World!`





## 关于集合迭代器

`Iterator`迭代器，是所有实现于`Iterator`类通用的一种方式，例如`Collector`集合。

创建一个集合对象

```java
Collection list = new HashSet();
```

添加一些元素

```java
String eles = "HelloWorld!";
for (int i = 0; i < eles.length(); i++) {
  c.add(eles.charAt(i));
}
```

对`Collection`进行迭代，得到`Collection`的迭代器`Iterator`,在源码中，`Iterator`迭代器在内部类中实现。

```java
Iterator iterator = c.iterator();
```

### 迭代器中的方法

1. [`boolean hasNext();`](#hasnext) 检查集合中是否还有元素，没有元素返回`false`，否则返回`true`。

2. [`E next();`](#next)使得迭代器前进一位，并返回当前迭代器指针指向的元素。统一返回`Object`(存进去什么类型取出来就是什么类型)



### hasNext()

查看是否是一个可迭代的状态。

```java
boolean hn = iterator.hasNext();
```

\-- `true`



### next()

使迭代器前进一位，并返回当前迭代器指针指向的元素。统一返回`Object`类型,**由于[Set](#setmap集合存储元素的特点)集合特点是无序不可重复的，所以每次执行可能元素位置不一。**

```java
Object item = iterator.next();
```

\-- `!`



### 使用循环

若想一个一个取出集合中的元素，可以使用循环来控制迭代器。

```java
while (iterator.hasNext()) {
  System.out.println(iterator.next());
}

```

  \-- `!`
  \-- `r`
  \-- `d`
  \-- `e`
  \-- `W`
  \-- `H`
  \-- `l`
  \-- `o`



### 迭代器执行原理内存图

没执行`next()`方法之前，`Iterator`指针不会指向任何元素

![image.png](http://qiniu-note-image.ctong.top/note/images/image-a68a038270f34891889afbb6900e7a6f.png)



第一次循环开始，执行`next()`之后,指针向前一位，并将当前指针指向的元素`!`返回


![image.png](http://qiniu-note-image.ctong.top/note/images/image-5af85f3f650c4e868ed628e3c56d87bf.png)

第二次循环开始后，执行`next()`，指针又向前移一位，将当前指针指向的元素`r`返回


![image.png](http://qiniu-note-image.ctong.top/note/images/image-250be5d231e44940b604645153495946.png)

循环直到`hasNext()`方法返回`false`之后结束,指针在最后一位元素的位置。
![image.png](http://qiniu-note-image.ctong.top/note/images/image-9b321cb77b5843bcadfcb771980f5351.png)

当迭代完成之后，`hasNext()`方法会一直处于`false`的状态,如果强行使用`next()`,那么你程序的生命也就到头了。

```java
while (iterator.hasNext()) {
  System.out.println(iterator.next());
}
System.out.println(iterator.next());
```

Exception in thread "main" java.util.NoSuchElementException
	at java.base/java.util.HashMap\$HashIterator.nextNode(HashMap.java:1586)
	at java.base/java.util.HashMap\$KeyIterator.next(HashMap.java:1607)
	at com.upyou.collection.CollectionTest01.main(CollectionTest01.java:26)



当集合的结构发生改变时，迭代器必须重新获取，如果还是用以前老的迭代器，会出现异常：`java.util.ConcurrentModificationException`

使用`Collection.remove`方法之后，集合结构发生了改变，此时迭代器没有更新。

```java
Collection c = new ArrayList();
c.add(1);
c.add(2);
c.add(3);
Iterator it = c.iterator();

while (it.hasNext()) {
  Object o = it.next();
  c.remove(o);
  System.out.println(o);
}
```

而如果迭代时需要删除元素，最好的办法就是使用迭代器来删除，如何使用迭代器来进行删除元素呢？别急，迭代器给我门提供了这样的一个方法: `remove`

```java
Collection c = new ArrayList();
c.add(1);
c.add(2);
c.add(3);
Iterator it = c.iterator();

while (it.hasNext()) {
  it.next();
  it.remove();
}
```

`Iterator.remove`删除迭代器当前指向的元素，而`next`方法控制迭代器的指针，所以使用`remove`时必须先使用`next`，因为迭代器开始是不会指向任何元素。

> 迭代器是通用的

## List接口方法

1. List集合存储元素特点
   - 有序可重复
2. List既然是Collection接口的子接口，那么肯定List接口有自己“特色”的方法
   - [`void add(int index, E element);`](#add-1)向集合中指定位置插入元素。
   - [`Object get(int index);`](#get)通过指定下标获取集合中的元素
   - [`int indexOf(Object o);`](#indexof)获取指定对象第一次出现处的索引,如果没有返回`-1`
   - [`int lastIndexOf(Object o)`](#lastIndexof)获取指定对象最后一次出现的索引，如果没有返回`-1`
   - [`Object remove(int index)`](#remove)删除指定下标位置的元素
   - [`Object set(int index, Object element)`](#set)修改指定下标位置的元素

### add()

向集合中指定位置插入元素

```java
List l = new ArrayList();
l.add(1);
l.add(2);
l.add(3);
System.out.println(l);
l.add(1, "Hello World!");
System.out.println(l);
```

\-- `[1, 2, 3]`

\-- `[1, Hello World!, 2, 3]`

这个方法使用不多，因为对于ArrayList集合来说效率比较低。



### get()

通过指定下标获取集合中的元素

```java
List l = new ArrayList();
l.add(1);
l.add("Hello World!");
l.add(3);
System.out.println(l.get(1));
```

\-- `Hello World!`





### indexOf()

获取指定对象第一次出现处的索引,如果没有返回`-1`

```java
List l = new ArrayList();
l.add(1);
l.add("Hello World!");
l.add(3);
System.out.println(l.indexOf(3));
System.out.println(l.indexOf(4));
```

\-- `2`

\-- `-1`



### lastIndexOf()

获取指定对象最后一次出现的索引，如果没有返回`-1`

```java
List l = new ArrayList();
l.add(1);
l.add("Hello World!");
l.add("Hello World!");
l.add("Hello World!");
l.add(3);
System.out.println(l.lastIndexOf("Hello World!"));
System.out.println(l.lastIndexOf(0));
```

\-- `3`

\-- `-1`



### remove()

删除指定下标位置的元素

```java
List l = new ArrayList();
l.add(1);
l.add("Hello World!");
l.add("Hello World!");
l.add(3);
l.remove(1);
System.out.println(l);
```

\-- `[1, Hello World!, 3]`



### set()

修改指定下标位置的元素

```java
List l = new ArrayList();
l.add(1);
l.set(0, "Hello monkey");
System.out.println(l);
```

\-- `[Hello monkey]`



## ArrayList集合初始化及扩容

1. 默认初始化容量10（底层先创建了一个长度为0的数组，当添加第一个元素的时候，初始化容量10）

      ```java
      private static final int DEFAULT_CAPACITY = 10;
      private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
      ```

2. 集合底层是一个Object[]数组

3. 构造方法
   `new ArrayList();`

   `new ArrayList(20);`

   将HashSet集合转为List集合

   `Collection c = new HashSet();`

   `c.add(1);`

   `new ArrayList(c);`

4. ArrayList集合的容量

   - 增长到原容量的1.5倍
   - 尽可能少扩容。因为数组扩容效率比较低，建议在使用ArrayList集合的时候预估计元素的个数，给定一个初始化容量。

5. 数组优缺点

   - 优点：检索效率比较高

   - 缺点：随机增删元素效率低

     数组无法存储大数据量，因为很难找到一块非常大的连续的内存空间

6. 向数组末尾添加元素，效率很高，不受影响。



### 将ArrayList转换成线程安全

如何将一个非线程安全的ArrayList集合转换成线程安全的呢？

- 使用集合工具类
  `java.util.Collections;`
  注意`java.util.Collection;`是集合接口

非线程安全

```java
List list = new ArrayList();
list.add("upyou");
```

转换成线程安全的

```java
List list = new ArrayList();
Collections.synchronizedList(list);
list.add("upyou");
```


## 单向链表数据结构

1. 对于链表数据结构来说，基本的单元是节点Node。

2. 对于单链表来说，任何一个节点Node中都有两个属性

   - 存储的数据
   - 下一节点的内存地址

3. 链表优缺点

   - 优点
     随机增删元素效率较高，因为增删元素不涉及到大量元素位移

   - 缺点

     不能通过数学表达式计算被查找元素的内存地址，每一次查找时都需要一个一个的去找

![image.png](http://qiniu-note-image.ctong.top/note/images/image-c5db5bec2f244f2b96d4499565c5dcd0.png)



## Map接口的常用方法 

1. Map和Collection没有继承关系
2. Map集合以key和value的方式存储数据：俗称键值对
   - key和value都是引用数据类型
   - key和value都是存储对象的内存地址
   - key起到主导的地位，value是key的一个附属品
3. Map集合中的常用方法
   - `void clear()` 清空Map
   -  `boolean containsValue(Object value)` 判断Map中是否包含某个value
   - `V get(Object key)` 通过key获取value
   - `boolean isEmpty()` 判断Map集合中的元素个数是否为0
   - `Set<K> keySet()` 获取Map集合所有的Key 
   - `V put(K key, V value)` 向Map集合中添加键值对
   - `V remove(Object key)` 通过key删除键值对
   - `int size()`获取集合长度
   - `Collection<V> values()`获取Map集合中所有的value，返回一个Collection
   - `Set<Map.Entry<K,V>> entrySet()`将Map集合转成Set集合



## 哈希表数据结构

1. HashMap集合底层是哈希表的数据结构。

2. 哈希表是一个数组和单向链表的结合体 。

3. 哈希值是key的 `hashCode` 方法的执行结果。

4. 哈希表HashMap使用不当时无法发挥性能！
   假设将所有的hashCode返回值固定为某个值，那么会导致底层哈希表变成了纯单向链表。这种情况我们称为：散列分布不均匀。

   - 什么是散列分布均匀？
     假设有100个元素，10个单向链表，那么每个单向链表上有10个节点，这是最好的，是散列分布均匀的。
   - 假设将所有的hashCode方法返回值都设定为不一样的值，可以吗，有什么问题？
     不行，因为这样的话导致底层哈希表就成为一维数组了，没有链表的概念了。也是散列分布不均匀。
   - 散列分布均匀需要你重写hashCode方法时有一定的技巧。
   
5. 放在`HashMap`集合key部分的元素，以及放在`HashSet`集合中的元素，需要同时重写`hashCode`和`equals`方法。

6. HashMap集合的默认初始化容量是16，默认加载因子是0.75，这个默认加载因子是当HashMap集合底层数组的容量达到75%的时候，数组开始扩容。

   HashMap集合初始化容量必须是2的倍数，这也是官方推荐的，这是因为达到散列均匀，为了提高HashMap集合的存取效率所必须的。

7. 向Map集合中存取，都是先调用key的hashCode方法，让后再调用equals方法！equals方法有可能调用，也有可能不调用。

   - 拿put(k, v)举例，什么时候equals不会调用？
     k.hashCode()方法返回哈希值，哈希值经过哈希算法转换成数组下标。

     数组下标位置上如果是null，equals不需要执行。

   - 拿get(k)举例，什么时候equals不会调用？

     k.hashCode()方法返回哈希值，哈希值经过哈希算法转换成数组下标。数组下标位置上如果是null，equals不需要执行。

8. 放在HashMap集合key部分的，以及放在HashSet集合中的元素，需要同时重写hashCode方法和equals方法。



### put(k，v)实现原理

```java
 public V put(K key, V value){...}
```

第一步：将`k`和`v`封装到`Node`对象中

第二步：底层会调用`k`的`hashCode`方法得出hash值，然后通过哈希函数/哈希算法，将hash值转成数组的下标，下标位置上如果没有任何元素，就把Node添加到这个位置上。如果下标对应的位置上有链表，此时会拿着`k`和链表上每一个节点中的`k`进行`equals`，如果所有的`equals`方法返回都是`false`，那么这个新节点将被添加到链表的末尾，如果其中有一个`equals`返回了`true`，那么这个节点的`value`将会被覆盖。



### get(k)实现原理



```java
public V get(Object key) {...}
```

第一步：先调用k的hashCode方法得出哈希值，通过哈希算法转换成数组下标，通过数组下标快速定位到某个位置上，如果这个位置上什么也没有，返回null，如果这个位置上有单向链表，那么对单向链表上的每个节点中的k进行equals，如果所有equals方法返回`false`，那么get方法返回`null`，只要其中有一个节点的k和参数k equals的时候返回`true`，那么此时这个节点的value就是需要找的value。get方法最终返回这个要找的value。



## 属性类Properties

Properties是一个Map集合，继承Hashtable，Properties的key和value都是String类型

Properties被称为属性类对象

Properties是线程安全的



### 存储数据

properties 通过setProperty方法来存储数据

```java
pro.setProperty("name", "UpYou");
```

### 取数据

properties 通过getProperty方法来获取数据

```java
System.out.println(pro.getProperty("name"));
```


## TreeSet 集合

1. TreeSet集合底层实际上是一个TreeMap
2. TreeMap集合底层是一个二叉树
3. 放到TreeSet集合中的元素，等同于放到TreeMap集合key部分了
4. TreeSet集合中的元素：无序不可重复，但是可以按照元素的大小顺序自动排序，称为：可排序集合

```java
TreeSet<String> ts = new TreeSet<>();
ts.add("a");
ts.add("d");
ts.add("b");
ts.add("h");
ts.add("m");
ts.add("s"); 
```

输出

-- `a`
-- `b`
-- `d`
-- `h`
-- `m`
-- `s`

**对TreeSet来说，自定义类型可以排序吗？**

```java
import java.util.TreeSet;
public class Learn {
  public static void main(String[] args) {
    // System.out.println("Hello World!");
    Person p1 = new Person(1);
    Person p2 = new Person(2);

    TreeSet<Person> persons = new TreeSet<>();
    persons.add(p1);
    persons.add(p2);

    for (Person p : persons) {
      System.out.println(p);
    }
  }
}

class  Person{
  int age;
  Person(int age) {
    this.age = age;
  }
  public String toString() {
    return "年龄是：" + this.age;
  }
}
```

程序执行出现错误 `Exception in thread "main" java.lang.ClassCastException: Person cannot be cast to java.lang.Comparable`。

对于`Person`类型来说是无法排序的，因为没有指定Person对象之间的比较规则。对于Person排序需要实现`java.lang.Comparable`接口。

```java
class Person implements Comparable<Person>{
  int age;

  Person(int age) {
    this.age = age;
  }
  // 实现Comparable中的comparaTo方法
  // 需要在这个方法中编写比较的逻辑、或者说比较的规则，按照什么比较！
  
  @Override
  public int compareTo(Person o) {
    return this.age - o.age;
  }

  public String toString() {
    return "年龄是：" + this.age;
  }
}
```

输出结果

-- `年龄是：1`
-- `年龄是：2`

`k.compareTo(t.key)`
拿着参数k和集合中的每一个k进行比较，返回值可能是 >0 <0 =0，比较规则最终还是由我们自己规定。例如：按照年龄升序、降序。

> 放在TreeSet集合中的元素需要实现java.lang.Comparable接口，并且实现compareTo方法。
>
> TreeSet中的自动排序是通过二叉树的方式进行排序。



### TreeSet使用自定义比较器

TreeSet可以使用自定义比较器来比较，TreeSet底层是TreeMap，我就不过多讲述了，就把源码中重要的代码copy出来！

```java
private final Comparator<? super K> comparator;

public TreeMap(Comparator<? super K> comparator) {
  this.comparator = comparator;
}

final Entry<K,V> getEntry(Object key) {
  // Offload comparator-based version for sake of performance
  if (comparator != null) // 使用自定义比较器
    return getEntryUsingComparator(key);
  if (key == null)
  	......
  return null;
}
```

#### 比较器实现

比较器需要实现`java.util.Comparator`接口。（Comparable是java.lang包下的。Comparator是java.util包下的）

```java
class PersonComparator implements Comparator<Person> {
  @Override
  public int compare(Person o1, Person o2) {
    return o1.age - o2.age;
  }
}
```

`Person`就不需要实现`Comparable`接口了，因为我们自定义了一个比较器，把我们自定义的比较器给到`TreeSet`去使用就好：

```java
TreeSet<Person> persons = new TreeSet<>(new PersonComparator());
```

#### 内部类实现比较器

为了减少java文件，也可以使用内部类的方式来实现比较器：

```java
TreeSet<Person> persons = new TreeSet<>(new Comparator<Person>(){
  @Override
  public int compare(Person o1, Person o2) {
    System.out.println("使用的是内部类！");
    return o1 - o2;
  }
});
```



> 放到TreeSet或者TreeMap集合key部分的元素想做到排序，包括两种方式：
>
> 1. 放在集合中的元素实现java.lang.Comparable接口
> 2. 在构造TreeSet或者TreeMap集合的时候给它传一个比较器对象。
>
> Comparable和Comparator如何选择？
>
> 1. 如果比较规则不会轻易改变那么选择Comparable
> 2. 如果比较规则不定，或者需要多个比较规则之间频繁切换，那么使用Comparator，Comparator接口的设计符合OCP原则。
>


## 自平衡二叉树数据结构

1. TreeSet、TreeMap是自平衡二叉树，遵循左小右大的原则。

2. 遍历二叉树的时候有三种方式

   - 前序遍历：根左右
   - 中序遍历：左根右
   - 后序遍历：左右根

   > 前中后说的是“根”的位置。根在前面是前序，根在中间是中序，根在后面是后序。

3. TreeSet集合/TreeMap集合采用的是：中序遍历方式，Iterator迭代器采用的是中序遍历方式。左根右。

4. 100，200，50，60，80，120，140，130，135，180，666，40，55
   ![image.png](http://qiniu-note-image.ctong.top/note/images/image-6957ab7ce01a43709f42df30f0593788.png)


## 集合工具类

Java.util.Collection 集合接口

Java.util.Collections 集合工具类，方便集合的操作。

### synchronizedList

可将不是线程安全的集合变成线程安全的：

```java
List<String> arrList = new ArrayList<>();
Collections.synchronizedList(arrList);
```

### sort

可对`List`集合进行排序

```java
List<String> arrList = new ArrayList<>();
Collections.sort(arrList);
```

sort 也支持传入比较器

```java
List<Person> arrList = new ArrayList<>();
arrList.add(new Person(1));
arrList.add(new Person(3));
arrList.add(new Person(9));
arrList.add(new Person(2));
// 自定义比较器
Collections.sort(arrList, new Comparator<Person>() {
  @Override
  public int compare(Person o1, Person o2) {
    return o1.age - o2.age;
  }
});

```

对Set集合进行排序的时，需要将Set集合转成List集合

```java
Set<String> s = ne-qw HashSet<>();
List<String> arrList = new ArrayList<>(s);
Collections.sort(arrList);
```

## 集合最主要掌握什么内容？

1. 每个集合对象的创建（new）
2. 向集合中添加元素
3. 从集合中取出某个元素
4. 遍历集合
5. 主要的集合类
   - ArrayList
   - LinkedList
   - HashSet
   - TreeSet
   - HashMap
   - Properties
   - TreeMap




> 类在转成某个接口类型的时候不需要有继承关系。