---
title: "节点（Node）介绍"
date: 2026-04-27
draft: false
description: "什么是节点，基本结构是什么"          # 摘要（显示在列表页）
tags: ["ROS2","节点"]                  # 例如：["ROS2", "机器人"]
categories: ["ROS2"]
series: ["ROS2 通信基础"]
series_order: 1
weight: 1
cover:
  image: ""
  alt: ""    #图片等效文字
  caption: ""    #说明文字
  relative: false   #false: 表示使用绝对路径或外部链接，true: 表示使用相对路径（相对于当前页面）
author: "林非"
---

<!--more-->

## 一、什么是 ROS2 节点？

**节点（Node）** 是 ROS2 中最基本的可执行计算单元。你可以把它理解为一个 **独立运行的小程序**，它负责机器人系统中的一项具体任务，例如：

- 读取激光雷达数据
- 控制电机转速
- 规划导航路径
- 发布机器人状态信息

ROS2 程序通常采用 **“多节点协作”** 的模式：一个复杂的机器人系统会被拆分为多个功能单一的节点，这些节点通过 **话题（Topic）**、**服务（Service）** 或 **动作（Action）** 相互通信，最终完成整体任务。

这种设计带来了巨大的好处：

| 特点 | 说明 |
|------|------|
| **模块化** | 每个节点可以独立开发、测试、调试。 |
| **可复用** | 一个写好的节点可以在不同项目中被直接使用。 |
| **容错性** | 某个节点崩溃不影响其他节点的运行（除非强依赖）。 |
| **分布性** | 节点可以运行在同一台计算机的不同进程，也可以分布在不同机器上，ROS2 的通信机制自动处理网络传输。 |

> 🔑 一句话定义：**节点是 ROS2 图中执行计算的一个参与者，它通过 ROS2 中间件与其他节点交换数据。**

---

## 二、节点基本结构

### 2.1 概念结构

一个典型的 ROS2 节点在概念上包含以下几个部分：

```
┌─────────────────────────────────┐
│           节点                   │
│  ┌───────────────────────────┐  │
│  │  名称 (Name)               │  │
│  │  命名空间 (Namespace)       │  │
│  └───────────────────────────┘  │
│  ┌───────────────────────────┐  │
│  │  发布者 (Publisher)         │  │
│  │  订阅者 (Subscriber)        │  │
│  │  服务服务器 (Service Server) │  │
│  │  动作客户端 (Action Client)  │  │
│  │  参数 (Parameter)           │  │
│  └───────────────────────────┘  │
│  ┌───────────────────────────┐  │
│  │  定时器 (Timer)             │  │
│  │  回调组 (Callback Group)    │  │
│  │  执行器 (Executor)          │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

- **名称与命名空间**：节点在 ROS2 图中必须有**唯一的名称**，可以搭配命名空间来组织（如 `/robot1/lidar_node`）。
- **通信接口**：节点通过创建发布者、订阅者等对象与其他节点交互。
- **定时器**：用于周期性地执行某些任务（如每 0.1 秒读取一次传感器）。
- **执行器**：负责调度节点的回调函数（当消息到达或定时器触发时，执行相应函数）。

### 2.2 文件结构（Python 包为例）

一个 ROS2 Python 节点的源代码通常位于一个功能包（Package）中，典型的目录结构如下：

```
my_robot_package/
├── package.xml                 # 包的描述文件（依赖、版本等）
├── setup.py                    # Python 包的安装脚本
├── setup.cfg                   # 配置辅助文件
├── resource/                   # 资源文件（可选）
└── my_robot_package/           # 与包同名的 Python 模块目录
    ├── __init__.py
    ├── my_node.py              # 节点的源代码文件
    └── utils.py                # 其他辅助模块（可选）
```

- `my_node.py` 中通常会定义一个继承自 `rclpy.node.Node` 的类，并在 `main()` 函数中初始化节点并进入事件循环。

---

## 三、代码解读：以 Python 为例

下面展示一个最简单的 ROS2 节点，它只做一件事：启动后打印一条信息，然后保持运行。

```python
#!/usr/bin/env python3
import rclpy                     # ROS2 Python 客户端库
from rclpy.node import Node      # 节点基类

