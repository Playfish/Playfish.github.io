---
layout:     post
title:      "【Ubuntu】TeamViewer14 ubuntu 破解商业环境"
subtitle:   "Ubuntu teamviewer14"
date:       2019-04-21 22:29:25
author:     "Playfish"
header-img: "img/in-post/ubuntu-teamviewer14/teamviewer.png"
header-mask: 0.3
catalog:    true
tags:
    - Ubuntu
    - Teamviewer
---


> 说明:<br><br>
> 本文首发于 [Playfish Blog](http://carlzhang.club/2019/04/21/2019-04-21-Teamviewer14-ubuntu-bus/)，转载请保留链接。

## 前言
声明：本教程主要用于个人不小心进入商业环境下修改，如使用商业环境请使用正版。

在平时的使用中，一不小心进入到了商业环境，从而导致受限，无法工作，在此提供Ubuntu下Teamviewer14.1.9025破解方式。
## 确认当前网卡

首先查看当前受限IP日志，位置位于/var/log/teamviewer14/TeamViewer14_Logfile.log，例如：

```
Start:              2019/04/21 21:44:09.931 (UTC+8:00)
Version:            14.1.9025 
ID:                 0
Loglevel:           Info (100)
License:            0
Server:             master11.teamviewer.com
IC:                 xxxxxxxxxxxxx
CPU:                Intel(R) Core(TM) i7-3610QM CPU @ 2.30GHz
CPU extensions:     e9
OS:                 Lx Ubuntu 16.04.6 LT (x86_64)
IP:                 192.168.1.4,192.168.1.8
MID:                xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
MIDv:               0
Proxy-Settings:     Type=0 IP= User=
```

其中```192.168.1.4```为当前受限IP。
## 修改Mac

一般TV根据Mac地址锁定TV  ID号，因此，确定IP为哪个网卡来修改该网卡的Mac地址，使用ifconfig查找当前受限网卡，如：
```
enp0s25   Link encap:Ethernet  HWaddr a0:xx:xx:xx:xx:xx  
          inet addr:192.168.1.4  Bcast:xxx.xxx.x.x  Mask:255.255.255.0
          inet6 addr: xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx/64 Scope:Global
          inet6 addr: xxxx::xxxx:xxxx:xxxx:xxxx/64 Scope:Link
          inet6 addr: xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx/64 Scope:Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:176657 errors:0 dropped:0 overruns:0 frame:0
          TX packets:99859 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:190258934 (190.2 MB)  TX bytes:11587608 (11.5 MB)
          Interrupt:20 Memory:d4400000-d4420000 
```

根据ifconfig找到受限IP的网卡Mac为：a0:xx:xx:xx:xx:xx，修改Mac如下：

网卡不知怎么修改可以将原网卡mac进行加1操作。
```
sudo ifconfig enp0s25 down #（停用网卡）

sudo ifconfig enp0s25 hw ether XX:XX:XX:XX:XX:XX ##(需要更改的MAC地址)

sudo ifconfig enp0s25 up ###（启用网卡）
```
## 初始化TV

修改Mac后，需要删除原TV信息，删除位置共两处，运行：

sudo  rm  -rf /etc/teamviewer/global.conf
rm -rf ~/.local/share/teamviewer14/logfiles/

删除文件后，重启TV服务，运行：

teamviewer daemon restart

## 重启TV

所有配置完成后，重启TV:

teamviewer

重启TV会提示：

进入到初始化环境后，选择接受许可证协议后，即可进入TV，再次连接将不会出现检测到商业环境。
![](/img/in-post/ubuntu-teamviewer14/teamviewer.png)

注：如再次遇到商业环境同样的方法操作即可。

再次声明：如使用商业，请使用正版。
