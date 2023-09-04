title: 3、Java 控制语句
categories: Java 基础
tags: [学习笔记,JAVA]
date: 2022-01-02 13:37:05
---
## 控制语句

Java 控制语句可以分为 7 种：

- 控制选择结构语句
  - if、if else
  - switch
- 控制循环结构语句
  - for
  - while
  - do while
- 改变控制语句顺序
  - break
  - continue

### 选择结构语句

#### if

如果发生了某件事那么就做什么...

如果 age 满 18，那么输出一段话。if只接受具有比较性的语句，最终比较结果为 Boolean。当满足一定的条件之后，做一件事情。

```java
public static void main(String[] args) {
    int age = 18;
    if(age >= 18){
        System.out.printf("我满 %d 岁啦！", age);
    }
}
```

#### if else

当然，如果不满足某一条件之后，再做另一件事情。

还是刚刚那个问题，如果未满18，那么提示 您未满18.

```java
public static void main(String[] args) {
    int age = 18;
    if(age >= 18){
        System.out.printf("我满 %d 岁啦！", age);
    } else {
        System.out.println("不好意思，您未满18！");
    }
}
```

当然，第一个条件为满足，还可以有第二个条件。当如果第一个条件未满足时再看第二个条件是否也满足。第一个条件不满足享受一种待遇，第二个条件满足又是享受另一种待遇。

```java
public static void main(String[] args) {
    int age = 17;
    if(age >= 18){
        System.out.printf("我满 %d 岁啦！", age);
    } else if(age >= 16){
        System.out.println("你已经满16岁啦，可以办理我们的业务啦！");
    } else {
        System.out.println("不好意思，您不满足任何条件！");
    }
}
```
- **重点：**对于Java中的if语句来说，只要有一个分支执行，整个if语句全部结束。
- **注意：**以上第一种编写方式和第二种编写方式都带有else分支，这两种方式可以100%保证会有分支执行。
- “所有的控制语句”都是可以嵌套使用的，只要合理嵌套执行。


#### switch

关于 `switch` 语句：

1. `switch` 语句也属于选择结构，也是分支语句。
2. `switch` 语句的语法结构：
    - 一个完整的 `switch` 语句应该这样编写：
```java
string tar = "Hello World!";
switch(tar){
  case "Hello World!"{
    ...
    break;
  }
}
```
3. `switch` 接受 `int` 或 `string` 类型的字面值或变量。
4. `case` 是 `switch` 匹配项，一个 `switch` 可以有无数个 `case`：
    - `case` 具有穿透性，必须使用关键字 `break` 阻止穿透。
    - `case` 接受int或string字面值或变量，不需要使用小括号`()`包裹。
5. `switch` 可以有 0 个或 1 个 `default`
    - 当所有 `case` 不匹配时，默认匹配该选项




### 循环结构语句

#### for

当某个条件不满足时，一直重复执行，直到循环条件满足。

一个变量 i 初始值为0，如果它小于10，那么执行一次循环体内的语句。执行完后，i++一次。然后 又执行一次，这时 i++ 等于1，1 < 10又继续执行，直到i不小于10后结束循环。

```java
public static void main(String[] args) {
    for (int i = 0;i < 10; i++){
        System.out.printf("现在i=: %d", i);
    }
}
```

#### while

while 循环语句：
  1. `while` 循环的语法结构：
      `whhile(布尔表达式){...循环体...}`
  2. `while` 执行原理:
      - 先判断布尔表达式执行结果为 `true` 或 `false`，如果为 `true`，执行循环体。`false` 循环结束。
  3. `while` 可能执行 0 次，也可能执行 n 次。
  4. 如果 `while` 表达式为true，那么会造成死循环，编译检测该程序都无法被执行，所以变异报错。

```java
public static void main(String[] args) {
    int i =0;
    while(i == 0){
        System.out.println("1");
        i++;
    }
}
```


#### do while

`do while` 循环语句：
1. `do while` 循环的语法结构
   `do{...循环体...}while(...布尔表达式...)`
2. 执行次数：
    - 一开始是先执行 `do` 语句，所以该循环最少被执行 1 次，最多被执行 n 次。
3. 执行原理：
    - 先执行 `do` 语句中的循环体，最后执行 `while`，如果 `while` 表达式为true，那么会造成死循环，编译检测该程序都无法被执行，所以变异报错。


​    
```java
public static void main(String[] args) {
    int i = 0;
    do{
        System.out.println("i 小于 100 i == %d", i);
        i++;
    } while (i < 100);
}
```

### 控制循环语句

#### break

关于Java 控制语句当中的 break语句：
   1. `break`是Java语言当中的关键字，被翻译为“中断、折断”
   2. `break` + ';' 可以成为一个当读的完整的Java语句： break;
   3. `break` 可以用在：
       - `switch` 语句中，可以终止 `switch` 语句执行。
       - 使用在循环语句当中，可以使当前整个循环语句终止循环。

下列程序，如果i等于3时，退出当前循环。
```java
for(int  = 0; i < 10; ++) {
    if (i == 3) {
        break;
    }
    System.out.println('i == %d', i);
}
```

#### continue

`continue`语句：
   1. `continue` 表示： 继续、go、on、下一个
   2. `continue` 也是一个 `continue` 关键字，加一个分号(;)可以构成一个单独的语句: `continue;`
   3. `break` 和 `continue` 的区别（用在循环语句中）：
       - `break` 可以终止当前整个循环语句
       - `continue` 可以终止、跳出当前循环，但是整个循环语句不会终止

下列程序，如果 i 等于 3，那么跳过当前循环执行下一个循环。

```java
for(int  = 0; i < 10; ++) {
    if (i == 3) {
        continue;
    }
    System.out.println('i == %d', i);
}
```