class MinimalNode(Node):
    """自定义节点类，继承自 Node"""
    def __init__(self):
        # 调用父类构造函数，节点的名称设置为 "minimal_node"
        super().__init__('minimal_node')
        # 使用日志输出工具打印信息
        self.get_logger().info('节点已启动，等待中...')

def main(args=None):
    # 1. 初始化 ROS2 客户端库
    rclpy.init(args=args)

    # 2. 创建节点实例
    node = MinimalNode()

    # 3. 让节点持续运行（阻塞直到收到关闭信号，如 Ctrl+C）
    rclpy.spin(node)

    # 4. 清理资源
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 逐行解读

| 行号(范围) | 解释 |
|----------|------|
| `#!/usr/bin/env python3` | Shebang 行，指定解释器，让脚本可以被直接执行（`chmod +x` 后）。 |
| `import rclpy` | 导入 ROS2 的 Python 接口库。 |
| `from rclpy.node import Node` | 导入节点基类。 |
| `class MinimalNode(Node):` | 定义自己的节点类，继承自 `Node`。 |
| `super().__init__('minimal_node')` | 调用父类构造器，为节点起名为 `minimal_node`。节点名在 ROS2 图中必须唯一。 |
| `self.get_logger().info(...)` | 获取节点的日志器并输出 INFO 级别日志（会显示在终端）。 |
| `def main(args=None):` | 主函数入口。 |
| `rclpy.init(args=args)` | 初始化 ROS2 通信层（必须在使用任何 ROS2 功能前调用）。 |
| `node = MinimalNode()` | 实例化节点。 |
| `rclpy.spin(node)` | 让节点进入事件循环。当有消息到达或定时器触发时，执行回调；进程会阻塞在这里直到收到中断信号（如 Ctrl+C）。 |
| `node.destroy_node()` | 销毁节点，释放资源。 |
| `rclpy.shutdown()` | 关闭 ROS2 上下文。 |
| `if __name__ == '__main__':` | Python 入口点，当脚本直接运行时调用 `main()`。 |

### 运行节点

编译后，可以通过以下命令运行（功能包  节点名）：

```bash
ros2 run my_robot_package minimal_node
```

预期输出：

```
[INFO] [minimal_node]: 节点已启动，等待中...
```

按 `Ctrl+C` 停止节点。

---

## 四、节点的生命周期与管理

### 4.1 节点状态

ROS2 节点在运行时有一个内部状态机，涉及以下主要状态（由 `rclcpp` / `rclpy` 管理，开发者一般不直接操作）：

- **Unconfigured**：节点刚被创建，但未配置。
- **Inactive**：节点已配置但不能处理数据（收发通信处于休眠状态）。
- **Active**：正常运行，处理消息、定时器等。
- **Finalized**：节点已关闭。

通常我们使用 `rclpy.spin(node)` 让节点一直处于 Active 状态。

### 4.2 多个节点同时运行

可以在**同一个程序**中创建多个节点实例（极少如此做），更常见的是**每个可执行文件启动一个节点**，然后通过多个 terminal 运行多个节点，或者使用 **Launch 文件** 同时启动一组节点。

---

## 五、核心注意事项与最佳实践

### 5.1 节点名称必须唯一

在同一个 ROS2 图中（即同一个 DDS 域），节点名称（包括命名空间）必须唯一。如果启动两个同名的节点，后启动的节点会因为名称冲突而失败。

✅ 推荐做法：为节点起有意义且不易冲突的名字，例如 `lidar_processor`，`motor_controller`。必要时使用命名空间区分，如 `/robot1/lidar_processor`。

### 5.2 处理好资源释放

虽然 Python 有垃圾回收，但 ROS2 的底层资源（如发布者、订阅者）在节点销毁时需要显式释放。使用 `rclpy.spin()` 结束时，务必调用 `destroy_node()` 和 `rclpy.shutdown()`。

