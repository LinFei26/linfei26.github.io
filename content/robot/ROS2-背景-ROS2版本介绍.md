---
title: "ROS2版本介绍"
date: 2026-04-24
draft: false
description: "介绍一下ROS2当前发布的版本"          # 摘要（显示在列表页）
tags: ["ROS2","背景"]                  # 例如：["ROS2", "机器人"]
categories: ["ROS2"]
cover:
  image: ""
  alt: ""    #图片等效文字
  caption: ""    #说明文字
  relative: false   #false: 表示使用绝对路径或外部链接，true: 表示使用相对路径（相对于当前页面）
author: "LinFei"
showToc: true    #显示目录
tocOpen: false    #目录默认折叠
---

<!--more-->

ROS2发展至今，已经发行多个版本了，让我们一起看一下

# 1. **什么是发行**
  ROS 发行版是一组有版本的 ROS 包。这些类似于 Linux 发行版（例如 Ubuntu）。ROS 发行版的目的是让开发者能够在相对稳定的代码库中工作，直到准备好推进所有内容。因此，一旦发行版发布，我们会尽量限制修改，仅限于修复漏洞和核心包（所有在 ros-desktop-full 以下的内容）的非破坏性改进。
# 2. **发行版明细**
  ## 2.1 **常规发行版**
  | 发行版   | 发行日期   | logo   |终止日期   |
  |---------|---------|---------|---------|
  | Kilted Kaiju   | May 23, 2025   |  <img src="/images/Kaijulogo.png"  style="height: 40px; vertical-align: middle;">  |December 2026   |
  | Jazzy Jalisco   | May 23, 2024   | <img src="/images/jazzy-small.png"  style="height: 40px; vertical-align: middle;">   |May 2029   |
  | Iron Irwini    | 	May 23, 2023   | <img src="/images/iron-small.png"  style="height: 40px; vertical-align: middle;">   |December 4, 2024   |
  | Humble Hawksbill   | May 23, 2022   |  <img src="/images/humble-small.png"  style="height: 40px; vertical-align: middle;">   |May 2027  | 
  | Galactic Geochelone   | May 23, 2021   | <img src="/images/galactic-small.png"  style="height: 40px; vertical-align: middle;">   |December 9, 2022   | 
  | Foxy Fitzroy   | June 5, 2020   | <img src="/images/foxy-small.png"  style="height: 40px; vertical-align: middle;">   |June 20, 2023   |      
---

  发行版分为短期版本和长周期支持版本。短期支持一年，长周期支持5年

  每年5月23日会发布新的ROS2版本
  ## 2.2 **即将发行**
  即将到来的是 Lyrical Luth，是一个长周期支持版本
  | 发行版   | 发行日期   | logo   |终止日期   |
  |---------|---------|---------|---------|
  | Lyrical Luth   | May 2026   |  TBD  |May 2031   |
  ## 2.3 **滚动发行版**
  ROS 2 Rolling Ridley 是 ROS 2 的滚动开发发行版，首次于 2020 年 6 月推出。
  ROS 2 的滚动分发有两个目的：
  1. 它是未来稳定 ROS 2 分布的中转区
  2. 它是最新开发版本的合集

顾名思义，Rolling 是持续更新的， 并且可以包含包含重大变更在内的原地更新 。我们建议大多数人使用最新的稳定发行版

在滚动发行版中发布的包将自动发布到未来的稳定版 ROS 2。 将 ROS 2 包发布到滚动发行版中，遵循与其他 ROS 2 发行版相同的程序。

# 3. **发行版对于通信的影响**
  1. 节点并不保证能够跨分布式通信。例如，一个构建并运行于 Humble 的节点，并不保证能与一个构建并运行于 Iron 的节点正确通信。它可能有效也可能无效，但没有支持，不应依赖它。
  2. 跨厂商（单一发行版）通信也不保证。