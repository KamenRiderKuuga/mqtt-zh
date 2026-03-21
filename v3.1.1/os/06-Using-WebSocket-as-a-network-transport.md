# 6 Using WebSocket as a network transport

如果 `MQTT` 通过 `WebSocket` [RFC6455] 连接传输，则适用以下条件：

- `MQTT Control Packets` 必须在 `WebSocket` binary data frames 中发送。如果收到任何其他类型的数据帧，接收方必须关闭 `Network Connection` [MQTT-6.0.0-1]。

- 单个 `WebSocket` 数据帧可以包含多个或部分的 `MQTT Control Packets`。接收方不应该假设 `MQTT Control Packets` 在 `WebSocket` 帧边界上对齐 [MQTT-6.0.0-2]。

- `Client` 必须在它提供的 `WebSocket` Sub Protocols 列表中包含 "mqtt" [MQTT-6.0.0-3]。

- `Server` 选择并返回的 `WebSocket` Sub Protocol 名称必须是 "mqtt" [MQTT-6.0.0-4]。

- 用于连接 `Client` 和 `Server` 的 `WebSocket` URI 对 `MQTT` 协议没有影响。

## 6.1 IANA Considerations

本规范请求 IANA 在 "WebSocket Subprotocol Name" 注册表中注册 `WebSocket` `MQTT` 子协议，数据如下：

**图 6.1 - IANA WebSocket Identifier**

| Subprotocol Identifier | mqtt |
|---|---|
| Subprotocol Common Name | mqtt |
| Subprotocol Definition | http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.html |