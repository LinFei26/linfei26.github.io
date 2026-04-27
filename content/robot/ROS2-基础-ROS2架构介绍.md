---
title: "ROS2架构介绍"
date: 2026-04-24
draft: false
description: "介绍一下ROS2的架构"          # 摘要（显示在列表页）
tags: ["ROS2","基础"]                  # 例如：["ROS2", "机器人"]
categories: ["ROS2"]
cover:
  image: ""
  alt: ""    #图片等效文字
  caption: ""    #说明文字
  relative: false   #false: 表示使用绝对路径或外部链接，true: 表示使用相对路径（相对于当前页面）
author: "林非"
showToc: true    #显示目录
tocOpen: false    #目录默认折叠
---

<!--more-->

# 1.架构总览
<img src="\images\ROS2-structure.png">

通过这张图，我们要清楚自己程序的运行思路，最上面的是我们写的程序和系统自带的程序。程序会通过封装好的C++或者python接口，进行传递，一直传递到DDS层。发布者在DDS层发出信息，订阅者就会收到相应的信息，再向上传递，反映到对应的程序里。

1. 系统操作层

Windows、Linux、Mac系统

提供各种硬件的驱动：网卡驱动，USB驱动，摄像头驱动等。

2. DDS实现层

数据分发服务Data Distribution Service 

该服务基于实时发布订阅协议（Real-time Publish-Subscribe, RTPS ）

[什么是RTPS以及常见厂商](什么是RTPS以及常见厂商.md)

3. DDS接口层

因为要支持不同厂家的DDS，同时又需要对外保持一致，所以ROS2定义了RMW(ROS Middleware Interface，ROS 中间件接口)，再由不同 DDS进行实现，为 ROS2客户端层提供统一的调用接口。举例来说，DDS接口层类似于USB接口的标准，而DDS实现层就是不同厂家根据标准生产的USB设备

在ROS 2中切换DDS非常简单，只需通过设置`RMW_IMPLEMENTATION`环境变量即可，无需修改任何代码。

```Shell
# 临时切换
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

# 或永久切换（添加至 ~/.bashrc）
echo "export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp" >> ~/.bashrc
```


4. ROS2 客户端层

这一层是ROS2的客户端库（ROS2 client library）

实际这里包含两层，一层是C语言的客户端库（包含与DDS接口层的对接），另一层是C++，python等语言的客户端库（包含与C语言的对接）

5. 应用层

所有基于RCL开发的程序，都在这一层。比如自带的海龟模拟器，后续自开发的程序。或者rviz2 ，QT等


# 2. 需要注意
1. 有一类特殊的包，叫ros_to_dds，是从用户应用层直接与DDS层联通
2. 