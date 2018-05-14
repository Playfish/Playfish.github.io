---
layout:     post
title:      "【ROS总结】点激光扫描仪数据发布（驱动编写）"
subtitle:   "Wirte laser scan data publisher(ROS driver)"
date:       2017-11-25 17:05:58
author:     "Playfish"
header-img: "img/in-post/write-laserscan-publisher/laserscan.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - ROS1
    - Release
    - driver
---


> 说明:<br><br>
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/78632555)，转载请保留链接。

## 前言

在使用ROS的过程中，不可避免的会遇到要自己手动编写驱动、节点、行为库、服务、消息等内容，尤其是需要一些传感器，而所需传感器没有相关ROS驱动时，下面将讲述的是在使用的传感器中，没有ROS驱动时，我们将如何使用该传感器。

 

在接下来的博客中将根据本身经验来编写一些ROS相关内容，权当是作为记忆来分享。


## ROS驱动——点激光扫描仪


下面将讲述使用ROS官方未有的传感器来作为样例，我使用的是莱旭光电的CHT-10点激光传感器作为 介绍，首先这款传感器通信协议部分比较简单，基本上打开串口就可以读出数据，在将数据进行转换就可以得到想要的距离，比较方便，免去复杂的部分，对教程也有帮助，毕竟教程主要讲述的是ROS驱动部分。


相关ROS驱动代码可以在github上找到进行下载：https://github.com/Playfish/cht10_node


CHT-10是点激光，扫描范围是0.05-10m，真实的应该是0.15-10m，10m未测试，不过0.15盲区还是有的。

下图是CHT-10的通信协议部分介绍：

![](/img/in-post/write-laserscan-publisher/laserscan_protocol.png)

可以看到想要的数据在9~12位（四字节），得到的数据需要除以1000才能得到真实的距离（米），ROS消息中基本单位是米。

### 消息选定

关于选择什么消息作为该传感器的ROS消息，这个相对来说不是很重要，不过作为通用项来说，尽量向通用靠近（即使用ROS官方消息定义，而不是自己定义消息）。目前激光消息的话，有两个选择，第一个是sensor_msgs/LaserScan消息类型，不过该消息类型适合有角度的传感器，即180°或360°的2D雷达或激光，而目前使用的激光是单点类型，和激光笔差不多，只有一个点，那么可以选择sensor_msgs/Range消息类型，该消息类型适用于超声波传感器、红外传感器单点类型，CHT-10就用这个类型。


关于该消息：可以查看[sensor_msgs/Range][1]

### 准备工作

传感器选型选定、通信消息选定，那么接下来还需要做：

 * Frame_id：Frame在ROS中作用至关重要，消息将和TF绑定才可以读取数据，在这里作为通用可配置，暂定内容为：laser，用户可自定义设置（通过ROS Parameters设置）。
 * 串口：避免和其他传感器串口冲突，因此在这里预留出一个串口设置参数，用户可以自定义设置（通过ROS Parameters设置）。
 * 话题：消息内容需要通过话题发布，并且话题需要唯一，不然容易崩溃，在这里选择话题为“range”

当然以上部分可以不考虑，这个是作为通用型需要考虑的，暂时可以先忘记以上部分，下面开启代码部分

### 测试代码编写

在这部分，主要先编写测试代码，如果测试代码无问题，就可以写到node中，测试代码和ROS无关系，只是通过串口读取数据，并且在终端显示出来，整体思路如下：

传感器选择 -> 测试程序测试（类似串口助手） -> ROS node绑定（将测试程序读取部分加入到ROS中） -> 将node加入到nodelet中封装

