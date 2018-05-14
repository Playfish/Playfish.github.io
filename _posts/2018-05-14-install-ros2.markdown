---
layout:     post
title:      "【ROS翻译】Ubuntu下安装ROS2"
subtitle:   "Installation for ROS2 in Ubuntu"
date:       2018-05-14 18:14:34
author:     "Playfish"
header-img: "img/in-post/release-ros2-into-rosdistro/ros2_logo.png"
header-mask: 0.3
catalog:    true
tags:
    - ROS2
    - Installation
---


> 说明:<br><br>
> 本文首发于 [Playfish Blog](http://carlzhang.club/2018/05/14/install-ros2/)，转载请保留链接。


## 前言

在Beta 2中，我们正在为Ubuntu Xenial构建Debian软件包。它们在一个临时存储库中进行测试。下面的链接和说明参考了最新版本——目前是ardent。

资源:

 * [Jenkins实例][1]

 * [存储库][2]

 * 状态页面([amd64][3],[arm64][4])

## 安装源

要安装Debian软件包，你需要将我们的Debian存储库添加到apt源。首先你需要授权我们的gpg密钥，就像这样:
```
sudo apt update && sudo apt install curl
curl http://repo.ros2.org/repos.key | sudo apt-key add -
```

然后将存储库添加到你的源列表中:
```
sudo sh -c 'echo "deb [arch=amd64,arm64] http://repo.ros2.org/ubuntu/main xenial main" > /etc/apt/sources.list.d/ros2-latest.list'
```

## 安装ROS2包

下面的命令安装除了`ros-ardent-ros1-bridge`和`ros-ardent-turtlebot2-*`之外的所有`ros-ardent-*`包，因为它们需要ROS 1的依赖项。下面看看如何安装这些。
```
sudo apt update
sudo apt install `apt list "ros-ardent-*" 2> /dev/null | grep "/" | awk -F/ '{print $1}' | grep -v -e ros-ardent-ros1-bridge -e ros-ardent-turtlebot2- | tr "\n" " "`
```

## 环境设置
```
source /opt/ros/ardent/setup.bash
```

如果你已经安装了Python包`argcomplete`(版本0.8.5或更高版本，请参见下面的Xenial说明)，你可以从以下文件中获取以下文件以完成诸如`ros2`这样的命令行工具:
```
source /opt/ros/ardent/share/ros2cli/environment/ros2-argcomplete.bash
```

### (可选)在Ubuntu 16.04上安装argcomplete >= 0.8.5
如果你需要在Ubuntu 16.04 (Xenial)上安装`argcomplete`，那么你将需要使用pip，通过`apt-get`可用的版本将不会工作，因为该版本的`argcomplete`有一个bug:
```
sudo apt install python3-pip
sudo pip3 install argcomplete
```

## 选择RMW实现
默认情况下，将使用RMW实现`FastRTPS`。通过设置环境变量`RMW_IMPLEMENTATION=rmw_opensplice_cpp`，你也可以切换到使用OpenSplice。

## 使用ROS1包的其他包
`ros1_bridge`和Turtlebot机器人演示使用的是ROS 1包。为了能够安装它们，请先在[这里][5]添加记录的ROS 1源。

如果你正在使用Docker进行隔离，你可以从镜像`ros:kinetic`或`osrf/ros:kinetic-desktop`开始:这也将避免设置ros源，因为它们已经被集成了。

现在你可以安装其余的包:
```
sudo apt update
sudo apt install ros-ardent-ros1-bridge ros-ardent-turtlebot2-*
```

[1]: build.ros2.org
[2]: http://repo.ros2.org/
[3]: http://repo.ros2.org/status_page/ros_ardent_default.html
[4]: http://repo.ros2.org/status_page/ros_ardent_uxv8.html
[5]: wiki.ros.org/Installation/Ubuntu?distro=kinetic
