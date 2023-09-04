title: MyBatis - 学习笔记
categories: 其它
tags: [学习笔记,MySQL,JAVA,SQL]
date: 2022-01-02 18:10:21
---
# MyBatis 简介

- 原是Apache的一个开源项目iBatis，2010年6月这个项目由Apache Software Foundation迁移到Google Code，随着开发团队转投Google Code旗下。iBatis3.x正式更名为MyBatis，代码于2013年11月迁移到Github
- iBatis一词来源于“internet”和“abatis”的组合。是一个基于Java的持久层框架。iBatis提供的持久层框架包括SQL Maps和Data Access Objects（DAO）

- MyBatis是支持定制化SQL、存储过程以及高级映射的优秀持久层框架
- MyBatis避免了几乎所有的JDBC代码的手动设置参数以及获取结果集
- MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO映射成数据库中的记录

[Github](https://github.com/mybatis/mybatis-3/releases)



# HelloWorld

[MyBatis Demo Github](https://github.com/YouChuantong/mybatis_learn_demo)

 

## 导包

```xml
<!--  jdbc包  -->
mysql-connector-java-8.0.21.jar
<!--  mybatis核心包  -->
mybatis-3.5.7.jar
<!-- 日志框架，导入这个包之后在关键的位置可以打印日志信息，若想使用这个日志框架，需要在类路径下存在一个log4j.xml配置文件 -->
log4j-1.2.17.jar
```



## 配置

1. 第一个配置文件被称为mybatis的全局配置文件，指导mybatis如何正确运行。比如连接到哪个数据库。
2. 第二个配置文件，用于编写每一个方法都如何向数据库发送sql语句、如何执行....可以看作接口的实现类

### 第一个配置文件

在`resources`文件夹下创建一个`jdbc.properties`文件，用于配置jdbc信息。

```properties
# 用户名
jdbc.user=root
# 密码
jdbc.pass=123
# 驱动
jdbc.driver=com.mysql.cj.jdbc.Driver
# 连接地址
jdbc.url=jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=UTF8&zeroDateTimeBehavior=convertToNull&serverTimezone=Asia/Shanghai
```

在`resources`文件夹下创建mybatis的配置文件`mybatis-config.xml`，

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <!-- 配置一个资源配置文件 -->
    <properties resource="jdbc.properties"/>

    <settings>
        <!--    开启驼峰命名自动映射    -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
  
    <typeAliases>
        <!-- 这里配置的是一个类型 -->
        <package name="com.ctong.learn.domain"/>

    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <!--  配置连接池  -->
            <dataSource type="POOLED">
                <!-- 这里根据`jdbc.properties`的配置来配置连接数据库 -->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.user}"/>
                <property name="password" value="${jdbc.pass}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <!-- 这里配置的是你的dao层的包名 -->
        <package name="com.ctong.learn.dao"/>
    </mappers>
</configuration>
```



### 第二个配置文件

假如有这么一个类`top.ctong.learn.domain.Employee`，他是数据库中的某个表的映射:

```java
@Data
public class Employee {

    /**
     * id
     */
    private Integer id;

    /**
     * 员工名
     */
    private String empName;

    /**
     * 性别
     */
    private Short gender;

    /**
     * 邮箱
     */
    private String email;

}
```

那么对应的，需要为这张表创建一个dao层`top.ctong.learn.dao.EmployeeDao`

```java
@Mapper
public interface EmployeeDao extends GenericDao<Employee> { }
```

> `GenericDao`接口只是提供了常用的crud操作。



正常情况下需要为`EmployeeDao`创建一个实现类，但是这个实现类是通过配置的方式，让mybatis帮我们创建。在`resources`文件夹中创建这个配置文件`resources/mapping/EmployeeDao.xml`。`namespace`是用来指定接口的全类名，这样mybatis才知道它这个配置实现的是哪个接口。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.ctong.learn.dao.EmployeeDao">
</mapper>
```

写完配置文件后mybatis还是不知道这个文件是干嘛用的，所以需要到`mybati-config.xml`配置文件中注册它。

```xml
<configuration>
  <mappers>
    <!--    引入每一个接口的实现配置文件    -->
    <mapper resource="mapper/EmployeeDao.xml" />
  </mappers>
</configuration>
```



### 测试

写一个简单查询。

- `id` 方法名，相当于这个配置是对某个方法的实现
- `resultType`  指定方法执行后的返回值类型，查询操作必须指定返回值类型。
- `#{id}`代表取出传递过来的某个参数值

```xml
<!--  T query(Integer id);  -->
<select id="query" resultType="top.ctong.learn.domain.Employee">
  select *
  from t_employee
  where id = #{id}
</select>
```

> 每个基于 MyBatis 的应用都是以一个 SqlSessionFactory 的实例为核心的。SqlSessionFactory 的实例可以通过 SqlSessionFactoryBuilder 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。

需要通过`SqlSessionFactory`来创建一个`SqlSession`

- `SqlSessionFactory` 是SqlSession工厂，负责创建SqlSession对象。

- `SqlSession` 代表sql会话（和数据库的一次会话）

```java
public class TestEmployeeDao {

  private SqlSessionFactory sqlSessionFactory;

  /**
     * MyBatis配置文件路径
     */
  private static final String MYBATIS_CONFIG_PATH = "mybatis-config.xml";

  /**
   * 初始化SqlSessionFactory
   */
  @Before
  public void initMyBatis() throws IOException {
    InputStream resourceAsStream = Resources.getResourceAsStream(MYBATIS_CONFIG_PATH);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
  }

}
```

测试

```java
/**
 * 测试查询接口
 */
@Test
public void testQuery() {
  SqlSession sqlSession = sqlSessionFactory.openSession();
  // 获取Dao接口实现
  EmployeeDao mapper = sqlSession.getMapper(EmployeeDao.class);
  Employee param = new Employee();
  param.setId(8);
  Employee query = mapper.query(param);
  //        Employee query = mapper.query(8);
  System.out.println(query.toString());
}
```

结果

```json
{"id":8,"empName":"张三它亲戚李四","gender":0,"email":"lisi@qq.com"}
```

---

1. `mybatis-config.xml`是全局配置文件。指导MyBatis正确运行的一些全局设置
2. `EmployeeDao.xml`是一个SQL映射文件，可以看作是对Dao层接口的一个实现描述，实际上MyBatis会根据这个文件为`EmployeeDao.java`创建一个代理对象。
3. `SqlSessionFactory` 用来创建`SqlSession`对象，Factory只需要new一次就行
4. `SqlSession` 相当于使用`connection`和数据库进行交互，称为“和数据库的一次会话”。既然是和数据库的**一次**会话，那么每次使用时就应该创建一个新的`SqlSession`



# 全局配置文件

MyBatis的配置文件包含了会深深影响MyBatis的设置和属性信息。

- configuration（配置）
  - properties（属性）
  - settings（设置）
  - typeAliases（类型别名）
  - typeHandlers（类型处理器）
  - objectFactory（对象工厂）
  - plugins（插件）
  - environments（环境配置）
    - environment（环境变量）
      - transactionManager（事务管理器）
      - dataSource（数据源）
  - databaseIdProvider（数据库厂商标识）
  - mappers（映射器）



## 属性(properties)

引用外部配置文件，除了引入外部配置文件外，还可以在`properties`元素的子元素中进行设置

- `resource`： 引用指定类路径下的properties文件，不需要通过`classpath:`指定
- `url`： 引用磁盘路径或网络路径中的资源文件

```xml
<properties resource="jdbc-config.properties">
    <property name="jdbc.username" value="root"/>
</properties>
```

在MyBatis配置文件中，可以通过`${xx}`的方式取出`properties`引入或定义的属性值例如:`${jdbc.username}`



## 设置(settings)

以下这些是MyBatis中非常重要的设置，他可以改变MyBatis运行时默认行为。

| 设置名                           | 描述                                                         | 有效值                                                       | 默认值                                                |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------- |
| cacheEnabled                     | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true                                                  |
| lazyLoadingEnabled               | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false                                                 |
| aggressiveLazyLoading            | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false                                                | false （在 3.4.1 及之前的版本中默认为 true）          |
| multipleResultSetsEnabled        | 是否允许单个语句返回多结果集（需要数据库驱动支持）。         | true \| false                                                | true                                                  |
| useColumnLabel                   | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。 | true \| false                                                | true                                                  |
| useGeneratedKeys                 | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。 | true \| false                                                | False                                                 |
| autoMappingBehavior              | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                               |
| autoMappingUnknownColumnBehavior | 指定发现自动映射目标未知列（或未知属性类型）的行为。`NONE`: 不做任何反应`WARNING`: 输出警告日志（`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`）`FAILING`: 映射失败 (抛出 `SqlSessionException`) | NONE, WARNING, FAILING                                       | NONE                                                  |
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                |
| defaultStatementTimeout          | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。         | 任意正整数                                                   | 未设置 (null)                                         |
| defaultFetchSize                 | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。 | 任意正整数                                                   | 未设置 (null)                                         |
| defaultResultSetType             | 指定语句默认的滚动策略。（新增于 3.5.2）                     | FORWARD_ONLY \| SCROLL_SENSITIVE \| SCROLL_INSENSITIVE \| DEFAULT（等同于未设置） | 未设置 (null)                                         |
| safeRowBoundsEnabled             | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true \| false                                                | False                                                 |
| safeResultHandlerEnabled         | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。 | true \| false                                                | True                                                  |
| mapUnderscoreToCamelCase         | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | False                                                 |
| localCacheScope                  | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。 | SESSION \| STATEMENT                                         | SESSION                                               |
| jdbcTypeForNull                  | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。              | OTHER                                                 |
| lazyLoadTriggerMethods           | 指定对象的哪些方法触发一次延迟加载。                         | 用逗号分隔的方法列表。                                       | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage         | 指定动态 SQL 生成使用的默认脚本语言。                        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler           | 指定 Enum 使用的默认 `TypeHandler` 。（新增于 3.4.5）        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls               | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 | true \| false                                                | false                                                 |
| returnInstanceForEmptyRow        | 当返回行的所有列都是空时，MyBatis默认返回 `null`。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2） | true \| false                                                | false                                                 |
| logPrefix                        | 指定 MyBatis 增加到日志名称的前缀。                          | 任何字符串                                                   | 未设置                                                |
| logImpl                          | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置                                                |
| proxyFactory                     | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。            | CGLIB \| JAVASSIST                                           | JAVASSIST （MyBatis 3.3 以上）                        |
| vfsImpl                          | 指定 VFS 的实现                                              | 自定义 VFS 的实现的类全限定名，以逗号分隔。                  | 未设置                                                |
| useActualParamName               | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 `-parameters` 选项。（新增于 3.4.1） | true \| false                                                | true                                                  |
| configurationFactory             | 指定一个提供 `Configuration` 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为`static Configuration getConfiguration()` 的方法。（新增于 3.2.3） | 一个类型别名或完全限定类名。                                 | 未设置                                                |
| shrinkWhitespacesInSql           | 从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5) | true \| false                                                | false                                                 |
| defaultSqlProviderType           | Specifies an sql provider class that holds provider method (Since 3.5.6). This class apply to the `type`(or `value`) attribute on sql provider annotation(e.g. `@SelectProvider`), when these attribute was omitted. | A type alias or fully qualified class name                   | Not set                                               |

