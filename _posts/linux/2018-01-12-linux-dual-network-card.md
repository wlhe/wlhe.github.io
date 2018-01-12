---
layout: post
title:  "Linux双网卡设置"
date:   2018-01-12
categories: Linux
tags: linux
---

1. 安装 brctl工具

    `sudo apt-get install bridge-utils`

2. 添加桥
```
# brctl addbr br0 #创建桥接 br0
# brctl addif br0 eth0 eth0 #添加 eth0, eth1 到 br0
# ifconfig br0 192.168.1.1 netmask 255.255.255.0 broadcast 192.168.1.255 up
```

3. 打开ip转发
```
sudo vim /etc/systcl.conf
net.ipv4.ip_forward=1
```
去掉该行前面的注释符#
```
sudo sysctl -p /etc/sysctl.conf 
```
文件立即生效

3. 修改配置

    `vim /etc/network/interfaces`
```bash
auto lo br0
iface lo inet loopback
auto eth0
iface eth0 inet manual
auto eth1
iface eth1 inet manual
iface br0 inet dhcp
bridge_ports eth0 eth1
或者使用静态IP
iface br0 inet static
bridge_ports eth0 eth1
address 192.168.1.2
broadcast 192.168.1.255
netmask 255.255.255.0
gateway 192.168.1.1
```

4. 重启机器

Ref: https://wiki.debian.org/BridgeNetworkConnections