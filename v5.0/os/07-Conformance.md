# 7 一致性

`MQTT` 规范定义了 `MQTT` `Client` 实现和 `MQTT` `Server` 实现的一致性。`MQTT` 实现可以同时作为 `MQTT` `Client` 和 `MQTT` `Server` 一致。

## 7.1 一致性条款

### 7.1.1 `MQTT` `Server` 一致性条款

有关 `Server` 的定义，请参阅术语部分中的 [Server](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#Server)。

`MQTT` `Server` 只有在满足以下所有声明时才符合本规范：

1. `Server` 发送的所有 `MQTT` 控制报文的格式与[第 2 章](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)和[第 3 章](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packets)中描述的格式匹配。

2. 它遵循 [4.7 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0-os/mqtt-v5.0-os.html#_Topic_Names_and)中描述的主题匹配规则和 [4.8 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Subscriptions)中的订阅规则。

3. 它满足以下章节中标识的必须级别要求，但仅适用于 `Client` 的要求除外：

   - [第 1 章 - 简介](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Introduction)
   - [第 2 章 - `MQTT` 控制报文格式](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)
   - [第 3 章 - `MQTT` 控制报文](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packets)
   - [第 4 章 - 操作行为](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Operational_behavior)
   - [第 6 章 - 使用 `WebSocket` 作为网络传输](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Using_WebSocket_as)

4. 它不需要使用规范之外定义的任何扩展即可与其他一致性实现互操作。

### 7.1.2 `MQTT` `Client` 一致性条款

有关 `Client` 的定义，请参阅术语部分中的 [Client](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#Client)。

`MQTT` `Client` 只有在满足以下所有声明时才符合本规范：

1. `Client` 发送的所有 `MQTT` 控制报文的格式与[第 2 章](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)和[第 3 章](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packets)中描述的格式匹配。

2. 它满足以下章节中标识的必须级别要求，但仅适用于 `Server` 的要求除外：

   - [第 1 章 - 简介](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Introduction)
   - [第 2 章 - `MQTT` 控制报文格式](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)
   - [第 3 章 - `MQTT` 控制报文](https://docs.oasis-open.org/mqtt/mqtt/v5.0-os/mqtt-v5.0-os.html#_MQTT_Control_Packets)
   - [第 4 章 - 操作行为](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Operational_behavior)
   - [第 6 章 - 使用 `WebSocket` 作为网络传输](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Using_WebSocket_as)

3. 它不需要使用规范之外定义的任何扩展即可与其他一致性实现互操作。