---
layout: post
title:  "树莓派GPIO开发"
date:   2018-01-06
categories: Raspi
tags: raspberrypi
---
# 树莓派GPIO开发

树莓派提供了40Pin的IO接口，除了电源和地，还有大量GPIO以及各类通信接口，可供用户开发或学习。  
先试一试最简单的GPIO应用，点亮一个LED以及检测按键输入，硬件上，LED连接在3引脚，按键链接在5引脚，也就是BCM2和BCM3  

### C库  

树莓派官方当然也提供了C语言库，用来访问底层接口的编程  
1. 安装C库
    下载[BCM2835库](http://www.airspayce.com/mikem/bcm2835/bcm2835-1.50.tar.gz)得到文件`bcm2835-1.50.tar.gz`  
    然后安装库
    ```bash
    tar zxvf bcm2835-1.50.tar.gz
    cd bcm2835-1.50/
    ./configure 
    make
    sudo make install
    ```
    于是编译好的 libbcm2835.a被安装在 /usr/local/lib/文件夹，  
    头文件bcm2835.h文件在/usr/local/include文件夹下
*    库使用[说明文档](http://www.airspayce.com/mikem/bcm2835/index.html).  
2. 用法  
    包含头文件  
    `#include <bcm2835.h>`  
    程序开始调用init函数，结束调用close函数  
    `bcm2835_init();`  
    `bcm2835_close();`  
    编译加链接选项  
    `-l bcm2835`  
    如果使用gpio以外的其他功能，运行程序需要root超级权限，使用sudo运行
3. 测试LED  
    下面写一个简单的程序，点亮LED灯程序来测试以及说明用法  

    ```c
        #include <bcm2835.h>

        int main(int argc, char **argv)
        {
            if (!bcm2835_init())
            {
                return 1;
            }

            bcm2835_gpio_fsel(RPI_GPIO_P1_11, BCM2835_GPIO_FSEL_OUTP);

            bcm2835_gpio_write(RPI_GPIO_P1_11, HIGH);

            bcm2835_delay(2000);

            bcm2835_gpio_write(RPI_GPIO_P1_11, LOW);

            bcm2835_close();

            return 0;
        }  
    ```  
    
    程序中首先调用`bcm2835_init()`初始化，然后设置11Pin为输出模式，接着输出写高电平，
    点亮LED，等待2秒后，关闭LED，调用`bcm2835_close()`关闭初始化中的相关设置，程序完成。  
    然后编译运行  

    ```
    gcc -o led led.c -l bcm2835
    ./led
    ```  
    运行后就能看到接到11引脚的LED亮2秒钟然后西门，程序退出  
4. 按键输入
    当用按键功能的时候，gpio设置输入模式，并根据实际需要设置上下拉电阻，然后读取引脚状态即可获取按键值  
    
    ```c
    bcm2835_gpio_fsel(RPI_GPIO_P1_05, BCM2835_GPIO_FSEL_INPT);
    bcm2835_gpio_set_pud(RPI_GPIO_P1_05, BCM2835_GPIO_PUD_UP);
    uint8_t key = bcm2835_gpio_lev(RPI_GPIO_P1_05);
    ```  
    
    稍微修改一下上面才程序，测试按键，如下  
    
    ```c
    #include <bcm2835.h>

    int main(int argc, char **argv)
    {
        if (!bcm2835_init())
        {
            return 1;
        }

        bcm2835_gpio_fsel(RPI_GPIO_P1_11, BCM2835_GPIO_FSEL_OUTP);

        bcm2835_gpio_fsel(RPI_GPIO_P1_05, BCM2835_GPIO_FSEL_INPT);

        bcm2835_gpio_set_pud(RPI_GPIO_P1_05, BCM2835_GPIO_PUD_UP);

        bcm2835_gpio_write(RPI_GPIO_P1_11, HIGH);

        while (bcm2835_gpio_lev(RPI_GPIO_P1_05) != 0)
        {
            bcm2835_delay(10);
        }

        bcm2835_gpio_write(RPI_GPIO_P1_11, LOW);

        bcm2835_close();

        return 0;
    }
    ```

    点亮LED后程序持续扫描按键，当按键被按下，while循环退出，关闭LED，程序结束。

*   [参考文档](http://www.airspayce.com/mikem/bcm2835/)