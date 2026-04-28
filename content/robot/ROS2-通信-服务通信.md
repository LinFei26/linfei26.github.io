---
title: "ROS2 服务（Service）通信完全指南"
date: 2026-04-28
draft: false
description: "什么是服务通信"          # 摘要（显示在列表页）
tags: ["ROS2","服务","通信"]                  # 例如：["ROS2","机器人"]
categories: ["ROS2"]
series: ["ROS2 通信基础"]
series_order: 3
weight: 3
cover:
  image: ""
  alt: ""    #图片等效文字
  caption: ""    #说明文字
  relative: false   #false: 表示使用绝对路径或外部链接，true: 表示使用相对路径（相对于当前页面）
author: "林非"

---


## 一、什么是服务通信？

**服务（Service）** 是 ROS2 中一种**同步请求‑响应**的通信模型。它由**客户端（Client）** 发起请求，**服务端（Server）** 处理请求并返回响应。

与话题不同：
- 话题是**异步**的：发布者只管发，订阅者被动收，没有反馈。
- 服务是**同步**的：客户端发送请求后，会**等待**服务端响应，得到明确的结果。

> 🔧 **适用场景**：需要触发性操作并获取结果的场合，例如：
> - 拍照并返回图像
> - 加法计算器（传入两个数，返回和）
> - 设置参数并返回是否成功
> - 查询机器人当前状态

---

## 二、与话题的对比

| 特性 | **话题（Topic）** | **服务（Service）** |
|------|-------------------|---------------------|
| 通信模型 | 发布‑订阅（异步） | 请求‑响应（同步） |
| 数据流向 | 单向（发布者 → 订阅者） | 双向（客户端 → 服务端 → 客户端） |
| 是否等待响应 | 否 | 是（客户端阻塞直到收到响应） |
| 频率 | 持续流式数据 | 按需触发 |
| 典型用途 | 传感器数据、指令流 | 查询、设置、触发动作 |
| 一对多 | 一个发布者对应多个订阅者 | 一个服务端对应多个客户端 |

---

## 三、服务的基本结构

```text
┌──────────┐    Request    ┌──────────┐
│  Client  │ ────────────→ │  Server  │
│   Node   │               │   Node   │
│          │ ←──────────── │          │
└──────────┘   Response    └──────────┘
```

- **请求（Request）**：客户端发送的数据结构，包含输入参数。
- **响应（Response）**：服务端返回的数据结构，包含执行结果或输出数据。
- **服务名称**：字符串，如 `/add_two_ints`，在ROS2图中唯一标识一个服务。

服务的接口由两部分组成：**请求消息类型** 和 **响应消息类型**。ROS2 将它们打包成一个 **`.srv` 文件**。

---

## 四、服务接口定义（.srv 文件）

### 4.1 标准服务接口举例

以 ROS2 内置的 `example_interfaces/srv/AddTwoInts` 为例：

```text
int64 a
int64 b
---
int64 sum
```

- `---` 上方是请求结构，下方是响应结构。表示客户端将a 和 b 发给服务端，服务端返回sum。

### 4.2 查看服务接口

```bash
ros2 interface show example_interfaces/srv/AddTwoInts
```

### 4.3 常用内置服务

| 服务类型 | 请求 | 响应 | 用途 |
|----------|------|------|------|
| `std_srvs/Empty` | --- | --- | 无参数触发（例如清空数据） |
| `std_srvs/SetBool` | `bool data` | `bool success`<br>`string message` | 设置布尔值并返回结果 |
| `std_srvs/Trigger` | --- | `bool success`<br>`string message` | 触发某个动作 |
| `example_interfaces/AddTwoInts` | `int64 a, b` | `int64 sum` | 加法服务 |

---

## 五、代码解读（Python）

下面我们分别编写**服务端**（提供加法服务）和**客户端**（调用服务）。

### 5.1 服务端节点

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts   # 导入服务接口

class AddTwoIntsServer(Node):
    def __init__(self):
        super().__init__('add_two_ints_server')
        # 创建服务：服务名，服务类型，回调函数
        self.srv = self.create_service(AddTwoInts, 'add_two_ints', self.add_callback)
        self.get_logger().info('加法服务已启动，等待请求...')

    def add_callback(self, request, response):
        """当客户端发送请求时，此函数被调用"""
        # request 包含 a 和 b
        response.sum = request.a + request.b
        self.get_logger().info(f'收到请求: {request.a} + {request.b} = {response.sum}')
        return response   # 必须返回响应对象

def main(args=None):
    rclpy.init(args=args)
    node = AddTwoIntsServer()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

**关键点：**

- `create_service(ServiceType, service_name, callback)`：创建服务端，必须保存到实例变量（否则会被回收）。
- 回调函数 `add_callback(request, response)`：
  - `request` 自动填充请求数据。
  - `response` 是服务类型的响应部分实例（需填充）。
  - 函数**必须返回** `response`。
- 服务端可以同时处理多个客户端请求，ROS2 会自动为每个请求创建一个线程或使用执行器。

### 5.2 客户端节点

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from example_interfaces.srv import AddTwoInts
import sys

class AddTwoIntsClient(Node):
    def __init__(self):
        super().__init__('add_two_ints_client')
        # 创建客户端：服务名，服务类型
        self.client = self.create_client(AddTwoInts, 'add_two_ints')
        # 等待服务可用（可选，但推荐）等待服务端响应
        while not self.client.wait_for_service(timeout_sec=1.0):
            self.get_logger().info('等待服务...')
        self.get_logger().info('服务已准备好')

    def send_request(self, a, b):
        # 创建请求对象
        request = AddTwoInts.Request()
        request.a = a
        request.b = b
        # 异步调用服务（返回 future 对象）
        self.future = self.client.call_async(request)
        # 可以添加回调或者直接等待 future 完成（见 main 函数）

