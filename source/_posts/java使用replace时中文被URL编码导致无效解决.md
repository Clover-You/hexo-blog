title: java使用replace时中文被URL编码导致无效解决
categories: JAVA
tags: [JAVA,随笔]
date: 2022-01-23 17:53:00
---
在使用 `"".replace("中文", "")` 时，如果 `target` 中存在中文就会被进行 URL 编码，例如

![源信息](http://qiniu-note-image.ctong.top/note/images/202201231739592.png)

![URL编码后](http://qiniu-note-image.ctong.top/note/images/202201231739458.png)

以上例子是想用 `queryString.replace("&attrs=44_以官网信息为准", "");` 将 `&attrs=44_以官网信息为准` 替换为空字符，但被URL编码了，编码后相当于 ` queryString.replace("&attrs=44_%E4%.....", "");` ，这时候就匹配不到我们的目标字符串。

可以将中文进行 **UTF-8** 编码就可以了

```java
// 对字符串进行UTF-8编码
String str = URLEncoder.encode("44_以官网信息为准", "UTF-8");
String targetStr = queryString.replace("&attrs=" + str, "");
```

在进行编码后，浏览器将空格替换为 `%20`，而java将空格替换为 `+` ,所以需要解决他们的差异

```java
// 解决java与浏览器的差异
str.replace("+", "%20");
```