在MyBatis中，默认不会开启驼峰命名规则，例如查询时，数据库字段为`emp_name`，而映射实体为`empName`。那么这种情况下，MyBatis默认是封装不进去的，因为他们字段不对应，可以开启驼峰命名规则，以让MyBatis能正确封装我们的实体。

- `name`：设置属性名称
- `value`：属性值

```xml
<settings>
  <setting name="mapUnderscoreToCamelCase" value="true"/>
</settings>
```



## 类型别名(typeAliases)

在MyBatis中，需要通过全类名的方法去指定一个类型，例如：

```xml
<insert parameterType="top.ctong.learn.domain.Employee"></insert>
```

如果直接通过类名的方式，是无法找到这个类的，这时，可以在配置文件`typeAliases`元素中配置你每一个类型的别名，MyBatis可以通过这些别名快速定位到指定类型。

- `type` ：类型完整类名
- `alias`：类型别名，使用时别名对大小写不敏感

```xml
<typeAliases>
  <typeAlias type="top.ctong.learn.domain.Employee" alias="Employee"/>
</typeAliases>
```

设置了类型别名之后，使用这些别名时，按住command 点击`Employee`可以快速定位到类型

```xml
<insert parameterType="Employee"></insert>
```

MyBatis默认为许多常见的java类型起好了别名

| 别名       | 映射的类型 |
| :--------- | :--------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |



## 类型处理器(typeHandlers)

> MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。下表描述了一些默认的类型处理器。

