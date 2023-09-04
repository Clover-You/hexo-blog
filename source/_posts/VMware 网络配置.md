title: VMware 网络配置
categories: 随笔
tags: [随笔]
date: 2022-02-019 22:54:21
---
# VMware 网络配置

运行 `dhcpd` 命令分配一个 ip 地址。

通过 `ifconfig` 或 `ip addr` 命令拿到刚刚分配的IP：192.168.135.128（以你自己的为准）

修改 ` /etc/sysconfig/network-scripts/ifcfg-ens33` 文件

将原来的 `BOOTPROTO=dhcp` 修改为 `BOOTPROTO=static` ，目的是使用我们指定的ip作为默认IP

添加 `ONBOOT=yes`

指定我们刚刚分配的IP `IPADDR=192.168.135.128`

设置子网腌码，一般是255.255.255.0，`NETMASK=255.255.255.0`

![截屏2022-02-19 下午10.45.55](https://qiniu-note-image.ctong.top/note/images/202202192246384.png)

最后设置网关，通过 `cat /Library/Preferences/VMware\ Fusion/vmnet8/nat.conf` 查看vmware给我们分配的网关

![网关地址](https://qiniu-note-image.ctong.top/note/images/202202192250941.png)

也可以通过`ifconfig`命令查看网关：

![网关地址](https://qiniu-note-image.ctong.top/note/images/202202192251644.png)
