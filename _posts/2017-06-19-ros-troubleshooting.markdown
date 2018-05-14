---
layout:     post
title:      "【ROS总结】ROS故障排除"
subtitle:   "ROS usage troubleshooting"
date:       2017-06-03 15:17:12
author:     "Playfish"
header-img: "img/in-post/release-ros1-into-rosdistro/ros_logo.jpg"
header-mask: 0.3
catalog:    true
tags:
    - ROS1
    - Troubleshooting
---


> 说明:<br><br>
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/72852251)，转载请保留链接。

## 前言

在ROS学习与开发中，难免遇到各种各样的问题，除了一些可以解决问题（语法问题）外，其他一些问题有时很难发现并解决，在这里，总结下在开发中遇到的各种问题，不定期更新中。。。

为了方便和管理，每一个标题都是一些问题模块，现在有catkin_make编译模块。

## catkin_make问题

**问题1**：在使用catkin_make过程中，明明有需要编译的包，但是系统提示找不到该路径，具体输出如下：
```
Traceback (most recent call last):
  File "/opt/ros/indigo/share/genjava/cmake/../../../lib/genjava/genjava_gradle_project.py", line 14, in <module>
    genjava.main(sys.argv)
  File "/opt/ros/indigo/lib/python2.7/dist-packages/genjava/genjava_main.py", line 82, in main
    gradle_project.create(args.package, args.output_dir)
  File "/opt/ros/indigo/lib/python2.7/dist-packages/genjava/gradle_project.py", line 152, in create
    raise IOError("could not find %s on the ros package path" % msg_pkg_name)
IOError: could not find turtlebot2i_marker_manipulation on the ros package path
make[2]: *** [turtlebot2i_marker_manipulation/java/turtlebot2i_marker_manipulation/build.gradle] Error 1
make[1]: *** [turtlebot2i_marker_manipulation/CMakeFiles/turtlebot2i_marker_manipulation_generate_messages_java_gradle.dir/all] Error 2
make[1]: *** Waiting for unfinished jobs....
Traceback (most recent call last):
  File "/opt/ros/indigo/share/genjava/cmake/../../../lib/genjava/genjava_gradle_project.py", line 14, in <module>
    genjava.main(sys.argv)
  File "/opt/ros/indigo/lib/python2.7/dist-packages/genjava/genjava_main.py", line 82, in main
    gradle_project.create(args.package, args.output_dir)
  File "/opt/ros/indigo/lib/python2.7/dist-packages/genjava/gradle_project.py", line 152, in create
    raise IOError("could not find %s on the ros package path" % msg_pkg_name)
IOError: could not find turtlebot2i_marker_manipulation on the ros package path

```

IOError: <span style="color:red;"> could not find </span> turtlebot2i_marker_manipulation <span style="color:red;">on the ros package path. </span>

**解决方案：**
下载genjava就可以正常编译。原因不明。命令如下：
```
$ sudo apt-get remove ros-indigo-genjava 
```

## Moveit! 问题
**问题1：** 当运行
```
roslaunch moveit_setup_assistant setup_assistant.launch
```

选择模型加载时，会突然系统崩溃。提示如下信息：
```
[rospack] Error: no package given
[librospack]: error while executing command
[ INFO] [1496720764.674712823]: Running 'rosrun xacro xacro.py /home/carl/Desktop/Project/turtlebot2i/src/turtlebot2i/turtlebot2i_description/robots/kobuki_interbotix_zr300.urdf.xacro'...
[ INFO] [1496720765.546447152]: Loaded turtlebot robot model.
[ INFO] [1496720765.546519891]: Setting Param Server with Robot Description
[ INFO] [1496720765.555161202]: Robot semantic model successfully loaded.
[ INFO] [1496720765.555195476]: Setting Param Server with Robot Semantic Description
[ INFO] [1496720765.570819680]: Loading robot model 'turtlebot'...
[ INFO] [1496720765.570888991]: No root/virtual joint specified in SRDF. Assuming fixed joint
================================================================================REQUIRED process [moveit_setup_assistant-1] has died!
process has died [pid 8123, exit code -11, cmd /opt/ros/indigo/lib/moveit_setup_assistant/moveit_setup_assistant __name:=moveit_setup_assistant __log:=/home/carl/.ros/log/689c9c26-4a64-11e7-8883-d85de2b5c407/moveit_setup_assistant-1.log].
log file: /home/carl/.ros/log/689c9c26-4a64-11e7-8883-d85de2b5c407/moveit_setup_assistant-1*.log
Initiating shutdown!
================================================================================
[moveit_setup_assistant-1] killing on exit
shutting down processing monitor...
... shutting down processing monitor complete
done
```

说明：

 1. 单独加载模型没问题.

 2. 使用moveit_setup加载其他模型有的没问题（例如，Turtlebot）。

 3. 只是使用有些模型会出现突然崩溃的情况。

**解决方案：**

**方式1：**检查模型中模型的<collision>属性，里面的<geometry>是否加载的是.dae或.stl文件，如果有，将.dae或.stl替换成urdf本身自有的属性，例如<cylinder>，<box>等。
举例：将

```
      <collision>
        <origin xyz="0.0 0.0 0.0" rpy="0 0 0"/>
        <geometry>
          <mesh filename="package://turtlebot2i_description/meshes/sensors/sr300.stl" scale="2.0 2.0 2.0" />
        </geometry>
      </collision>
```

