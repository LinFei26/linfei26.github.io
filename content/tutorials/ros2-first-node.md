---
title: "ROS2 入门：5分钟搭建第一个节点"
date: 2026-04-23T21:00:00+08:00
draft: false
description: "从零开始，在 Ubuntu 上安装 ROS2 Humble 并运行第一个 Python 节点。"
tags: ["ROS2", "入门", "Python"]
categories: ["robot"]
difficulty: "beginner"
duration: "20 min"
prerequisites: ["Ubuntu 22.04", "基础 Python"]
author: "LinFei"
showToc: true
tocOpen: true
---

## 📋 前置准备

- Ubuntu 22.04 LTS
- 基础的 Linux 命令行操作

## 🎯 学习目标

1. 成功安装 ROS2 Humble
2. 创建并运行一个简单的 Publisher 节点

---

## 第一步：安装 ROS2 Humble

```bash
# 设置 locale
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8

# 添加 ROS2 仓库
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
  -o /usr/share/keyrings/ros-archive-keyring.gpg

# 安装 ROS2
sudo apt update
sudo apt install ros-humble-desktop
```

## 第二步：创建 Publisher 节点

```python
# my_publisher.py
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalPublisher(Node):
    def __init__(self):
        super().__init__('minimal_publisher')
        self.publisher_ = self.create_publisher(String, 'topic', 10)
        timer_period = 0.5
        self.timer = self.create_timer(timer_period, self.timer_callback)

    def timer_callback(self):
        msg = String()
        msg.data = 'Hello, ROS2!'
        self.publisher_.publish(msg)
        self.get_logger().info(f'Publishing: {msg.data}')

def main(args=None):
    rclpy.init(args=args)
    node = MinimalPublisher()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

## 第三步：运行

```bash
source /opt/ros/humble/setup.bash
python3 my_publisher.py
```

---

## ✅ 小结

你已经成功运行了第一个 ROS2 节点，恭喜！下一步可以学习 Subscriber 和服务通信。
