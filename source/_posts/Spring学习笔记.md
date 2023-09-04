title: Spring学习笔记
categories: Spring
tags: [学习笔记,JAVA,Spring]
date: 2022-01-02 18:03:37
---
# Spring

资料： https://pan.baidu.com/s/1aS4B69iA8-AtXqT7D9obXA 提取码: rczx

- Spring 是一个开源框架

- Spring为简化企业级开发而生，使用Spring，javaBean就可以实现很多以前要靠EJB才能实现的功能。同样的功能，在EJB中要通过繁琐的配置和复杂的代码才能实现，而在Spring中却非常的优雅和简洁。
- Spring是一个IOC（DI）和AOP容器框架
- Spring的优良特性
  1. 非侵入式
     基于Spring开发的应用中的对象可以不依赖于Spring的API
  2. 依赖注入
     DI——Dependency Injection，反转控制（IOC）最经典的实现
  3. 面向切面编程
     Aspect Oriented Programming——AOP
  4. 容器
     Spring是一个容器，因为它包含并且管理应用对象的生命周期
  5. 组件化
     Spring实现了使用简单的组件配置组合成一个复杂的应用。在Spring中可以使用XML和Java注解组合这些对象
  6. 一站式
     在IOC和AOP的基础上可以整合各种企业应用的开源框架和优秀的第三方类库（实际上Spring自身也提供了表述层的SpringMVC和持久层的SpringJDBC）



## Spring 模块化分

