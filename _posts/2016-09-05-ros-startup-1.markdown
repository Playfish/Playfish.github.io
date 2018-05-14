---
layout:     post
title:      "【ROS总结】Turtlebot ROS 开机自启动设置"
subtitle:   "ROS startup setting 1"
date:       2016-09-05 10:14:08
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
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/52437927)，转载请保留链接。

## 前言

关于ROS自启动设置，之前查看了很多相关文章，并不是很适用，尤其是开机启动时候加载minimal.launch和amcl.launch的时候，目前开启自启动的ROS包有[upstart][1]，该包是通过创建一个服务来启动minimal.launch等基本的launch文件，要是启动多个launch文件，可以通过把多个launch文件写入到一个launch中这样的方法来启动。

还有其他方式是利用Ubuntu自带的开机启动文件来达到多launch文件启动效果，一种是服务，方法和upstart类似，如果想要多launch文件分别启动，例如minimal.launch和amcl.launch两个launch文件，当启动完minimal.launch后，再启动amcl.launch会出现waiting for device...字样，导致的结果就是amcl.launch自动不了，还得重新启动。因此，把多个launch文件写入到一个launch启动就不会提示waiting for device...。

## rc.local

另外一种是在开机启动文件rc.local文件中直接写入ros命令来达到开机启动效果，把minimal和amcl两个launch文件写成一个launch命名为auto_start.launch，然后在rc.laoch中写入如下内容：
```
source /opt/ros/indigo/setup.bash
roslaunch /etc/auto_start.launch 
```
其中auto_start.launch文件所在位置并不重要，需要写在exit 0的前面即可，不过这个不能启动用户目录下的配置，如果启动用户目录下的launch文件，需要source ~/.bashrc。

我用的方法是，把所有需要的配置和launch文件写到了一个脚本中，通过开机启动脚本来达到启动效果，不过经过测试，单独运行两个launch文件还是有错误，即运行完minimal后运行amcl.launch的时候，会卡在waiting for device...。目前解决方法是把minimal和amcl写到了一个文件中命名为minimal_amcl.launch。

启动minimal_amcl.launch文件是在开机的第20秒左右，amcl.launch和minimal.launch共启动了两次，基本完全启动差不多是一到两分钟左右。目前不知道是什么原因，如果有知道的请告知，谢谢！ 

首先，在家目录下创建一个名为aotu_runing的目录，在该目录下创建脚本文件minimal，内容如下：

```
<pre name="code" class="plain">#!/bin/bash
source /opt/ros/indigo/setup.bash
source ~/.bashrc
export TURTLEBOT_BASE=kobuki
export TURTLEBOT_3D_SENSOR=kinect
export TURTLEBOT_MAP_FILE=<path_of_map_file>
source ~/catkin_ws/devel/setup.bash
export ROS_HOSTNAME=10.42.0.1
export ROS_MASTER_URI=http://10.42.0.1:11311
d='data +%Y%m%d-%H:%M:%S'
echo "[$d]:minimal runing" >> ~/print_log.txt
roslaunch turtlebot_bringup minimal_amcl.launch
```

其中首先source ros路径，然后source bashrc和用户目录下的catkin_ws，因为catkin_ws目录下包含了路径规划算法。

```
<pre name="code" class="plain">d='data +%Y%m%d-%H:%M:%S'
echo "[$d]:minimal runing" >> ~/print_log.txt
```

打印出信息到家目录下。

然后修改/etc/rc.local文件：

```
#!/bin/sh -e

cd ~/aotu_runing
./minimal
```
配置完成，当再次开机的时候，等待1到2分钟，会听到Turtlebot响声，amcl也一起启动。

[1]: http://docs.ros.org/jade/api/robot_upstart/html/
