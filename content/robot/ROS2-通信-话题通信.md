---
title: "话题通信介绍"
date: 2026-04-28
draft: false
description: "什么是话题通信"          # 摘要（显示在列表页）
tags: ["ROS2","话题","通信"]                  # 例如：["ROS2","机器人"]
categories: ["ROS2"]
cover:
  image: ""
  alt: ""    #图片等效文字
  caption: ""    #说明文字
  relative: false   #false: 表示使用绝对路径或外部链接，true: 表示使用相对路径（相对于当前页面）
author: "林非"

---

<!--more-->
# ROS2 话题与订阅通信完全指南

## 一、什么是话题通信？

**话题（Topic）** 是 ROS2 中最基础的**异步数据流通信**模型。它遵循 **发布‑订阅（Publish‑Subscribe）** 模式：

- **发布者（Publisher）** 节点将数据发送到某个话题。
- **订阅者（Subscriber）** 节点从同一话题接收数据。
- 发布者和订阅者**彼此不知道对方的存在**，通过话题名称自动匹配。

这种设计实现了**解耦**：你可以随时增加或移除发布者/订阅者，不会影响其他节点。

### 1.1 适用场景

| 典型应用 | 说明 |
|----------|------|
| 传感器数据分发 | 激光雷达、摄像头、IMU 等周期性数据 |
| 状态广播 | 机器人位姿、电池电量、任务状态 |
| 低延迟控制指令 | 速度指令、关节角度指令 |
| 日志/调试信息 | 多节点同时接收调试输出 |

> ✅ **话题通信是 ROS2 中使用最频繁的通信机制**，掌握它是学习 ROS2 的第一步。

---

## 二、话题通信的基本结构

```text
┌──────────┐          ┌──────────┐
│ Publisher│          │Subscriber│
│   Node   │          │   Node   │
└────┬─────┘          └────▲─────┘
     │                     │
     │   publish()         │   callback()
     │                     │
     ▼                     │
┌─────────────────────────────────┐
│           Topic                  │
│  (name: "/example_topic")        │
│  (message type: std_msgs/String) │
└─────────────────────────────────┘
```

- **话题名称**：字符串，例如 `/cmd_vel`、`/scan`。建议以斜杠开头。
- **消息类型**：ROS2 定义好的数据结构，如 `std_msgs/String`、`geometry_msgs/Twist`。
- **QoS（Quality of Service）**：一组配置策略，控制消息的可靠性、历史记录、持久性等（后续章节详解）。

---

## 三、代码解读（Python）

下面以一个简单的“发布者‑订阅者”对为例，逐步解释每个部分。

### 3.1 发布者节点

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String   # 导入消息类型

class Talker(Node):
    def __init__(self):
        super().__init__('talker')
        # 创建发布者
        # 参数：消息类型，话题名称，队列深度
        self.publisher = self.create_publisher(String, 'chatter', 10)
        # 创建定时器，每0.5秒触发一次回调
        self.timer = self.create_timer(0.5, self.timer_callback)
        self.counter = 0

    def timer_callback(self):
        msg = String()               # 创建消息对象
        msg.data = f'Hello, count: {self.counter}'
        self.publisher.publish(msg)  # 发布消息
        self.get_logger().info(f'Publishing: "{msg.data}"')
        self.counter += 1

def main(args=None):
    rclpy.init(args=args)
    node = Talker()
    rclpy.spin(node)                # 保持节点运行
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

**关键点注释：**

- `create_publisher(msg_type, topic, qos_depth)`：创建发布者对象，必须保存为实例变量，（就是使用`self.`，这样变量publisher与节点生命周期相同，否则会被垃圾回收）。
- `publish(msg)`：将消息发布到话题上。
- 定时器模拟了周期性发布；实际场景中，传感器驱动节点会在收到硬件数据时直接 `publish()`。

