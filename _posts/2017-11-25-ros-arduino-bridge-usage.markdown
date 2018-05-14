---
layout:     post
title:      "ros_arduino_bridge使用总结"
subtitle:   "Usage summary for ros-arduino-bridge"
date:       2017-11-25 17:04:46
author:     "Playfish"
header-img: "img/in-post/ros-arduino-bridge-usage/arduono_logo.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - ROS1
    - ros_arduino_bridge
---


> 说明:<br><br>
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/77528798)，转载请保留链接。

## 前言

关于ROS和Arduino通信方式，刚开始大多使用的是rosserial_arduino这个库，随后又诞生了一个新的库名为ros_arduino_bridge，两者对比如下：

 * [rosserial_arduino][1]：由于将Arduino内的程序写成ros节点形式，所以能够快速的通过ROS控制Arduino，并且可以忽略通信协议层，与之相同的还有rosserial_stm32f1库，但是由于将Arduino作为ROS节点，不可避免的产生通信延时较大的问题，而且运行该库是通过rosserial_python包内的serial_node.py启动，该脚本使用了tcpip协议，有点大材小用，uno速率会跟不上，导致启动过程中一崩溃很难在连接上的问题，学习ROS可以使用。
 * [ros_arduino_bridge][2]：是自定义通信协议方式来与Arduino进行通信，和大多数控制形式一致，将通信协议固定可以让初学者在使用过程中了解通信协议，由于是自定义通信协议形式，可以即插即用，和rosserial_arduino有本质差别，不过在使用过程中需要自己配置，所以需要有一定的arudino编程基础，早期的EAI dashgo D1采用的就是这种形式，使用这个库可以缩短底盘开发时间，并且Arduino有很多开源包供使用，会相对轻松一些。

关于如何使用rosserial_arduino_bridge，作者在github的[Readme][3]中写的很清楚，当然，是使用英文编写，如果大家想要查看中文，[这里][4]是一个很好的方式，我做的小车就是参考这篇博客，写的很清楚。

## 注意事项
### 编码器

关于如何使用我就不在赘述了，大家可以参考上面博客进行查看，作者代码在github上也有给出。查看作者代码发现作者应该用的Arduino mega板，电机控制器是L298N模块，我使用的是Arduino Uno板作为测试，电机控制器也选用的L298N，所以更改代码如下：

  将[ros_arduino_bridge/ros_arduino_firmware/src/libraries/ROSArduinoBridge/encoder_driver.h][5]的内容：
```
#ifdef ARDUINO_MY_COUNTER  
  #define LEFT_ENC_A 2  
  #define LEFT_ENC_B 22  
  #define RIGHT_ENC_A 21  
  #define RIGHT_ENC_B 24  
  void initEncoders();  
  void leftEncoderEvent();  
  void rightEncoderEvent();  
#endif 
```
修改为
```
    #ifdef ARDUINO_MY_COUNTER  
      #define LEFT_ENC_A 2  
      #define LEFT_ENC_B 4  
      #define RIGHT_ENC_A 3  
      #define RIGHT_ENC_B 11  
      void initEncoders();  
      void leftEncoderEvent();  
      void rightEncoderEvent();  
    #endif  
```
关于这点需要注意，由于Arduino Uno只有两个中断，所以只能将两个中断分开给左编码器和右编码器，即Uno两个中断对应硬件分别为2和3，其中引脚2对应中断号为0，引脚3对应中断号为1，如下图 

![](/img/in-post/ros-arduino-bridge-usage/arduino_pin.jpeg)

修改完成后，还需修改[ros_arduino_bridge/ros_arduino_firmware/src/libraries/ROSArduinoBridge/encoder_driver.ino][6]

将内容
```
    void initEncoders(){  
      pinMode(LEFT_ENC_A, INPUT);  
      pinMode(LEFT_ENC_B, INPUT);  
      pinMode(RIGHT_ENC_A, INPUT);  
      pinMode(RIGHT_ENC_B, INPUT);  
      
      attachInterrupt(0, leftEncoderEvent, CHANGE);  
      attachInterrupt(2, rightEncoderEvent, CHANGE);  
```

修改为
```
    void initEncoders(){  
      pinMode(LEFT_ENC_A, INPUT);  
      pinMode(LEFT_ENC_B, INPUT);  
      pinMode(RIGHT_ENC_A, INPUT);  
      pinMode(RIGHT_ENC_B, INPUT);  
      
      attachInterrupt(0, leftEncoderEvent, CHANGE);  
      attachInterrupt(1, rightEncoderEvent, CHANGE);  
```

注意变化，mega有四个中断，而Uno只有两个，所以需要修改为中断0、中断1。不然造成的结果是有一个电机编码器无数据，一直为0。

同样的，如果引脚类型不对应也会造成错误，如右电机，我将编码器A接上引脚3，编码器B接上引脚12，造成结果是右编码器数据一直在累加，即使电机没有转动，这是由于引脚3作为中断，类型为PWN引脚，而引脚12则为数字引脚，所以需要将编码器B接上引脚11，这是由于引脚11也为PWM类型，这样就不会造成错误。

[1]: http://wiki.ros.org/rosserial_arduino
[2]: http://wiki.ros.org/rosserial_arduino_bridge
[3]: https://github.com/hbrobotics/ros_arduino_bridge/blob/indigo-devel/README.md
[4]: blog.csdn.net/github_30605157/article/details/51344150/
[5]: https://github.com/Sunlcy/ros_arduino_bridge/tree/indigo-devel-saturnbot-pid-tuning/ros_arduino_firmware/src/libraries/ROSArduinoBridge/encoder_driver.h
[6]: https://github.com/Sunlcy/ros_arduino_bridge/tree/indigo-devel-saturnbot-pid-tuning/ros_arduino_firmware/src/libraries/ROSArduinoBridge/encoder_driver.ino
