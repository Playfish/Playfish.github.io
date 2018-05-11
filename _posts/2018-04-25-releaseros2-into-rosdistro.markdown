---
layout:     post
title:      "【ROS总结】发布ROS2包到ROS版本"
subtitle:   "Release ROS2 package into rosdistro"
date:       2018-04-25 11:40:44
author:     "Playfish"
header-img: "img/in-post/release-ros2-into-rosdistro/ros2_logo.png"
header-mask: 0.3
catalog:    true
tags:
    - ROS2
    - Release
---


> 说明:<br><br>
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/80075689)，转载请保留链接。


## 前言

在上一总结中，讲述了如何将[ROS1的包发布到ROS版本](https://blog.csdn.net/u011118482/article/details/80039148)（Indigo、jade、kinetic、lunar等），在这一节中，讲述如何把ROS2的包发送到ROS社区，比如发布到ROS2版本（ardent）。

这个页面描述了如何准备在公共ROS 2 buildfarm上发布存储库。在你创建了一个包之后，这是将你的包引入到公开可用的Debian软件包(即：你将能够通过apt-get安装包。这个页面包含了ROS 2特定的指令，它取代了在ROS Wiki上的[Bloom发布教程的第2部分](http://wiki.ros.org/cn/bloom/Tutorials/FirstTimeRelease)。

注意：早期的ROS 2版本的发布过程主要依赖于`git-bloom-release`的子命令`bloom-release`，而不是完整的开放版本工作流，所以可能会有问题。

## 对比ROS1 bloom不同

如果你在ROS1之前使用bloom发布过ROS包，ROS2的先决条件和ROS1差不多。然而，ROS 2还没有为你自动进行标记和版本控制的工具（没有等价于`catkin_create_changelog`工具）。


### 要求工具

对于ROS2的Ardent来说：

 * bloom >= 0.6.2
 * catkin_pkg >= 0.4.0

### 过程

#### 第一步：changelog（可选）

创建/更新CHANGELOG.rst，使用即将到来的新格式。注意，changelog严格来说是可选的，但它是非常推荐的。

注意你的changelog格式中的错误可能会导致你的包发布问题。提交并将更改提交给变更日志。

#### 第二步：标记包版本号

在`<package.xml>`中更新包的版本。版本号必须比前一个版本高。对于你的第一个版本，我们建议0.0.1或1.0.0。提交并推动这个变更。

注意，你不能使用以前使用的版本号(参见下面)。一些包释放到ROS 1和ROS 2，但是由于这个需求，必须使用不同的版本控制系列。ROS包通常不遵循严格的[语义版本][1]控制，所以不要过分担心。如果你想了解其他人已经做了什么，请使用ROS 1发布包中的0.x.x或1.x.x系列和ROS 2发布包中的1.x.x或2.x.x系列。

#### 第三步：标记你的包

创建一个与你刚刚输入到package.xml中的版本号相匹配的标记，在提交时，会遇到版本号。现在你知道了不能重用版本号的原因——git只允许在存储库中使用给定名称的一个标记。

#### 第四步：确保你的bloom和catkin_pkg是最新版本

查看以上版本要求，运行以下命令追踪当前版本：

```
sudo apt-get install python-catkin-pkg python-bloom 
```

#### 第五步：设置ROS 2环境变量

ROS 2使用的是全新的存储库，该版本的所有索引保存在https://github.com/ros2/rosdistro，forked该存储库。你可以通过设置ROSDISTRO_INDEX_URL环境变量来配置bloom。

```
export ROSDISTRO_INDEX_URL='https://raw.githubusercontent.com/ros2/rosdistro/ros2/index.yaml'
```

导出这个之后，你将能够在你的bloom-release终端命令中使用ROS 2发行版的`ardent`, `bouncy`等名称。

### 下一步

现在，你的存储库设置完毕。你已经手动完成ROS 2在[这个界面][2]的内容

返回到ROS Wiki上的[Bloom发布][3]教程，并继续“创建一个发布存储库”。

[1]: https://semver.org/
[2]: http://wiki.ros.org/cn/bloom/Tutorials/PrepareUpstream
[3]: http://wiki.ros.org/cn/bloom/Tutorials/FirstTimeRelease
