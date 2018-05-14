---
layout:     post
title:      "【ROS总结】Turtlebot ROS 开机自启动设置(2)"
subtitle:   "Turtlebot ROS startup Settings (2)"
date:       2017-08-01 19:31:00
author:     "Playfish"
header-img: "img/in-post/turtlebot-startup-setting-2/turtlebot2.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - ROS1
    - Turtlebot2
    - Startup
---


> 说明:<br><br>
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/76549144)，转载请保留链接。

## 前言

之前写过一篇[Turtlebot下的开机自启动设置][1]，这几天由于项目原因重新整理下，虽然已经有过先例，但是还是碰到了不少错误(keng)，前一篇写的是rc.local方式写入制成自启动脚本，现在换一种方式，改用启动内置控制台终端tty形式作为自启动设置，适用于桌面。

开机启动方式很多，不唯一，我所知道的是：

 * rc.local开启启动
 * linux内置服务启动，包括上一篇的ROS的upstart包。
 * 内置控制台终端启动tty
 * 开机界面后启动终端，例如Startup Application.等等

下面以项目为例讲述开机启动，rc.local就不在赘述了，目前两个功能进行实现，一种是开启自启动地图创建程序，另一种是开机自启动跟随脚本，两个脚本还是和之前一样，所有的节点全部都在一个脚本中。

原本觉得很容易，结果很悲剧的我想当然了，尤其是地图创建程序，涉及到两台电脑通信，如果把两个脚本（minimal和gmapping_demo）合并的话，就可以发现，程序挂起成功，在进程中可以看到，但是，没法用另外一台电脑查看，也就是说，另一台电脑和turtlebot上电脑没有master通信，不用怀疑，ROS的系统变量是正确的，并且两台电脑是同一网段，就是没有办法用另一台电脑使用rosnode list来查看turtlebot上电脑的节点。如果有知道原因的还请告知，个人理解应该是gmapping的代码导致master外部通信阻塞缘故。

好，言归正转，地图创建自启动程序分开运行，原因如上，也就是说分别开两个终端运行minimal和gmapping。如下

## 内置控制台终端tty自启动

这个方式只对gmapping使用就可以了，其他都是正常的，因为没有把脚本分开，所以使用两个tty控制台。

首先打开终端，输入如下命令：
```
$  sudo vi /etc/init/tty1.config 
```

整个文件如下，修改部分请对照好。

```
    # tty1 - getty  
    #  
    # This service maintains a getty on tty1 from the point the system is  
    # started until it is shut down again.  
      
    start on stopped rc RUNLEVEL=[2345] and (  
                not-container or  
                container CONTAINER=lxc or  
                container CONTAINER=lxc-libvirt)  
      
    stop on runlevel [!2345]  
      
    respawn  
    exec /sbin/getty -8 38400 tty1  
    exec /sbin/mingetty --autologin ubuntu tty1  

```
同样的保存退出后，修改另一个终端，输入如下命令：

```
$  sudo vi /etc/init/tty2.config 
```
内容如下：

```
# tty2 - getty  
#  
# This service maintains a getty on tty2 from the point the system is  
# started until it is shut down again.  
  
start on runlevel [23] and (  
            not-container or  
            container CONTAINER=lxc or  
            container CONTAINER=lxc-libvirt)  
  
stop on runlevel [!23]  
  
respawn  
exec /sbin/getty -8 38400 tty2  
exec /sbin/mingetty --autologin ubuntu tty2
```

随后修改登陆命令，改成不需要密码登陆，输入

```
$  sudo vi /etc/sudoers 
```

全部文本如下：

```
#  
# This file MUST be edited with the 'visudo' command as root.  
#  
# Please consider adding local content in /etc/sudoers.d/ instead of  
# directly modifying this file.  
#  
# See the man page for details on how to write a sudoers file.  
#  
Defaults        env_reset  
Defaults        mail_badpass  
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"  
  
# Host alias specification  
  
# User alias specification  
  
# Cmnd alias specification  
  
# User privilege specification  
root    ALL=(ALL:ALL) ALL  
  
# Members of the admin group may gain root privileges  
%admin ALL=(ALL) ALL  
  
# Allow members of group sudo to execute any command  
%sudo   ALL=(ALL:ALL) ALL  
  
# See sudoers(5) for more information on "#include" directives:  
  
#includedir /etc/sudoers.d  
  
ubuntu ALL=NOPASSWD:ALL
```

