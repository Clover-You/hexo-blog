title: 手撸Spring整合MyBatis玩具
categories: 随笔
tags: [SpringMvc,随笔]
date: 2022-01-02 18:17:52
---
源码在我GitHub上，有兴趣可以去观望一下：[传送门](https://github.com/YouChuantong/mybatis_learn_demo)

这个只是个**玩具、玩具、玩具**，正经的还得看官方提供的整合包
先说下这个玩具的问题所在吧！====>>> 无法提交事务和无法关闭SqlSession，嗯嗯嗯~~~

开始吧～～

---

## 难点

若想使用Spring容器提供的强大功能，例如：`Autowired`，那么就需要让Spring容器将其管理。

没整合之前，每次都需要通过`SqlSessionFactory`将其创建，一个两个还好，写多了整个人都麻了...

所以我就想，能不能在service层直接注入呢？

让Spring注入的话就得注册到容器，Dao层数量和命名都无法确定，怎么注册Bean呢？..嘶～～～

~~嗯。。。使用工厂模式加上自动扫包好像能实现..~~

说干就干...

```java
@Configuration
public class MyBatisSqlSessionFactory implements Serializable {

    private static final long serialVersionUID = 7157094465332447639L;

    /**
     * MyBatis配置文件路径
     */
    private static final String MYBATIS_CONFIG_PATH = "mybatis-config.xml";

    /**
     * 注册一个全局SqlSessionFactory组件
     */
    @Bean
    public SqlSessionFactory getSqlSessionFactory() throws IOException {
        System.out.println("getSqlSessionFactory");
        try (InputStream configStream = getConfigStream()) {
            return new SqlSessionFactoryBuilder().build(configStream);
        }
    }

    /**
     * 懒加载Bean工厂
     * @param daoClass Bean类型
     * @param <T> Bean类型
     */
    @Bean
    public static <T> T getMapping(Class<T> daoClass) {
        SpringUtils springUtils = new SpringUtils();
        SqlSessionFactory bean = springUtils.getBean(SqlSessionFactory.class);
        SqlSession sqlSession = bean.openSession();
        return sqlSession.getMapper(daoClass);
    }

    /**
     * 获取配置文件文件流
     * @return 文件流
     */
    public InputStream getConfigStream() throws IOException {
        return Resources.getResourceAsStream(MYBATIS_CONFIG_PATH);
    }

}

```



嗯？我该怎么为`getMapping`注册为懒加载呢???还有`Class<T> daoClass`我该怎么拿到当前`getBean`时要找的类型呢？

嘶～～～

嘶～～～

嘶～～～



## 更换方案

翻了翻`ClassPathXmlApplication`源码，启动的时候它去执行了个后置处理器`org.springframework.beans.factory.config.BeanFactoryPostProcessor`。这个处理器需要实现一个`postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory)`，这个`ConfigurableListableBeanFactory`就是那个还未启动完成的容器。嗯嗯嗯...

它里面有一个`registerSingleton`方法，可以注册单实例Bean，可我需要注册多实例...先不管了

```java
@Override
public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory)
  throws BeansException {
  this.configurableListableBeanFactory = configurableListableBeanFactory;
  if (factory == null) {
    factory = configurableListableBeanFactory.getBean(SqlSessionFactory.class);
  }
  RegisterMyBatisFactory registerMyBatisFactory = configurableListableBeanFactory.getBean(
    RegisterMyBatisFactory.class);

  if (registerMyBatisFactory == null) return;
  String mapperPackage = registerMyBatisFactory.getPackage();
  // 获取指定包下所有class
  List<Class<?>> allClass = getClassByPackage(mapperPackage);
  // 通过@Mapper过滤无效class
  List<Class<?>> interfaceByWithMyBatis = getInterfaceByMapper(allClass);
  // 将所有Mapper注册到IOC中
  registerMyBatis(interfaceByWithMyBatis);
}
```



- ` this.configurableListableBeanFactory = configurableListableBeanFactory;`  保存这个容器

- `RegisterMyBatisFactory registerMyBatisFactory = configurableListableBeanFactory.getBean(RegisterMyBatisFactory.class);` 这里定义了个接口，通过这个借口拿到Dao所在的包。

  ```java
  /**
   * mybatis，将其注册在容器中，整合器就可以获取到指定包名
   */
  public interface RegisterMyBatisFactory {
  
    /**
     * 获取mapper所在的包
     */
    String getPackage();
  
  }
  ```

- `List<Class<?>> allClass = getClassByPackage(mapperPackage);` 通过包名找这个包下所有的类，这个方法就不说了，就一个普通的文件查找而已。有兴趣就拉源码吧！

- `List<Class<?>> interfaceByWithMyBatis = getInterfaceByMapper(allClass);`需要过滤没有用的类，也就是说可能不是Dao。通过MyBatis提供的`@Mapper`注解来识别了，懒得自己写注解，先把功能实现。就一个简简单单的反射

  ```java
  /**
   * 通过指定注解获取接口
   * @return 过滤结果
   */
  private List<Class<?>> getInterfaceByMapper(List<Class<?>> clazz) {
    return clazz.stream().filter(aClass -> {
      if (!aClass.isInterface()) return false;
      Mapper annotation = aClass.getAnnotation(Mapper.class);
      return annotation != null;
    }).collect(Collectors.toList());
  }
  ```

- `registerMyBatis(interfaceByWithMyBatis);` 接下来就将过滤后的Mapper注册到容器中了。如果是注册的多实例Bean，那么就可以通过切面去搞定这个事务问题。

  ```java
  /**
   * 将MyBatis注册到IOC
   */
  private void registerMyBatis(List<Class<?>> mappers) {
    mappers.forEach(clazz -> {
      SqlSession sqlSession = factory.openSession();
      Object mapper = sqlSession.getMapper(clazz);
      configurableListableBeanFactory.registerSingleton(clazz.getName(), mapper);
    });
  }
  ```





## 使用演示

Dao层用`@Mapper`标注

```java
@Mapper
public interface EmployeeDao extends GenericDao<Employee> {...}
```

service层直接注入。

```java
@Service
public class EmployeeServiceImpl implements Serializable, EmployeeService {

    private static final long serialVersionUID = 1863674392751816610L;

    @Autowired
    private EmployeeDao employeeDao;
}
```

现在就可以了.....

```json
{"id":1,"empName":"Clover","gender":0,"email":"cloveryou.ctong@qq.com"}
```


