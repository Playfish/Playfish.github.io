---
layout:     post
title:      "【ROS翻译】ROS2 Ament教程"
subtitle:   "ROS2 ament tutorial"
date:       2018-05-17 14:30:00
author:     "Playfish"
header-img: "img/in-post/release-ros2-into-rosdistro/ros2_logo.png"
header-mask: 0.3
catalog:    true
tags:
    - ROS2
    - Ament
    - Tutorial
---


> 说明:<br><br>
> 本文首发于 [Playfish Blog](http://carlzhang.club/2018/05/17/ament-tutorial/)，转载请保留链接。


## 概述

这将为你提供一个快速总结，如何使用一个工作空间来启动和运行。它将是一个实用的教程，并不是用来取代核心文档的。

### 背景

Ament是catkin元构建工具的一个迭代。欲了解更多有关Ament设计的资料，请参阅[本文件][1]。

该源可以在[ament Github组织中找到][2]。

### 先决条件

### 开发环境

请确保根据build-from-source[指令][3]设置开发环境。

### 基础知识

一个ament空间是一个具有特定结构的目录，通常有一个`src`子目录，在该子目录中，源代码将位于其中，一般情况下，目录开始为空。

Ament是由源生成的。默认情况下，它将创建一个和`src`并行的目录：`build`和`install`目录。`build`目录将是存储中间文件的地方。对于每个包，将在其中创建一个子文件夹，其中将调用CMake。`install`目录是每个包被安装到的地方。

注意:与catkin相比，没有`devel`目录。

### 创建目录结构

在`~/ros2_ws`下创建基本目录结构：

```bash
mkdir -p ~/ros_ws/src
cd ~/ros2_ws
```

这是`~/ros2_ws`的目录结构，你可以在这一点上看到:
```
.
└── src

1 directory, 0 files
```

### 添加源码
首先，我们需要安装一个没有任何ROS2安装的底层。

```bash
wget https://raw.githubusercontent.com/ros2/ros2/master/ros2.repos
vcs import ~/ros2_ws/src < ros2.repos
```

这是`~/ros2_ws`的目录结构，你可以期望在添加了数据源之后(请注意，具体的结构和目录的数量可能会随着时间而变化)


```
.
├── ros2.repos
└── src
    ├── ament
    │   ├── ament_cmake
    │   ├── ament_index
    |   ...
    │   ├── osrf_pycommon
    │   └── uncrustify
    ├── eProsima
    │   ├── Fast-CDR
    │   └── Fast-RTPS
    ├── ros
    │   ├── class_loader
    │   └── console_bridge
    └── ros2
        ├── ament_cmake_ros
        ├── common_interfaces
        ├── demos
        ...
        ├── urdfdom
        ├── urdfdom_headers
        └── vision_opencv

51 directories, 1 file
```

### 运行build

因为这是一个引导环境，我们需要通过它的完整路径调用ament.py。

注意:在将来，一旦在你的系统上安装了ament，或者在工作环境下，这将不再是必要的。

由于在ament上没有`devel`空间，因此需要安装它支持的每个包`--symlink-install`。

这允许通过将`source`空间中的文件(例如Python文件或其他未编译的资源)更改为更快的迭代，从而更改已安装的文件。

```bash
src/ament/ament_tools/scripts/ament.py build --build-tests --symlink-install
```

### 运行测试

要运行你刚刚构建的测试，使用`--build-test`选项，运行以下操作:

```bash
src/ament/ament_tools/scripts/ament.py test
```

如果你在包括测试之前建好了(并且安装了)工作区，你可以跳过构建和安装步骤来加快进程。

```bash
src/ament/ament_tools/scripts/ament.py test --skip-build --skip-install
```

### source环境

当构造成功完成构建时，输出将在`install`目录中。

要使用你需要的可执行文件和库，请将`install/bin`目录添加到你的路径中。

Ament将在`install`目录中生成bash文件，以帮助设置环境。

这些文件将向你的路径和库路径添加所需的元素，并提供由包导出的bash或shell命令。

```bash
. install/local_setup.bash
```

NB:这与catkin略有不同。

`local_setup.*`文件与`setup.*`文件略有不同。它只会应用当前工作区的设置。

当使用多个工作空间时，你仍然会提供`setup.*`文件获取环境，包括所有的父工作区。

### 尝试演示

通过环境来源，你现在可以运行由ament构建的可执行文件。
```bash
ros2 run demo_nodes_cpp listener &
ros2 run demo_nodes_cpp talker
```

你会看到数字在增加。

让我们删除节点并尝试创建我们自己的工作区覆盖。
```bash
^-C
kill %1
```

### 开发你的功能包

在[REP 140][4]中，Ament使用和catkin一样的指定文件`package.xml`。

你可以在`src`目录下创建你自己的功能包，然而，在迭代的时候要用一个叠加来进行迭代。

### 创建覆盖

现在你已经设置好了你的引导，你也会发现`ament`在你的路径上。

让我们创建一个新的覆盖目录： `~/ros2_overlay_ws`。

```bash
mkdir -p ~/ros2_overlay_ws/src
cd ~/ros2_overlay_ws/src
```

并且我们开始覆盖[ros2/examples 存储库][5]：

```bash
# If you know that you're using the latest branch of all
# repositories in the underlay, you can also get the latest
# version of the ros2/examples repository, with this command:
#   git clone https://github.com/ros2/examples.git
# Otherwise, clone a copy from the underlay source code:
git clone ~/ros2_ws/src/ros2/examples
```

构建覆盖层，但是让我们使用debug构建，这样我们就可以确保得到调试符号：

```bash
cd ~/ros2_overlay_ws
ament build --cmake-args -DCMAKE_BUILD_TYPE=Debug

```

现在这个覆盖层在现有的覆盖层之上，所以你会发现`talker`现在指的是来自底层的那个。

如果你source `~/ros2_overlay_ws/install/local_setup.bash`将更改为在覆盖层中引用talker。

如果你正在返回一个新的终端到你的开发，并且想要在你的覆盖层上进行开发，你可以简单地提供`~/ros2_overlay_ws/install/setup.bash`将自动地为所有的父工作区环境提供源。


## 创建自己的包

你可以创建自己的包。
等效的`catkin_create_package`将被移植到ament，但目前还没有。

空间支持多种构建类型。
推荐的构建类型是`ament_cmake`和`ament_python`。
也支持纯`cmake`包。
它将增加对更多[构建类型][6]的支持。

`ament_python`构建的一个示例是[`ament_tools`包][7]，其中setup.py是构建的主要入口点。

像[`demo_nodes_cpp`][8]这样的包使用`ament_cmake`构建类型，并使用CMake作为构建工具。

## 提示

- 如果你不想构建一个特定的包，那么在目录中放置一个名为“AMENT_IGNORE”的空文件，它将不会被索引。

    "Catch all"选项，例如:--cmake-args应该放在其他选项之后，或用'--'分隔：
	
```bash
ament build . --force-cmake-configure --cmake-args -DCMAKE_BUILD_TYPE=Debug -- --ament-cmake-args -DCMAKE_BUILD_TYPE=Release
```

<br>

- 如果你想从包运行单个特定的测试:
```
ament test --only-packages YOUR_PKG_NAME --ctest-args -R YOUR_TEST_IN_PKG
```

[1]: http://design.ros2.org/articles/ament.html
[2]: https://github.com/ament
[3]: https://github.com/ros2/ros2/wiki/Installation
[4]: http://www.ros.org/reps/rep-0140.html
[5]: https://github.com/ros2/examples
[6]: https://github.com/ament/ament_tools/blob/master/doc/development/build_types.rst
[7]: https://github.com/ament/ament_tools
[8]: https://github.com/ros2/demos/tree/master/demo_nodes_cpp
