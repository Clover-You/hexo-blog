title: JavaScript 判断0非空问题
categories: 随笔
tags: [TypeScript,js,随笔,ts,JavaScript,前端]
date: 2022-01-02 18:22:37
---
用uniapp做商城购物车时有个需求：类似饿了么中的选商品规格功能，只不过我们需求是多选，我是这么做的：
用一个对象记录选中的'规格'，例如：
```javascript
dataSet: {}
```
点击规格时，将当前被点击项v-for的index得到，判断`dataSet`是否有这个index，有代表删除，没有就代表需要添加
```javascript
(index) {
  if(dataSet[index]) delect dataSet[index];
  else dataSet[index] = index;
}
```
非常完美，但测试时却无法删除索引为0的数据。
原因是Number类型的0等于false
![image.png](http://qiniu-note-image.ctong.top/note/images/202112271117895.png)

只要将0索引转为字符串即可

![截屏20200917 16.30.01.png](http://qiniu-note-image.ctong.top/note/images/202112271117211.png)

```javascript
(index) {
	index = index + '';
  if(dataSet[index]) delect dataSet[index];
  else dataSet[index] = index;
}
```

---

当Number类型的0和空字符串''判断时，结果为`true`

![截屏20200917 16.32.08.png](http://qiniu-note-image.ctong.top/note/images/202112271117275.png)