title: MySQL查询区分大小写敏感问题
categories: 随笔
tags: [MySQL,SQL]
date: 2022-01-02 15:49:00
---
由于mysql是不区分大小写的，所以当你查询的时候，例如数据库里有条数据用户名为UpYou（用户名唯一），当你输入：upyou时发现也可以查询，在某些需求下这样是不允许的，可以在查询语句中加入binary关键字过滤，例如
```sql
select id, username, password, type, createtime, comm from user where binary username = 'upyou'
```
以下是测试查询没有加`binary`执行的查询语句

![没加binary的java代码](http://qiniu-note-image.ctong.top/note/images/202112271103340.png)

执行结果，可以看到返回的data为true，证明这个用户存在，我可以很确定这个用户并不存在

![没加binary的结果](http://qiniu-note-image.ctong.top/note/images/202112271103932.png)

这是加了binary的查询
![加了binary的java代码](http://qiniu-note-image.ctong.top/note/images/202112271103406.png)

执行结果
![加了binary的查询结果](http://qiniu-note-image.ctong.top/note/images/202112271103456.png)

再试试正确的还好不好使

![正确结果](http://qiniu-note-image.ctong.top/note/images/202112271103644.png)