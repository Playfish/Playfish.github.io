---
layout:     post
title:      "【ROS总结】发布ROS1包之文档编写"
subtitle:   "Release ROS1 package with document writed"
date:       2018-04-22 18:35:30
author:     "Playfish"
header-img: "img/in-post/release-ros1-with-document/ros_org.png"
header-mask: 0.3
catalog:    true
tags:
    - ROS1
    - Release
    - Document
    - wiki
---


> 说明:<br><br>
> 本文首发于 [CSDN](https://blog.csdn.net/u011118482/article/details/80041201)，转载请保留链接。

## 前言

在上一节中，讲述了如何发布一个包到ROS社区，这一节将讲述，如何在wiki界面编写ROS包说明文档、教程。

步骤如下：

 1. 登录wiki官网，创建界面编辑账号
 2. 向ROS社区提交wiki页面编辑权限，取得白名单
 3. 编辑ROS包文档

## 创建wiki账号

登录wiki.ros.org，选择登录，创建账户

![](/img/in-post/release-ros1-with-document/wiki_login.png)

输入用户名密码
![](/img/in-post/release-ros1-with-document/wiki_create_account.png)

创建完成后，选择登录，登录成功会将显示：
![](/img/in-post/release-ros1-with-document/wiki_whitelists.png)

点击界面获取白名单。

## wiki白名单

在github上，登录当前github账号，在白名单注册界面：https://github.com/ros-infrastructure/roswiki/issues/139

发布请求，比如我的wiki用户为playfish，输入，comment即可。
![](/img/in-post/release-ros1-with-document/get_whitelists.png)

如果一切正常，大家可以登录http://wiki.ros.org/UserGroup，查看自己的名字是否加入。

## 编辑文档

现在编写自己ROS包的使用文档，该文档地址应当在package.xml里面表明，比如我的cht10_node包的package.xml地址为：
![](/img/in-post/release-ros1-with-document/package_xml.png)

因此，输入当前第一个网址 http://ros.org/wiki/cht10_node

![](/img/in-post/release-ros1-with-document/package_template.png)

点击packageTemplate，来创建界面。
![](/img/in-post/release-ros1-with-document/write_wiki.png)

在packageHeader中添加自己的cht10_node包名，该界面会自动链接到cht10_node包。

创建完成后，即可生成界面，文档编辑结束。