| 类型处理器                   | Java 类型                       | JDBC 类型                                                    |
| :--------------------------- | :------------------------------ | :----------------------------------------------------------- |
| `BooleanTypeHandler`         | `java.lang.Boolean`, `boolean`  | 数据库兼容的 `BOOLEAN`                                       |
| `ByteTypeHandler`            | `java.lang.Byte`, `byte`        | 数据库兼容的 `NUMERIC` 或 `BYTE`                             |
| `ShortTypeHandler`           | `java.lang.Short`, `short`      | 数据库兼容的 `NUMERIC` 或 `SMALLINT`                         |
| `IntegerTypeHandler`         | `java.lang.Integer`, `int`      | 数据库兼容的 `NUMERIC` 或 `INTEGER`                          |
| `LongTypeHandler`            | `java.lang.Long`, `long`        | 数据库兼容的 `NUMERIC` 或 `BIGINT`                           |
| `FloatTypeHandler`           | `java.lang.Float`, `float`      | 数据库兼容的 `NUMERIC` 或 `FLOAT`                            |
| `DoubleTypeHandler`          | `java.lang.Double`, `double`    | 数据库兼容的 `NUMERIC` 或 `DOUBLE`                           |
| `BigDecimalTypeHandler`      | `java.math.BigDecimal`          | 数据库兼容的 `NUMERIC` 或 `DECIMAL`                          |
| `StringTypeHandler`          | `java.lang.String`              | `CHAR`, `VARCHAR`                                            |
| `ClobReaderTypeHandler`      | `java.io.Reader`                | -                                                            |
| `ClobTypeHandler`            | `java.lang.String`              | `CLOB`, `LONGVARCHAR`                                        |
| `NStringTypeHandler`         | `java.lang.String`              | `NVARCHAR`, `NCHAR`                                          |
| `NClobTypeHandler`           | `java.lang.String`              | `NCLOB`                                                      |
| `BlobInputStreamTypeHandler` | `java.io.InputStream`           | -                                                            |
| `ByteArrayTypeHandler`       | `byte[]`                        | 数据库兼容的字节流类型                                       |
| `BlobTypeHandler`            | `byte[]`                        | `BLOB`, `LONGVARBINARY`                                      |
| `DateTypeHandler`            | `java.util.Date`                | `TIMESTAMP`                                                  |
| `DateOnlyTypeHandler`        | `java.util.Date`                | `DATE`                                                       |
| `TimeOnlyTypeHandler`        | `java.util.Date`                | `TIME`                                                       |
| `SqlTimestampTypeHandler`    | `java.sql.Timestamp`            | `TIMESTAMP`                                                  |
| `SqlDateTypeHandler`         | `java.sql.Date`                 | `DATE`                                                       |
| `SqlTimeTypeHandler`         | `java.sql.Time`                 | `TIME`                                                       |
| `ObjectTypeHandler`          | Any                             | `OTHER` 或未指定类型                                         |
| `EnumTypeHandler`            | Enumeration Type                | VARCHAR 或任何兼容的字符串类型，用来存储枚举的名称（而不是索引序数值） |
| `EnumOrdinalTypeHandler`     | Enumeration Type                | 任何兼容的 `NUMERIC` 或 `DOUBLE` 类型，用来存储枚举的序数值（而不是名称）。 |
| `SqlxmlTypeHandler`          | `java.lang.String`              | `SQLXML`                                                     |
| `InstantTypeHandler`         | `java.time.Instant`             | `TIMESTAMP`                                                  |
| `LocalDateTimeTypeHandler`   | `java.time.LocalDateTime`       | `TIMESTAMP`                                                  |
| `LocalDateTypeHandler`       | `java.time.LocalDate`           | `DATE`                                                       |
| `LocalTimeTypeHandler`       | `java.time.LocalTime`           | `TIME`                                                       |
| `OffsetDateTimeTypeHandler`  | `java.time.OffsetDateTime`      | `TIMESTAMP`                                                  |
| `OffsetTimeTypeHandler`      | `java.time.OffsetTime`          | `TIME`                                                       |
| `ZonedDateTimeTypeHandler`   | `java.time.ZonedDateTime`       | `TIMESTAMP`                                                  |
| `YearTypeHandler`            | `java.time.Year`                | `INTEGER`                                                    |
| `MonthTypeHandler`           | `java.time.Month`               | `INTEGER`                                                    |
| `YearMonthTypeHandler`       | `java.time.YearMonth`           | `VARCHAR` 或 `LONGVARCHAR`                                   |
| `JapaneseDateTypeHandler`    | `java.time.chrono.JapaneseDate` | `DATE`                                                       |

MyBatis几乎囊括了数据库所有的类型，如果你需要自定义类型处理器，可以实现`org.apache.ibatis.type.TypeHandler`接口或者继承一个很便利的类`org.apache.ibatis.type.BaseTypeHandler`。

```java
public class MyTypeHandler implements TypeHandler<MyEnum> { }

enum MyEnum {
    ADMIN,
    USER
}
```



```java
public class MyTypeHandler extends BaseTypeHandler<MyEnum> { }

enum MyEnum {
    ADMIN,
    USER
}
```



## 插件(plugins)

- 插件时MyBatis提供的一个非常强大的机制，我们可以通过插件来修改MyBatis的一些核心行为。**插件通过动态代理机制**，可以介入四大对象的任何一个方法的执行。
- `Execulor(update, query, flushStatements, commt, rollback, getTransaction, close, isClosed)`是一个执行器，用于负责执行sql语句，它是mybatis四大对象之一。
- `ParameterHandler(getparameterobject, setParameters)` 是一个参数处理器，它是mybatis四大对象之一
- `ResultSetHandler(handleResultSets, handleOutputParameters)` 是一个结果集处理器，它负责将查询出来的数据的结果集封装成你指定的JavaBean对象或者你指定的其他类型。它是四大对象之一
- `StatementHandler(prepare, parameterize, batch, update, query)` 它是一个预编译处理器，是四大对象之一



> 这些类中方法的细节可以通过查看每个方法的签名来发现，或者直接查看 MyBatis 发行包中的源代码。 如果你想做的不仅仅是监控方法的调用，那么你最好相当了解要重写的方法的行为。 因为在试图修改或重写已有方法的行为时，很可能会破坏 MyBatis 的核心模块。 这些都是更底层的类和方法，所以使用插件的时候要特别当心。



## 环境配置(environments)

> MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中使用相同的 SQL 映射。还有许多类似的使用场景。
>
> **不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**
>
> 所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，



`environments`元素定义如何配置环境和使用哪个环境。

- `default` 默认使用哪个环境，它的值是子级的`id`值

```xml
<environments default="xxx"></environments>
```

具体环境信息在子级`environment`中定义。它有两个子级，`transactionManager`和`dataSource`。

无论是哪种环境，它都需要一个事物管理器和一个数据源。

`transactionManager`是用于配置事物管理器

- `type` 事务类型，指使用那种事务管理器进行管理事务
  - `JDBC` – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。
  - `MANAGED` – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。

```xml
<environment id="dev">
  <transactionManager type="JDBC"/>
</environment>
```

>  如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

`dataSource` 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

MyBatis有三种内建的数据源类型:

1. **UNPOOLED**– 这个数据源的实现会每次请求时打开和关闭连接。虽然有点慢，但对那些数据库连接可用性要求不高的简单应用程序来说，是一个很好的选择。 性能表现则依赖于使用的数据库，对某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

   - `driver` – 这是 JDBC 驱动的 Java 类全限定名（并不是 JDBC 驱动中可能包含的数据源类）。

   - `url` – 这是数据库的 JDBC URL 地址。

   - `username` – 登录数据库的用户名。

   - `password` – 登录数据库的密码。

   - `defaultTransactionIsolationLevel` – 默认的连接事务隔离级别。

   - `defaultNetworkTimeout` – 等待数据库操作完成的默认网络超时时间（单位：毫秒）。查看 `java.sql.Connection#setNetworkTimeout()` 的 API 文档以获取更多信息。作为可选项，你也可以传递属性给数据库驱动。只需在属性名加上“driver.”前缀即可，例如：

     - `driver.encoding=UTF8`

     这将通过 DriverManager.getConnection(url, driverProperties) 方法传递值为 `UTF8` 的 `encoding` 属性给数据库驱动。

