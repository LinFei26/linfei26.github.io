---
title: "future的用法"
date: 2026-04-29
draft: false
description: "future的用法"          # 摘要（显示在列表页）
tags: ["ROS2","节点"]                  # 例如：["ROS2", "机器人"]
categories: ["ROS2"]
cover:
  image: ""
  alt: ""    #图片等效文字
  caption: ""    #说明文字
  relative: false   #false: 表示使用绝对路径或外部链接，true: 表示使用相对路径（相对于当前页面）
author: "林非"
---
在 ROS2 的服务客户端中，`client.call_async(request)` 返回的 **`future`** 对象是一个**异步操作的占位符**。它本身不包含结果，而是承诺“将来某个时刻”会得到结果。

---

## 一、为什么需要 `future`？

服务调用需要等待服务端处理并返回响应。如果使用**同步调用** `client.call(request)`，程序会**卡在那里**直到收到响应，这期间无法做任何其他事情（比如处理其他消息、响应新请求）。

使用 **异步调用** `call_async(request)` + `future` 的好处：
- 调用**立即返回**，程序不会被阻塞。
- 你可以在等待结果的同时做其他工作（例如继续处理订阅者的消息）。
- 当结果真正到达时，可以通过 `future` 获取。

可以将 `future` 想象成一个 **“空盒子”**：
- 你发起调用后，马上拿到一个空的盒子。
- 服务端处理完成后，会把结果**放进盒子**。
- 你可以时不时检查盒子是否被填满（`future.done()`），或者注册一个回调函数，等盒子被填满时自动触发。

---

## 二、`future` 的几个关键方法

| 方法 | 作用 |
|------|------|
| `future.done()` | 返回 `True` / `False`，表示结果是否已经到达。 |
| `future.result()` | 获取结果。**如果结果还没到，调用此方法会阻塞等待**（但通常我们会先检查 `done()` 或使用回调避免阻塞）。 |
| `future.add_done_callback(callback)` | 注册一个函数，当未来结果到达时自动调用该函数（参数就是 future 本身）。 |

---

## 三、在 ROS2 代码中如何使用 `future`

### 常见写法1：轮询 `future.done()`

```python
self.future = self.client.call_async(request)

while rclpy.ok():
    rclpy.spin_once(node, timeout_sec=0.1)   # 让节点处理其他回调
    if self.future.done():
        response = self.future.result()
        # 处理响应
        break
```

这种写法在简单的脚本中可见，但需要注意 `rclpy.spin_once` 的存在是为了让 ROS2 通信得以处理（否则 `future` 永远不会变为 done）。

### 常见写法2：使用回调

```python
self.future = self.client.call_async(request)
self.future.add_done_callback(self.response_callback)

def response_callback(self, future):
    response = future.result()
    self.get_logger().info(f'结果: {response.sum}')
```

回调方式更优雅，不占用主循环。

---

## 四、`future` 与 `spin` 的关系

ROS2 的异步通信依赖 **执行器（Executor）** 和 **事件循环**。  
`call_async` 只是把请求发出，并创建了一个 future。当服务端响应回到客户端时，底层 DDS 会把响应传递给 ROS2 客户端库，然后库会**设置 future 的结果**。这个过程需要 `rclpy.spin()` 或 `spin_once()` 来驱动。

所以，在使用 future 的代码中，必须调用 `spin()` 或定期调用 `spin_once()`，否则 future 永远不会完成。

---

## 五、通俗比喻

你给朋友发了一条微信消息，然后：

- `call_async` 相当于**发送消息**。
- `future` 相当于**手机上的“消息已读回执”占位符**（还没有回执，但你知道将来会有一个）。
- `future.done()` 相当于检查对方是否已读。
- `future.result()` 相当于点开消息查看回复内容。
- `spin()` 相当于保持在当前APP等着收消息，不干别的。
- `spin_once()` 相当于每隔timeout_sec这么久，打开APP查看消息，两次查看的间隔，还可以干别的去。
- `add_done_callback` 回调函数，则是发送完消息就干别的去。等对方回了消息，系统会自动弹窗告诉你。

你也可以注册回调：设置“当对方回复时，自动弹窗显示回复内容”，这就是 `add_done_callback`。

---

## 六、总结

- **`future` 是异步调用的结果容器**，用于获取将来才会到达的数据。
- 它允许你在等待结果时不阻塞主线程。
- 必须配合 `spin()` / `spin_once()` 驱动 ROS2 底层通信，否则 future 永远无法完成。
- 常用方法：`done()`、`result()`、`add_done_callback()`。

掌握 future 后，就能写出既响应及时又不阻塞的 ROS2 客户端程序。