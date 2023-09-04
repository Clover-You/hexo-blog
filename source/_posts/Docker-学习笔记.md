---
title: Docker - 学习笔记
date: 2022-04-03 14:35:16
tags: Docker
categories: 其它
---

# Docker

docker 的出现开启了一个新的时代：容器化时代

市面上有很多开发语言，如：Java、C++、JavaScript，如果你要运行，你可能拿到的是一个 `.jar` `.exe` `.sh` 文件甚至你需要自己编译源码。没有一个统一的标准进行管理。

而 Docker 却可以将这些应用 build 成一个镜像，你只要是一个 Docker 镜像，那么 Docker 就可以直接给你跑起来。不仅如此， 这些镜像还很容易分享，例如将某些镜像放到指定的地方，别人就可以找到这个镜像下载和运行，所以 Docker 的推出还统一了标准。

## 虚拟化技术

在 Docker 之前，非常流行一种“虚拟化技术”，这种技术其实就是在一台机器中安装多个虚拟机，我们部署的应用都在这些安装的虚拟机中。如果有应用出现问题，比如内存泄露，顶多把他自个的机子内存占满，不会影响到别的应用。

![虚拟化技术](https://qiniu-note-image.ctong.top/note/images/202203311724774.png)

虚拟化技术有一下痛点：

1. 基础镜像GB级别
2. 创建使用比较复杂
3. 启动速度慢
4. 移植与分享不方便

优点是隔离性很强

## 容器化技术

![容器化技术](https://qiniu-note-image.ctong.top/note/images/202203311906410.png)

1. 基础镜像MB级别
2. 创建简单
3. 隔离性强
4. 启动速度秒级
5. 移植与分享方便

## 架构

![架构图](https://qiniu-note-image.ctong.top/note/images/202203312031389.png)

- Docker_Host

  如果需要使用 Docker，那么需要在使用的机器上安装 Docker。安装了 Docker 的机器称为 Docker 主机。

- Docker Daemon

  安装了 Docker 后，主机启动那么后台就会启动一个 Docker 的进程 `Docker Daemon`

- Client 
  如果需要操作 Docker，那么需要使用命令行，这个命令行被称为Docker 客户端

- Images

  镜像，一个带环境打包好的程序，可以直接启动运行

- Registry
  镜像仓库，官方的镜像仓库是 Docker Hub

- Containers

  容器，由镜像启动起来正在运行中的程序

## Docker 安装

卸载旧版

```shell
sudo yum remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-engine
```

设置 Docker yum 源所在的位置

```shell
 sudo yum install -y yum-utils
 sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

安装 Docker

```shell
sudo yum -y install docker-ce docker-ce-cli containerd.io
```

安装成功后将 docker 设置为开机启动

```shell
[root@i-ul0sb6hn ~]# systemctl enable docker --now
Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
```

国内使用默认的镜像仓库可能会很慢，我们可以替换成阿里云的地址，`registry-mirrors` 用你自己的就行

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://gnsz5r5r.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 镜像操作

### **下载镜像**

可以到中央镜像仓库  `https://hub.docker.com/` 找你需要的镜像。

第一步首先找到你需要的镜像然后点击进去

![搜索](https://qiniu-note-image.ctong.top/note/images/202203312149988.png)

右边会有一个下载命令，我们可以直接使用这个命令来下载这个容器，这命令默认是下载的最新版本。

![下载命令](https://qiniu-note-image.ctong.top/note/images/202203312150831.png)

可以点 **tags** 来找你需要的版本

![切换tags](https://qiniu-note-image.ctong.top/note/images/202203312151865.png)

找到需要的版本后，可以使用 `docker pull <image>:<tag>` 命令下载，在 Docker 中，版本名还被称为标签

```shell
docker pull nginx:1.21.6-perl
```

### **查看已下载的镜像**

可以使用 `docker images` 命令查看已下载的镜像

```shell
[root@i-ul0sb6hn ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    605c77e624dd   3 months ago   141MB
```

### **删除镜像**

可以使用 `docker rmi <image>:<tag>` 命令移除镜像

```shell
[root@i-ul0sb6hn ~]# docker rmi nginx
Untagged: nginx:latest
Untagged: nginx@sha256:0d17b565c37bcbd895e9d92315a05c1c3c9a29f762b011a10c54a66cd53c9b31
Deleted: sha256:605c77e624ddb75e6110f997c58876baa13f8754486b461117934b24a9dc3a85
Deleted: sha256:b625d8e29573fa369e799ca7c5df8b7a902126d2b7cbeb390af59e4b9e1210c5
Deleted: sha256:7850d382fb05e393e211067c5ca0aada2111fcbe550a90fed04d1c634bd31a14
Deleted: sha256:02b80ac2055edd757a996c3d554e6a8906fd3521e14d1227440afd5163a5f1c4
Deleted: sha256:b92aa5824592ecb46e6d169f8e694a99150ccef01a2aabea7b9c02356cdabe7c
Deleted: sha256:780238f18c540007376dd5e904f583896a69fe620876cabc06977a3af4ba4fb5
Deleted: sha256:2edcec3590a4ec7f40cf0743c15d78fb39d8326bc029073b41ef9727da6c851f
```

再检查镜像列表，就可以看到已经被移除了

```shell
[root@i-ul0sb6hn ~]# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

###  **启动容器**

在下载了一个镜像后，可以使用 `docker run [OPTIONS] IMAGE [COMMAND] [ARG...]` 启动指定镜像

例如启动一个 nginx

```shell
docker run --name=nginx -d --restart=always nginx:latest
```

- `--name` 启动别名
- `-d` 后台启动
- `--restart` 容器是否开机自启，默认是 `no` ，如果需要设置为开启自启，可以设置为 `always`

###  **查看正在运行的容器**

如果需要查看正在运行的容器，可以使用 `docker ps` 进行查看

```shell
[root@i-ul0sb6hn ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS     NAMES
0546ab754a92   nginx:latest   "/docker-entrypoint.…"   29 seconds ago   Up 29 seconds   80/tcp    nginx
```

###  **删除容器**

删除一个已经停止的容器，通过这个命令 `docker rm <container_id | name>`

```shell
docker rm nginx
```

###  **停止一个正在运行的容器**

`docker stop <container_Id | name>` 命令可以停止一个正在运行的容器

```shell
docker stop nginx
```

###  **启动已停止运行的容器**

使用 `docker start <container_Id | name>` 命令

```shell
docker start nginx
```

###  **修改已启动容器的设置项**

`docker update [OPTIONS] CONTAINER [CONTAINER...] `,例如将一个容器修改为开启自启

```shell
docker update nginx --restart=always
```

###  **端口映射**

docker 容器其实是一个小的Linux，里面的nginx暴露了一个端口为80，但是它不是在我们主机上暴露的，所以我们无法通过 ip 去访问这个nginx。需要在 docker 做一层端口映射，将主机的某个端口映射到 docker 容器中的指定端口上。

```shell
docker run --name=nginx -d -p 88:80 nginx
```

 启动后检查 `PORTS` 是不是成功的 

```shell
[root@i-ul0sb6hn ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                               NAMES
8e6d1342b9e8   nginx     "/docker-entrypoint.…"   51 seconds ago   Up 50 seconds   0.0.0.0:88->80/tcp, :::88->80/tcp   nginx
```

![公网访问Nginx](https://qiniu-note-image.ctong.top/note/images/202204010843412.png)

###  **进入容器**

如果需要修改容器的配置或者什么东西，由于 Docker 的资源隔离是非常高的，所以不能直接修改，需要进入这个容器的控制台，类似 ssh 远程连接某个服务器。

```shell
docker exec -it nginx /bin/bash
```

```shell
[root@i-ul0sb6hn ~]# docker exec -it nginx /bin/bash
root@8e6d1342b9e8:/#
```

进到容器后，可以找到例如nginx的部署目录并修改内容

```shell
root@8e6d1342b9e8:/# cd /usr/share/nginx/html
root@8e6d1342b9e8:/usr/share/nginx/html# echo "<h1>Hello World</h1>" > index.html
```

修改后刷新浏览器访问

![修改index.html](https://qiniu-note-image.ctong.top/note/images/202204010900085.png)

改完后直接使用 `exit` 退出即可

###  **提交改变**

可以将自己已经修改好的镜像提交，例如配置nginx容器，配置了很多次配好了，如果移到新的服务器上，还需要重新配置。那么可以使用 `docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]` 命令将当前容器重新生成为一个新的镜像。

```shell
[root@i-ul0sb6hn ~]# docker commit -a "Clover You" -m "修改index.html" nginx clover-nginx:v0.0.1
sha256:3c5ae6aa7604e19944106c35299f2b317ae454a4a34bbf0a6875aad7aabcc296
```

可以再检查一下已下载的镜像

```shell
[root@i-ul0sb6hn ~]# docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
clover-nginx   v0.0.1    3c5ae6aa7604   27 seconds ago   141MB
nginx          latest    605c77e624dd   3 months ago     141MB
```

测试一下，将之前在运行的nginx删掉，直接运行我们刚刚 commit 的镜像

```shell
docker run -d -p 88:80 --name=clover-nginx clover-nginx:v0.0.1
```

再访问，它保留了我们上次修改容器的结果

![commit 镜像启动结果](https://qiniu-note-image.ctong.top/note/images/202204010917406.png)

###  **镜像传输**

当我们做好一个镜像后，如果我们需要给别人分享，有两种办法，一种是物理传输，还有一种是推送至远程仓库。

可以先使用 `docker save [OPTIONS] IMAGE [IMAGE...]` 命令将镜像压缩

```shell
[root@i-ul0sb6hn ~]# docker save -o clover-nginx.tar  clover-nginx:v0.0.1
[root@i-ul0sb6hn ~]# ls
clover-nginx.tar
```

之后直接将 `clover-nginx.tar` 分享出去就行了

再通过 `docker load [OPTIONS]` 命令加载这个镜像

```shell
[root@i-ul0sb6hn ~]# docker load -i ./clover-nginx.tar
Loaded image: clover-nginx:v0.0.1
```

检查镜像

```shell
[root@i-ul0sb6hn ~]# docker images
REPOSITORY     TAG       IMAGE ID       CREATED          SIZE
clover-nginx   v0.0.1    3c5ae6aa7604   19 minutes ago   141MB
```

---

推送到远程仓库，先创建一个镜像仓库

![创建仓库](https://qiniu-note-image.ctong.top/note/images/202204010939283.png)

填好仓库信息后点【create】就好

![创建](https://qiniu-note-image.ctong.top/note/images/202204010941163.png)

给我们本地镜像加个标签 `docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]`

```shell
docker tag clover-nginx:v0.0.1 cloveryou/clover-nginx:v0.0.1
```

最后通过 `docker push` 命令将镜像推送

```shell
docker push cloveryou/clover-nginx:v0.0.1
```

以上操作是需要登录的，如果没有登录需要使用 `docker login` 登录

```shell
[root@i-ul0sb6hn ~]# docker login -u <user name> -p <password>
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

登录成功后再重新推送

推送成功刷新仓库就可以看到了

![推送结果](https://qiniu-note-image.ctong.top/note/images/202204010951531.png)

如果是公开的仓库，也可以通过搜索搜到

![搜索结果](https://qiniu-note-image.ctong.top/note/images/202204010952767.png)

最后需要用就直接下载就好啦

```shell
docker pull cloveryou/clover-nginx:v0.0.1
```

###  **挂载数据到外部**

像我们之前修改nginx，是需要进入到容器里修改，其实可以将容器中的某个文件夹挂载到主机的某个位置。`docker run --name=xxx -d -v [target path]:[source path]:[缺省，ro容器内只读，rw容器内可读写]`

```shell
docker run --name=clover-nginx -d -p 88:80 -v /data/html:/usr/share/nginx/html:ro clover-nginx:v0.0.1
```

- `-v` 将容器中某个目录挂在到主机上，如果挂载的是一个文件，那么这个文件在主机上必须存在

启动后我们只要在主机的 `/data/html`

### **检查容器日志**

`docker logs <container>`

### 复制容器文件到主机

`docker cp <container>:<source path> <target path>`

例如将nginx中的文件复制出来

```shell
docker cp 52924565fcfc:/etc/nginx /data/nginx
```

```shell
[root@i-ul0sb6hn data]# ls
-->  nginx
```

然后将nginx的配置文件挂载到本地

```shell
docker run --name=clover-nginx -d -p 88:80 \
-v /usr/share/html:/data/html:ro \
-v /etc/nginx:/data/nginx:ro \
cloveryou/clover-nginx:v0.0.1
```

## 将现有应用打包成 Docker 镜像

以 SpringBoot 为例，先将我们的项目打包成 `.jar` 包。然后编写一个 `Dockerfile` 将这个 `.jar` 打包成 Docker 镜像。

```dockerfile
FROM openjdk:8-jdk-slim
LABEL maintainer=CloverYou

COPY target/**.jar /app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```

由于 Docker 镜像都是一个很小的 Linux 系统，并没有我们的运行环境，所以需要指定安装jdk环境 `openjdk:8-jdk-slim`。

然后将项目打包结果复制到 Linux 的某个位置，`ENTRYPOINT` 用于 Docker 启动时执行的命令，也可以使用 `CMD`。让 Docker 启动时执行 `java -jar /app.jar` 启动我们的项目。

Dockerfile 文件编写好后，需要到一个有 Docker 环境的机子上执行。

执行 `docker build [OPTIONS] <工作路径>` 命令打包

```shell
docker build -t spring-boot-hello-world:v1.0.0 .
```

**注意：如果出现以下错误，那是因为 Dockerfile 不能放在 `/` 目录**

```
error checking context: 'file ('/proc/1500/fd/7') not found or excluded by .dockerignore'.
```

打包完成后查看镜像，就可以看到我们打包的镜像

```shell
[root@i-ul0sb6hn ~]# docker images
REPOSITORY                TAG          IMAGE ID       CREATED         SIZE
spring-boot-hello-world   v1.0.0       a9c63c4b9ad4   6 seconds ago   313MB
```

启动容器

```shell
docker run -d -p 80:8080 --name=helloworld spring-boot-hello-world:v1.0.0
```

检查是否启动成功

```shell
[root@i-ul0sb6hn ~]# docker ps
CONTAINER ID   IMAGE                            COMMAND                CREATED         STATUS         PORTS                                   NAMES
7f62352ef378   spring-boot-hello-world:v1.0.0   "java -jar /app.jar"   3 seconds ago   Up 3 seconds   0.0.0.0:80->8080/tcp, :::80->8080/tcp   helloworld
```

然后访问我们的服务就可以了

![执行结果](https://qiniu-note-image.ctong.top/note/images/202204011533435.png)