2. **POOLED**– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。

   除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：

   - `poolMaximumActiveConnections` – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
   - `poolMaximumIdleConnections` – 任意时间可能存在的空闲连接数。
   - `poolMaximumCheckoutTime` – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
   - `poolTimeToWait` – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。
   - `poolMaximumLocalBadConnectionTolerance` – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过 `poolMaximumIdleConnections` 与 `poolMaximumLocalBadConnectionTolerance` 之和。 默认值：3（新增于 3.4.5）
   - `poolPingQuery` – 发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动出错时返回恰当的错误消息。
   - `poolPingEnabled` – 是否启用侦测查询。若开启，需要设置 `poolPingQuery` 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句），默认值：false。
   - `poolPingConnectionsNotUsedFor` – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。

3. **JNDI** – 这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。这种数据源配置只需要两个属性：

   - `initial_context` – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
   - `data_source` – 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。和其他数据源配置类似，可以通过添加前缀“env.”直接把属性传递给 InitialContext。比如：
     - `env.encoding=UTF8` 这就会在 InitialContext 实例化时往它的构造方法传递值为 `UTF8` 的 `encoding` 属性。

如果需要自定义连接池，可以实现`org.apache.ibatis.datasource.DataSourceFactory`接口来实现或使用第三方连接池。

```java
public class MyDataSource implements DataSourceFactory {

    @Override
    public void setProperties(Properties properties) {
        
    }
    @Override
    public DataSource getDataSource() {
        return null;
    }

}
```

将自定义的连接池配置到`<dataSource>`中，使用全类名的方式

```xml
<environments default="dev">
  <environment id="dev">
    <transactionManager type="JDBC"/>
    <dataSource type="top.ctong.learn.utils.mybatis.MyDataSource"/>
  </environment>
</environments>
```



## 数据库厂商标识(databaseIdProvider)

`databaseIdProvider`是MyBatis用来考虑数据库移植性的。每一个数据库的关键字、语法不同，可能切换数据库时，新数据库不支持旧数据库的语法而发生严重错误。

这种多厂商的支持是基于映射语句中的 `databaseId` 属性。 MyBatis 会加载带有匹配当前数据库 `databaseId` 属性和所有不带 `databaseId` 属性的语句。 如果同时找到带有 `databaseId` 和不带 `databaseId` 的相同语句，则后者会被舍弃。

```xml
<databaseIdProvider type="DB_VENDOR" />
```

配置数据库厂商

- `name` 指定数据库厂商名字
- `value` 给这个数据库厂商起别名

```xml
<databaseIdProvider type="DB_VENDOR">
  <property name="MySql" value="mysql"/>
  <property name="Oracle" value="oracle"/>
  <property name="SQL Server" value="sqlserver"/>
</databaseIdProvider>
```



```xml
<select id="query" resultMap="employee">
  select * from t_employee where id = #{id}
</select>
<select id="query" resultMap="employee" databaseId="sqlserver">
  select * from t_employee where id = #{id}
</select>
<select id="query" resultMap="employee" databaseId="oracle">
  select * from t_employee where id = #{id}
</select>
```





## mappers

当你写了一个`xxDao.xml`文件时，你需要告诉MyBatis这是一个映射文件。

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

这些配置会告诉 MyBatis 去哪里找映射文件



# SQL映射文件

> 新增、修改删除都不需要设置返回值，只有查询需要设置返回值，因为除查询外，其他语句返回的都是受影响行数。



MyBatis的SQL映射文件中能写以下列举出来的标签

- `cache` 和缓存有关
- `cache-ref` 和缓存有关
- `delete`、`update`、`insert`、`select` 用来做增删改查
- `parameterMap` 用来做复杂参数映射，iBatis更新到MyBatis后废弃
- `resultMap` 结果映射，自定义结果集的封装规则
- `sql` 抽取可重用的sql



## Insert, Update, Delete 元素的属性

| 属性               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| `id`               | 在命名空间中唯一的标识符，可以被用来引用这条语句。           |
| `parameterType`    | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。 |
| `parameterMap`     | 用于引用外部 parameterMap 的属性，目前已被废弃。请使用行内参数映射和 parameterType 属性。 |
| `flushCache`       | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。 |
| `timeout`          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
| `statementType`    | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| `useGeneratedKeys` | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。 |
| `keyProperty`      | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| `keyColumn`        | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
| `databaseId`       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。 |



## CRUD操作

> 只演示Insert，因为其他三个标签一样。



使用MyBatis向数据库中插入一条数据。

先写Dao层的`insert`

```java
public interface EmployeeDao { 
	int insert(Employee dao);
}
```

写这个Dao层的`xml`配置文件

`#{xxx}` 代表获取指定参数名中的值

`parameterType` 代表传过来的参数类型，它是一个缺省属性

默认返回受影响行数，0表示失败1+表示新增成功1或多条数据

```xml
<insert id="insert" parameterType="top.ctong.learn.domain.Employee">
  insert into t_employee(emp_name, gender, email)
  value (#{empName}, #{gender}, #{email})
</insert>
```

由于sql执行成功后，除了`select`语句，其他语句返回的全是受影响行数，如果`insert`的时候需要获取插入数据后的自增id，那么可以使用MyBatis提供的以下两个属性:

- `useGeneratedKeys` 取出由数据库内部生成的主键
- `keyProperty` 告诉MyBatis，将刚才自增的id封装给哪个属性

```xml
<insert id="insert" parameterType="top.ctong.learn.domain.Employee"  useGeneratedKeys="true" keyProperty="id">
  insert into t_employee(emp_name, gender, email)
  value (#{empName}, #{gender}, #{email})
</insert>
```

**注意，是个传过来的参数封装,例如`int insertReturnKey(Employee dao);`，那么就给dao里边的`keyProperty`封装。**



## selectKey

用于查询主键，当一个数据库不支持自增id时，可以使用`selectKey`模拟自增，当然它可能还能做更多的事。

- `order` 指定一个运行时机，他有两个属性
  1. `AFTER` 在sql语句执行之后执行`selectKey`中的sql
  2. `BEFORE` 在sql执行之前执行`selectKey`中的sql

如果当前做的是一个全字段插入，并且数据库并不支持自增，可以这么解决：

```xml
<insert id="insertReturnKey">
  <selectKey order="BEFORE" keyProperty="id" resultType="integer">
    select max(id) + 1 from t_employee
  </selectKey>
  insert into t_employee(id, emp_name, gender, email)
  value (#{id}, #{empName}, #{gender}, #{email})
</insert>
```



## 参数的各种取值

```java
public interface EmployeeDao extends GenericDao<Employee> {
    Employee queryByNameAndId(String empName, Integer id);
}
```

```xml
<select id="queryByNameAndId" resultType="Employee">
  select *
  from t_employee
  where emp_name = #{empName} and id = #{id}
</select>
```