![Spring模块划分图](http://qiniu-note-image.ctong.top/note/images/202112271048716.png)



1. Test
   Spring单元测试模块

2. Core Container

   核心容器(IOC)，黑色代表这部分的功能由哪些jar包组成。要使用这部分的完整功能，这些jar包都要导入。

3. AOP、Aspects
   面向切面编程模块

4. Data Access/Integration
   数据访问

5. Web
   Spring开发web应用模块



## IOC和DI

- IOC 容器
  可以用来整合大多数第三方框架
- AOP 面向切面编程
  通常用来完成一些，声明式事务。例如操作数据库等…

![IOC和DI](http://qiniu-note-image.ctong.top/note/images/202112271048728.png)





### IOC

IOC全称（Inversion of control或控制反转）。

- 控制
  指资源的获取方式，获取方式又分为一下两点

  1. 主动式：
     要什么资源都需要自己创建。这种方式在创建简单对象时非常方便， 但在创建复杂对象时，是非常麻烦的。

     ```java
     class TestClass {
       Object obj = new Object();
     }
     ```

  2. 被动式
     资源的获取不是我们自己创建，而是交给容器创建和设置。

     ```java
     class TestClass {
       Object obj;
       
       public void test() {
         obj.xxx();
       }
     }
     ```

     容器管理所有的组件。假设`TestClass`受容器管理，`Object`也受容器管理，容器可以自动的检查出哪些组件需要用到另一些组件。容器会帮我们创建出`TestClass`对象并把Object对象赋值过去。

- 反转
  主动获取资源变为被动的接受资源

  



### DI

DI（Dependency Injection或依赖注入）。容器能知道哪些组件运行的时候需要使用到另外一个组件。容器会通过反射的方式将容器中准备好的`Object`注入到`TestClass`。

以前是自己new对象，现在所有的对象交给容器创建。「**给容器中注册组件**」



> 只要是容器管理的组件，都能使用容器提供的强大功能。



## HelloWorld

### 导包

需要导入所有核心jar包

```
commons-logging-1.1.3.jar
spring-beans-4.0.0.RELEASE.jar
spring-context-4.0.0.RELEASE.jar
spring-core-4.0.0.RELEASE.jar
spring-expression-4.0.0.RELEASE.jar
```



### 编写配置文件

spring的配置文件中，集合了spring的ioc容器管理的所有组件。以下为spring 配置文件基础格式。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

</beans>
```

注册一个对象，Spring会自动创建这个对象。一个Bean标签可以注册一个组件

```java
package top.clover.spring.bean;
public class Person {

    private String name;

    private Integer age;

    private String gender;

    private String email;

}
```

在Bean标签中，有一个class，它表示你要注册哪一个组件。这里需要的是一个全类名。

在这个bean标签中，还有一个id，这是这个组件的唯一标识。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- 注册一个bean组件 -->
    <bean class="top.clover.spring.bean.Person" id="person"/>
</beans>
```

如果你还想为组件赋值，可以使用`property`标签。`name`表示属性名，`value`表示属性值。

```xml
<bean class="top.clover.spring.bean.Person" id="person">
  <property name="age" value="19"/>
  <property name="name" value="clover"/>
  <property name="gender" value="男"/>
  <property name="email" value="2621869236@qq.com"/>
</bean>
```

除了调用**`set`**进行赋值外，还可以使用`constructor-arg`标签调用有参构造器进行赋值。

```java
public Person(String name, Integer age, String gender, String email) {
  this.name = name;
  this.age = age;
  this.gender = gender;
  this.email = email;
}
```

```xml
<bean class="top.clover.spring.bean.Person" id="person">
    <constructor-arg name="name" value="Clover"/>
    <constructor-arg name="age" value="19"/>
    <constructor-arg name="gender" value="男"/>
    <constructor-arg name="email" value="2621869236@qq.com"/>
</bean>
```

`constructor-arg`也可以使用索引进行赋值

```xml
<bean class="top.clover.spring.bean.Person" id="person1">
  <constructor-arg index="0" value="Clover"/>
  <constructor-arg index="1" value="19"/>
  <constructor-arg index="2" value="男"/>
  <constructor-arg index="3" value="2621869236@qq.com"/>
</bean>
```



### 编写测试类

在配置文件写好后，可以使用`JUnit`来进行测试

```java
@Test
public void getPersonBean() {
  ApplicationContext app = new ClassPathXmlApplicationContext("ioc.xml");
  Person bean = app.getBean(Person.class);
  System.out.println(bean.getAge());
}
```

`ApplicationContext`代表ioc容器。在spring中，有很多种容器，其中有两个最为常用`org.springframework.context.support.ClassPathXmlApplicationContext`和`org.springframework.context.support.FileSystemXmlApplicationContext`。

`ClassPathXmlApplicationContext`表示在xml配置文件在当前项目类路径下，`FileSystemXmlApplicationContext` 获取其它位置的xml位置文件。

```java
app.getBean(Person.class);
```

可以通过类的方式获取到对应的bean，也可以通过刚刚定义的id去获取

```java
app.getBean("person");
```

 

### 总结

- 容器中对象的创建都是在容器创建完成的时候就已经创建好了。
- 同一个组件在ioc容器中是单实例的
- 如果组件并不在ioc中，则会出现`NoSuchBeanDefinitionException`异常，无论是以id的形式还是class的形式获取。
- 在`getBean(Class<?>)`的时候，如果ioc中存在多个同类型组件时，则会抛出`NoUniqueBeanDefinitionException`异常
- 如果使用`property`对bean赋值，那么该属性必须存在**`set`**方法。



## P名称空间

ioc可以通过p名称空间进行赋值，在xml中名称空间是用来防止标签重复的。在**`beans`**标签中添加`xmlns:p="http://www.springframework.org/schema/p"`。添加后就可以使用**`p`**名称空间进行赋值。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  <bean class="top.clover.spring.bean.Person" id="person"
        p:age="19" p:name="Clover"
        p:email="2621869236@qq.com" p:gender="男"/>
</beans>
```



## 复杂属性赋值



### null值

在java中，除了基本类型，其它类型在不赋值的情况下默认值都是null。如果需要给一个属性null值，不能通过`property`标签的`value`属性来赋值，可以通过**`null`**标签。

```xml
<bean class="top.clover.spring.bean.Person">
  <property name="name">
    <null/>
  </property>
</bean>
```



### 自定义复杂类型

```java
public class Person {

  private String name;

  private Integer age;

  private String gender;

  private String email;

  private Car car;

  getter/setter....

}
```

在`Person`中，存在一个`Car`类型，这个类型属于我们自定义的复杂类型。可以使用两种方式进行赋值

#### 使用其它Bean

在`propertu`中有一个`ref`属性，这个属性值必须是某一个`bean`的id，spring可以通过这个属性将指定的`bean`对指定的属性进行赋值。

```xml
<bean class="top.clover.spring.domain.Car" id="MyCar">
  <property name="cardName" value="小轿车"/>
  <property name="color" value="五彩斑斓的黑"/>
</bean>
<bean class="top.clover.spring.bean.Person">
  <property name="car" ref="MyCar"/>
</bean>
```

```java
@Test
public void testBeanPropertyRef() {
  ApplicationContext app = new ClassPathXmlApplicationContext("ioc.xml");

  Person bean = app.getBean(Person.class);
  Car car = bean.getCar();
  System.out.println("汽车：" + car.getCardName());
  System.out.println("颜色：" + car.getColor());
}
```

```
汽车：小轿车
颜色：五彩斑斓的黑
```



#### 引用内部bean

```xml
<bean class="top.clover.spring.bean.Person">
  <property name="car">
    <bean class="top.clover.spring.domain.Car" id="MyCar">
      <property name="cardName" value="小轿车"/>
      <property name="color" value="五彩斑斓的黑"/>
    </bean>
  </property>
</bean>
```

这种方式相当于`car = new Car();`。与外部无关，所以`getBean`的时候会找不到这个car



### List类型

在ioc中对`List`类型进行赋值，可以使用**`list`**标签。

```xml
<bean class="top.clover.spring.bean.Person">
  <property name="car">
    <list>
      <bean class="top.clover.spring.domain.Car" >
        <property name="cardName" value="小轿车"/>
        <property name="color" value="五彩斑斓的黑"/>
      </bean>
      <bean class="top.clover.spring.domain.Car" >
        <property name="cardName" value="小轿车"/>
        <property name="color" value="五彩斑斓的黑"/>
      </bean>
      <bean class="top.clover.spring.domain.Car" >
        <property name="cardName" value="小轿车"/>
        <property name="color" value="五彩斑斓的黑"/>
      </bean>
    </list>

  </property>
</bean>
```

除了内部Bean的方式，还可以使用**`ref`**标签引用外部Bean。

```xml
<bean class="top.clover.spring.domain.Car" id="MyCar">
  <property name="cardName" value="小轿车"/>
  <property name="color" value="五彩斑斓的黑"/>
</bean>

<!-- 注册一个bean组件 -->
<bean class="top.clover.spring.bean.Person">
  <property name="car">
    <list>
      <ref bean="MyCar"></ref>
      <ref bean="MyCar"></ref>
      <ref bean="MyCar"></ref>
      <ref bean="MyCar"></ref>
    </list>
  </property>
</bean>
```





### Map类型

同样的，使用List类型有对应的list标签，那么Map类型也有一个map标签。spring底层使用的是`LinkedHashMap`。

```xml
<bean class="top.clover.spring.bean.Person">
  <property name="maps">
    <map>
      <entry key="name" value="Clover" />
      <entry key="age" value="19" />
    </map>
  </property>
</bean>
```

一个`entry`对于一个KV，map也提供了一个**`value-ref`**属性用于引用外部Bean。用法与其它**`ref`**属性一致。



## Util名称空间

使用这个名称空间，需要在**`beans`**中添加`xmlns:util="http://www.springframework.org/schema/util"`，还需要在xsi中添加两个与util名称相关的链接`http://www.springframework.org/schema/util， http://www.springframework.org/schema/util/spring-util-4.0.xsd`。可以为`Map`、`List`、`set`、`properties`等类型创建对应的Bean。

以map为例，`util:map`相当于new 一个`LinkedHashMap`。不需要再使用map标签

```xml
<util:map id="diyMap">
  <entry key="name" value="clover"/>
  <entry key="age" value="19"/>
</util:map>
```

```java
ApplicationContext app = new ClassPathXmlApplicationContext("ioc.xml");
Map<String, Object> diyMap =(Map<String, Object>) app.getBean("diyMap");
System.out.println(diyMap);
```

```
{name=clover, age=19}
```



## Bean之间的依赖

```xml
<bean class="top.clover.spring.bean.Person" id="person"></bean>
<bean class="top.clover.spring.bean.Book" id="book"></bean>
```

**`Bean`**的创建默认是按照配置文件中配置的先后顺序。

```
person创建了...
book被创建...
```

 如果你注册的bean需要依赖到某个bean时，若是你的bean先被创建，可能会发生不可预料的错误。那么这时候就需要用到spring提供的`depends-on`属性。它需要一个id，也可以通过**`,`**隔开传多个id。表示我当前这个bean依赖到了xxx。

```xml
<bean class="top.clover.spring.bean.Person" id="person" depends-on="book"></bean>
<bean class="top.clover.spring.bean.Book" id="book"></bean>
```

```
book被创建...
person创建了...
```



## Bean的多实例和单实例

在spring中，bean默认是单实例，若想讲bean组册为多实例，需要使用**`bean`**标签提供的`scope`属性。该属性有两个属性值

- `prototype`

  表示当前bean是多实例的

  1. 容器启动默认不会去创建bean
  2. 多实例bean只有在获取的时候被创建
  3. 每次获取都会创建一个新的对象

- `singleton`
  表示当前bean是单实例的

  1. 在容器启动完成之前就已经创建好对象保存在容器中了。
  2. 在任何时候获取都是获取之前创建好的那个对象。



## 静态工厂和实例工厂

```java
public class AirPlane {

  /**
     * 机长名称
     */
  private String captainName;

  /**
     * 副驾驶
     */
  private String coPilot;

  /**
     * 发动机
     */
  private String engine;

  /**
     * 载客量
     */
  private Integer total;

}
```

### 工厂模式

工厂是一个专门帮我们创建对象的类



### 静态工厂

工厂本身不需要创建对象。都是通过静态方法调用：`obj = xxxFactory.getxxx(String className)`

```java
public class AirPlaneStaticFactory {

  /**
     * 飞机工厂
     * @param captainName 机长名称
     * @return 飞机实例
     */
  public static AirPlane getAirPlane(String captainName) {
    AirPlane airPlane = new AirPlane();
    airPlane.setCaptainName(captainName);
    airPlane.setCoPilot("Clover");
    airPlane.setEngine("太行");
    airPlane.setTotal(300);
    return airPlane;
  }

}
```



#### 通过IOC调用静态工厂注册Bean

创建好工厂后，需要通过bean去调用这个工厂，最后将工厂的返回值注册为一个Bean对象。

**`bean`**标签**`class`**属性指向工厂全类名，需要配合**`factory-method`**属性让ioc调用工厂方法。如果工厂方法需要传参，可以使用`constructor-arg`赋值。

```xml
<bean id="AirPlane" class="top.clover.spring.factory.AirPlaneStaticFactory" factory-method="getAirPlane">
  <constructor-arg name="captainName" value="张三"/>
</bean>
```

```java
@Test
public void testAirPlaneStaticFactory() {
    AirPlane airPlaneBean = (AirPlane)app.getBean("AirPlaneByAirPlaneStaticFactory");
    System.out.println("通过静态工厂创建的bean对象：" + airPlaneBean);
}
```

```
通过静态工厂创建的bean对象：AirPlane{captainName='张三', coPilot='Clover', engine='太行', total=300}
```





### 实例工厂

工厂本身需要创建对象：`obj = new xxxFactory();`



```java
public class AirPlaneInstanceFactory {

  /**
     * 飞机工厂
     *
     * @param captainName 机长名称
     * @return 飞机实例
     */
  public AirPlane getAirPlane(String captainName) {
    AirPlane airPlane = new AirPlane();
    airPlane.setCaptainName(captainName);
    airPlane.setCoPilot("Clover");
    airPlane.setEngine("太行");
    airPlane.setTotal(300);
    return airPlane;
  }

}
```



#### 通过IOC调用实例工厂注册Bean

1. 先配置出实例工厂对象
2. 配置要创建的AirPlane使用哪个工厂创建



```xml
<!-- 创建一个实例工厂 -->
<bean id="airPlaneInstanceFactory" class="top.clover.spring.factory.AirPlaneInstanceFactory"/>
<!-- 使用指定工厂创建对象 -->
<bean id="airPlaneByAirPlaneInstanceFactory" factory-bean="airPlaneInstanceFactory" factory-method="getAirPlane">
  <constructor-arg name="captainName" value="李四"/>
</bean>
```

通过**`factory-bean`**属性来指定使用哪个工厂实例创建`bean`

```java
@Test
public void testAirPlaneInstanceFactory() {
  AirPlane airPlaneBean = (AirPlane)app.getBean("airPlaneByAirPlaneInstanceFactory");
  System.out.println("通过实例工厂创建的bean对象：" + airPlaneBean);
}
```

```
通过实例工厂创建的bean对象：AirPlane{captainName='李四', coPilot='Clover', engine='太行', total=300}
```



### FactoryBean

FactoryBean是Spring规定的一个接口，只要是这个接口接口的实现类，Spring都认为是一个工厂。

- `getObject` 获取工厂创建的对象
- `getObjectType` 获取对象类型
- `isSingleton` 当前工厂注册的Bean是否为单例

> **`FactoryBean`**创建的对象不管是单实例还是多实例，都是在获取的时候才创建对象。

```java
public class MyFactoryBean implements FactoryBean<AirPlane> {
 
  @Override
  public AirPlane getObject() throws Exception {
    AirPlane airPlane = new AirPlane();
    airPlane.setCaptainName("王五");
    airPlane.setCoPilot("Clover");
    airPlane.setEngine("太行");
    airPlane.setTotal(300);
    return airPlane;
  }

  @Override
  public Class<?> getObjectType() {
    return AirPlane.class;
  }

  @Override
  public boolean isSingleton() {
    return true;
  }

}
```

实现了`FactoryBean`接口后还需要在配置文件中进行注册

```xml
<bean id="airPlaneBySpringFactoryBean" class="top.clover.spring.bean.MyFactoryBean"/>
```

```java
@Test
public void testMyFactoryBean() {
  AirPlane airPlaneBean = (AirPlane)app.getBean("airPlaneBySpringFactoryBean");
  System.out.println(
    "通过FactoryBean创建的bean对象：" + airPlaneBean);
}
```

```
通过FactoryBean创建的bean对象：AirPlane{captainName='王五', coPilot='Clover', engine='太行', total=300}
```



## Bean的后置处理器

- 编写后置处理器实现类`BeanPostProcessor`
- 将后置处理器注册在配置文件中



```java
public class MyBeanPostProcessor implements BeanPostProcessor {

  /**
     * bean在初始化的时候会调用这个方法
     * @param bean bean对象
     * @param beanName bean注册的名称
     * @return 要注册的bean对象
     */
  @Override
  public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    System.out.println("「"+ bean +"」bean准备开始初始化了 --->> " + beanName);
    return bean;
  }

  /**
     * 初始化方法之后被调用
     * @param bean bean对象
     * @param beanName bean名称
     * @return 要注册的bean对象
     */
  @Override
  public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
    System.out.println("「"+ bean +"」bean初始化方法调用完了 --->> " + beanName);
    return bean;
  }

}
```

在配置文件注册

```xml
<bean class="top.clover.spring.bean.MyBeanPostProcessor"/>
```

```
「AirPlane{captainName='张三', coPilot='Clover', engine='太行', total=300}」bean准备开始初始化了 --->> airPlaneByAirPlaneStaticFactory
「AirPlane{captainName='张三', coPilot='Clover', engine='太行', total=300}」bean初始化方法调用完了 --->> airPlaneByAirPlaneStaticFactory
```



## Dao、Controller、Service

通过给bean上添加某些注解，可以快速的将bean加入到ioc容器中。
分别使用Spring提供的这四个注解

- **`@Controller`**
  用来给控制层加上

- **`@Service`**
  业务逻辑层的组件添加这个注解

- **`@Repository`**
  给数据库层（持久化层/Dao层）添加这个注解

- **`@Component`**

  可以给有功能的类添加上这个注解，表示这个类是一个组件

> 注解可以随便加，Spring底层不会去验证你的这个组件是否如你注解所说就是一个dao层或者就是一个servlet层的组件。

使用注解将组件快速的加入到容器需要这几步：

1. 给组件上标注所需注解
2. 告诉Spring自动扫描加了注解的组件。这个功能依赖了context名称空间
3. 导入aop包，这个包支持注解模式



在配置文件中加入`context`名称空间：`xmlns:context="http://www.springframework.org/schema/context"`

加入之后这个名称空间提供了一个**`context:compnent-scan`**标签，用来自动扫描组件。这个标签中有一个**`base-package`**属性，用来指定扫描的基础包，把基础包及子包下的所有加了注解的类自动的扫描进ioc容器中。



> 默认注册为单例模式。若想注册为多例，需要使用**`@Scope`**注解





## @Autowired自动装配

被`@Autowired`标注的属性，spring会根据类型到ioc容器中查找并赋值给被标注的属性。前提是你这个属性所在的类和目标类型已经被注册进容器中。

```java
@Controller
public class BookController {
  
  @Autowired
  private BookService bookService;

  public void doGet() {
    bookService.saveBook();
  }

}
```

```java
@Service
public class BookService {
  
  @Autowired
  private BookDao bookDao;

  public void saveBook() {
    System.out.println("调用bookDao中的saveBook保存图书....");
    bookDao.saveBook();
  }

}
```

```java
@Repository
public class BookDao {

  public void saveBook() {
    System.out.println("保存书籍成功....");
  }

}
```

测试这个controller

```java
/**
 * 测试自动注入
 */
@Test
public void testAutowired() {
  BookController controller = app.getBean(BookController.class);
  controller.doGet();
}
```

```
调用bookDao中的saveBook保存图书....
保存书籍成功....
```

### 在方法上使用

`@Autowired`可以标注在很多地方

> ```java
> @Target({ElementType.CONSTRUCTOR, ElementType.FIELD, ElementType.METHOD, ElementType.ANNOTATION_TYPE})
> ```

如果作用在方法上

- 这个方法会在bean创建的时候自动运行
- 这个方法上的每一个参数都会自动注入



```java
@Controller
public class BookController {
  
  @Autowired
  public void testAutowired(BookService bookService) {
    System.out.println("容器创建，autowired运行：---》"+bookService);
  }

}
```

```
容器创建，autowired运行：---》top.ctong.service.BookService@5e21e98f
```





### 大致原理

- `@Autowired` 默认按照类型去容器中找到对应的组件
  - 找到一个就直接赋值
  - 没有找到就抛异常`BeanCreationException`
  - 如果找到多个
    1. 会根据变量名最为id再找一次`app.getBean(String id)`
    2. 如果这时候还是匹配不上，那么还是抛出错误。



> `@Qualifier` 可以修改`@Autowired`的哦人行为，`@Autowired`会根据`@Qualifier` 的值进行查找。`app.getBean(String id);`

---

自动装配有三种

1. `@Autowired`
   这是spring自己的注解，相比其它两个，它更强大
2. `@Resource`
   这是javax提供的，是一个标准。相比其它，他的扩展性更强。如果换成其它框架，`@Resource`还可以使用，`@Autowired`可能就不行了。
3. `@Inject`

### 泛型注入

在同类型组件中，`@Autowired`可以根据泛型去匹配

例如有一个`BaseDao`

```java
public abstract class BaseDao<T>  {
  abstract void save();
}
```

有两个类实现`BaseDao`

```java
@Repository
public class BookDao extends BaseDao<Book> {
  @Override
  public void save() {
    System.out.println("保存图书");
  }
}
```

```java
@Repository
public class UserDao extends BaseDao<User> {
  @Override
  public void save() {
    System.out.println("用户保存");
  }
}
```

有一个`BaseService`去使用泛型注入对应Dao

```java
public class BaseService<T> {

  @Autowired
  private BaseDao<T> baseDao;

  public void save() {
    System.out.println("baseDao类型是：" + baseDao);
  }

}
```

分别有`UserService`和`BookService`继承`BaseService`

```java
@Service
public class BookService extends BaseService<Book>{
  public void saveBook() {
    System.out.println("调用bookDao中的saveBook保存图书....");
  }
}
```

```java
@Service
public class UserService extends BaseService<User>{
  public UserService() {
    // 防止存在一个或多个有参构造器时反射通过无参构造起实例化发生异常
  }
}
```

最后测试结果

```java
@Test
public void testBaseService() {
  UserService userBean = app.getBean(UserService.class);
  BookService BookBean = app.getBean(BookService.class);
  userBean.save();
  BookBean.save();
}
```

```
baseDao类型是：top.ctong.dao.UserDao@6692b6c6
baseDao类型是：top.ctong.dao.BookDao@1cd629b3
```

 

## AOP

AOP(Aspect Oriented Programming或面向切面编程) 。这是一种新的编程思想。他不是用来替代OOP的，他是一种基于OOP基础之上衍生出的一种新的思想。

面向切面编程是指在程序运行期间，**将某段代码动态的切入**到指定方法的指定位置进行运行。



## AOP与动态代理

1. 动态代理写起来代码复杂
2. jdk默认的动态代理，如果目标对象没有实现任何接口，是无法为它创建代理对象的。
3. spring实现的AOP功能，底层就是动态代理
4. spring可以一句代码都不写的去创建动态代理 
5. spring实现简单，而且没有强制要求目标对象必须实现接口
6. 原生动态代理就是切面编程，而Spring简化了面向切面编程



## AOP的几个专业术语

- 横切关注点
  每一个方法的开始

- 通知方法
  每一个横切关注点触发的方法

- 切面类
  通知方法所在的类

- 连接点
  每一个方法的每一个位置都是一个连接点

- 切入点

  在众多连接点中选出我们感兴趣的地方。例如：方法结束需要记录日志，方法异常需要记录日志，方法返回需要记录日志的地方。

- 切入点表达式

![AOP专业术语](http://qiniu-note-image.ctong.top/note/images/202112271048731.png)



## 导包

若想spring支持面向切面编程，需要导入`spring-aspects-4.0.0.RELEASE.jar`，这是spring提供的基础版jar包。

由于spring提供的包比较基础，可以使用第三方加强包「即使目标对象没有实现任何接口也能创建动态代理」。

```
com.springsource.org.aopalliance-1.0.0.jar
com.springsource.net.sf.cglib-2.2.0.jar
com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar
```



## 配置

- 将目标类和切面类（装了通知方法的类）加入到ioc容器

  ```java
  @Service
  public class MyMathCalculator implements Calculator {
  
    @Override
    public int add(int i, int j) {
      return i + j;
    }
  
    @Override
    public int sub(int i, int j) {
      return i - j;
    }
  
    @Override
    public int mul(int i, int j) {
      return i * j;
    }
  
    @Override
    public int div(int i, int j) {
      return i / j;
    }
  
  }
  ```

- 使用`@Aspect`注解告诉spring哪个是切面类

  ```java
  @Aspect
  @Component
  public class LogUtils {
    /**
       * 方法执行之前的日志记录器
       */
    @Before("execution(public int top.ctong.aop.impl.MyMathCalculator.*(int, int))")
    public static void methodStartBefore() {
      System.out.println("方法执行前....");
    }
  
    @After("execution(public int top.ctong.aop.impl.MyMathCalculator.*(int, int)))")
    public static void methodAfter() {
      System.out.println("方法执行完成....");
    }
  
    @AfterReturning("execution(public int top.ctong.aop.impl.MyMathCalculator.*(int, int)))")
    public static void methodReturning() {
      System.out.println("方法执行返回值....");
    }
  
    @AfterThrowing("execution(public int top.ctong.aop.impl.MyMathCalculator.*(int, int)))")
    public static void methodException() {
      System.out.println("方法执行完成....");
    }
  }
  ```

  

- 告诉spring切面类里面的每一个方法都什么时候运行。

  1. `@Before` 在目标方法执行之前运行「前置通知」
  2. `@After`在目标方法结束之后运行「后置通知」
  3. `@AfterReturning` 在目标方法正常返回之后运行「返回通知」
  4. `@AfterThrowing` 在目标方法抛出异常之后运行「异常通知」
  5. `@Around` 环绕「环绕通知」

- 标注好所需注解后，还需要告诉spring，你这个切面在哪里运行，需要写切入点表达式

  ```java
  excution(访问权限符 返回值类型 方法签名)
  ```

- 在配置文件中 开启基于注解的AOP功能

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xmlns:aop="http://www.springframework.org/schema/aop"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
         http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
      <context:component-scan base-package="top.ctong"/>
      <aop:aspectj-autoproxy />
  </beans>
  ```





## 测试

```java
@Test
public void testLogUtilsAspect() {
  Calculator bean = this.app.getBean(Calculator.class);
  bean.add(1,1);
}
```

```
方法执行前....
方法执行完成....
方法执行返回值....
```

注意：当前`bean`还是一个代理对象，因为AOP底层就是原生`Proxy`，所以不能通过`MyMathCalculator.class`来获取。但当被代理类没有任何父类的时候，又可以通过`MyMathCalculator.class`进行获取。



在没有任何父类的时候，是cglib帮我们创建的代理对象。cglib为我们创建的一个内部类，内部类中实现了所有`MyMathCalculator`的方法

`class top.ctong.aop.impl.MyMathCalculator$$EnhancerByCGLIB$$1a9d9ab2`



## 切入点表达式

切入点表达式的固定写法

```java
excution(访问权限符 返回值类型 方法签名(参数列表))
```



### 通配符

- **`*`**
  1.  匹配一个或多个字符
  2. 匹配任意一个参数
  3. 如果放在路径上，那么只能匹配一层路径
  4. 权限不能表示
- **`..`**
  1. 匹配任意多个参数，任意类型参数
  2. 匹配任意多层路径
-  **`&&`**切入的位置要同时满足两种表达式
- **`||`**切入的位置最少要满足一种表达式
- **`!`**不满足指定表达式触发



### 抽取表达式

抽取可重用的切入点表达式

1. 随便声明一个没有实现的返回`void`的空方法。
2. 给方法上标注`@Pointcut`注解

```java
@Aspect
@Component
public class AspectLogUtils {

  @Pointcut("execution(* top.ctong..add(..))")
  public static void aspectAddMethod() {
    // 抽取切入点表达式
  }

  @After("aspectAddMethod()")
  public static void aspectAllAddMethod() {
    System.out.println("方法执行了...");
  }

}
```



## 目标方法信息

只需要为通知方法的参数列表上写一个`JoinPoint`类型的参数。它封装了目标方法的详细信息

```java
@After("execution(* top.ctong.aop.impl.MyMathCalculator.*(..))")
public static void methodAfter(JoinPoint joinPoint) {
  // 获取方法签名
  Signature signature = joinPoint.getSignature();
  System.out.println("「" + signature.getName() + "」方法执行完成....使用的参数：「" + Arrays.asList(joinPoint.getArgs()) + "」");
}
```

```
「add方法执行完成....使用的参数：「[1, 1]」
```

获取方法返回值，需要在通知注解中指定，在`@AfterReturning`注解中，有一个`returning`属性，用来指定参数名

```java
@AfterReturning(value = "execution(public int top.ctong.aop.impl.MyMathCalculator.*(int, int)))", returning = "result")
public static void methodReturning(JoinPoint joinPoint, Object result) {
  // 获取方法签名
  Signature signature = joinPoint.getSignature();
  System.out.println("「" + signature.getName() + "」方法执行返回值===>>" + result);
}
```

获取异常和获取返回值操作一样

```java
@AfterThrowing(value = "execution(* top.ctong.aop.impl.MyMathCalculator.*(..))", throwing = "exception")
public static void methodException(Exception exception) {
  exception.getCause();
}
```



## Around环绕通知

`@Around`是spring最强大的通知。它相当于我们手写动态代理。

```java
@Around("aspectAddMethod()")
public Object aspectAround(ProceedingJoinPoint pjp) throws Throwable {
  Object[] args = pjp.getArgs();
  // 类似method.invoke()
  return pjp.proceed(args);
}
```



## 基于配置文件的AOP

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
  <bean id="myMathCalculator" class="top.ctong.utils.MyMathCalculator"></bean>
  <bean id="aspectLogUtils" class="top.ctong.utils.AspectLogUtils"/>

  <aop:config>
    <!--  指定切面  -->
    <aop:aspect ref="aspectLogUtils">
      <!-- 抽取表达式 -->
      <aop:pointcut id="MyMathCalculatorAllMethod" expression="execution(* top.ctong..MyMathCalculator.*(..))"/>
      <aop:before method="aspectBefore" pointcut="execution(* top.ctong..MyMathCalculator.*(..))"/>
      <aop:after method="aspectAfter" pointcut-ref="MyMathCalculatorAllMethod"/>
      <aop:after-throwing method="aspectThrowing" pointcut-ref="MyMathCalculatorAllMethod"/>
      <aop:after-returning method="aspectReturning" pointcut-ref="MyMathCalculatorAllMethod"/>
    </aop:aspect>
  </aop:config>
</beans>
```

若想使用基于配置文件的aop，需要引入`aop`空间。

- `aop:aspect`指定切面类
- `aop:pointcut`抽取切入点表达式，这个标签如果放在`aop:aspect`中，那么只能在当前切面可使用，若想全局使用，可以将其放置在切面外，也就是`aop:config`直接子级。
- `aop:before`指定前置切入
- `aop:after`指定后置切入
- `aop:after-throwing` 指定异常切入
- `aop:after-returning`指定返回时切入





## SpringIOC源码

1. IOC是一个容器
2. 容器启动的时候创建所有单实例对象
3. 我们可以直接从容器中获取到这个对象



### SpringIOC启动过程

- `ClassPathXmlApplicationContext`所有构造器调用的都是这个构造器

  ```java
  public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
    throws BeansException {
  
    super(parent);
    setConfigLocations(configLocations);
    if (refresh) {
      refresh();
    }
  }
  ```

- 在构造器中，最重要的代码是 `refresh();`。它负责创建所有单实例bean和启动容器

  1. 在第一行代码中，`synchronized (this.startupShutdownMonitor) ` 有个同步锁，是为了保证在多线程环境中IOC容器只被创建一次。
		
      >  生产环境默认是多线程的。
  
  2. `prepareRefresh();`解析配置文件，以准备刷新上下文
  
  3. `ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();`准备一个Bean工厂，在此工厂中准备好了待初始化的Bean、系统环境信息等基础信息用于后来创建Bean。也就是说在这一步已经解析好了xml配置文件。
      ![ConfigurableListableBeanFactory继承图](http://qiniu-note-image.ctong.top/note/images/202112271048736.png)
  
  4. `prepareBeanFactory(beanFactory);`设置工厂的所有可能用到的外置工厂，例如注解解析器，类加载器等其它工厂
  
  5. `postProcessBeanFactory(beanFactory)`工厂的后置处理设置
  
  6. `invokeBeanFactoryPostProcessors(beanFactory);`调用bean工厂的后置处理器
  
  7. `registerBeanPostProcessors(beanFactory);`注册所有`BeanPostProcessor.class`类型的Bean。也就是说在这一步把所有`BeanPostProcessor.class`实例化。
  
  8. `initMessageSource();`支持国际化语言服务
  
  9. `initApplicationEventMulticaster();` 初始化事件传唤器，spring有创建、销毁bean时会产生非常多的事件，转换器是为了能让其它组件感知到spring触发了哪个事件。
  
  10. `onRefresh();`这是一个空的方法，留给子类的一个初始化方法。例如你重写IOC容器，可以重写这个方法进行初始化某些必要的Bean。
  
  11. `registerListeners();`注册监听器，注册的是`ApplicationListener.class`类型的监听器，这是一个spring事件监听器。
  
  12. `finishBeanFactoryInitialization(beanFactory);`完成所有剩余的bean实例的初始化
  
       1. `beanFactory.preInstantiateSingletons();`初始化所有剩余的bean
  
          > `org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons`
  
          1. `RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);`根据id拿到Bean的定义信息。里面包含了bean所有的详细信息。
  
          2. `if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) { ... }`如果当前bean不是一个抽象类，并且是单实例和不是懒加载。那么就创建它。
  
             > 懒加载就是在使用的时候才创建。
  
          3. `if (isFactoryBean(beanName)) {...}
             else {getBean(beanName);}`如果是工厂bean那么就通过工厂去创建这个bean。否则直接创建。
  
          4. 所有的`getBean`调用的都是`org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean`，直接看它就好
  
             1. `final String beanName = transformedBeanName(name);`获取Bean名称
  
             2. `Object sharedInstance = getSingleton(beanName);`从单例缓存池中检查是否存在这个bean。如果存在就直接拿这个bean。
  
             3. `final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);`获取当前bean的定义信息。
  
             4. `String[] dependsOn = mbd.getDependsOn();`获取当前bean是有依赖了谁。也就是在xml文件中bean标签的`depends-on`属性。如果有那么循环它，再调用`getBean`方法。也就是说，先创建好所依赖的对象再创建自己。
  
                ```java
                String[] dependsOn = mbd.getDependsOn();
                if (dependsOn != null) {
                  for (String dependsOnBean : dependsOn) {
                    if (isDependent(beanName, dependsOnBean)) {
                      throw new BeanCreationException("Circular depends-on relationship between '" +
                                                      beanName + "' and '" + dependsOnBean + "'");
                    }
                    registerDependentBean(dependsOnBean, beanName);
                    getBean(dependsOnBean);
                  }
                }
                ```
  
             5. `sharedInstance = getSingleton(...)` 创建bean。
  
                1. `singletonObject = singletonFactory.getObject();`一系列验证当前bean不存在之后开始创建。
  
                   > `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean`
                   >
                   > 通过反射创建：
                   >
                   > `org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBeanInstance`
  
                2. `addSingleton(beanName, singletonObject);`将创建好的bean对象放到单实例缓存池中
  
                   > 在项目中使用`getBean`其实也是在这个缓存池中获取对应的bean
  
          

## BeanFactory和ApplicationContext的区别

ApplicationContext是BeanFactory的子接口

- BeanFactory是一个工厂接口，也是Spring最底层的接口。负责创建Bean实例，容器里面保存的所有单例bean其实都是在一个map中。
- ApplicationContext是容器接口，更多的是负责容器功能的实现。（可以基于BeanFactory创建好的对象之上完成强大的容器）容器可以从map中获取这个bean，并且aop、di在ApplicationContext接口的下面的这些类里面。



> BeanFactory是最底层的接口，而ApplicationContext更多是留给我们使用的ioc容器接口。