最后修改~/.bashrc文件，在最后添加： 

```
$  gedit ~/.bashrc
```

添加：
```
    ##auto start  
        Terminal=`tty`  
        case $Terminal in  
            "/dev/tty1") roslaunch turtlebot_bringup minimal.launch;;  #在tty1 启动minimal  
            "/dev/tty2") sleep 10;roslaunch turtlebot_navigation gmapping_demo.launch;;  #等待10秒后，在tty2启动地图创建  
        esac  
```

## 测试

在开始测试之前，首先确保你的bashrc文件中，修改正确了ROS_HOSTNAME和ROS_IP以及ROS_MASTER_URI的值，并且和另一台测试电脑在同一网段。

turtlebot电脑运行成功提示，假设当前ip为192.168.0.100，另一台测试电脑ip为192.168.0.101，配置如下：

turtlebot电脑的.bashrc中，应该有

```
    export ROS_HOSTNAME=192.168.0.100  
    export ROS_IP=192.168.0.100  
    export ROS_MASTER_URI=http://192.168.0.100:11311
```

测试电脑.bashrc，也应该有 
```
export ROS_HOSTNAME=192.168.0.101  
export ROS_IP=192.168.0.101  
export ROS_MASTER_URI=http://192.168.0.100:11311 
```

重新启动turtlebot电脑，等待完成后，使用另一台电脑测试，运行

```
roslaunch turtlebot_rviz_launchers view_navigation.launch
```

就可以在viz中看到当前地图创建情况，控制运行：

```
    roslaunch turtlebot_teleop keyboard_teleop.launch
```
地图创建自启动完成。

## 其他方式自启动

### Startup Applications
只讲述linux应用设置，关于ROS包，教程太多就不说了，最简单的自启动方式就是在桌面上，按下“开始”键位，在弹出的搜索框中输入“Startup Applications"，应用弹出后，选择Add，然后在command行中输入：/usr/bin/gnome-terminal -x /home/ubuntu/aotu_runing/minimal_follower.sh即可。

minimal_follower.sh内容如下:
```
#! /bin/bash
if test -z "$DBUS_SESSION_BUS_ADDRESS" ; then
      ## if not found, launch a new one
      eval `dbus-launch --sh-syntax --exit-with-session`
      echo "D-Bus per-session daemon address is: $DBUS_SESSION_BUS_ADDRESS"
fi
d="$(date +%Y%m%d-%H:%M:%S)"
echo "[$d]:source opt " > /home/$USER/aotu_runing/print_log.txt

source /opt/ros/indigo/setup.bash
echo "[$d]:source bashrc " >> /home/$USER/aotu_runing/print_log.txt

source /home/ubuntu/.bashrc
echo "[$d]:source catkin_ws " >> /home/$USER/aotu_runing/print_log.txt

source /home/$USER/catkin_ws/devel/setup.bash

export TURTLEBOT_BASE=kobuki
export TURTLEBOT_3D_SENSOR=astra
export ROS_HOSTNAME=localhost
export ROS_MASTER_URI=http://localhost:1311
echo "[$d]:source catkin_ws again " >> /home/$USER/aotu_runing/print_log.txt

source /home/$USER/catkin_ws/devel/setup.bash
echo "[$d]:minimal runing " > /home/$USER/aotu_runing/print_log.txt
roslaunch roch_bringup minimal_follower.launch  >>  /home/$USER/aotu_runing/print_log.txt
echo "[$d]:minimal runing successfully. " >> /home/$USER/aotu_runing/print_log.txt

```

