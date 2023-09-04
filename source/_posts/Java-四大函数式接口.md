title: Java 四大函数式接口
categories: JAVA
tags: [JAVA,随笔]
date: 2022-01-31 03:25:01
---
# Java 四大函数式接口

| 接口名                           | 参数类型 | 返回类型 | 用途                                                         |
| -------------------------------- | -------- | -------- | ------------------------------------------------------------ |
| Consumer\<T\><br />消费型接口    | T        | void     | 对类型为T的对象应用操作，包含方法：`void accept(T t)`        |
| Supplier\<T><br />供给型接口     | 无       | T        | 返回类型为T的对象，包含方法：`T get()`                       |
| Function\<T, R\><br />函数型接口 | T        | R        | 对类型为T的对象应用操作，并返回结果。结果是R类型的对象，包含方法：`R apply(T t)` |
| Predicate\<T\><br />断定型接口   | T        | boolean  | 确定类型为T的对象是否满足约束，并返回 boolean 值。包含方法 `boolean test(T t)` |


