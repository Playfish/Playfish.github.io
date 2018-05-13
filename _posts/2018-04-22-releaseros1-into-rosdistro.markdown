---
layout:     post
title:      "【ROS总结】发布ROS1包到ROS版本"
subtitle:   "Release ROS1 package into rosdistro"
date:       2018-04-22 17:48:13
author:     "Playfish"
header-img: "img/in-post/release-ros1-into-rosdistro/ros_logo.jpg"
header-mask: 0.3
catalog:    true
tags:
    - ROS1
    - Release
---


> 说明:<br><br>
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/80039148)，转载请保留链接。


## 前言
在ROS开发过程中，想要发布自己的包贡献到ROS社区，也就是想要自己的包可以通过apt-get的形式进行下载，这样可以避免每次源码编译的时候会遇到很多坑的情况，不过想要发布ROS包到ROS社区需要如下能力：

 * 长期维护能力：ROS包会一直迭代更新，因此自己的包也应具有长期维护能力，当然，稳定版本后可以不用长期维护。
 * 开源精神：因为ROS包遵循BSD协议，会要求将包开源后才可以上传到ROS社区，所以将自己的包发布到ROS社区，最好有开源精神。
 * 文档更新：包的更新也伴随这文档更新，当然也不是绝对的，不过最好有包的使用文档，也就是wiki上。

如果以上的可以做到，那么，恭喜你，可以作为开源成员加入到ROS大家庭来，开源自己代码，共享社区。

本教程只是单纯的介绍如何发布自己的包到ROS社区，并且，发布后如何通过apt-get获取下载，关于文档界面编写，也就是wiki说明，会在后面进行说明。

一般发布自己ROS包有如下步骤：

 1. 具有github账号与要发布的存储库地址，发布存储库(bloom-release生成的包路径)，wiki白名单(文档界面做准备)
 2. 发布前准备工作，bloom安装(ROS1推荐使用bloom-release进行一键操作)
 3. 同步到ROS社区
 4. 等待ROS社区的ROS包版本迭代
 5. 编辑wiki文档教程

以上步骤完成后，就可以进行apt-get形式下载自己的包了。假如自己的包为cht10_node，那么可以使用如下命令进行安装：

```
sudo apt-get install ros-<rosdistro>-cht10-node 
```

其中将<rosdistro>替换成自己的ROS版本，也就是自己的包发布到的ROS版本中，比如自己的包发布到了kinetic中，那么就是：

```
sudo  apt-get  install ros-kinetic-cht10-node
```

注意：并不是自己把包发布到ROS社区就等于所有的ROS版本都可以下载，只有发布到对应ROS版本才可以。也就是说要想把自己的包发布到ROS所有版本，必须把自己的包发布到所有的版本中，比如Indigo、Jade、Kinetic、Luna、Melodic中。

ROS发布需要bloom-release包，我已经将ROS的bloom-release包如何使用翻译到wiki上，大家可以查看教程

http://wiki.ros.org/cn/bloom

或者查看英文教程：

http://wiki.ros.org/bloom

以后会编写如何发布ROS2的包到ROS社区教程。

大家也可以看Mastering ROS for Robotics Programming中的Maintaining the ROS package 部分。

Mastering ROS for Robotics Programming书籍下载：https://download.csdn.net/download/u011118482/10402380 (包含英文和中文翻译)。

没有0积分下载，大家也可以进入ROS群进行下载。

## 发布前的准备

### 创建github存储库

登录http://github.com/，创建自己的github账号，创建完成后，创建要发布的ROS存储库，例如包名为之前教程的cht10_node，那么创建cht10_node目录。

我的存储库为：https://github.com/Playfish/cht10_node

![](/img/in-post/release-ros1-into-rosdistro/cht10_repo.png)

创建完成后，上传自己的代码到github的cht10_node存储库中。完成如上图。

注意：发布ROS包应有如下内容，否则无法发布：

 * CHANGELOG.rst：必须有该文件，该文件内容，可以查看我写的，格式一般为包+版本(日期)+分隔符+修订日志
 * package.xml：内容必须有maintainer子项，版本号必须与CHANGELOG中的版本一致，比如都为0.0.1，包名也一样。
 * tag：版本标记。

生成tag用于以下发布ROS包追踪：

点击当前界面上的release按钮，创建一个release tag：

![](/img/in-post/release-ros1-into-rosdistro/cht10_release.png)