最终这个脚本启动的是turtlebot_bringup下的 minimal_follower.launch，minimal_follower.launch内容如下：
```
<launch>
  <!-- TURTLEBOT Configure -->
  <arg name="base"                      default="$(env TURTLEBOT_BASE)"                              doc="mobile base type [TURTLEBOT]"/>
  <arg name="stacks"                    default="$(env TURTLEBOT_STACKS)"                    doc="stack type displayed in visualisation/simulation [standard]"/>
  <arg name="3d_sensor"                 default="$(env TURTLEBOT_3D_SENSOR)"                 doc="3d sensor types [kinect, asus_xtion_pro, r200]"/>
  <arg name="serialport"                default="$(env TURTLEBOT_SERIAL_PORT)"               doc="used by create to configure the port it is connected on [/dev/ttyUSB0, /dev/ttyS0]"/>
  <arg name="simulation"        default="$(env TURTLEBOT_SIMULATION)"                doc="set flags to indicate this turtle is run in simulation mode."/>
  <arg name="robot_name"        default="$(env TURTLEBOT_NAME)"                      doc="used as a unique identifier and occasionally to preconfigure root namespaces, gateway/zeroconf ids etc."/>
  <arg name="robot_type"        default="$(env TURTLEBOT_TYPE)"                      doc="just in case you are considering a 'variant' and want to make use of this."/>


  <param name ="/use_sim_time" value="$(arg simulation)"/>

  <!-- Load robot model -->
  <include file="$(find turtlebot_bringup)/launch/includes/robot.launch.xml">
    <arg name="base" value="$(arg base)" />
    <arg name="stacks" value="$(arg stacks)" />
    <arg name="3d_sensor" value="$(arg 3d_sensor)" />
  </include>

  <!-- Load robot base-->
  <include file="$(find turtlebot_bringup)/launch/includes/mobile_base.launch.xml">
    <arg name="base" value="$(arg base)" />
    <arg name="serialport" value="$(arg serialport)" />
  </include>

 <group unless="$(arg simulation)"> <!-- Real robot -->
    <include file="$(find turtlebot_follower)/launch/includes/velocity_smoother.launch.xml">
      <arg name="nodelet_manager"  value="/mobile_base_nodelet_manager"/>
      <arg name="navigation_topic" value="twist_mux/input/navi"/>
    </include>

    <include file="$(find turtlebot_bringup)/launch/sensor.launch">
      <arg name="rgb_processing"                  value="true"/>  <!-- only required if we use android client -->
      <arg name="depth_processing"                value="true"/>
      <arg name="depth_registered_processing"     value="false"/>
      <arg name="depth_registration"              value="false"/>
      <arg name="disparity_processing"            value="false"/>
      <arg name="disparity_registered_processing" value="false"/>
    </include>
  </group>
  <group if="$(arg simulation)">
    <!-- Load nodelet manager for compatibility -->
    <node pkg="nodelet" type="nodelet" ns="camera" name="camera_nodelet_manager" args="manager"/>

    <include file="$(find turtlebot_follower)/launch/includes/velocity_smoother.launch.xml">
      <arg name="nodelet_manager"  value="camera/camera_nodelet_manager"/>
      <arg name="navigation_topic" value="/twist/input/navi"/>
    </include>
  </group>

  <param name="camera/rgb/image_color/compressed/jpeg_quality" value="22"/>

  <!-- Make a slower camera feed available; only required if we use android client -->
  <node pkg="topic_tools" type="throttle" name="camera_throttle"
        args="messages camera/rgb/image_color/compressed 5"/>


  <!--  Real robot: Load turtlebot follower into the  sensors nodelet manager to avoid pointcloud serializing -->
  <!--  Simulation: Load turtlebot follower into nodelet manager for compatibility -->
  <node pkg="nodelet" type="nodelet" name="turtlebot_follower"
        args="load turtlebot_follower/TurtlebotFollower camera/camera_nodelet_manager">
    <remap from="turtlebot_follower/cmd_vel" to="follower_velocity_smoother/raw_cmd_vel"/>
    <remap from="depth/points" to="camera/depth/points"/>
    <remap from="depth/image_rect" to="/camera/depth/image_rect"/>
    <param name="enabled" value="true" />
    <param name="x_scale" value="7.0" />
    <param name="z_scale" value="2.0" />
    <param name="min_x" value="-0.35" />
    <param name="max_x" value="0.35" />
    <param name="min_y" value="0.1" />
    <param name="max_y" value="0.5" />
    <param name="max_z" value="1.2" />
    <param name="goal_z" value="0.6" />
  </node>
  <!-- Launch the script which will toggle turtlebot following on and off based on a joystick button. default: on -->
  <node name="switch" pkg="turtlebot_follower" type="switch.py"/>
</launch>
                                                                                          
```
使用桌面程序弊端在于修改的话需要重新连接显示屏，比较麻烦，不过不长改。