✅ 推荐做法：将节点创建和应用逻辑放在 `try-finally` 或使用上下文管理器（Python 的 `with` 不直接支持，但可以自己封装）。最简单的保证方式是：

```python
def main():
    rclpy.init()
    node = MyNode()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()
```
try的部分，即使遭受了系统崩溃，也依然会执行finally部分的程序。

### 5.3 回调函数中避免耗时操作

节点的订阅者回调函数、定时器回调函数等是在 ROS2 的执行器线程中被调用的。如果你在回调中执行耗时操作（如长时间循环、大规模文件 I/O），会阻塞其他回调的执行，导致消息积压。

✅ 推荐做法：回调函数应尽快返回。耗时工作可以：
- 另起线程处理（把回调函数当作一个开关，打开就继续往下走。又新的线程执行任务）
- 将任务拆分成多个小步骤，使用多个定时器
- 使用 ROS2 的 **动作（Action）** 机制处理长时间任务

### 5.4 合理使用执行器

`rclpy.spin(node)` 使用默认的 **单线程执行器**（`SingleThreadedExecutor`）。如果你的节点有多个订阅者且消息频率很高，单线程可能处理不过来，此时可改用 **多线程执行器**（`MultiThreadedExecutor`）。

```python
from rclpy.executors import MultiThreadedExecutor

executor = MultiThreadedExecutor()
executor.add_node(node)
executor.spin()
```
[如何更好的理解多线程执行器](如何更好的理解多线程执行器.md)

### ※ 5.5 不要在构造函数中执行阻塞代码 

`__init__` 中应该只做节点初始化的轻量工作（创建发布者、订阅者等）。不要在其中调用 `rclpy.spin()` 或执行长时间循环，否则节点永远不会进入事件循环。

---

## 六、完整示例：发布者与订阅者节点

为了展示节点的实际通信能力，下面给出一个简单的发布者节点（发送消息）和一个订阅者节点（接收并打印）。

### 6.1 发布者节点 `talker.py`

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class Talker(Node):
    def __init__(self):
        super().__init__('talker')
        # 创建发布者，话题为 'chatter'，消息类型 String，队列大小 10
        self.publisher = self.create_publisher(String, 'chatter', 10)
        # 定时器，每 0.5 秒调用一次 timer_callback
        self.timer = self.create_timer(0.5, self.timer_callback)
        self.counter = 0

    def timer_callback(self):
        msg = String()
        msg.data = f'Hello ROS2, count: {self.counter}'
        self.publisher.publish(msg)
        self.get_logger().info(f'Publishing: "{msg.data}"')
        self.counter += 1

def main(args=None):
    rclpy.init(args=args)
    node = Talker()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 6.2 订阅者节点 `listener.py`

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class Listener(Node):
    def __init__(self):
        super().__init__('listener')
        # 创建订阅者，订阅 'chatter' 话题，回调函数是 self.callback
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

### 6.3 运行二者（两个终端）

```bash
# 终端1
ros2 run my_package talker

# 终端2
ros2 run my_package listener
```

观察输出：talker 每 0.5 秒发布一条消息，listener 会实时打印收到的消息。

---

## 七、总结

| 知识点 | 核心要点 |
|--------|----------|
| **定义** | 节点是 ROS2 中最小的计算单元，独立运行，通过通信机制协作。 |
| **创建** | 继承 `rclpy.node.Node`，在 `__init__` 中初始化通信接口和定时器。 |
| **运行** | `rclpy.init()` → 创建节点 → `rclpy.spin()` → 清理。 |
| **命名** | 节点名称必须在图中唯一，推荐使用有意义的名字。 |
| **回调** | 回调函数应简短，避免阻塞执行器。 |
| **多节点** | 可以用 Launch 文件一次性启动多个节点。 |

节点是 ROS2 编程的起点。理解节点的结构、生命周期和最佳实践，你就能写出健壮、可复用的机器人软件。下一步可以学习如何创建 launch 文件、自定义消息接口，以及使用参数服务器等更高级的功能。