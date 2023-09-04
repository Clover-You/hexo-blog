title: Junit5 单元测试 - 学习笔记
categories: 其它
tags: [JAVA,学习笔记,Junit]
date: 2022-01-02 18:13:28
---
在SpringBoot2.2.0+开始引入Junit5 作为单元测试默认库

> JUnit5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

**JUnit Plaform** 是在JVM上启动测试框架的基础，不仅支持Junit自制的测试引擎，其它测试引擎也都可以接入。

**JUnit Jupiter** 提供了JUnit5 的新的编程模型，时JUnit5 新特性的核心。内部包含了一个**测试引擎**，用于在Junit Platform上运行。

**JUnit Vintage** 为老版本提供了兼容。

![截屏2021-07-11 21.38.27](http://qiniu-note-image.ctong.top/note/images/202112271054568.png)

在创建SpringBoot项目的时候，会自动引入junit 5并帮我们自动配好

```java
package com.ctong.learnspringboot;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
class LearnSpringBootApplicationTests {
  @Test
  void contextLoads() { }
}
```

> 在SpringBoot2.4中，移除了Junit Vintage，也就是说我们无法兼容3、4的版本，如果需要继续兼容需要手动加入Junit Vintage依赖
>
> ```xml
> <dependency>
> <groupId>org.junit.vintage</groupId>
> <artifactId>junit-vintage-engine</artifactId>
> <scope>test</scope>
> <exclusions>
>  <exclusion>
>    <groupId>org.hamcrest</groupId>
>    <artifactId>hamcrest-core</artifactId>
>  </exclusion>
> </exclusions>
> </dependency>
> ```



## 使用

在SpringBoot整合Junit后，

- 编写测试方法时需要使用`@Test`标注，需要使用**Junit5**版本的注解
- Junit类具有Spring的功能，例如`@Autowired`、`@Transactional`



## Junit5 常用注解

[JUnit5官方文档](https://junit.org/junit5/docs/current/user-guide/)

> JUnit5 的注解和Junit4的注解有所变化

- `@Test`表示方法是测试方法，但是与Junit4的`@Test`不同，他的职责非常单一，不能声明任何属性，拓展的测试将会由Jupiter提供额外测试
- `@ParameterizedTest`  表示方法是参数化测试
- `@DisplayName` 为测试类或测试方法设置展示名称
- `@BeforeEach` 在每个单元测试之前执行
- `@AfterEach` 在每个单元测试之后执行
- `@BeforeAll` 在所有单元测试开始之前执行
- `@AfterAll` 在所有单元测试结束之后执行
- `@Tag` 给单元测试进行分类，类似JUnit4中的`@Categories`
- `Disabled` 表示当前测试方法不执行，类似JUnit4中的`@Ignore` 
- `@RepeatedTest` 重复执行当前测试方法



## 断言 （assertions）

 断言（assertions）是测试方法中的核心部分，用来对测试需要满足的条件进行验证。这些断言方法都是**org.junit.jupiter.api.Assertions**的静态方法。Junit5内置的断言可以分成如下几个类别：

> 检查业务逻辑返回的数据是否合理。



### 简单断言

用来对单个值进行简单的验证

| 方法            | 说明                                 |
| --------------- | ------------------------------------ |
| assertEquals    | 判断两个对象或两个原始类型           |
| assertNotEquals | 判断两个对象或两个原始类型是否不相等 |
| assertSame      | 判断两个对象引用是否指向同一个对象   |
| assertNotSame   | 判断两个对象引用是否指向不同的对象   |
| assertTrue      | 判断给定的布尔值是否为true           |
| assertFalse     | 判断给定的布尔值是否为false          |
| assertNull      | 判断给定的对象引用是否为null         |
| assertNotNull   | 判断给定的对象引用是否不为null       |

**简单示例**

```java
@Test
@DisplayName("测试简单断言")
void testSimpleAssertions() {
  int result = cal(1, 3);
  Assertions.assertEquals(3, result, "期望的测试结果为3， 但最终结果是：" + result);
}
```

 

```java
org.opentest4j.AssertionFailedError: 期望的测试结果为3， 但最终结果是：4 ==> 
Expected :3
Actual   :4
```



### 数组断言

`assertArrayEquals`方法用来判断两个对象或原始类型的数组是否相等

```java
@Test
@DisplayName("数组断言")
void testArrayAssertions() {
  Assertions.assertArrayEquals(new int[] {1, 2}, new int[] {2, 3});
}
```



```java
org.opentest4j.AssertionFailedError: array contents differ at index [0], expected: <1> but was: <2>
```





### 组合断言

`assertAll`方法接受多个`org.junit.jupiter.api.function.Executable`函数式借口的实例作为要验证的断言，可以通过lambada表达式很容易的提供这些断言



```java
@Test
@DisplayName("组合断言")
void testAssertAll() {
  Assertions.assertAll("test",
                       () -> Assertions.assertTrue(true),
                       () -> Assertions.assertEquals(1, 1));
}
```



### 异常断言

在Junit4时期，想要测试方法的异常情况时， 需要用`@Rule`注解的Eexpected Exception变量还是比较麻烦的。而JUnit5提供了一种新的断言方式`Assertions.assertThrows()`配合函数式编程就可以进行使用。(业务逻辑不抛出异常断言就失败...)

```java
@Test
@DisplayName("异常断言")
void testException() {
  Assertions.assertThrows(ArithmeticException.class, ()-> {
    int i = 10 / 0;
  }, "因为太过正常导致与他们格格不入~~~");
}
```



### 超时断言

`@Timeout` 当前测试方法运行如果超过了指定的时间测试将会不通过

```java
@Test
@Timeout(value = 500, unit = TimeUnit.MILLISECONDS)
void testTimeout() throws InterruptedException {
  log.info("执行了test timeout方法...");
  Thread.sleep(600);
}
```



```java
java.util.concurrent.TimeoutException: testTimeout() timed out after 500 milliseconds
```



### 快速失败

能够使一个单元测试快速失败

```java
@Test
@DisplayName("快速失败")
void testFail() {
  Assertions.fail("测试失败");
}
```



## 前置条件（assumptions）

JUnit5中的潜质条件（assumptions【假设】）类似于断言，不同之处在于**不满足的断言会使得测试方法失败，而不满足的前置条件只会使得测试方法的执行终止**。前置条件可以看成是测试方法执行的前提，当前提不满足时，就没有继续执行的必要。

```java
@Test
@DisplayName("assumptions 前置条件")
void testAssumptions() {
  Assumptions.assumeTrue(false, "结果不为true");
  log.info("前置条件方法体执行...");
}
```





### 嵌套测试

JUnit5可以通过Java中的内部类和`@Nested`注解实现嵌套测试，从而可以更好的把相关方法组织在一起。在内部类中可以使用`@BeforeEach`和`@AfterEach`注解，而且嵌套的层次没有限制。

> 嵌套测试的情况下，外层的Test不能驱动内层的Before(After)Each/All之类的方法。



```java
@DisplayName("嵌套测试")
public class TestingAStack implements Serializable {

  private static final long serialVersionUID = 8946641935343845060L;

  public TestingAStack() {
    // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
  }

  Stack<Object> stack;

  @Test
  @DisplayName("is instantiated with new Stack()")
  void isInstantiatedWithNew() {
    new Stack<>();
  }

  @Nested
  @DisplayName("when new")
  class WhenNew {

    @BeforeEach
    void createNewStack() {
      stack = new Stack<>();
    }

    @Test
    @DisplayName("is empty")
    void isEmpty() {
      assertTrue(stack.isEmpty());
    }

    @Test
    @DisplayName("throws EmptyStackException when popped")
    void throwsExceptionWhenPopped() {
      assertThrows(EmptyStackException.class, stack::pop);
    }

    @Test
    @DisplayName("throws EmptyStackException when peeked")
    void throwsExceptionWhenPeeked() {
      assertThrows(EmptyStackException.class, stack::peek);
    }

    @Nested
    @DisplayName("after pushing an element")
    class AfterPushing {

      String anElement = "an element";

      @BeforeEach
      void pushAnElement() {
        stack.push(anElement);
      }

      @Test
      @DisplayName("it is no longer empty")
      void isNotEmpty() {
        assertFalse(stack.isEmpty());
      }

      @Test
      @DisplayName("returns the element when popped and is empty")
      void returnElementWhenPopped() {
        assertEquals(anElement, stack.pop());
        assertTrue(stack.isEmpty());
      }

      @Test
      @DisplayName("returns the element when peeked but remains not empty")
      void returnElementWhenPeeked() {
        assertEquals(anElement, stack.peek());
        assertFalse(stack.isEmpty());
      }
    }
  }

}
```



## 参数化测试

参数化测试是JUnit5很重要的一个特性，它使得用不同的参数多次运行测试成为了可能，也为我门的单元测试带来许多便利。

利用`@ValueSource`等注解，指定入参，我们将可以使用不同的参数进行多次单元测试，而不需要每新增一个参数就新增一个单元测试，省去了很多冗余代码。

`@ValueSource` 为参数化测试指定入参来源，支持八大基础类以及String类型，Class类型

`@NullSource`表示为参数测试提供一个null的入惨

`@EnumSource` 表示为参数化测试提供一个枚举入参

`@CsvFileSource` 表示读取指定CSV文件内容作为参数化测试入参

`@MethodSource` 表示读取指定方法的返回值作为参数化测试入参「返回方法需要是一个流」



> 参数化测试之所以强大是因为它可以支持外部的各类入参，如：CSV、YML、JSON文件甚至方法的返回值也可以作为入参。只需要去实现Arguments Provider接口，任何外部文件都可以作为他的入参。





### ValueSource

```java
@ParameterizedTest
@ValueSource(ints = {1, 2, 3, 4, 5})
void testParameterized(int i) {
  log.info("当前i的值是: {}", i);
}
```





### MethodSource

```java
@ParameterizedTest
@DisplayName("方法参数化测试")
@MethodSource("stringProvider")
void testStreamParameterized(String str) {
  log.info("方法参数化测试的值：{}", str);
}

static Stream<String> stringProvider() {
  return Stream.of("apple", "banana");
}
```