# 6 使用 WebSocket 作为网络传输

如果 MQTT 通过 WebSocket [RFC6455] 连接传输，则适用以下条件：

- MQTT 控制报文必须在 WebSocket 二进制数据帧中发送。如果接收到任何其他类型的数据帧，接收者必须关闭网络连接 [MQTT-6.0.0-1]。

- 单个 WebSocket 数据帧可以包含多个或部分 MQTT 控制报文。接收者禁止假设 MQTT 控制报文在 WebSocket 帧边界上对齐 [MQTT-6.0.0-2]。

- Client 必须在其提供的 WebSocket 子协议列表中包含 "mqtt" [MQTT-6.0.0-3]。

- Server 选择并返回的 WebSocket 子协议名称必须是 "mqtt" [MQTT-6.0.0-4]。

- 用于连接 Client 和 Server 的 WebSocket URI 对 MQTT 协议没有影响。

## 6.1 IANA 考虑

本规范请求 IANA 使用以下数据在"WebSocket 子协议名称"注册表中修改 WebSocket MQTT 子协议的注册：

**表 6.6-1 - IANA WebSocket 标识符**

| 子协议标识符 | mqtt |
|--------------|------|
| 子协议通用名称 | mqtt |
| 子协议定义 | http://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html |