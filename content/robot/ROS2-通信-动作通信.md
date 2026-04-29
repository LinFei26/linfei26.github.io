---
title: "ROS2 动作（Action）通信完全指南"
date: 2026-04-28
draft: false
description: "什么是动作通信"          # 摘要（显示在列表页）
tags: ["ROS2","动作","通信"]                  # 例如：["ROS2","机器人"]
categories: ["ROS2"]
series: ["ROS2 通信基础"]
series_order: 4
weight: 4
cover:
  image: ""
  alt: ""    #图片等效文字
  caption: ""    #说明文字
  relative: false   #false: 表示使用绝对路径或外部链接，true: 表示使用相对路径（相对于当前页面）
author: "林非"

---

## 一、什么是动作通信？

**动作（Action）** 是 ROS2 中用于**长时间任务**的通信模型，它结合了**服务**和**话题**的特点：

- 像**服务**一样：客户端发起一个目标（goal），服务端执行并最终返回结果（result）。
- 像**话题**一样：在执行过程中可以**周期性反馈（feedback）** 进度。
- 额外支持**取消（cancel）** 正在执行的任务。

> 🎯 **适用场景**：需要较长时间才能完成，且用户希望获得进度反馈或能中途取消的任务。典型例子：
> - 导航到目标点（“还有 10 米…5 米…到达”）
> - 机械臂抓取物体（“正在接近…夹取…提起…完成”）
> - 自动泊车（“寻找车位…倒车…完成”）
> - 拍照并处理（“拍照中…处理中…完成”）

---

## 二、动作与服务、话题的对比

| 特性 | **话题（Topic）** | **服务（Service）** | **动作（Action）** |
|------|-------------------|---------------------|--------------------|
| 通信模型 | 异步发布‑订阅 | 同步请求‑响应 | 异步带反馈的请求‑响应 |
| 数据流向 | 单向 | 双向（一次来回） | 双向（目标+反馈+结果） |
| 是否等待响应 | 否 | 是（阻塞或异步） | 是（异步，可监听反馈） |
| 能否取消 | 不适用 | 否 | **是** |
| 进度反馈 | 无 | 无 | **有（周期性反馈）** |
| 典型用途 | 传感器数据流 | 短时计算、参数设置 | 长时任务、移动、操作 |

简单总结：
- **话题**：广播，不管有没有人收。
- **服务**：问一次、答一次，立即知道结果。
- **动作**：下达一个目标，执行过程中不断汇报进度，最后告知结果，且随时可以取消。

---

## 三、动作的基本结构

```text
┌──────────┐   Goal           ┌──────────┐
│  Client  │ ───────────────→ │  Server  │
│  (行动端) │                  │  (行动端)│
│          │ ←── Feedback ─── │          │
│          │                  │          │
│          │ ←── Result ────  │          │
└──────────┘                  └──────────┘
```

动作通信包含**三个并行的通信通道**，底层依然由三个话题实现：

1. **目标（goal）**：客户端发布目标，服务端接收。
2. **反馈（feedback）**：服务端周期性地发布进度，客户端订阅。
3. **结果（result）**：服务端执行完成后发布最终结果，客户端接收。
4. **取消（cancel）**：客户端可通过独立的服务请求（底层也是服务）要求服务端取消当前目标。

对开发者而言，ROS2 的 `ActionClient` 和 `ActionServer` 抽象封装了这些细节。

---

## 四、动作接口定义（.action 文件）

动作接口由三部分组成，用 `---` 分隔：

```text
# 目标（Goal）: 客户端发送的请求
# 例如：目标位置坐标
float32 x
float32 y
---
# 反馈（Feedback）: 服务端周期性发送的进度信息
# 例如：剩余距离
float32 remaining_distance
---
# 结果（Result）: 最终返回给客户端的信息
# 例如：是否成功
bool success
string message
```

### 4.1 内置动作示例

ROS2 提供了一些标准动作接口，例如 `turtlesim/action/RotateAbsolute`：

```bash
ros2 interface show turtlesim/action/RotateAbsolute
```

输出：
```text
# 目标：绝对角度（弧度）
float32 theta
---
# 反馈：当前已旋转的角度（弧度）
float32 remaining
float32 actual
---
# 结果：无特别数据（空结构）
---
```

### 4.2 查看活动动作

```bash
ros2 action list -t        # 列出所有动作及类型
ros2 action info /turtle1/rotate_absolute   # 查看详情
```

---

## 五、代码解读（Python）

下面我们以一个**长时间循环动作**为例：服务端模拟一个需要10秒完成的任务，每1秒发送一次进度（已完成百分比），客户端可取消任务。

