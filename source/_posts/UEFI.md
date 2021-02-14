---
title: UEFI
date: 2020-04-04 14:44:56
tags: System
---

windows启动模式包括两种：UEFI+GPT、Legacy+MBR。
在此介绍3中在windows上查看系统启动模式的方法。

<!--more-->  

# （一）、diskpart判断UEFI方法

win+r打开运行输入diskpart命令。
<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/diskpart1.png"/>

GPT一行下面带*表示使用的GPT分区表+UEFI启动模式。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/diskpart2.png"/>


# （二）系统信息判断

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/systeminfo1.png"/>

也可以win+r打开运行，输入msinfo32命令来执行打开系统信息。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/msinfo321.png"/>

在系统摘要中可以看到bios的模式

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/msinfo322.png"/>

# （三）、磁盘管理判断UEFI

win+r打开，输入compmgmt.msc，打开管理。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/compmgmt.msc1.png"/>

在磁盘管理中将鼠标放到引导硬盘的分区上，是UEFI引导会有相应的提示，否则为传统模式。

<img src="https://blog-image-host.oss-cn-shanghai.aliyuncs.com/gyqblog/compmgmt.msc2.png"/>


**参考资料**
https://jingyan.baidu.com/article/49ad8bce1bf31a1935d8fa09.html