替换成
```
      <collision>
        <origin xyz="0 0 0" rpy="0 0 0"/>
        <geometry>
          <cylinder length="0.006" radius="0.170"/>
        </geometry>
      </collision>
```

**方式2：**测试了下moveit_setup_assistant，发现模型加载mesh时，会出错，方式1是将现有属性替换，另外一种方式是改变自己制作的dae或stl文件名，不清楚什么原因，moveit_setup_assistant加载dae或stl时，模型名必须为小写，如果有大写出现就会报错，例如将Test_Finger.dae修改为test_finger.dae就不会造成崩溃。

**方式3：**如果是在加载别人做的dae或stl文件时，需要将dae或stl文件重写，我使用的是blender，将stl重新导出stl就不会造成崩溃。其他3D建模也可以尝试下，3D Max，solidworks，inventor等等。

其他解决方案如果有请告知。

## ROS_Control + Gazebo问题
**问题1：**使用ros_control为模型添加Gazebo接口时，模型单独显示没有问题，在Gazebo中添加会错误，从而导致加载不了ros_control接口，报错如下：

```
[ERROR] [1497406213.064550049, 0.172000000]: No valid hardware interface element found in joint 'wheel_joint_fl'.
[ERROR] [1497406213.064681421, 0.172000000]: Failed to load joints for transmission 'wheel_trans_fl'.
[ERROR] [1497406213.064711210, 0.172000000]: No valid hardware interface element found in joint 'caster_joint_fl'.
[ERROR] [1497406213.064765673, 0.172000000]: Failed to load joints for transmission 'caster_trans_fl'.
[ERROR] [1497406213.064781965, 0.172000000]: No valid hardware interface element found in joint 'wheel_joint_fr'.
[ERROR] [1497406213.064794358, 0.172000000]: Failed to load joints for transmission 'wheel_trans_fr'.
[ERROR] [1497406213.064806668, 0.172000000]: No valid hardware interface element found in joint 'caster_joint_fr'.
[ERROR] [1497406213.064815775, 0.172000000]: Failed to load joints for transmission 'caster_trans_fr'.
[ERROR] [1497406213.064826099, 0.172000000]: No valid hardware interface element found in joint 'wheel_joint_bl'.
[ERROR] [1497406213.064837826, 0.172000000]: Failed to load joints for transmission 'wheel_trans_bl'.
[ERROR] [1497406213.064850074, 0.172000000]: No valid hardware interface element found in joint 'caster_joint_bl'.
[ERROR] [1497406213.064861368, 0.172000000]: Failed to load joints for transmission 'caster_trans_bl'.
[ERROR] [1497406213.064874445, 0.172000000]: No valid hardware interface element found in joint 'wheel_joint_br'.
[ERROR] [1497406213.064884656, 0.172000000]: Failed to load joints for transmission 'wheel_trans_br'.
[ERROR] [1497406213.064893785, 0.172000000]: No valid hardware interface element found in joint 'caster_joint_br'.
[ERROR] [1497406213.064902338, 0.172000000]: Failed to load joints for transmission 'caster_trans_br'.
[ERROR] [1497406213.064913362, 0.172000000]: No valid hardware interface element found in joint 'arm_joint_1'.
[ERROR] [1497406213.064923217, 0.172000000]: Failed to load joints for transmission 'arm_trans_1'.
[ERROR] [1497406213.064932655, 0.172000000]: No valid hardware interface element found in joint 'arm_joint_2'.
[ERROR] [1497406213.064942810, 0.172000000]: Failed to load joints for transmission 'arm_trans_2'.
[ERROR] [1497406213.064956180, 0.172000000]: No valid hardware interface element found in joint 'arm_joint_3'.
[ERROR] [1497406213.064965970, 0.172000000]: Failed to load joints for transmission 'arm_trans_3'.
[ERROR] [1497406213.064976051, 0.172000000]: No valid hardware interface element found in joint 'arm_joint_4'.
[ERROR] [1497406213.064985608, 0.172000000]: Failed to load joints for transmission 'arm_trans_4'.
[ERROR] [1497406213.064995977, 0.172000000]: No valid hardware interface element found in joint 'arm_joint_5'.
[ERROR] [1497406213.065005730, 0.172000000]: Failed to load joints for transmission 'arm_trans_5'.
[ERROR] [1497406213.065017694, 0.172000000]: No valid hardware interface element found in joint 'gripper_finger_joint_l'.
[ERROR] [1497406213.065026496, 0.172000000]: Failed to load joints for transmission 'gripper_finger_l_trans'.
[ERROR] [1497406213.065036557, 0.172000000]: No valid hardware interface element found in joint 'gripper_finger_joint_r'.
[ERROR] [1497406213.065046327, 0.172000000]: Failed to load joints for transmission 'gripper_finger_r_trans'.

```

**解决：**该问题原因是ros_control接口更新，有的旧的会不兼容，通过给<joint>选项添加<hardwareinterface>标签，这个错误会消失，比如，在rrbot.xacro文件中，改变：

```
<joint name="joint1"/> 
```

为：
```
<transmission name="tran1">
  <type>transmission_interface/SimpleTransmission</type>
  <joint name="joint1">
    <hardwareInterface>EffortJointInterface</hardwareInterface>
  </joint>
</transmission>
```