### 3.2 订阅者节点

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class Listener(Node):
    def __init__(self):
        super().__init__('listener')
        # 创建订阅者
        # 参数：消息类型，话题名称，回调函数，队列深度
        self.subscription = self.create_subscription(
            String, 'chatter', self.callback, 10)

    def callback(self, msg):
        self.get_logger().info(f'I heard: "{msg.data}"')

def main(args=None):
    rclpy.init(args=args)
    node = Listener()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

**关键点注释：**

- `create_subscription(msg_type, topic, callback, qos_depth)`：创建订阅者对象，**必须保存返回值**，否则订阅者会被销毁。
- `callback(msg)`：当消息到达时，ROS2 调用此函数，参数为接收到的消息对象。
- 可以多个订阅者订阅同一个话题，每个都会收到独立拷贝。

### 3.3 运行

```bash
# 终端1
ros2 run your_package talker

# 终端2
ros2 run your_package listener
```

你会看到 `listener` 实时打印出 `talker` 发布的每条消息。

---

## 四、消息（Message）详解

### 4.1 什么是消息？

消息是 ROS2 中**传输的数据结构**，类似于 C 语言的 `struct` 或 Python 的 `dataclass`。ROS2 提供了一套**内置消息类型**（位于 `std_msgs`、`geometry_msgs`、`sensor_msgs` 等包中），你也可以**自定义消息**。

### 4.2 常用内置消息示例

| 消息类型 | 定义 | 用途 |
|----------|------|------|
| `std_msgs/String` | `string data` | 字符串数据 |
| `std_msgs/Int32` | `int32 data` | 整数 |
| `geometry_msgs/Twist` | `Vector3 linear`<br>`Vector3 angular` | 速度指令（线速度+角速度） |
| `sensor_msgs/LaserScan` | `float32[] ranges`<br>`float32 angle_min` 等 | 激光雷达扫描数据 |

### 4.3 在 Python 中使用消息

```python
from geometry_msgs.msg import Twist

msg = Twist()
msg.linear.x = 1.0      # 前进速度 1 m/s
msg.angular.z = 0.5     # 旋转速度 0.5 rad/s
```

### 4.4 查看消息结构

在终端中可以使用命令查看：

```bash
ros2 interface show std_msgs/msg/String
```

输出：
```
string data
```

---

## 五、QoS（服务质量）配置

QoS 决定话题通信的**可靠性、历史记录、持久性**等行为。这是 ROS2 相比 ROS1 的重要增强。

### 5.1 常用 QoS 策略

| 策略 | 可选值 | 含义 |
|------|--------|------|
| **可靠性 (Reliability)** | `RELIABLE`<br>`BEST_EFFORT` | 确保消息一定送达（类似 TCP）<br>尽力送达，可能丢包（类似 UDP） |
| **历史记录 (History)** | `KEEP_LAST`<br>`KEEP_ALL` | 只保留最近 N 条消息<br>保留全部消息 |
| **深度 (Depth)** | 整数 (如 10) | 当 `KEEP_LAST` 时，保留的条数 |
| **持久性 (Durability)** | `VOLATILE`<br>`TRANSIENT_LOCAL` | 消息只对当前已存在的订阅者有效<br>晚到的订阅者也能收到最后一条消息 |

### 5.2 默认 QoS

创建发布者/订阅者时，如果不指定 QoS，会使用 **系统默认 QoS**：

```python
# 默认 QoS：Reliability = RELIABLE, History = KEEP_LAST, Depth = 10
self.pub = self.create_publisher(String, 'topic', 10)
```

上面代码中的 `10` 实际上就是 `KEEP_LAST` 的深度，其它使用默认值。

### 5.3 自定义 QoS

```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy

qos = QoSProfile(
    reliability=ReliabilityPolicy.RELIABLE,
    history=HistoryPolicy.KEEP_LAST,
    depth=5
)
self.pub = self.create_publisher(String, 'topic', qos)
```

### 5.4 兼容性规则

