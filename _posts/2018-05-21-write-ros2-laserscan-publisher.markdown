---
layout:     post
title:      "【ROS2总结】点激光扫描仪数据发布（驱动编写）"
subtitle:   "Wirte laser scan data publisher(ROS2 driver)"
date:       2018-05-21 10:34:25
author:     "Playfish"
header-img: "img/in-post/write-laserscan-publisher/laserscan.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - ROS2
    - Release
    - driver
---


> 说明:<br><br>
> 本文首发于 [Playfish Blog](http://carlzhang.club/2018/05/21/write-ros2-laserscan-publisher/)，转载请保留链接。

## 前言

在上一篇博客中，介绍了如何[在ROS1下编写激光测距传感器节点](http://carlzhang.club/2017/11/25/write-laserscan-publisher/)，在此基础上我们下面讲述如何在ROS2下编程来创建激光测距传感器节点。

在接下来的博客中将根据本身经验来编写一些ROS相关内容，权当是作为记忆来分享。


## ROS2驱动——点激光扫描仪


下面将讲述使用ROS官方未有的传感器来作为样例，我使用的是莱旭光电的CHT-10点激光传感器作为 介绍，首先这款传感器通信协议部分比较简单，基本上打开串口就可以读出数据，在将数据进行转换就可以得到想要的距离，比较方便，免去复杂的部分，对教程也有帮助，毕竟教程主要讲述的是ROS2驱动部分。
为了不妨碍与上一篇的衔接问题，这里我们重新开始从头介绍，大家就可以不用看上一篇博客。


相关ROS2驱动代码可以在github上找到进行下载：https://github.com/Playfish/cht10_node/tree/ros2


CHT-10是点激光，扫描范围是0.05-10m，真实的应该是0.15-10m，10m未测试，不过0.15盲区还是有的。

下图是CHT-10的通信协议部分介绍：

![](/img/in-post/write-laserscan-publisher/laserscan_protocol.png)

可以看到想要的数据在9~12位（四字节），得到的数据需要除以1000才能得到真实的距离（米），ROS消息中基本单位是米。

### 消息选定

关于选择什么消息作为该传感器的ROS2消息，这个相对来说不是很重要，不过作为通用项来说，尽量向通用靠近（即使用ROS官方消息定义，而不是自己定义消息）。目前激光消息的话，有两个选择，第一个是sensor_msgs/LaserScan消息类型，不过该消息类型适合有角度的传感器，即180°或360°的2D雷达或激光，而目前使用的激光是单点类型，和激光笔差不多，只有一个点，那么可以选择sensor_msgs/Range消息类型，该消息类型适用于超声波传感器、红外传感器单点类型，CHT-10就用这个类型。
注意：在ROS2中，头文件将采用<消息包>/<通信机制>/消息名称.hpp形式，拿本教程为例，

| ROS1 |  ROS2 |
|------|-------|
|sensor_msgs/Range.h|sensor_msgs/msg/range.hpp|


关于该ROS2消息,官方暂未提供API文档，大家可以查看该文件来查看API，基本语法与ROS1一致。

### 准备工作

传感器选型选定、通信消息选定，那么接下来还需要做：

 * Frame_id：Frame在ROS1中作用至关重要，消息将和TF绑定才可以读取数据，在这里作为通用可配置，暂定内容为：laser，用户可自定义设置（通过ROS2 Parameters设置）。
 * 串口：避免和其他传感器串口冲突，因此在这里预留出一个串口设置参数，用户可以自定义设置（通过ROS2 Parameters设置）。
 * 话题：消息内容需要通过话题发布，并且话题需要唯一，不然容易崩溃，在这里选择话题为“range”

当然以上部分可以不考虑，这个是作为通用型需要考虑的，暂时可以先忘记以上部分，下面开启代码部分

### 测试代码编写

在这部分，主要先编写测试代码，如果测试代码无问题，就可以写到node中，测试代码和ROS无关系，只是通过串口读取数据，并且在终端显示出来，整体思路如下：

传感器选择 -> 测试程序测试（类似串口助手） -> ROS2 node绑定（将测试程序读取部分加入到ROS2中）

下面是测试程序部分，也可以在cht10_node/test中找到，名为[test_cht10.cpp][2]：
```
#include <string>
#include <cht10_node/serial_func.hpp>
#include <cstdlib>

#include <iostream>
#include <stdint.h>
#define BUFSIZE 17

int main(int argc, char** argv){
  cht10_serial_func::Cht10Driver cht10driver_;

  std::string serialNumber_;
  serialNumber_ = "/dev/ttyUSB0";
  int baudRate_ = 115200;

  std::stringstream ostream;
  int fd, len, rcv_cnt;
  bool success_flag;
  char buf[40], temp_buf[BUFSIZE],result_buf[BUFSIZE];
  unsigned int laser_data=0;
  char data_buf[4];
  rcv_cnt = 0;
  success_flag = false;
  memset(buf, 0xba, sizeof(buf));
  memset(temp_buf, 0xba, sizeof(temp_buf));
  memset(result_buf, 0xba, sizeof(result_buf));

  fd = open(serialNumber_.c_str(), O_RDWR | O_NOCTTY | O_NDELAY );
  if(fd < 0){
    std::cout<<"Open Serial: "<<serialNumber_.c_str()<<" Error!";
    exit(0);
  }
  
  cht10driver_.UART0_Init(fd,baudRate_,0,8,1,'N');
  while(1){
    len = cht10driver_.UART0_Recv(fd, buf,40);
    if(len>0){
      for(int i = 0; i < len; i++){
        if(rcv_cnt<=BUFSIZE){  
          result_buf[rcv_cnt++] = buf[i];
          if(rcv_cnt == BUFSIZE){
            success_flag = true;
          }
        }//end if
        else{
          /****
          *  checkout received data
          */
          success_flag = false;

          for(int count = 0; count < 4; count++){
            data_buf[count] = result_buf[9+count];
          }
          sscanf(data_buf, "%x", &laser_data);
          std::cout<<"sensor data:"<<laser_data<<", Distance: "<<(double)laser_data/1000<<std::endl;
          /****
           *  data writing end
           */
          if('~' == buf[i]){
            rcv_cnt = 0;
            result_buf[rcv_cnt++] = buf[i];
          }
        }//end else
      }//end for    
    }
  }
}
```
我将串口部分封装成一个类名为`Cht10Driver`，该类包含串口读/写部分、初始化。

通过使用`ament build`可以得到一个名为```test_cht10```的可执行文件，使用`./test_cht10`可以运行，将CHT-10插入到电脑上，并且电脑只有一个ttyUSB0，就可以读取数据，数据将显示为sensor data: 传感器毫米数据， Distance: 传感器米数据。

### ROS2驱动编写

上部分讲述了测试程序编写，主要作用是通过串口读取传感器数据，那么得到传感器数据之后，就可以将传感器数据填充到ROS消息中，然后通过话题形式发布出去，如下是将传感器数据填充到ROS消息并生成节点的主要部分：

完整代码可以查看： [cht10_node.cpp][3]
```
/**
 * @file /cht10_node/src/cht10/cht10_node.cpp
 *
 * @brief Implementation for dirver with read data from Cht10 nodelet
 *
 * @author Carl
 *
 **/

/*****************************************************************************
 ** Includes
 *****************************************************************************/
#include <chrono>
#include <cstdio>
#include <memory>
#include <string>

#include <rclcpp/rclcpp.hpp>
#include <rcutils/logging_macros.h>

#include <functional>

#include <std_msgs/msg/string.hpp>
#include <sensor_msgs/msg/range.hpp>

#include <cht10_node/cht10_node.h>

#define ROS_ERROR RCUTILS_LOG_ERROR
#define ROS_INFO RCUTILS_LOG_INFO
#define ROS_ERROR_THROTTLE(sec, ...) RCUTILS_LOG_ERROR_THROTTLE(RCUTILS_STEADY_TIME, sec, __VA_ARGS__)

using namespace cht10_serial_func;

  Cht10Func::Cht10Func(rclcpp::Node::SharedPtr & node):node_(node), 
serialNumber_("/dev/USB0"),
frame_id("laser"){

    msg_ = std::make_shared<sensor_msgs::msg::Range>();

    scan_pub_ = node_->create_publisher<sensor_msgs::msg::Range>("range");

    parameter_service_ = std::make_shared<rclcpp::ParameterService>(node_);

    node_->get_parameter("serialNumber", serialNumber_);
    node_->get_parameter("baudRate", baudRate_);
    node_->get_parameter("frame_id", frame_id);

    rcv_cnt = 0;
    success_flag = 0;

    fd = open(serialNumber_.c_str(), O_RDWR | O_NOCTTY | O_NDELAY );
    if(fd < 0){
      ROS_ERROR("Open Serial: %s Error!",serialNumber_.c_str());
      exit(0);
    }

    memset(buf, 0, sizeof(buf));
    memset(temp_buf, 0, sizeof(temp_buf));
    memset(result_buf, 0, sizeof(result_buf));
    Cht10driver_.UART0_Init(fd,baudRate_,0,8,1,'N');
    ROS_INFO("Open serial: [ %s ], successfully, with idex: %d.", serialNumber_.c_str() ,fd);

    update();
  }

  Cht10Func::~Cht10Func(){
  }

  double Cht10Func::data_to_meters(unsigned int &data, int scale){
    return (double)data/scale;
  }

  void Cht10Func::publish_scan(double nodes, builtin_interfaces::msg::Time start,
                    std::string frame_id){

    float final_range;
    std::shared_ptr<sensor_msgs::msg::Range> range_msg;
    range_msg->field_of_view = 0.05235988;
    range_msg->max_range = 10.0;
    range_msg->min_range = 0.05;
    range_msg->header.frame_id = frame_id;
    range_msg->radiation_type = sensor_msgs::msg::Range::INFRARED;
    if(nodes > range_msg->max_range){
      final_range = std::numeric_limits<float>::infinity();
    }else if(nodes < range_msg->min_range){
      final_range = -std::numeric_limits<float>::infinity();
    }else{
      final_range = nodes;
    }
    range_msg->header.stamp = start;
    range_msg->range = final_range;

    scan_pub_->publish(range_msg);

  }

  bool Cht10Func::get_scan_data(){
    len = Cht10driver_.UART0_Recv(fd, buf,40);
    if(len>0){
      for(int i = 0; i < len; i++){
        if(rcv_cnt<=BUFSIZE){  
          result_buf[rcv_cnt++] = buf[i];
          if(rcv_cnt == BUFSIZE){
            success_flag = true;
          }
        }//end if
        else{
          /****
          *  checkout received data
          */
          success_flag = false;
           for(int count = 0; count < 4; count++){
            data_buf[count] = result_buf[9+count];
          }
          sscanf(data_buf, "%x", &laser_data);
          
          //std::cout<<"sensor data:"<<laser_data<<std::endl;

          /****
           *  data writing end
           */
          if('~' == buf[i]){
            rcv_cnt = 0;
            result_buf[rcv_cnt++] = buf[i];
          }
        }//end else
      }//end for    
    }
    return success_flag;
  }

  void Cht10Func::update(){
    rcv_cnt = 0;
    success_flag = 0;
    laser_data = 0;
    memset(buf, 0, sizeof(buf));
    memset(temp_buf, 0, sizeof(temp_buf));
    memset(result_buf, 0, sizeof(result_buf));
    ROS_INFO("Begin receive data from %s, with idex: %d.",serialNumber_.c_str(),fd);
    fd = open(serialNumber_.c_str(), O_RDWR | O_NOCTTY | O_NDELAY );
    Cht10driver_.UART0_Init(fd,baudRate_,0,8,1,'N');
    // Create a function for when messages are to be sent.
    auto publish_message =
      [this]() -> void
      {
//        start_scan_time = 
        rclcpp::Time(start_scan_time);
        success_flag = get_scan_data();
          
        //Send data
        publish_scan(data_to_meters(laser_data,SCALE),
                         start_scan_time, frame_id);
        countSeq++;
      };

    ROS_INFO("Shotdown and close serial: %s.", serialNumber_.c_str());
    Cht10driver_.UART0_Close(fd);

    // Use a timer to schedule periodic message publishing.
    timer_ = node_->create_wall_timer(300ms, publish_message);
  }
```

其中get_scan_data()函数，就是测试代码内容，得到数据后将数据发送到publish_scan()函数中，就可以发布传感器数据了。

编写可执行程序节点，创建名为cht10_node_ros.cpp，内容为：
```
#include <cht10_node/cht10_node.h>

int main(int argc, char * argv[])
{
  // Initialize any global resources needed by the middleware and the client library.
  // You must call this before using any other part of the ROS system.
  // This should be called once per process.
  rclcpp::init(argc, argv);

  // Create a node.
  rclcpp::Node::SharedPtr node = rclcpp::Node::make_shared("cht10_node");

  cht10_serial_func::Cht10Func cht(node);

  // spin will block until work comes in, execute work as it becomes available, and keep blocking.
  // It will only be interrupted by Ctrl-C.
  rclcpp::spin(node);

  rclcpp::shutdown();
  return 0;
}
```

### 测试

代码完成后，按照格式进行编写CMakeList、package文件，运行catkin_make就可以得到名为cht10_node.so动态库，运行驱动节点，命令如下：
```
ros2 run cht10_node cht10_node_ros 
```
如果一切政策，运行ros2 topic list 可以得到/range话题。

使用ros2 topic echo /range就可以得到传感器的ROS2消息内容，包括传感器距离。

目前ROS2功能还未完全开发完毕，一些图形化还需借助ROS1来完成。

[2]: https://github.com/Playfish/cht10_node/blob/ros2/test/test_cht10.cpp
[3]: https://github.com/Playfish/cht10_node/blob/ros2/src/cht10_node.cpp
