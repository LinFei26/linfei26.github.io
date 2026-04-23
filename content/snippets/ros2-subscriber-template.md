---
title: "ROS2 话题订阅 - 最简模板"
date: 2026-04-23T21:00:00+08:00
draft: false
description: "ROS2 Python Subscriber 最简实现，直接复用"
tags: ["Python", "ROS2", "节点"]
categories: ["robot"]
language: "python"
author: "LinFei"
showToc: false
---

## 功能说明

ROS2 Python Subscriber 最简实现，订阅 `/topic` 话题并打印消息。

## 使用方法

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class MinimalSubscriber(Node):
    def __init__(self):
        super().__init__('minimal_subscriber')
        self.subscription = self.create_subscription(
            String,
            'topic',
            self.listener_callback,
            10)

    def listener_callback(self, msg):
        self.get_logger().info(f'Received: {msg.data}')

def main(args=None):
    rclpy.init(args=args)
    rclpy.spin(MinimalSubscriber())
    rclpy.shutdown()
```

## 示例输出

```
[minimal_subscriber]: Received: Hello, ROS2!
```
