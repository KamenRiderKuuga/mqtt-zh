# 1 简介（Introduction）

## 1.1 MQTT 章节结构

本规范共分为七章：

| 章节 | 标题 | 链接 |
|---|---|---|
| 1 | 简介 | [01-Introduction.md](01-Introduction.md) |
| 2 | MQTT 控制报文格式 | [02-MQTT-Control-Packet-format.md](02-MQTT-Control-Packet-format.md) |
| 3 | MQTT 控制报文 | [03-MQTT-Control-Packets.md](03-MQTT-Control-Packets.md) |
| 4 | 操作行为 | [04-Operational-behavior.md](04-Operational-behavior.md) |
| 5 | 安全性 | [05-Security.md](05-Security.md) |
| 6 | 使用 WebSocket 作为网络传输 | [06-Using-WebSocket-as-a-network-transport.md](06-Using-WebSocket-as-a-network-transport.md) |
| 7 | 一致性 | [07-Conformance.md](07-Conformance.md) |

## 1.2 术语说明

本规范中的关键词 “必须（MUST）”、“禁止（MUST NOT）”、“要求（REQUIRED）”、“应当（SHALL）”、“不应当（SHALL NOT）”、“应该（SHOULD）”、“不应该（SHOULD NOT）”、“推荐（RECOMMENDED）”、“可以（MAY）”、“可选（OPTIONAL）” 均按照 IETF RFC 2119 的定义进行解释。详见：[RFC2119](../01-Introduction.md#rfc2119)。

### 网络连接（Network Connection）
MQTT 所使用的底层传输协议提供的连接构造。它将客户端（Client）与服务器（Server）连接起来，并提供双向有序、无丢失的字节流传输。

### 应用消息（Application Message）
通过 MQTT 协议在网络上传输的应用数据。每条应用消息都关联有服务质量（QoS）和主题名称（Topic Name）。

### 客户端（Client）
使用 MQTT 的程序或设备。客户端总是主动与服务器建立网络连接，并可以：
- 发布应用消息（Publish）
- 订阅应用消息（Subscribe）
- 取消订阅（Unsubscribe）
- 断开连接（Disconnect）

### 服务器（Server）
作为中介，负责在发布应用消息的客户端和订阅消息的客户端之间转发消息。服务器可以：
- 接受客户端的网络连接
- 接收客户端发布的应用消息
- 处理客户端的订阅和取消订阅请求
- 转发匹配订阅的应用消息

### 订阅（Subscription）
由主题过滤器（Topic Filter）和最大 QoS 组成。每个订阅关联一个会话（Session），一个会话可以包含多个订阅。

### 主题名称（Topic Name）
应用消息附带的标签，用于与服务器已知的订阅进行匹配。服务器会将消息发送给所有匹配订阅的客户端。

### 主题过滤器（Topic Filter）
订阅中包含的表达式，用于表示对一个或多个主题的兴趣。可以包含通配符。

### 会话（Session）
客户端与服务器之间的有状态交互。有些会话只持续网络连接期间，有些则可跨多个连接。

### MQTT 控制报文（MQTT Control Packet）
通过网络连接发送的信息包。MQTT 规范定义了 14 种不同类型的控制报文，其中 PUBLISH 报文用于传递应用消息。

## 1.3 规范性引用（Normative references）

- [RFC2119](https://www.ietf.org/rfc/rfc2119.txt)：定义了规范文档中的关键词含义。

---
如需进一步了解术语和章节内容，请参考本仓库其他章节的 Markdown 文件。