如果执行以上操作就会抛出一个异常：

```
Cause: org.apache.ibatis.binding.BindingException: Parameter 'empName' not found. Available parameters are [arg1, arg0, param1, param2]
```

在MyBatis中，存在多个参数的时候不能使用参数名获取参数指定参数值需要使用参数索引，例如`#{arg0}`、`#{arg1}`或者`#{param1}`、`#{param2}`

```xml
<select id="queryByNameAndId" resultType="Employee">
  select *
  from t_employee
  where emp_name = #{param1} and id = #{param2}
</select>
```

- 传入当个参数

  - 基本类型
    取值： `#{随便写}`

- 传入多个参数

  `#{xxx}`无效，而取值方式是通过索引进行取值。

  原因是只要传入了多个参数，mybatis会自动将这些参数封装在一个map中，封装时使用的key就是参数的索引。

  可以使用`@Param`注解告诉MyBatis封装时使用我自己指定的key

  ```java
  Employee queryByNameAndId(@Param("empName") String empName, @Param("id") Integer id);
  ```

- 传入Map
  相当于告诉mybatis，你别封装了，直接使用我的map

- 传入pojo
  可以使用`#{xxx}`取指定参数的参数值



## \#{}和${}

实际上在mybatis中，有两种取值方式，一种是`#{xxx}`另一种是`${xxx}`。

再执行以下sql时

```sql
select *
from t_employee
where emp_name = ${empName} and id = #{id}
```

控制台输出执行语句：

```sql
select * from t_employee where emp_name = Clover and id = ?
```

再执行以下sql

```sql
select *
from t_employee
where emp_name = #{empName} and id = #{id}
```

输出执行语句

```sql
select * from t_employee where emp_name = ? and id = ?
```

---

- `#{xxx}` 是参数预编译的方式，参数的位置都是**`?`**替代
- `${xxx}` 是以字符串拼接的方式。这种方式存在sql注入问题。



## 查询返回list

如果查询所有员工，那么该结果必定是一个`List`

- `resultType` 如果返回的是一个集合，那么写的是集合里面元素的类型，而不是`List`。MyBatis会自动封装为`List`

```xml
<select id="queryAll" resultType="Employee">
    select *
    from t_employee
</select>
```

```java
/**
 * 查询所有<T>数据
 * @return <T> 数据查询结果
 */
List<T> queryAll();
```





## resultMap自定义封装规则

MyBatis默认的自动封装结果集

- 按照列名和属性名一一对应的规则进行封装（不区分大小写）
- 如果不对应那么就可能封装不上，有三种解决方法
  1. 在MyBatis配置中开启`mapUnderscoreToCamelCase`设置
  2. 给数据库字段起别名，例如：`select emp_name as empName from t_employee;`
  3. 自定义结果集，自定义每一列数据和Javabean的映射规则。



- `type`指定 这个结果集映射到哪个javaBean
- `id` 唯一标识，能够让别人通过它进行引用
- `extends` 可继承已存在的规则

```xml
<resultMap id="employee" type="Employee"></resultMap>
```

指定`id`字段映射关系

- `column` 数据库中字段名
- `property` javaBean中对应的字段名
- `javaType` 字段类型
- `jdbcType` 这个字段在数据库中的类型
- `typeHandler` 使用哪个类型处理器解析

```xml
<resultMap id="employee" type="Employee">
  <id column="id" property="id" javaType="java.lang.Integer"/>
</resultMap>
```

映射普通字段，属性和用法与`<id>`一直

```xml
<resultMap id="employee" type="Employee">
  <result column="emp_name" property="empName" javaType="java.lang.String"/>
</resultMap>
```

完整示例

```xml
<resultMap id="employee" type="top.ctong.learn.domain.Employee">
  <id column="id" property="id" javaType="java.lang.Integer" typeHandler=""/>
  <result column="emp_name" property="empName" javaType="java.lang.String"/>
  <result column="gender" property="gender" javaType="java.lang.Short"/>
  <result column="email" property="email" javaType="java.lang.String"/>
</resultMap>
```

使用方法也很简单

- `resultMap` 查出数据时使用自定义规则

```xml
<select id="queryAll" resultMap="employee">
  select *
  from t_employee
</select>
```



### 联合查询

```java
@Data
public class Salary implements Serializable {

    private static final long serialVersionUID = 4589851267399554580L;

    private Integer id;

    private Double money;

    private Integer empId;

    private Employee emp;

}
```

```java
@Data
public class Employee implements Serializable {

    private static final long serialVersionUID = -6915954877566070374L;

    /**
     * id
     */
    private Integer id;

    /**
     * 员工名
     */
    private String empName;

    /**
     * 性别
     */
    private Short gender;

    /**
     * 邮箱
     */
    private String email;
  
}
```

#### 联合查询

以上`Salary`级联查询时需要这样定义自定义规则：

- `emp.email` 代表`emp`对象下的`email`字段

```xml
<resultMap id="SalaryResultMap" type="top.ctong.learn.domain.Salary">
  <id column="id" property="id"/>
  <result column="money" property="money"/>
  <result column="emp_id" property="empId"/>
  <result property="emp.id" column="emp_id"/>
  <result property="emp.email" column="email"/>
  <result property="emp.empName" column="emp_name"/>
  <result property="emp.gender" column="gender"/>
</resultMap>
```

```xml
<select id="querySalaryAndEmp">
  select t_salary.id as salary_id, t_salary.money, te.id as emp_id,
         te.gender, te.emp_name, te.email
  from t_salary
       left join t_employee te on t_salary.emp_id = te.id
</select>
```



#### association

`association` 用于定义一个复杂的类型关联规则，这也是MyBatis官方推荐的。`association`中定义的`<id>`、`<result>`都是对`association`的`property`指定的对象的定义，与外界无关。

- `property` javaBean 属性名
- `javaType` 指定javaBean类型

```xml
    <resultMap id="SalaryResultMap" type="top.ctong.learn.domain.Salary">
        <id column="id" property="id"/>
        <result column="money" property="money"/>
        <result column="emp_id" property="empId"/>
        <association property="emp" javaType="top.ctong.learn.domain.Employee">
            <id column="emp_id" property="id"/>
            <result column="email" property="email"/>
            <result column="emp_name" property="empName"/>
            <result column="gender" property="gender"/>
        </association>
    </resultMap>
```

```xml
<select id="querySalaryAndEmp">
  select t_salary.id as salary_id, t_salary.money, te.id as emp_id,
         te.gender, te.emp_name, te.email
  from t_salary
       left join t_employee te on t_salary.emp_id = te.id
</select>
```



`association` 有个缺点，它无法定义集合属性`List<Object>`。



#### collection

`collection` 可用于定义集合元素的封装

- `ofType` 用于指定集合元素的类型
- `property`指定哪个属性是集合属性

