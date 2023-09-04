title: Tomcat 安装
categories: 随笔
tags: [JAVA,随笔]
date: 2022-01-02 18:31:06
---
## Tomcat

到官网下载 [tomcat](http://maven.apache.org/download.cgi)
![image.png](http://qiniu-note-image.ctong.top/note/images/202112271120666.png)

下载完成解压后，将该文件夹命名为 Tomcat，移动到资源库（/Library）

![image.png](http://qiniu-note-image.ctong.top/note/images/202112271120671.png)

打开终端转到 Tomcat /bin 路径,给所有 .sh 文件755权限。

`sudo chmod 755 *.sh`

启动 tomcat，输入 sudo sh ./startup.sh 后回车：

`sudo sh ./startup.sh`


![image.png](http://qiniu-note-image.ctong.top/note/images/202112271121914.png)

tomcat 启动后打开浏览器，输入[localhost:8080](localhost:8080)

![image.png](http://qiniu-note-image.ctong.top/note/images/202112271121258.png)

如果启动失败请检查8080端口是否被占用，如果已经被占用需要将8080杀死。

关闭Tomcat，用终端输入sh ./shutdown.sh