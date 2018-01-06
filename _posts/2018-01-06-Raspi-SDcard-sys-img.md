---
layout: post
title:  "树莓派SD卡系统制作"
date:   2018-01-06
categories: Raspi
tags: 
---
# 树莓派SD卡系统制作

1. 下载系统镜像  
    树莓派使用SD卡作为系统硬盘，支持多种系统，可到[**官方网站**](https://www.raspberrypi.org/downloads/)下载  
    此处使用[**Rasbian**](https://www.raspberrypi.org/downloads/raspbian/)，下载得到文件`2017-01-11-raspbian-jessie.zip`.

2. 解压文件  
    `unzip 2017-01-11-raspbian-jessie.zip`  
    得到文件  
    `2017-01-11-raspbian-jessie.img`  
    正是要下载到SD卡的文件

3. 准备SD卡，8G以上  
    `df -h`  
    查看当前磁盘设备，插入SD卡，再次输入  
    `df -h`  
    则可看到多出来的设备也就是SD卡，卸载SD卡已有的所有分区  
    `sudo umount /dev/sdc1`  

4. 烧写系统镜像文件  
    进入镜像文件存放的文件夹  
    `sudo dd bs=4M if=2017-01-11-raspbian-jessie.img of=/dev/sdc`  
    大概需要花几分到十几分钟等待完成烧写

5. 扩展系统分区  
    烧写完成后，拔掉SD卡重插，可以看到有两个分区，分区1是boot分区，分区2是系统分区  
    此时SD卡的空间并没有完全分配，下面的命令将剩余空间分配给分区2，充分利用SD卡的存储空间，避免出现硬盘空间不足  
    ```
    sudo umount /dev/sdc2  
    sudo parted /dev/sdc unit % resizepart 2 100 unit MB print  
    sudo resize2fs -f /dev/sdc2  
    ```

6. 启动系统
    将SD卡插入树莓派卡槽，接通电源，树莓派即可正常启动

7. 链接wifi无线网络
    链接显示器可直接桌面操作  
    命令行操作
    `sudo vi /etc/wpa_supplicant/wpa_supplicant.config`
    在末尾加入
    ```
    network={
        ssid="wifi_name"
        psk="pass_word"
    }
    ```
    然后重启网卡  
    ```
    sudo ifdown wlan0
    sudo ifup wlan0
    ```  
    或直接重启系统，就可以正常上网了