```xml
<resultMap id="SalaryResultMap" type="top.ctong.learn.domain.Salary">
  <id column="id" property="id"/>
  <result column="money" property="money"/>
  <result column="emp_id" property="empId"/>
  <collection property="emp" ofType="top.ctong.learn.domain.Employee">
    <id column="emp_id" property="id"/>
    <result column="email" property="email"/>
    <result column="emp_name" property="empName"/>
    <result column="gender" property="gender"/>
  </collection>
</resultMap>
```

```xml
<select id="querySalaryAndEmp">
  select *
  from t_employee
       left join t_salary ts on t_employee.id = ts.emp_id
  where ts.money in (select distinct salary.money from t_salary as salary);
</select>
```

```json
{
  "xxx": "xxx",
  "xxx": "xxx",
  "xxx": "xxx",
  "List<xxx>": [
    {"xxx":"xxx"},
    {"xxx":"xxx"},
    {"xxx":"xxx"},
    {"xxx":"xxx"}
  ]
}
```



#### select分步查询

它不是连表查询也不是联合国查询，而是通过调用另外一个sql语句进行查询，最后将查询结果封装到指定javaBean。

- `select` 指定一个查询sql的id，MyBatis自动调用指定的sql。
- `column` 告诉MyBatis将哪一列数据传递过去。若需要传递多个参数，需要使用KV的方式：`{key1=列名,key2=列名}`

```xml
<resultMap id="SalaryResultMap" type="top.ctong.learn.domain.Salary">
  <id column="id" property="id"/>
  <result column="money" property="money"/>
  <result column="emp_id" property="empId"/>
  <association property="emp" select="queryEmp"/>
</resultMap>
```

```xml
<select id="queryEmp">
  select * from t_employee where id=#{id}
</select>

<select id="query" resultMap="SalaryResultMap">
  select *
  from t_salary
</select>
```

这种方式存在一个问题，当我们不需要查询`emp`的时候，他也会去查，这样会造成严重性能问题，资源浪费。



> `collection`也可以使用分布查询



#### 按需加载和延迟加载

需要到配置文件中开启按需加载和延迟加载

- `lazyLoadingEnabled`：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。

  ```xml
  <setting name="lazyLoadingEnabled" value="true" />
  ```

- `aggressiveLazyLoading`：开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。

  ```xml
  <setting name="aggressiveLazyLoading" value="false"/>
  ```



开启这两个配置之后，**select分步查询**的`association`不会立马执行。当有一天你调用了`emp`相关的属性或方法之后，会自动去查这些信息。

当开启按需加载后，如果你不需要按需加载了，可以通过`association`的`fetchType`属性取消按需加载。

- `fetchType`在开启按需加载时生效，它一共有两个属性
  1. `eager` 立刻查询，不需要按需加载
  2. `lazy` 这是默认的，使`association`按需加载



> 这里只是拿`association`做例子，`collection`也可以这么用



# 动态SQL

- 动态SQL时MyBatis最强大的特性之一。极大的简化我们拼装SQL的操作
- 动态SQL元素和使用JSTL或其他类似基于XML的文本处理器相似。
- MyBatis采用功能强大的基于OGNL的表达式来简化操作。
  - `if`
  - `choose`、`when`、`otherwise`
  - `trim`、`where` 、`set`
  - `foreach`



## if标签

```java
/**
 * 通过一个dao层信息查询所有与其相关的数据
 * @param dao dao层信息
 * @return 通过dao层查询结果集
 */
List<T> queryAll(T dao);
```

通过条件拼接sql

- `test` 判断条件，它支持`==`、`!=`、`and`、`&&` 、`or` 、`||`
  注意`&`需要转译

```xml
<select id="queryAll" parameterType="Employee" resultMap="employee">
  select *
  from t_employee
  where 1=1
  <if test="empName!=null">
    and emp_name=#{empName}
  </if>
  <if test="gender!=null">
    and gender=#{gender}
  </if>
  <if test="email!=null">
    and email=#{email}
  </if>
</select>
```

`where 1=1`是为了让拼接后的sql能正确运行，因为这些条件你并不知道哪些是能用哪些不能用。如果条件这么写，那么`empName`不为null时是没问题的。

```xml
<if test="empName!=null">
  emp_name=#{empName}
</if>
<if test="gender!=null">
  and gender=#{gender}
</if>
<if test="email!=null">
  and email=#{email}
</if>
```

一但`empName`为null那么它sql拼接的结果就是这样：

```sql
where and gender=xxx
```

这样会出现语法错误，所以先有个默认条件`1=1`就可以了

```sql
where 1=1 and gender=xxx
```



## where标签

在使用`if`标签时，需要通过`1=1`的方式避免语法错误，而`where`标签就是为了解决这些问题而出现。他会将你错误的语法去除(前面的语法)。

```xml
<select id="queryAll" parameterType="Employee" resultMap="employee">
  select *
  from t_employee
  <where>
    <if test="empName!=null">
      and emp_name=#{empName}
    </if>
    <if test="gender!=null">
      and gender=#{gender}
    </if>
    <if test="email!=null">
      and email=#{email}
    </if>
  </where>
</select>
```

```sql
select * from t_employee WHERE emp_name=?
```



## trim标签

`trim`用来截取字符串，他有4个属性

- `prefix` 为整体整体添加一个前缀
- `prefixOverrides` 去除整体字符串前面多余的字符串
- `suffix` 为整体字符串添加后缀
- `suffixOverrides`去除整体字符串末尾多余的字符串

```xml
<select id="queryAll" parameterType="Employee" resultMap="employee">
  select *
  from t_employee
  <trim prefix="where " prefixOverrides="and " suffixOverrides=" and" suffix=";">
    <if test="empName!=null">
      and emp_name=#{empName}
    </if>
    <if test="gender!=null">
      and gender=#{gender}
    </if>
    <if test="email!=null">
      and email=#{email}
    </if>
    and
  </trim>
</select>
```

```sql
select * from t_employee where emp_name=?;
```



## foreach标签

假如有这么一个需求，通过前端传过来的id查询员工信息，而这个id是个`List`。如果通过sql语句查询的话应该是这样：

```sql
select * from t_employee where id in (1, 2, 3);
```

像这种需求，`List<id>`是无法后端是不知道的，都是前端传过来的，它不一定是三个。

像这种需求可以使用`foreach`进行字符串拼接的方式操作。

- `collection` 指定要遍历的集合

- `item` 当前被遍历出的数据的变量名，相当于以下代码的id：

  ```java
  for (Integer id : ids)
  ```

- `index` 遍历索引，相当于`for`循环的 `i` 

  - 如果遍历的是一个`list`，那么`index`指定的变量就是当前索引
  - 如果遍历的是一个`map`，那么`index`指定的变量就是`key`

- `open` 遍历开始时要拼接的字符串

- `close` 遍历结束后要拼接的字符串

- `separator` 每次遍历一次后通过指定字符串隔开

```xml
<select id="queryEmployeeByIds" resultMap="employee">
  select * from t_employee
  where id in
  <foreach collection="ids" open="(" close=")" index="index" item="id" separator=",">
    #{id}
  </foreach>
</select>
```





## Choose标签