注意：$USER千万不要用在/etc/rc.local下，否则会造成rc.local自启动不成功，没有反应，这一点坑了我很长时间。

如果启动后，无法跟随，说明astra没有启动成功，可以在/var/log/apport.log中看到输出消息，解决办法就是在脚本中添加：

```
    if test -z "$DBUS_SESSION_BUS_ADDRESS" ; then  
          ## if not found, launch a new one  
          eval `dbus-launch --sh-syntax --exit-with-session`  
          echo "D-Bus per-session daemon address is: $DBUS_SESSION_BUS_ADDRESS"  
    fi  
```
就可以。

### robot_upstart 网络
#### 配置
今天使用前几个方式在笔记本上进行自启动不成功，原因应该在于网络，节点/话题什么都有，但是无法使用rosnode list，因此，使用了robot_upstart来尝试，首先安装robot_upstart包，安装步骤比较简单：

```
$ sudo apt-get install ros-indigo-robot-upstart
```

安装完成后，大家可以查看[使用文档][2]来进行使用，方式也比较简单，不过是单一launch启动，由于clearpath的机器人譬如husky只提供了底盘启动的缘故，不过大家可以将多个launch合并为一个来进行启动，启动方式也比较简单，首先，安装需要自启动的launch文件，使用方式如下：

```
$ rosrun robot_upstart install myrobot_bringup/launch/base.launch
```

其中myrobot_bringup是要启动launch文件所在的包，base.launch是要启动的launch文件，只需要提供包到launch文件路径即可。需要注意的是，运行这个命令需要在sudo root权限下，这个步骤将写到系统中。

安装launch文件完成后，将提示有很多文件被放到了系统中，如下输出：

```
    Preparing to install files to the following paths:  
      /etc/init/turtlebot.conf  
      /etc/ros/indigo/turtlebot.d/.installed_files  
      /etc/ros/indigo/turtlebot.d/minimal.launch  
      /usr/sbin/turtlebot-start  
      /usr/sbin/turtlebot-stop  
    Now calling: /usr/bin/sudo /opt/ros/indigo/lib/robot_upstart/mutate_files  
    Filesystem operation succeeded. 
```

现在启动服务的话，只是单机模式，即ROS_HOSTNAME/ROS_IP/ROS_MASTER_URI都是localhost，如果想要另外一个同一网段的计算机查看当前计算机的node/topic时，就需要对脚本进行修改，重要的脚本为  /usr/sbin/turtlebot-start，将该脚本中的

```
    28  export ROS_HOSTNAME=$(hostname)  
      
    30  export ROS_MASTER_URI=http://127.0.0.1:11311  
```

修改为当前电脑局域网内的IP，比如我的为192.168.0.106，最好将上面的注释掉，重新书写一遍：

```
28  export ROS_HOSTNAME=192.168.0.106  
  
30  export ROS_MASTER_URI=http://192.168.0.106:11311 
```

现在运行服务，运行操作共有如下：
```
$ sudo service myrobot start  #启动脚本服务  
$ sudo service myrobot stop  #停止脚本服务  
$ sudo service myrobot restart  #重启脚本服务 
```

其中myrobot为安装launch文件中提示的，比如我的为turtlebot，查看/usr/sbin/turtlebot-start名字可以知道。启动完成后，在本地运行终端：
```
$ rostopic list
```

将会有输出，也可以在同一网段内的电脑上，配置好网络后，运行

```
$ rostopic list
```

也可以得到输出。服务输出日志文件为/var/log/upstart/myrobot.log，其中myrobot在我的电脑上为turtlebot.log。如果运行服务后没有输出，可以查看这个文件进行调试。

#### 卸载

卸载命令如下
```
rosrun robot_upstart uninstall [-h] [--rosdistro DISTRO] JOBNAME
```

其中，JOBNAME是要卸载的脚本，比如我的是turtlebot，DISTRO为指定ROS版本，存在多个脚本时需要用到，比如：
```
rosrun robot_upstart uninstall turtlebot
```

[1]: https://blog.csdn.net/u011118482/article/details/52437927
[2]: http://docs.ros.org/jade/api/robot_upstart/html/