保存。

### 创建github release存储库

创建要发布的ROS完成后，为ROS包创建生成的release存储库，我创建的名为cht10_node_release，最好创建为包名_release。这个存储库为空即可，必须勾选初始化ReadeMe.md选项，生成的包版本更新日志将存放在这里。
![](/img/in-post/release-ros1-into-rosdistro/cht10_release_repo.png)

完成后即可，不用再管这个存储库，后期发布完成后，可以查看内容。
### fork ROS社区版本包存储库

保证自己的github账号处于登录状态，点击：http://github.com/ros/rosdistro 。随后点击fork。
![](/img/in-post/release-ros1-into-rosdistro/fork_rosdistro.png)

完成后，可以看到自己有了rosdistro存储库：
![](/img/in-post/release-ros1-into-rosdistro/playfish_rosdistro.png)

## 发布ROS包
发布前的准备完成后，在自己的Ubuntu下，安装bloom_release包，安装命令如下

```
sudo apt-get install python-bloom
```

安装完成后，使用如下命令进行发布：

以下配置只会在第一次产生。

运行以下命令进行发布与配置ROS Release包，比如把cht10_node发布到kinetic版本上：

```
bloom-release --rosdistro kinetic --track kinetic cht10_node
```

其中--rosdistro后的选项为发布到kinetic版本，--track选项为追踪选项，默认为ROS分布式版本，最后的cht10_node为当前存储库名称。

运行命令产生如下：

![](/img/in-post/release-ros1-into-rosdistro/release_0.png)

输入之前创建的发布的release存储库：https://github.com/Playfish/cht10_node_release.git

选择Y确定创建追踪。
![](/img/in-post/release-ros1-into-rosdistro/release_1.png)

输入当前存储库名称：cht10_node

输入当前存储库地址：https://github.com/Playfish/cht10_node.git

随后一路按下回车为默认选项。直到遇到输入用户名、密码为止，在此过程中会遇到很多次输入账号密码。

![](/img/in-post/release-ros1-into-rosdistro/release_2.png)

输入github账号密码，回车。
![](/img/in-post/release-ros1-into-rosdistro/release_3.png)

产生debian配置文件：y

输入当前github用户名密码继续。
![](/img/in-post/release-ros1-into-rosdistro/release_4.png)

产生debian包后，输入用户名密码发布tag。
![](/img/in-post/release-ros1-into-rosdistro/release_5.png)

编辑当前ROS包另一个配置，文档配置以及版本状态，输入默认即可。

注意：turn on pull request testing选择默认为N，如果选择y需要额外配置，不需要打开，我这里是为以后做准备。

额外的配置查看：https://github.com/ros/rosdistro/pull/17576

![](/img/in-post/release-ros1-into-rosdistro/release_6.png)

向rosdistro存储库的kinetic目录下的分布式文件添加当前包内容。

随后向rosdistro存储库提交请求，输入当前github账号密码。
![](/img/in-post/release-ros1-into-rosdistro/release_7.png)

生成请求日志。
![](/img/in-post/release-ros1-into-rosdistro/release_8.png)

完成后，将提示已经发布完成请求，大家可以点击最后的链接查看当前请求。
## 确认工作

请求完成后，大家打开之前创建的空白发布存储库，可以看到已经生成了很多生成deb包的规则文件，例如我的：

https://github.com/Playfish/cht10_node_release.git

![](/img/in-post/release-ros1-into-rosdistro/cht10_release_fill.png)

大家可以到https://github.com/ros/rosdistro/pulls，查看发布情况。
![](/img/in-post/release-ros1-into-rosdistro/report_rosdistro.png)

如果一切正常，ROS维护人员将合并当前请求，合并完成后。

到 讨论论坛中https://discourse.ros.org/查看当前版本下的包迭代情况，一般为一个月ROS包版本迭代一次。

迭代完成后，可以看到自己的包已经存放到了http://packages.ros.org下。

使用apt-get update 更新后，可以下载自己的包了，比如我的包可以用如下命令下载：

sudo  apt-get  install  ros-kinetic-cht10-node

ROS包的发布过程这一节告一段落，下一步讲述如何[编辑cht10_node生成的wiki文档][1]。

注意，如果要生成wiki文档，需要在package.xml里面添加url指向wiki。

[1]: https://blog.csdn.net/u011118482/article/details/80041201
