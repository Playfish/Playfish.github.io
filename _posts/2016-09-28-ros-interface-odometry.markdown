---
layout:     post
title:      "【ROS总结】 ROS接口——Odometry"
subtitle:   "ROS interface odometry"
date:       2016-09-28 11:23:57
author:     "Playfish"
header-img: "img/in-post/release-ros1-into-rosdistro/ros_logo.jpg"
header-mask: 0.3
catalog:    true
tags:
    - ROS1
    - ros_control
    - Odometry
---


> 说明:<br><br>
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/52688982)，转载请保留链接。

## 前言

版本：Indigo

  这篇文章主要是把平时用到的一些ROS接口梳理一下，避免无法和ros进行对接，首先ROS中相对重要的是里程计（Odometry），里程计的重要性不言而喻，如果没有里程计，不管是建立地图还是导航都不会很好的工作，相应的Odometry更新也有一些节点可以调用，本文主要用到的是[ros_controllers中的差分驱动控制器][1]，[介绍文档如下][2]，该控制器用处广泛，基本差分驱动都可以用到，从底层得到左右电机编码器数据，通过机器人运动学转换成Odometry消息发送出去，其他节点得到Odometry消息就可以在rviz或者Gazebo中更新机器人位置信息，也可以加上imu数据让机器人更准确。

## ros_controllers
安装方法为：

```
$ sudo apt-get install ros-indigo-ros-controllers
```

本文并没有直接使用Odometry消息，而是通过ros_controls中的[hardware_interface][3]来更新Odometry。相应教程说明如下，[链接地址][4]。直接更新hardware_interface中的joint就可以更新Odometry。在本地声明一个类为myrobot，继承于hardware_interface::RobotHw。

```
#include <hardware_interface/joint_command_interface.h>//关节命令接口，用于接收/cmd_vel数据
#include <hardware_interface/joint_state_interface.h>//关节状态接口，用于更新Odometry
#include <hardware_interface/robot_hw.h>

class MyRobot : public hardware_interface::RobotHW
{
public:
  MyRobot() 
 { 
   // connect and register the joint state interface
   hardware_interface::JointStateHandle state_handle_a("A", &pos[0], &vel[0], &eff[0]);
   jnt_state_interface.registerHandle(state_handle_a);

   hardware_interface::JointStateHandle state_handle_b("B", &pos[1], &vel[1], &eff[1]);
   jnt_state_interface.registerHandle(state_handle_b);

   registerInterface(&jnt_state_interface);

   // connect and register the joint position interface
   hardware_interface::JointHandle pos_handle_a(jnt_state_interface.getHandle("A"), &cmd[0]);
   jnt_pos_interface.registerHandle(pos_handle_a);

   hardware_interface::JointHandle pos_handle_b(jnt_state_interface.getHandle("B"), &cmd[1]);
   jnt_pos_interface.registerHandle(pos_handle_b);

   registerInterface(&jnt_pos_interface);
  }

private:
  hardware_interface::JointStateInterface jnt_state_interface;
  hardware_interface::PositionJointInterface jnt_pos_interface;
  double cmd[2];
  double pos[2];
  double vel[2];
  double eff[2];
};
```

其中"A"为机器人模型中的关节，一般为wheel_left。同理“B"也是轮子关节，一般为wheel_right。pos为当前机器人的位置，vel为机器人目前速度，eff为初始状态时机器人位置，cmd为接收cmd_vel消息用于控制机器人运动。需要注意的是pos为轮子所走过的弧度，因此上传时需要上传弧度值。

[Odometry.cpp][5]文件中更新Odometry的代码为：
```
 bool Odometry::update(double left_pos, double right_pos, const ros::Time &time)
  {
    /// Get current wheel joint positions:
    const double left_wheel_cur_pos  = left_pos  * wheel_radius_;
    const double right_wheel_cur_pos = right_pos * wheel_radius_;

    /// Estimate velocity of wheels using old and current position:
    const double left_wheel_est_vel  = left_wheel_cur_pos  - left_wheel_old_pos_;
    const double right_wheel_est_vel = right_wheel_cur_pos - right_wheel_old_pos_;

    /// Update old position with current:
    left_wheel_old_pos_  = left_wheel_cur_pos;
    right_wheel_old_pos_ = right_wheel_cur_pos;

    /// Compute linear and angular diff:
    const double linear  = (right_wheel_est_vel + left_wheel_est_vel) * 0.5 ;
    const double angular = (right_wheel_est_vel - left_wheel_est_vel) / wheel_separation_;

    /// Integrate odometry:
    integrate_fun_(linear, angular);

    /// We cannot estimate the speed with very small time intervals:
    const double dt = (time - timestamp_).toSec();
    if (dt < 0.0001)
      return false; // Interval too small to integrate with

    timestamp_ = time;

    /// Estimate speeds using a rolling mean to filter them out:
    linear_acc_(linear/dt);
    angular_acc_(angular/dt);

    linear_ = bacc::rolling_mean(linear_acc_);
    angular_ = bacc::rolling_mean(angular_acc_);

    return true;
  }
```

其中
```
const double left_wheel_cur_pos  = left_pos  * wheel_radius_;

```
当前的轮子所走过的弧度*轮子半径，得到当前轮子走过的距离，因此，这就是为什么pos要为弧度值。

[1]: https://github.com/ros-controls/ros_controllers/blob/indigo-devel/diff_drive_controller/src/odometry.cpp
[2]: http://wiki.ros.org/ros_control
[3]: http://wiki.ros.org/hardware_interface?distro=indigo
[4]: https://github.com/ros-controls/ros_control/wiki/hardware_interface
[5]: https://github.com/ros-controls/ros_controllers/blob/kinetic-devel/diff_drive_controller/src/odometry.cpp
