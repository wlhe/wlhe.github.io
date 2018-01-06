---
layout: post
title:  "树莓派 Python GPIO"
date:   2018-01-06
categories: Raspi
tags: raspberrypi
---
### **Python**

树莓派官方提供了完整的Python GPIO库gpiozero，并且一集随系统一起安装，可以直接使用，只需要导入库就行了。  
接下来定义个led变量,传入的参数是BCM引脚号
```py
from gpiozero import LED, Button
led = LED(2)
```
然后用led操作就可以控制硬件引脚点亮或关闭LED  

 ```py
 led.on()         #turn on led
 led.off()        #turn off led
 led.blink()      #blink led
 led.toggle()     #toggle led
 ```

按键的操作也比较类似，定义一个按键变量，对变量进行操作，同样传入BCM引脚编号作为参数  
```py
 button = Button(3)  
 button.wait_for_press()    
```
当按键被按下，该函数返回
也可以检测按键被按下或者被释放的信号
```py
 button.when_pressed
 button.when_released  
```
下面的程序实现按键按下，点亮LED，再按一下，关闭LED的功能  
```py
 while True:
      button.wait_for_press()
      sleep(.1)
      led.toggle()
      sleep(.1)
```

另外可以用PWM功能实现呼吸灯功能  
```py
from gpiozero import PWMLED

pwmled = PWMLED(2)
i = 0
p = True
while True:
    pwmled.value = i / 500.0
    if(p):
        i += 1
    else:
        i -= 1
    if(i >= 500):
        p = False
    elif(i <= 0):
        p = True
    sleep(0.002)
```

*[参考文档](https://gpiozero.readthedocs.io/en/v1.3.1/)