下面是测试程序部分，也可以在cht10_node/test中找到，名为[test_cht10.cpp][2]：
```
    #include <string>  
    #include <cht10_node/seiral_func.hpp>  
    #include <cstdlib>  
      
    #include <iostream>  
    #include <stdint.h>  
    #define BUFSIZE 17  
      
    int main(int argc, char** argv){  
      cht10_seiral_func::Cht10Driver cht10driver_;  
      
      std::string serialNumber_;  
      serialNumber_ = "/dev/ttyUSB0";  
      int baudRate_ = 115200;  
      
      std::stringstream ostream;  
      int fd, len, rcv_cnt;  
      bool success_flag;  
      char buf[40], temp_buf[BUFSIZE],result_buf[BUFSIZE];  
      int laser_data=0;  
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
    //          std::cout<<"Received data, with :[";  
    //          for(int j=0;j<BUFSIZE;j++){  
    //            printf("%c ",(unsigned char) result_buf[j]);  
    //          }  
    //          printf("] \n");  
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

通过使用`catkin_make`可以得到一个名为```test_cht10```的可执行文件，使用`./test_cht10`可以运行，将CHT-10插入到电脑上，并且电脑只有一个ttyUSB0，就可以读取数据，数据将显示为sensor data: 传感器毫米数据， Distance: 传感器米数据。

### ROS驱动编写

上部分讲述了测试程序编写，主要作用是通过串口读取传感器数据，那么得到传感器数据之后，就可以将传感器数据填充到ROS消息中，然后通过话题形式发布出去，如下是将传感器数据填充到ROS消息并封装成nodelet类的主要部分：

完整代码可以查看： [cht10_node.cpp][3]
```
    /** 
     * @file /Cht10_serial_func/src/Cht10_serial_func.cpp 
     * 
     * @brief Implementation for dirver with read data from Cht10 nodelet 
     * 
     * @author Carl 
     * 
     **/  
      
    /***************************************************************************** 
     ** Includes 
     *****************************************************************************/  
    #include <string>  
    #include <ros/ros.h>  
    #include <std_msgs/String.h>  
    #include <nodelet/nodelet.h>  
    #include <ecl/threads/thread.hpp>  
    #include <sensor_msgs/LaserScan.h>    
    #include <sensor_msgs/Range.h>    
    #include <pluginlib/class_list_macros.h>  
    #include <cht10_node/seiral_func.hpp>  
      
      
    namespace cht10_seiral_func{  
      
    class Cht10Func : public nodelet::Nodelet{  
    #define BUFSIZE 17  
    #define SCALE 1000  
      
    public:  
      Cht10Func() : shutdown_requested_(false),serialNumber_("/dev/USB0"),frame_id("laser"){}  
      
      ~Cht10Func(){  
        NODELET_DEBUG_STREAM("Waiting for update thread to finish.");  
        shutdown_requested_ = true;  
        update_thread_.join();  
      }  
      
      virtual void onInit(){  
      
        ros::NodeHandle nh = this->getPrivateNodeHandle();  
        std::string name = nh.getUnresolvedNamespace();  
        nh.getParam("serialNumber", serialNumber_);  
        nh.getParam("baudRate", baudRate_);  
        nh.getParam("frame_id", frame_id);  
        rcv_cnt = 0;  
        success_flag = 0;  
      
        fd = open(serialNumber_.c_str(), O_RDWR | O_NOCTTY | O_NDELAY );  
        if(fd < 0){  
          ROS_ERROR_STREAM("Open Serial: "<<serialNumber_.c_str()<<" Error!");  
          exit(0);  
        }  
      
        int countSeq=0;  
        scan_pub = nh.advertise<sensor_msgs::Range>("range",100);  
        memset(buf, 0, sizeof(buf));  
        memset(temp_buf, 0, sizeof(temp_buf));  
        memset(result_buf, 0, sizeof(result_buf));  
        Cht10driver_.UART0_Init(fd,baudRate_,0,8,1,'N');  
        ROS_INFO_STREAM("Open serial: ["<< serialNumber_.c_str() <<" ] successful, with idex: "<<fd<<".");  
        NODELET_INFO_STREAM("Cht10Func initialised. Spinning up update thread ... [" << name << "]");  
        update_thread_.start(&Cht10Func::update, *this);  
      }  
      
      double data_to_meters(int &data, int scale){  
        return (double)data/scale;  
      }  
      
      void publish_scan(ros::Publisher *pub,  
                        double nodes, ros::Time start,  
                        std::string frame_id){  
      
        float final_range;  
        sensor_msgs::Range range_msg;  
        range_msg.field_of_view = 0.05235988;  
        range_msg.max_range = 10.0;  
        range_msg.min_range = 0.05;  
        range_msg.header.frame_id = frame_id;  
        range_msg.radiation_type = sensor_msgs::Range::INFRARED;  
        if(nodes > range_msg.max_range){  
          final_range = std::numeric_limits<float>::infinity();  
        }else if(nodes < range_msg.min_range){  
          final_range = -std::numeric_limits<float>::infinity();  
        }else{  
          final_range = nodes;  
        }  
        range_msg.header.stamp = start;  
        range_msg.header.seq = countSeq;  
        range_msg.range = final_range;  
        scan_pub.publish(range_msg);  
      
      }  
      
      bool get_scan_data(){  
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
      }  
    private:  
      void update(){  
        rcv_cnt = 0;  
        success_flag = 0;  
        laser_data = 0;  
        ros::Rate spin_rate(50);  
        memset(buf, 0, sizeof(buf));  
        memset(temp_buf, 0, sizeof(temp_buf));  
        memset(result_buf, 0, sizeof(result_buf));  
        ROS_INFO_STREAM("Begin receive data from "<<serialNumber_.c_str()<<", with idex:"<<fd<<".");  
        fd = open(serialNumber_.c_str(), O_RDWR | O_NOCTTY | O_NDELAY );  
        Cht10driver_.UART0_Init(fd,baudRate_,0,8,1,'N');  
        while (! shutdown_requested_ && ros::ok())  
        {  
          start_scan_time = ros::Time::now();  
          success_flag = get_scan_data();  
                
          //Send data  
          publish_scan(&scan_pub, data_to_meters(laser_data,SCALE),  
                           start_scan_time, frame_id);  
          spin_rate.sleep();  
      
          countSeq++;  
        }  
      
        ROS_INFO_STREAM("Shotdown and close serial: "<<serialNumber_.c_str()<<".");  
        Cht10driver_.UART0_Close(fd);  
      }  
    private:  
      int fd, len, rcv_cnt;  
      int success_flag;  
      char buf[40], temp_buf[BUFSIZE],result_buf[BUFSIZE];  
        
      Cht10Driver Cht10driver_;  
      ecl::Thread update_thread_;  
      bool shutdown_requested_;  
      ros::Publisher scan_pub;  
      int laser_data;  
      char data_buf[4];  
      // ROS Parameters  
      std::string serialNumber_;  
      int baudRate_;  
      int countSeq;  
        
      std::string frame_id;  
      
      ros::Time start_scan_time;  
    };  
      
    } //namespace Cht10_serial_func  
    PLUGINLIB_EXPORT_CLASS(cht10_seiral_func::Cht10Func,  
    nodelet::Nodelet);  
```

其中get_scan_data()函数，就是测试代码内容，得到数据后将数据发送到publish_scan()函数中，就可以发布传感器数据了。

### 测试

代码完成后，按照格式进行编写CMakeList、package文件，运行catkin_make就可以得到名为cht10_node.so动态库，运行launch文件即可加载，命令如下：
```
roslaunch cht10_node standalone.launch 
```
如果一切成才，运行rostopic list 可以得到/range话题。

使用rostopic echo /range就可以得到传感器的ROS消息内容，包括传感器距离。

或者直接运行
```
roslaunch cht10_node view_range.launch
```
就可以看到图形化形式为传感器数据。

[1]: docs.ros.org/api/sensor_msgs/html/msg/Range.html
[2]: https://github.com/Playfish/cht10_node/blob/master/test/test_cht10.cpp
[3]: https://github.com/Playfish/cht10_node/blob/master/src/cht10_node.cpp