- 发布者的 QoS 与订阅者的 QoS **必须兼容**，否则无法建立通信。
- 一般规则：订阅者的可靠性要求不能高于发布者（例如发布者 `BEST_EFFORT`，订阅者 `RELIABLE` 会不兼容）。
- 建议：除非有特殊需求，否则使用默认 QoS 即可。

---

## 六、注意事项与最佳实践

### 6.1 必须保留 Publisher / Subscriber 对象

```python
# ❌ 错误：没有保存返回值
self.create_publisher(String, 'chatter', 10)

# ✅ 正确
self.publisher = self.create_publisher(String, 'chatter', 10)
```

如果不保存，对象会被 Python 垃圾回收，导致发布/订阅失效。

### 6.2 回调函数中不要做耗时操作

订阅者的回调函数在 ROS2 执行器线程中运行。如果回调函数执行时间过长，会**阻塞其他回调**，导致消息积压。

```python
# ❌ 错误示范
def callback(self, msg):
    time.sleep(5)          # 模拟耗时操作
    self.process_data(msg)

# ✅ 正确做法：仅存数据，另开线程处理
def callback(self, msg):
    self.latest_data = msg.data   # 简单赋值
```

如果必须做耗时任务，可以考虑：
- 使用 **多线程执行器**（`MultiThreadedExecutor`）
- 将耗时任务放入独立节点，通过另一个话题通信

### 6.3 队列深度（Queue Depth）的选取

- **深度太小**：在高频率话题下，消息可能被丢弃（新的覆盖旧的）。
- **深度太大**：占用更多内存，且当订阅者处理不及时时，会积压大量历史消息。

一般建议：
- 传感器数据（高频）：深度 = 1~5（只需要最新帧）
- 控制指令（低频但有历史需求）：深度 = 10~50

### 6.4 话题命名规范

- 使用小写字母、下划线分割，如 `/robot/cmd_vel`。
- 斜杠开头表示**全局话题**；不以斜杠开头是**相对话题**（相对于节点命名空间）。
- 避免使用特殊字符（`@`, `~` 等）。

### 6.5 调试命令

运行时可以用这些命令观察话题：

```bash
ros2 topic list                      # 列出所有活跃话题
ros2 topic echo /chatter             # 打印话题上的消息
ros2 topic info /chatter --verbose   # 查看发布者/订阅者信息
ros2 topic hz /chatter               # 查看发布频率
```

---

## 七、多线程订阅注意事项

默认 `rclpy.spin(node)` 使用**单线程执行器**。如果节点有多个订阅者，所有回调都在同一个线程中串行执行。

- 当某个回调阻塞时，其它话题的消息**无法被处理**。
- 解决方案：
  1. 确保回调函数简短。
  2. 使用 `MultiThreadedExecutor`：

```python
from rclpy.executors import MultiThreadedExecutor

executor = MultiThreadedExecutor()
executor.add_node(node)
executor.spin()
```

多线程后，不同消息回调可能**并发执行**，需要注意共享资源的线程安全（使用锁或回调组）。

---

## 八、总结

| 概念 | 要点 |
|------|------|
| **话题通信** | 异步、一对多、解耦的数据流模型。 |
| **发布者** | 创建时指定话题和消息类型，调用 `publish()` 发送数据。 |
| **订阅者** | 创建时指定话题和回调函数，ROS2 自动调用回调。 |
| **消息** | 传输的数据结构，有内置类型也支持自定义。 |
| **QoS** | 控制可靠性、历史记录等，默认配置足够初学者使用。 |
| **注意事项** | 必须保存发布者/订阅者对象；回调函数避免耗时操作；合理设置队列深度。 |

掌握了话题通信，你就能够搭建大部分 ROS2 机器人应用的数据流骨架。下一步可以学习 **服务（Service）** 和 **动作（Action）**，它们分别适合**同步请求‑响应**和**长时间任务**的场景。