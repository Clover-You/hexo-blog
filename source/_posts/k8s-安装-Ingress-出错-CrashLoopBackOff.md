---
title: k8s 安装 Ingress 出错 CrashLoopBackOff
date: 2022-03-25 11:25:32
tags: [随笔,运维]
categories: 随笔
---

# k8s 安装 Ingress 出错 CrashLoopBackOff

学习 k8s 在安装 Ingress 的时候一直出现 `CrashLoopBackOff` 

```shell
[root@k8s-node1 k8s]# kubectl get pods --all-namespaces
NAMESPACE       NAME                                READY   STATUS             RESTARTS   AGE
ingress-nginx   nginx-ingress-controller-kmkhx      0/1     CrashLoopBackOff   4          2m56s
ingress-nginx   nginx-ingress-controller-nncd7      1/1     Running            0          2m56s
```

**搞了我3天**

检查日志 `kubectl logs nginx-ingress-controller-kmkhx -n ingress-nginx` ，说80端口被占用。

```shell
[root@k8s-node1 k8s]# kubectl logs nginx-ingress-controller-kmkhx -n ingress-nginx
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:    0.20.0
  Build:      git-e8d8103
  Repository: https://github.com/kubernetes/ingress-nginx.git
-------------------------------------------------------------------------------

F0325 02:12:50.629951       9 main.go:72] Port 80 is already in use. Please check the flag --http-port
```

突然想到我这集群镜像是直接复制之前开发虚拟机的系统来搭建的集群，安装了很多奇奇怪怪得妖魔鬼怪...

之后检查 docker 发现了之前配置的 nginx在 80 端口启动（使用 `docker ps` 命令）。emmm...

![docker](https://qiniu-note-image.ctong.top/note/images/202203251118681.png)

由于这是 k8s master 主机，所以我直接把不相干的 docker 镜像全部卸载了（其它节点也全部干掉不相干的镜像）

如果容器在启动，那么停止掉它 `docker stop <container-id>`

移除这个容器 `docker rm  <container-id>`

最后删除这个镜像 `docker rmi <image-name:tags>`

最后问题就解决了

**你可能和我的情况不一样，具体情况需要根据日志错误提示来修复**
