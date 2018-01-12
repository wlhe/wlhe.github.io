---
layout: post
title:  "树莓派配置路由器"
date:   2018-01-06
categories: Raspi
tags: raspberrypi
---

### 1. 安装工具  
```
    sudo apt-get install hostapd
    sudo apt-get install isc-dhcp-server
```
### 2. 修改配置文件
    `sudo vim /etc/network/interfaces`

注释掉原来的部分，修改如下，ip同网段

```
    #allow-hotplug wlan0
    #iface wlan0 inet manual
    #    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
    #
    #allow-hotplug wlan1
    #iface wlan1 inet manual
    #    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
     
    iface wlan0 inet static
        address 192.168.1.10 
        netmask 255.255.255.0
```   

### 3. hostapd配置
修改hostapd默认配置文件  

    `sudo vim /etc/default/hostapd`

添加下面内容

    `DAEMON_CONF="/etc/hostapd/hostapd.conf"`  
    
/etc/hostapd/hostapd.conf 为hostapd的配置文件修改为
```
    interface=wlan0
    driver=nl80211
    ssid=RPI
    hw_mode=g
    channel=11
    wpa=2
    wpa_passphrase=12345678
    wpa_key_mgmt=WPA-PSK
    wpa_pairwise=CCMP
    rsn_pairwise=CCMP
    beacon_int=100
    auth_algs=3
    wmm_enabled=1
```

ssid是WIFI名称，wpa_passphrase是WIFI密码

重启服务

    `sudo service hostapd restart`

hostapd相关配置完成
### 4. dhcp配置

    `sudo vim /etc/dhcp/dhcpd.conf`

内容为
```
    default-lease-time 600;
    max-lease-time 7200;
    log-facility local7;
    subnet 192.168.1.0 netmask 255.255.255.0 {
    	range 192.168.1.200 192.168.1.250;
    	option routers 192.168.1.10;
    	option broadcast-address 192.168.1.255;
    	option domain-name-servers 8.8.8.8,8.8.4.4;
    	default-lease-time 600;
    	max-lease-time 7200;
    }
```

重启dhcp服务

    `sudo service  isc-dhcp-server restart`

配置初步完成，可以用手机或者笔记本搜索到名为RPI密码12345678的WIFI并且连接，但是只能连接还不能上网。

### 5. 配置上网

如果有线网卡链接有网络的网线，则可以配置上网，方法有很多  
先打开IP转发  

    `sudo vim /etc/sysctl.conf`  

去掉下面这句前的注释符#  

    `net.ipv4.ip_forward=1`

#### 1) IP转发
```
    sudo iptables -F
    sudo iptables -X
    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```  

此时WIFI已经能够上网，但是重启后需要再次运行上面命令才有效，可以将上面的命令写入脚本文件中每次开机自动执行即可。或者运行

```
    sudo bash
    iptables-save > /etc/iptables.up.rules
    exit
```

将当前iptable设置存入/etc/iptables.up.rules文件中

    `sudo vim /etc/network/if-pre-up.d/iptables`

输入：

```
    #!/bin/bash
    /sbin/iptables-restore < /etc/iptables.up.rules

```

该文件启动网络时会调用，将之前保存的设置恢复，相当于执行前面三行命令
给该文件添加权限

    `sudo chmod 755 /etc/network/if-pre-up.d/iptables`

最后

    `sudo sysctl -p`

就可以成功上网了
    
[Ref: http://shumeipai.nxez.com/2013/09/11/raspberry-pi-configured-as-a-wireless-router.html](http://shumeipai.nxez.com/2013/09/11/raspberry-pi-configured-as-a-wireless-router.html)
    
#### 2) 桥接bridg

通过建立双网卡桥接br0链接wlan0和eth0实现上网
    
[Ref: https://wiki.debian.org/BridgeNetworkConnections](https://wiki.debian.org/BridgeNetworkConnections)  
参见另一篇[桥接笔记]({{site.url}}/2018/01/linux-dual-network-card)