这是一个**分支选择**标签，功能类似于`switch`。它是需要和`when`、`otherwise`标签配合使用。

- `when` 表示一个选项，他有一个属性
  - `test` 条件验证
- `otherwise`表示默认值，如果`when`标签都没匹配上，那么就使用`otherwise`指定的值

```java
/**
 * 通过id列表查询指定员工，使用choose
 * @param emp 参数信息
 * @return 员工信息
 */
List<Employee> queryEmpByIdsUseChoose(@Param("emp") Employee emp);
```

```xml
<select id="queryEmpByIdsUseChoose" resultType="employee">
  select *
  from t_employee
  <where>
    <choose>
      <when test="emp.id!=null">
        id=#{emp.id}
      </when>
      <when test="emp.empName!=null">
        emp_name=#{emp.empName}
      </when>
      <when test="emp.gender!=null">
        gender=#{emp.gender}
      </when>
      <otherwise>
        gender=0
      </otherwise>
    </choose>
  </where>
</select>
```





## set标签

`set`标签是一个非常强大且实用的标签（虽然功能有点鸡肋），用来完成动态更新。以前使用sql更新时是属于全字段更新。而`set`标签可以动态更新字段。

当然，MyBatis提供的其他标签也能完成这些工作，但...`set`他是专业的。

```java
/**
 * 修改指定dao层
 * @param dao 修改后的dao层信息
 * @return 受影响行数
 */
int update(T dao);
```



```xml
<update id="update">
  update t_employee
  <set>
    <if test="empName!=null">
      emp_name=#{empName},
    </if>
    <if test="gender!=null">
      gender =#{gender},
    </if>
    <if test="email!=null">
      email=#{email} ,
    </if>
  </set>
  <where>
    id=#{id}
  </where>
</update>
```

```
DEBUG 08-12 20:38:08,104 ==>  Preparing: update t_employee SET emp_name=? WHERE id=?  (BaseJdbcLogger.java:137) 
DEBUG 08-12 20:38:08,138 ==> Parameters: Clover You(String), 1(Integer)  (BaseJdbcLogger.java:137) 
DEBUG 08-12 20:38:08,144 <==    Updates: 1  (BaseJdbcLogger.java:137) 
```



## sql和include标签

`sql`标签需要和`include`标签配合使用。`sql`标签用来抽取可重用的`sql`代码，然后使用`include`标签将这个`sql`引入到指定位置。

```xml
<sql id="queryEmpSql">
  select id, emp_name, gender, email
  from t_employee
</sql>
```

```xml
<select id="query" resultMap="employee">
  <include refid="queryEmpSql"/>
  where id = #{id}
</select>
```





# OGNL

OGNL是Object-Graph Navigation Language的缩写，它是一种功能强大的表达式语言，通过它简单一致的表达式语法，可以存取对象的任意属性，调用对象的方法，遍历整个对象的结构图，实现字段类型转化等功能。它使用相同的表达式去存取对象的属性。这样可以更好的取得数据。

- `person.name` 访问对象属性
- `person.getName()` 调用方法
- `@java.lang.Math@PI` 调用静态方法
- `new  xxx.xxx.Person('admin').name` 调用构造器方法并且访问对象属性
- `+-*/%` 运算符
- `int not in > >= < <= == !=` 逻辑运算符

> xml中特殊符号如：`> < &`等这些都是需要使用转译字符



访问集合伪属性

| 类型               | 伪属性           | 伪属性对应的java方法                                      |
| ------------------ | ---------------- | --------------------------------------------------------- |
| `List` `Set` `Map` | `size` `isEmpty` | `List` `Set` `Map.size()` or `List` `Set` `Map.isEmpty()` |
| `List` `Set`       | `iterator`       | `List.iterator()` `Set.iterator()`                        |
| `Map`              | `keys` `values`  | `Map.keySet()` `Map.values()`                             |
| `Iterator`         | `next` `hasNext` | `Iterator.next()` `iterator.hasNext()`                    |



在MyBatis中，除了传入的参数可以用来做判断外，还而外有两个属性：

- `_parameter` 代表传过来的参数
  - 在传入了单个参数的情况下，`_parameter`就可以代表这个参数
  - 如果是传了多个参数，那么`_parameter`就代表这些参数的集合`Map`
- `_databaseId` 代表当前数据库厂商





# MyBatis缓存机制

 缓存，说白了就是利用某个东西暂时存储一些数据。目的就是用来加快系统的查询速度，减少数据库压力提升用户体验。

在MyBatis中，使用了Map作为数据缓存池，并且使用了两种缓存方案：

- **一级缓存**：可以认为是线程级别的缓存，也称为是本地缓存和SqlSession级别的缓存。在MyBatis中它是默认开启的。
- **二级缓存**：全局范围的缓存，除了当前线程或者说当前SqlSession能用外，其他的也可以使用。



## 一级缓存

先搞一个查询

```java
/**
 * 从缓存中获取数据
 * @param id 员工id
 * @return 员工信息
 */
Employee queryInCache(@Param("id") Integer id);
```

```xml
<sql id="queryEmpSql">
  select id, emp_name, gender, email
  from t_employee
</sql>
<select id="queryInCache" resultMap="employee">
  <include refid="queryEmpSql"/>
  <where>
    <if test="id!=null">
      id=#{id}
    </if>
  </where>
</select>
```

测试

```java
@Test
public void testQueryEmpSql() {
  try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
    EmployeeDao mapper = sqlSession.getMapper(EmployeeDao.class);
    Employee employee = mapper.queryInCache(1);
    log.info(employee);
    log.info("------------ 分割线 ------------");
    Employee employee2 = mapper.queryInCache(1);
    log.info(employee2);
    log.info("------------ 分割线: 判断是否是同一个对象 ------------");
    log.info("判断结果：" + (employee == employee2));
  }
}
```

结果很明显可以看到，在同一个条件下，第二次查询结果不是通过数据库查的。而且第一次查询结果和第二次查询结果的内存地址是一致的。

```
DEBUG 08-13 13:47:07,196 ==>  Preparing: select id, emp_name, gender, email from t_employee WHERE id=?  (BaseJdbcLogger.java:137) 
DEBUG 08-13 13:47:07,222 ==> Parameters: 1(Integer)  (BaseJdbcLogger.java:137) 
DEBUG 08-13 13:47:07,244 <==      Total: 1  (BaseJdbcLogger.java:137) 
INFO  08-13 13:47:07,245 {"id":1,"empName":"Clover","gender":0,"email":"cloveryou.ctong@qq.com"}  (TestEmployeeDao.java:112) 
INFO  08-13 13:47:07,245 ------------ 分割线 ------------  (TestEmployeeDao.java:113) 
INFO  08-13 13:47:07,245 {"id":1,"empName":"Clover","gender":0,"email":"cloveryou.ctong@qq.com"}  (TestEmployeeDao.java:115) 
INFO  08-13 13:47:07,245 ------------ 分割线: 判断是否是同一个对象 ------------  (TestEmployeeDao.java:116) 
INFO  08-13 13:47:07,247 判断结果：true  (TestEmployeeDao.java:117) 
```