### 5.1 定义自定义动作接口

为了充分演示，我们先在包中创建一个自定义动作 `Countdown.action`。

**步骤**：
1. 在功能包根目录下创建 `action` 文件夹。
2. 创建文件 `Countdown.action`：

```text
# 目标：倒计时总秒数
int32 total_seconds
---
# 反馈：剩余秒数
int32 remaining_seconds
---
# 结果：是否成功完成
bool success
string message
```

3. 修改 `CMakeLists.txt`（如果使用 C++）或 `package.xml` 并添加依赖。
4. 对于 Python 包，需要添加 `action` 生成逻辑（参考官方文档）。简单起见，我们也可以使用内置动作演示，但为了更好地说明，这里仍使用自定义动作。

由于 Python 包中生成动作比较复杂，我们可以先使用 ROS2 内置的 `turtlesim/action/RotateAbsolute` 来演示客户端调用，服务端也可模拟类似接口。但为了完整展示自定义动作，示例中将给出伪代码思路，实际可运行代码请参考官方教程。

### 5.2 动作服务端（Python）

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from rclpy.action import ActionServer
from rclpy.action.server import ServerGoalHandle
from my_interfaces.action import Countdown   # 假设自定义动作
import time

class CountdownServer(Node):
    def __init__(self):
        super().__init__('countdown_server')
        # 创建动作服务器
        self._action_server = ActionServer(
            self,                     # 节点
            Countdown,                # 动作类型
            'countdown',              # 动作名称
            self.execute_callback     # 执行回调函数（核心）
        )
        self.get_logger().info('倒计时动作服务器已启动')

    def execute_callback(self, goal_handle: ServerGoalHandle):
        """当客户端发送目标时，此函数在一个专用线程中被调用"""
        total_seconds = goal_handle.request.total_seconds
        self.get_logger().info(f'开始倒计时 {total_seconds} 秒')

        feedback_msg = Countdown.Feedback()
        result_msg = Countdown.Result()

        # 模拟倒计时
        for i in range(total_seconds, 0, -1):
            # 检查客户端是否请求取消
            if goal_handle.is_cancel_requested:
                # 接受取消请求
                goal_handle.canceled()
                result_msg.success = False
                result_msg.message = f'倒计时在 {i} 秒时被取消'
                self.get_logger().info(result_msg.message)
                return result_msg

            # 发送反馈
            feedback_msg.remaining_seconds = i
            goal_handle.publish_feedback(feedback_msg)
            self.get_logger().info(f'反馈: 剩余 {i} 秒')
            time.sleep(1)   # 模拟耗时操作（实际应避免阻塞，但这里仅示例）

        # 正常完成
        goal_handle.succeed()
        result_msg.success = True
        result_msg.message = '倒计时完成'
        self.get_logger().info(result_msg.message)
        return result_msg

def main(args=None):
    rclpy.init(args=args)
    node = CountdownServer()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

**关键点：**

- `ActionServer` 需要传入：节点、动作类型、动作名称、执行回调 `execute_callback`。
- 执行回调在**专用线程**中运行（不会阻塞其他回调）。
- 通过 `goal_handle.is_cancel_requested` 检查取消请求，并调用 `goal_handle.canceled()` 接受取消。
- 通过 `goal_handle.publish_feedback()` 发送反馈。
- 正常结束时调用 `goal_handle.succeed()`，然后返回结果对象。

### 5.3 动作客户端（Python）

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from rclpy.action import ActionClient
from my_interfaces.action import Countdown
import sys

class CountdownClient(Node):
    def __init__(self):
        super().__init__('countdown_client')
        self._action_client = ActionClient(self, Countdown, 'countdown')

    def send_goal(self, total_seconds):
        goal_msg = Countdown.Goal()
        goal_msg.total_seconds = total_seconds

        # 等待动作服务器可用
        self._action_client.wait_for_server()

        # 异步发送目标，并附加反馈回调、结果回调
        self._send_goal_future = self._action_client.send_goal_async(
            goal_msg,
            feedback_callback=self.feedback_callback
        )
        # 添加回调，当服务器接受目标时调用
        self._send_goal_future.add_done_callback(self.goal_response_callback)

    def goal_response_callback(self, future):
        goal_handle = future.result()
        if not goal_handle.accepted:
            self.get_logger().info('目标被拒绝')
            return

        self.get_logger().info('目标被接受，开始执行动作')
        self._get_result_future = goal_handle.get_result_async()
        self._get_result_future.add_done_callback(self.result_callback)

    def feedback_callback(self, feedback_msg):
        remaining = feedback_msg.feedback.remaining_seconds
        self.get_logger().info(f'收到反馈: 剩余 {remaining} 秒')

    def result_callback(self, future):
        result = future.result().result
        self.get_logger().info(f'最终结果: success={result.success}, message="{result.message}"')
        rclpy.shutdown()   # 收到结果后可以关闭节点

