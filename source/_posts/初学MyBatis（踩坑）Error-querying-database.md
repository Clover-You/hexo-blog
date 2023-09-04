title: 初学MyBatis（踩坑）Error querying database
categories: 随笔
tags: [SpringMvc,MySQL,JAVA,SpringBoot,SQL,Spring,随笔]
date: 2022-01-02 18:14:49
---
最近在学习Mybatis，代码全部根据教程写好了，一运行结果报了一个错误，主要错误内容：
```java
Caused by: org.apache.ibatis.exceptions.PersistenceException: 
### Error querying database.  Cause: java.sql.SQLException: java.lang.ClassCastException: java.math.BigInteger cannot be cast to java.lang.Long
### The error may exist in com/ctong/crm/dao/UserDao.xml
### The error may involve com.ctong.crm.dao.UserDao.login
### The error occurred while executing a query
### Cause: java.sql.SQLException: java.lang.ClassCastException: java.math.BigInteger cannot be cast to java.lang.Long
	at org.apache.ibatis.exceptions.ExceptionFactory.wrapException(ExceptionFactory.java:30)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:149)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:140)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectOne(DefaultSqlSession.java:76)
	at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:87)
	at org.apache.ibatis.binding.MapperProxy$PlainMethodInvoker.invoke(MapperProxy.java:144)
	at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:85)
	at com.sun.proxy.$Proxy5.login(Unknown Source)
	at com.ctong.crm.service.impl.UserServiceImpl.login(UserServiceImpl.java:42)
	... 31 more
```
说什么BigInteger无法转Long？我跑去检查实体类和数据库，类型全部一一对应

![User.class和数据库对照表](http://qiniu-note-image.ctong.top/note/images/202112271102336.png)

断点调试跑底层去，发现是连接数据库出现了错误，然后我又跑去检查mybatis配置文件，没问题...

![MyBatis配置文件](http://qiniu-note-image.ctong.top/note/images/202112271102378.png)

终于在百度快搜烂了的情况下发现了一个帖子（maven依赖版本不对应）[传送门](https://www.imooc.com/qadetail/302176)

![原帖](http://qiniu-note-image.ctong.top/note/images/202112271102586.png)

我的`mysql-connector-java`版本为5.1.23，而我本地mysql版本8.0+，`mybatis`版本3.5，我把`mysql-connector-java`版本对应本地mysql之后就好了....

![错误版本](http://qiniu-note-image.ctong.top/note/images/202112271102571.png)

把依赖版本对应本地mysql版本

![对应mysql版本](http://qiniu-note-image.ctong.top/note/images/202112271102523.png)

他来啦他来啦！！

![查询结果](http://qiniu-note-image.ctong.top/note/images/202112271102820.png)

（不知道mybatis版本需不需要对应，你们试试再来告诉我\^.^）