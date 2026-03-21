# 7 一致性

`MQTT` 规范定义了 `MQTT Client` 实现和 `MQTT Server` 实现的一致性要求。

`MQTT` 实现**可以**同时符合 `MQTT Client` 和 `MQTT Server` 实现的要求。<span style="background:yellow">既接受入站连接又建立到其他 `Server` 的出站连接的 `Server` **必须**同时符合 `MQTT Client` 和 `MQTT Server` 的要求</span> <span style="color:red">[MQTT-7.0.0-1]</span>。

<span style="background:yellow">符合规范的实现**禁止**要求使用本规范之外定义的任何扩展才能与其他符合规范的实现互操作</span> <span style="color:red">[MQTT-7.0.0-2]</span>。

## 7.1 一致性目标

### 7.1.1 `MQTT Server`

有关 `Server` 的定义，请参阅术语部分中的 [Server](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#Server)。

`MQTT Server` 只有在满足以下所有条件时才符合本规范：

1. `Server` 发送的所有 `MQTT` 控制报文的格式符合[第 2 章](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)和[第 3 章](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packets)中描述的格式。

2. 它遵循 [4.7 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Topic_Names_and)中描述的主题匹配规则和 [4.8 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Subscriptions)中的订阅规则。

3. 它满足以下章节中标识的所有 MUST 级别要求，但仅适用于 `Client` 的要求除外：

   - [第 1 章 - 简介](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Introduction)
   - [第 2 章 - `MQTT` 控制报文格式](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)
   - [第 3 章 - `MQTT` 控制报文](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packets)
   - [第 4 章 - 操作行为](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Operational_behavior)
   - [第 6 章 - 使用 `WebSocket` 作为网络传输](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Using_WebSocket_as)

4. 它不需要使用规范之外定义的任何扩展即可与其他一致性实现互操作。

<span style="background:yellow">符合规范的 `Server` **必须**支持使用一种或多种底层传输协议，这些协议提供从 `Client` 到 `Server` 以及从 `Server` 到 `Client` 的有序、无损的字节流</span> <span style="color:red">[MQTT-7.1.1-1]</span>。然而，一致性并不取决于它是否支持任何特定的传输协议。`Server` **可以**支持第 4.2 节中列出的任何传输协议，或任何其他满足 [MQTT-7.1.1-1] 要求的传输协议。

### 7.1.2 `MQTT Client`

有关 `Client` 的定义，请参阅术语部分中的 [Client](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#Client)。

`MQTT Client` 只有在满足以下所有条件时才符合本规范：

1. `Client` 发送的所有 `MQTT` 控制报文的格式符合[第 2 章](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)和[第 3 章](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packets)中描述的格式。

2. 它满足以下章节中标识的所有 MUST 级别要求，但仅适用于 `Server` 的要求除外：

   - [第 1 章 - 简介](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Introduction)
   - [第 2 章 - `MQTT` 控制报文格式](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)
   - [第 3 章 - `MQTT` 控制报文](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packets)
   - [第 4 章 - 操作行为](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Operational_behavior)
   - [第 6 章 - 使用 `WebSocket` 作为网络传输](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Using_WebSocket_as)

3. 它不需要使用规范之外定义的任何扩展即可与其他一致性实现互操作。

<span style="background:yellow">符合规范的 `Client` **必须**支持使用一种或多种底层传输协议，这些协议提供从 `Client` 到 `Server` 以及从 `Server` 到 `Client` 的有序、无损的字节流</span> <span style="color:red">[MQTT-7.1.2-1]</span>。然而，一致性并不取决于它是否支持任何特定的传输协议。`Client` **可以**支持第 4.2 节中列出的任何传输协议，或任何其他满足 [MQTT-7.1.2-1] 要求的传输协议。