def main(args=None):
    rclpy.init(args=args)
    node = AddTwoIntsClient()

    # 解析命令行参数（假设传入两个整数），就是模拟在命令行上，第一个位置[0]那里输入脚本名称，第二、三个位置[1]\[2]那里，输入两个整数。
    # 这里是一个三元表达式，格式是 ：值A if 条件 else 值B
    a = int(sys.argv[1]) if len(sys.argv) > 1 else 2
    b = int(sys.argv[2]) if len(sys.argv) > 2 else 3

    node.send_request(a, b)

    # 让节点运行直到收到响应（也可以使用回调）
    while rclpy.ok():
        rclpy.spin_once(node, timeout_sec=0.1)
        if node.future.done():
            try:
                response = node.future.result()
                node.get_logger().info(f'结果: {a} + {b} = {response.sum}')
            except Exception as e:
                node.get_logger().error(f'服务调用失败: {e}')
            break

    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

**关键点：**

- `create_client(ServiceType, service_name)`：创建客户端，必须保存。
- `wait_for_service(timeout_sec)`：阻塞直到服务可用，避免调用时服务未启动。
- `call_async(request)`：**异步**调用，立即返回一个 **future 对象**。不会阻塞主线程。
- 要获得响应，需要等待 future 完成。常见做法：
  - 在循环中调用 `rclpy.spin_once()` 并定期检查 `future.done()`。
  - 或者为 future 添加回调：`future.add_done_callback(callback)`。

### 5.3 同步客户端（简单写法）

如果你只想写一个小脚本，可以用 **`call(request)`**（同步阻塞），但不推荐在复杂的多节点程序中使用，因为它会阻塞执行器。

```python
response = self.client.call(request)   # 阻塞直到服务返回
print(response.sum)
```

---

## 六、常用操作命令

```bash
# 列出所有服务
ros2 service list

# 查看服务类型
ros2 service type /add_two_ints

# 查看服务接口详情
ros2 service find example_interfaces/srv/AddTwoInts

# 手动调用服务（测试用）
ros2 service call /add_two_ints example_interfaces/srv/AddTwoInts "{a: 5, b: 3}"
```

手动调用后，服务端会返回 `{sum: 8}`。

---

## 七、注意事项与最佳实践

### 7.1 必须保存客户端/服务端对象

和发布者/订阅者一样，不保存返回值会导致对象被垃圾回收：

```python
self.server = self.create_service(...)   # ✅ 保存
# self.create_service(...)              # ❌ 错误
```

### 7.2 服务回调中不要做长时间阻塞操作

服务回调是在执行器线程中调用的。如果回调中睡眠或循环耗时过长，会阻塞其他回调（包括其他服务请求、订阅者回调）。

- 若确实需要长时间处理，可以将工作提交给另一个线程，并立即返回一个“处理中”的响应（但 ROS2 服务一次请求只能返回一次响应，无法中途反馈进度；如需进度反馈，请使用 **动作（Action）**）。

### 7.3 客户端应处理服务不可用的情况

服务可能未启动或崩溃。客户端应当：
- 使用 `wait_for_service()` 等待。
- 设置合理的超时。
- 使用异步方式，避免阻塞整个节点。

### 7.4 多个客户端同时请求

服务端默认按顺序处理请求（单线程执行器时），或并发处理（多线程执行器时）。如果服务端是无状态的，一般不需要额外同步。

### 7.5 服务与参数（Parameter）的区别

- **服务**：按需调用，可以有任意复杂的请求/响应。
- **参数**：持久存储的配置值，节点启动时加载，运行时可通过服务（实际也是服务）读写，但接口固定。

ROS2 中参数读写底层也使用服务（`/node_name/get_parameters` 等），但开发者使用 `rclpy` 的 `get_parameter()` 等更方便的 API。

---

## 八、服务 vs 动作（Action）

| 特性 | 服务 | 动作 |
|------|------|------|
| 反馈 | 无（只有最终响应） | 有（周期性反馈） |
| 取消 | 不支持 | 支持 |
| 执行时间 | 短（毫秒~秒级） | 长（秒~分钟） |
| 典型用例 | 计算、设置参数、查询状态 | 导航到目标点、机械臂轨迹规划 |

如果你的任务需要长时间执行并希望获得中间反馈，请使用 **动作**，而不是服务。

---

## 九、总结

| 概念 | 要点 |
|------|------|
| **服务通信** | 同步请求‑响应模型，客户端等待服务端返回结果。 |
| **.srv 文件** | 定义请求和响应数据结构，`---` 分隔。 |
| **服务端** | `create_service()` + 回调函数，返回响应。 |
| **客户端** | `create_client()` + `call_async()` (异步) 或 `call()` (同步)。 |
| **常用命令** | `ros2 service list`, `call`, `type` 等。 |
| **注意事项** | 保存客户端/服务端对象；回调避免阻塞；客户端需处理服务不可用情况；长时间任务用动作。 |

掌握了服务和话题，你已经能够搭建大部分 ROS2 应用的通信骨架。下一步可以学习 **动作（Action）**，它在服务基础上增加了反馈和取消能力，非常适合导航、机械臂运动规划等场景。