def main(args=None):
    rclpy.init(args=args)
    node = CountdownClient()
    total = int(sys.argv[1]) if len(sys.argv) > 1 else 10
    node.send_goal(total)
    rclpy.spin(node)   # 保持节点运行以接收反馈和结果

if __name__ == '__main__':
    main()
```

**关键点：**

- `ActionClient` 创建动作客户端。
- `send_goal_async(goal, feedback_callback)`：异步发送目标，立即返回一个 `Future`，不会阻塞。
- 通过 `feedback_callback` 处理周期性反馈。
- 通过 `goal_handle.get_result_async()` 获取最终结果。
- 可以使用 `goal_handle.cancel_goal_async()` 发送取消请求。

---

## 六、常用操作命令

```bash
# 列出所有动作
ros2 action list

# 查看动作类型
ros2 action type /countdown

# 查看动作接口详情
ros2 action info /countdown

# 手动发送动作目标（测试用）
ros2 action send_goal /countdown my_interfaces/action/Countdown "{total_seconds: 5}"
```

---

## 七、注意事项与最佳实践

### 7.1 动作是长时间任务，不要阻塞节点

动作服务端的 `execute_callback` 在独立线程中运行，但如果你在里面做阻塞 I/O 或无限循环，仍会占用该线程。建议：
- 对于纯时间等待，使用 `time.sleep` 仍是可接受的（因为是专用线程）。
- 对于更复杂的任务，可以将任务拆分为多个步骤，或使用异步编程模型（如 asyncio），但 ROS2 Python 客户端库原生不支持 asyncio，推荐使用线程。

### 7.2 处理好取消逻辑

服务端应定期检查 `goal_handle.is_cancel_requested`，并在适当时候执行清理操作，调用 `goal_handle.canceled()` 并返回。

### 7.3 反馈频率不宜过高

反馈也是通过话题发送，频率过高会增加网络负载。对于几秒的任务，0.5~1 秒一次的反馈足够了。

### 7.4 必须保存动作客户端/服务器对象

和发布者、订阅者、服务一样，`ActionClient` 和 `ActionServer` 对象如果未被保存（赋值给实例变量），会被垃圾回收导致动作无法通信。

```python
self._action_client = ActionClient(...)    # ✅ 保存
self._action_server = ActionServer(...)    # ✅ 保存
```

### 7.5 客户端处理服务器不可用的场景

使用 `wait_for_server(timeout_sec)` 等待，避免在服务器未启动时发送目标。

### 7.6 动作适合最终需要结果的任务

如果你的任务只需要发起后不关心结果，且不需要反馈，可以考虑使用话题或服务。

---

## 八、动作 vs 服务之间的选择

| 场景 | 推荐方案 | 理由 |
|------|----------|------|
| 短时间 (<1秒)，无反馈需求 | **服务** | 简单，阻塞调用方便。 |
| 长时间 (>1秒)，需要进度反馈 | **动作** | 内置反馈机制，支持取消。 |
| 长时间，不需要反馈，也不需取消 | 服务或动作均可 | 动作过于重量级，但也可以。 |
| 流式数据 | **话题** | 最适合。 |

---

## 九、总结

| 概念 | 要点 |
|------|------|
| **动作通信** | 用于长时间任务，支持反馈和取消。 |
| **.action 文件** | 定义目标、反馈、结果三部分，`---` 分隔。 |
| **动作服务端** | 使用 `ActionServer` + `execute_callback`，在该回调中处理目标、发送反馈、检查取消、返回结果。 |
| **动作客户端** | 使用 `ActionClient` + `send_goal_async`，设置反馈回调，获取最终结果。 |
| **常用命令** | `ros2 action list`, `info`, `send_goal` 等。 |
| **注意事项** | 保存对象；服务端定期检查取消；反馈频率适中；避免阻塞。 |

动作是 ROS2 中最为高级且实用的通信模型，掌握它就能处理绝大多数真实机器人任务（导航、抓取、巡检等）。如果你已经掌握了话题和服务，学习动作会非常自然。接下来可以尝试编写一个完整的导航动作客户端，与 Nav2 交互，体验机器人的自主移动。