- 只要之前查询过的数据，MyBatis就会保存在一个缓存中（Map）。下次查询时直接从缓存中拿。



 

### 一级缓存失效的几种情况

- 在不同的sql会话下缓存会失效，因为默认一级缓存是SqlSession级别的的缓存，不同的缓存他使用的是不同的一级缓存(缓存池/Map)。每个SqlSession都有自己的一级缓存池。
  例如你打开了两个不同的会话：

  ```java
  SqlSession sqlSession1 = sqlSessionFactory.openSession()
  SqlSession sqlSession2 = sqlSessionFactory.openSession()
  ```

  那么在这种情况下通过这两个会话进行查询，他们的结果已经不是同一个对象了

  ```java
  try (SqlSession sqlSession = sqlSessionFactory.openSession();
       SqlSession sqlSession2 = sqlSessionFactory.openSession()) {
    EmployeeDao mapper1 = sqlSession.getMapper(EmployeeDao.class);
    Employee employee1 = mapper1.queryInCache(1);
    log.info("employee1：" + employee1);
    log.info("------------ 分割线 ------------");
    EmployeeDao mapper2 = sqlSession2.getMapper(EmployeeDao.class);
    Employee employee2 = mapper2.queryInCache(1);
    log.info("employee2：" + employee2);
    log.info("------------ 分割线: 判断是否是同一个对象 ------------");
    log.info("判断结果：" + (employee1 == employee2));
  }
  ```

  结果 

  ```
  DEBUG 08-13 14:18:24,617 ==>  Preparing: select id, emp_name, gender, email from t_employee WHERE id=?  (BaseJdbcLogger.java:137) 
  DEBUG 08-13 14:18:24,651 ==> Parameters: 1(Integer)  (BaseJdbcLogger.java:137) 
  DEBUG 08-13 14:18:24,682 <==      Total: 1  (BaseJdbcLogger.java:137) 
  INFO  08-13 14:18:24,683 employee1：{"id":1,"empName":"Clover","gender":0,"email":"cloveryou.ctong@qq.com"}  (TestEmployeeDao.java:124) 
  INFO  08-13 14:18:24,684 ------------ 分割线 ------------  (TestEmployeeDao.java:125) 
  DEBUG 08-13 14:18:24,722 ==>  Preparing: select id, emp_name, gender, email from t_employee WHERE id=?  (BaseJdbcLogger.java:137) 
  DEBUG 08-13 14:18:24,723 ==> Parameters: 1(Integer)  (BaseJdbcLogger.java:137) 
  DEBUG 08-13 14:18:24,726 <==      Total: 1  (BaseJdbcLogger.java:137) 
  INFO  08-13 14:18:24,726 employee2：{"id":1,"empName":"Clover","gender":0,"email":"cloveryou.ctong@qq.com"}  (TestEmployeeDao.java:128) 
  INFO  08-13 14:18:24,727 ------------ 分割线: 判断是否是同一个对象 ------------  (TestEmployeeDao.java:129) 
  INFO  08-13 14:18:24,729 判断结果：false  (TestEmployeeDao.java:130) 
  ```

- 同一个方法，不同的参数，由于之前没查询过，所以在缓存中找不到对应数据，所以需要查询数据库。

- 在这个SqlSession期间执行了任何一次增删改操作，也会造成缓存失效。因为增删改操作会把缓存全部清空。因为增删改可能会对数据造成影响导致与数据库对应不上。

- 手动清空缓存

  ```java
  SqlSession sqlSession = sqlSessionFactory.openSession();
  sqlSession.clearCache();
  ```



`org.apache.ibatis.cache.Cache`这是MyBatis的缓存接口。一级缓存使用的是`org.apache.ibatis.cache.impl.PerpetualCache`实现。

```java
public class PerpetualCache implements Cache {
    private final String id;
    private final Map<Object, Object> cache = new HashMap();
}
```



> 每次查询前，先去一级缓存中查有没有对应的数据，如果没有那么就去数据库中查询，每个SqlSession都有自己的一级缓存。



## 二级缓存

- 二级缓存（second level cache），全局作用域缓存。
- 二级缓存默认不开启，需要手动配置
- MyBatis提供二级缓存的接口以及实现，缓存实现要求POJO实现`Serializable`接口
- **二级缓存在SqlSession关闭或提交之后才会生效**
- 永远不会出现一级缓存和二级缓存中有同一个数据的情况。因为在一级缓存关闭后二级缓存就有数据了，而第二次查询时，二级缓存中有这个数据，那么就不会去查一级缓存，如果二级没有此数据才会去查一级缓存，一级缓存如果也没有那么就去查数据库，数据库查完后将数据放到一级缓存。



由于MyBatis默认不开启二级缓存，若需要使用二级缓存，需要到配置文件中开启

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
</settings>
```

开启二级缓存之后，还需要到你需要使用二级缓存的**SQL映射文件**中开启二级缓存。开启`cacheEnabled`设置后，在SQL映射文件中加入`<cache/>`标签即可开启二级缓存支持。

`<cache/>`标签有如下属性：

- `eviction` 缓存回收策略，默认是`LRU`
  - `LRU` 最近最少使用的。移除最长时间不被使用的对象
  - `FIFO` 先进先出。按对象进入缓存的顺序来移除它们。
  - `SOFT` 软引用。移除基于垃圾回收器状态和软引用规则的对象
  - `WEAK` 弱引用。更积极的移除基于垃圾收集器状态和弱引用规则的对象。
- `flushInterval` 刷新间隔，单位毫秒
  - 默认情况下没有默认值，也就是没有刷新间隔，缓存仅仅调用语句时刷新
- `size` 引用数目，正整数
  - 代表缓存最多可以存储多少个对象，太大容易导致内存溢出
- `readOnly` 只读模式，`true/false`
  - `true` 表示只读缓存。会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。
  - `false`表示会读写缓存。会返回缓存对象的拷贝（通过序列化）。这会慢一些但是安全性更高，因此默认是`false`



## 缓存有关设置

- 全局setting的cacheEnable：配置二级缓存的开关，一级缓存默认就是打开的
- select标签的useCache属性：配置这个select是否使用耳机缓存。一级缓存默认开启
- sql标签的`flushCache`属性：增删改默认`flushCache="true"`。sql执行后会同时清空一级和二级缓存。查询默认`flushCache="false"`
- `sqlSession.clearCache()`：只是用来清除一级缓存
- 当在某一个作用域（一级缓存Session/二级缓存Namespace）进行了CUD操作后，默认该作用域下所有select中的缓存将被`clear`







---


> 商业转载请联系作者获得授权，非商业转载请注明出处。
> For commercial use, please contact the author for authorization. For non-commercial use, please indicate the source.
> 协议(License)：署名-非商业性使用-相同方式共享 4.0 国际 (CC BY-NC-SA 4.0)
> 作者(Author)：Clover
> 个人博客地址(BlogURL)：[http://www.ctong.top](http://www.ctong.top)