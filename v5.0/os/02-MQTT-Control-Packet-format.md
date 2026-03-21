# 2 MQTT Control Packet format

## 2.1 Structure of an MQTT Control Packet

MQTT 协议通过以定义的方式交换一系列 MQTT Control Packet 来运行。本节描述这些报文的格式。

一个 MQTT Control Packet 由最多三部分组成，始终按以下顺序排列，如下所示。

**图 2-1 MQTT Control Packet 的结构**

| 组成部分 |
|---------|
| `Fixed Header`，存在于所有 MQTT Control Packet |
| `Variable Header`，存在于某些 MQTT Control Packet |
| `Payload`，存在于某些 MQTT Control Packet |

### 2.1.1 Fixed Header

每个 MQTT Control Packet 都包含一个 `Fixed Header`，如下所示。

**图 2-2 `Fixed Header` 格式**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type | Flags specific to each MQTT Control Packet type |
| byte 2 | `Remaining Length` |

### 2.1.2 MQTT Control Packet type

**位置：** byte 1, bits 7-4。

表示为一个 4 位无符号值，取值如下所示。

**表 2-1 MQTT Control Packet 类型**

| Name | Value | Direction of flow | Description |
|------|-------|-------------------|-------------|
| Reserved | 0 | Forbidden | 保留 |
| `CONNECT` | 1 | `Client` to `Server` | 连接请求 |
| `CONNACK` | 2 | `Server` to `Client` | 连接确认 |
| `PUBLISH` | 3 | `Client` to `Server` or `Server` to `Client` | 发布消息 |
| `PUBACK` | 4 | `Client` to `Server` or `Server` to `Client` | 发布确认（`QoS` 1） |
| `PUBREC` | 5 | `Client` to `Server` or `Server` to `Client` | 发布已接收（`QoS` 2 传递第 1 部分） |
| `PUBREL` | 6 | `Client` to `Server` or `Server` to `Client` | 发布释放（`QoS` 2 传递第 2 部分） |
| `PUBCOMP` | 7 | `Client` to `Server` or `Server` to `Client` | 发布完成（`QoS` 2 传递第 3 部分） |
| `SUBSCRIBE` | 8 | `Client` to `Server` | 订阅请求 |
| `SUBACK` | 9 | `Server` to `Client` | 订阅确认 |
| `UNSUBSCRIBE` | 10 | `Client` to `Server` | 取消订阅请求 |
| `UNSUBACK` | 11 | `Server` to `Client` | 取消订阅确认 |
| `PINGREQ` | 12 | `Client` to `Server` | PING 请求 |
| `PINGRESP` | 13 | `Server` to `Client` | PING 响应 |
| `DISCONNECT` | 14 | `Client` to `Server` or `Server` to `Client` | 断开连接通知 |
| `AUTH` | 15 | `Client` to `Server` or `Server` to `Client` | 认证交换 |

### 2.1.3 Flags

`Fixed Header` 中 byte 1 的剩余位 [3-0] 包含每个 MQTT Control Packet 类型特定的标志，如下所示。当标志位标记为"Reserved"时，它保留供将来使用，**必须**设置为列出的值 **[MQTT-2.1.3-1]**。如果接收到无效的标志，则这是一个 `Malformed Packet`。有关处理错误的详细信息，请参阅第 4.13 节。

**表 2-2 标志位**

| MQTT Control Packet | `Fixed Header` flags | Bit 3 | Bit 2 | Bit 1 | Bit 0 |
|---------------------|----------------------|-------|-------|-------|-------|
| `CONNECT` | Reserved | 0 | 0 | 0 | 0 |
| `CONNACK` | Reserved | 0 | 0 | 0 | 0 |
| `PUBLISH` | Used in MQTT v5.0 | `DUP` | `QoS` | | `RETAIN` |
| `PUBACK` | Reserved | 0 | 0 | 0 | 0 |
| `PUBREC` | Reserved | 0 | 0 | 0 | 0 |
| `PUBREL` | Reserved | 0 | 0 | 1 | 0 |
| `PUBCOMP` | Reserved | 0 | 0 | 0 | 0 |
| `SUBSCRIBE` | Reserved | 0 | 0 | 1 | 0 |
| `SUBACK` | Reserved | 0 | 0 | 0 | 0 |
| `UNSUBSCRIBE` | Reserved | 0 | 0 | 1 | 0 |
| `UNSUBACK` | Reserved | 0 | 0 | 0 | 0 |
| `PINGREQ` | Reserved | 0 | 0 | 0 | 0 |
| `PINGRESP` | Reserved | 0 | 0 | 0 | 0 |
| `DISCONNECT` | Reserved | 0 | 0 | 0 | 0 |
| `AUTH` | Reserved | 0 | 0 | 0 | 0 |

`DUP` = `PUBLISH` 报文的重复分发

`QoS` = `PUBLISH` 服务质量

`RETAIN` = `PUBLISH` 保留消息标志

有关 `PUBLISH` 报文中 `DUP`、`QoS` 和 `RETAIN` 标志的描述，请参阅第 3.3.1 节。

### 2.1.4 Remaining Length

**位置：** 从 byte 2 开始。

`Remaining Length` 是一个 `Variable Byte Integer`，表示当前 Control Packet 中剩余的字节数，包括 `Variable Header` 和 `Payload` 中的数据。`Remaining Length` 不包括用于编码 `Remaining Length` 本身的字节。报文大小是 MQTT Control Packet 中的总字节数，等于 `Fixed Header` 的长度加上 `Remaining Length`。

## 2.2 Variable Header

某些类型的 MQTT Control Packet 包含一个 `Variable Header` 组件。它位于 `Fixed Header` 和 `Payload` 之间。`Variable Header` 的内容因报文类型而异。`Variable Header` 中的 `Packet Identifier` 字段在多种报文类型中都很常见。

### 2.2.1 Packet Identifier

许多 MQTT Control Packet 类型的 `Variable Header` 组件都包含一个双字节整数 `Packet Identifier` 字段。这些 MQTT Control Packet 包括：`PUBLISH`（当 `QoS` > 0 时）、`PUBACK`、`PUBREC`、`PUBREL`、`PUBCOMP`、`SUBSCRIBE`、`SUBACK`、`UNSUBSCRIBE`、`UNSUBACK`。

需要 `Packet Identifier` 的 MQTT Control Packet 如下所示：

**表 2-3 包含 `Packet Identifier` 的 MQTT Control Packet**

| MQTT Control Packet | `Packet Identifier` 字段 |
|---------------------|-------------------------|
| `CONNECT` | NO |
| `CONNACK` | NO |
| `PUBLISH` | YES（如果 `QoS` > 0） |
| `PUBACK` | YES |
| `PUBREC` | YES |
| `PUBREL` | YES |
| `PUBCOMP` | YES |
| `SUBSCRIBE` | YES |
| `SUBACK` | YES |
| `UNSUBSCRIBE` | YES |
| `UNSUBACK` | YES |
| `PINGREQ` | NO |
| `PINGRESP` | NO |
| `DISCONNECT` | NO |
| `AUTH` | NO |

如果 `PUBLISH` 报文的 `QoS` 值设置为 0，则该报文**禁止**包含 `Packet Identifier` **[MQTT-2.2.1-2]**。

每次 `Client` 发送新的 `SUBSCRIBE`、`UNSUBSCRIBE` 或 `PUBLISH`（当 `QoS` > 0 时）MQTT Control Packet 时，它**必须**为其分配一个当前未使用的非零 `Packet Identifier` **[MQTT-2.2.1-3]**。

每次 `Server` 发送新的 `PUBLISH`（当 `QoS` > 0 时）MQTT Control Packet 时，它**必须**为其分配一个当前未使用的非零 `Packet Identifier` **[MQTT-2.2.1-4]**。

在发送方处理了相应的确认报文后，`Packet Identifier` 变为可重用，定义如下。对于 `QoS` 1 `PUBLISH`，这是相应的 `PUBACK`；对于 `QoS` 2 `PUBLISH`，这是 `PUBCOMP` 或带有大于或等于 128 的 `Reason Code` 的 `PUBREC`。对于 `SUBSCRIBE` 或 `UNSUBSCRIBE`，这是相应的 `SUBACK` 或 `UNSUBACK`。

与 `PUBLISH`、`SUBSCRIBE` 和 `UNSUBSCRIBE` 报文一起使用的 `Packet Identifier` 在 `Session` 中分别为 `Client` 和 `Server` 形成一个统一的标识符集合。一个 `Packet Identifier` 在任何时候都不能被多个命令使用。

`PUBACK`、`PUBREC`、`PUBREL` 或 `PUBCOMP` 报文**必须**包含与最初发送的 `PUBLISH` 报文相同的 `Packet Identifier` **[MQTT-2.2.1-5]**。`SUBACK` 和 `UNSUBACK` **必须**分别包含在相应的 `SUBSCRIBE` 和 `UNSUBSCRIBE` 报文中使用的 `Packet Identifier` **[MQTT-2.2.1-6]**。

`Client` 和 `Server` 彼此独立地分配 `Packet Identifier`。因此，`Client`-`Server` 对可以使用相同的 `Packet Identifier` 参与并发的消息交换。

> **非规范性说明**
>
> `Client` 可能发送一个 `Packet Identifier` 为 0x1234 的 `PUBLISH` 报文，然后在收到其发送的 `PUBLISH` 报文的 `PUBACK` 之前，从其 `Server` 接收到另一个 `Packet Identifier` 为 0x1234 的 `PUBLISH` 报文。

```
Client                                                      Server

PUBLISH Packet Identifier=0x1234 ──>

                                <── PUBLISH Packet Identifier=0x1234

PUBACK Packet Identifier=0x1234 ──>

                                <── PUBACK Packet Identifier=0x1234
```

### 2.2.2 Properties

`CONNECT`、`CONNACK`、`PUBLISH`、`PUBACK`、`PUBREC`、`PUBREL`、`PUBCOMP`、`SUBSCRIBE`、`SUBACK`、`UNSUBSCRIBE`、`UNSUBACK`、`DISCONNECT` 和 `AUTH` 报文的 `Variable Header` 中的最后一个字段是一组 `Properties`。在 `CONNECT` 报文中，`Payload` 中的 Will Properties 字段还有一组可选的 `Properties`。

`Properties` 集合由 Property Length 和随后的 `Properties` 组成。

#### 2.2.2.1 Property Length

Property Length 编码为 `Variable Byte Integer`。Property Length 不包括用于编码自身的字节，但包括 `Properties` 的长度。如果没有属性，这**必须**通过包含值为零的 Property Length 来表示 **[MQTT-2.2.2-1]**。

#### 2.2.2.2 Property

`Property` 由定义其用途和数据类型的 Identifier 及随后的值组成。Identifier 编码为 `Variable Byte Integer`。包含对其报文类型无效的 Identifier 或包含非指定数据类型的值的 Control Packet 是 `Malformed Packet`。如果收到此类报文，请使用带有 `Reason Code` 0x81（`Malformed Packet`）的 `CONNACK` 或 `DISCONNECT` 报文，如第 4.13 节处理错误中所述。具有不同 Identifier 的 `Properties` 的顺序没有意义。

**表 2-4 - Properties**

| Identifier | | Name (usage) | Type | Packet / Will Properties |
|------------|---|--------------|------|-------------------------|
| Dec | Hex | | | |
| 1 | 0x01 | Payload Format Indicator | Byte | `PUBLISH`, Will Properties |
| 2 | 0x02 | Message Expiry Interval | `Four Byte Integer` | `PUBLISH`, Will Properties |
| 3 | 0x03 | Content Type | `UTF-8 Encoded String` | `PUBLISH`, Will Properties |
| 8 | 0x08 | Response Topic | `UTF-8 Encoded String` | `PUBLISH`, Will Properties |
| 9 | 0x09 | Correlation Data | `Binary Data` | `PUBLISH`, Will Properties |
| 11 | 0x0B | Subscription Identifier | `Variable Byte Integer` | `PUBLISH`, `SUBSCRIBE` |
| 17 | 0x11 | Session Expiry Interval | `Four Byte Integer` | `CONNECT`, `CONNACK`, `DISCONNECT` |
| 18 | 0x12 | Assigned Client Identifier | `UTF-8 Encoded String` | `CONNACK` |
| 19 | 0x13 | Server Keep Alive | `Two Byte Integer` | `CONNACK` |
| 21 | 0x15 | Authentication Method | `UTF-8 Encoded String` | `CONNECT`, `CONNACK`, `AUTH` |
| 22 | 0x16 | Authentication Data | `Binary Data` | `CONNECT`, `CONNACK`, `AUTH` |
| 23 | 0x17 | Request Problem Information | Byte | `CONNECT` |
| 24 | 0x18 | Will Delay Interval | `Four Byte Integer` | Will Properties |
| 25 | 0x19 | Request Response Information | Byte | `CONNECT` |
| 26 | 0x1A | Response Information | `UTF-8 Encoded String` | `CONNACK` |
| 28 | 0x1C | Server Reference | `UTF-8 Encoded String` | `CONNACK`, `DISCONNECT` |
| 31 | 0x1F | Reason String | `UTF-8 Encoded String` | `CONNACK`, `PUBACK`, `PUBREC`, `PUBREL`, `PUBCOMP`, `SUBACK`, `UNSUBACK`, `DISCONNECT`, `AUTH` |
| 33 | 0x21 | Receive Maximum | `Two Byte Integer` | `CONNECT`, `CONNACK` |
| 34 | 0x22 | Topic Alias Maximum | `Two Byte Integer` | `CONNECT`, `CONNACK` |
| 35 | 0x23 | Topic Alias | `Two Byte Integer` | `PUBLISH` |
| 36 | 0x24 | Maximum `QoS` | Byte | `CONNACK` |
| 37 | 0x25 | Retain Available | Byte | `CONNACK` |
| 38 | 0x26 | User Property | `UTF-8 String Pair` | `CONNECT`, `CONNACK`, `PUBLISH`, Will Properties, `PUBACK`, `PUBREC`, `PUBREL`, `PUBCOMP`, `SUBSCRIBE`, `SUBACK`, `UNSUBSCRIBE`, `UNSUBACK`, `DISCONNECT`, `AUTH` |
| 39 | 0x27 | Maximum Packet Size | `Four Byte Integer` | `CONNECT`, `CONNACK` |
| 40 | 0x28 | Wildcard Subscription Available | Byte | `CONNACK` |
| 41 | 0x29 | Subscription Identifier Available | Byte | `CONNACK` |
| 42 | 0x2A | Shared Subscription Available | Byte | `CONNACK` |

> **非规范性说明**
>
> 虽然 `Property Identifier` 定义为 `Variable Byte Integer`，但在此版本的规范中，所有 `Property Identifier` 都是一个字节长。

## 2.3 Payload

某些 MQTT Control Packet 包含一个 `Payload` 作为报文的最后部分。在 `PUBLISH` 报文中，这是 `Application Message`。

**表 2-5 - 包含 `Payload` 的 MQTT Control Packet**

| MQTT Control Packet | `Payload` |
|---------------------|-----------|
| `CONNECT` | Required |
| `CONNACK` | None |
| `PUBLISH` | Optional |
| `PUBACK` | None |
| `PUBREC` | None |
| `PUBREL` | None |
| `PUBCOMP` | None |
| `SUBSCRIBE` | Required |
| `SUBACK` | Required |
| `UNSUBSCRIBE` | Required |
| `UNSUBACK` | Required |
| `PINGREQ` | None |
| `PINGRESP` | None |
| `DISCONNECT` | None |
| `AUTH` | None |

## 2.4 Reason Code

`Reason Code` 是一个单字节无符号值，表示操作的结果。小于 0x80 的 `Reason Code` 表示操作成功完成。成功的正常 `Reason Code` 是 0。值为 0x80 或更高的 `Reason Code` 表示失败。

`CONNACK`、`PUBACK`、`PUBREC`、`PUBREL`、`PUBCOMP`、`DISCONNECT` 和 `AUTH` Control Packet 在 `Variable Header` 中有一个单独的 `Reason Code`。`SUBACK` 和 `UNSUBACK` 报文在 `Payload` 中包含一个或多个 `Reason Code` 列表。

`Reason Code` 共享一组公共值，如下所示。

**表 2-6 - Reason Codes**

| `Reason Code` | | Name | Packets |
|---------------|---|------|---------|
| Decimal | Hex | | |
| 0 | 0x00 | Success | `CONNACK`, `PUBACK`, `PUBREC`, `PUBREL`, `PUBCOMP`, `UNSUBACK`, `AUTH` |
| 0 | 0x00 | Normal disconnection | `DISCONNECT` |
| 0 | 0x00 | Granted `QoS` 0 | `SUBACK` |
| 1 | 0x01 | Granted `QoS` 1 | `SUBACK` |
| 2 | 0x02 | Granted `QoS` 2 | `SUBACK` |
| 4 | 0x04 | Disconnect with Will Message | `DISCONNECT` |
| 16 | 0x10 | No matching subscribers | `PUBACK`, `PUBREC` |
| 17 | 0x11 | No subscription existed | `UNSUBACK` |
| 24 | 0x18 | Continue authentication | `AUTH` |
| 25 | 0x19 | Re-authenticate | `AUTH` |
| 128 | 0x80 | Unspecified error | `CONNACK`, `PUBACK`, `PUBREC`, `SUBACK`, `UNSUBACK`, `DISCONNECT` |
| 129 | 0x81 | `Malformed Packet` | `CONNACK`, `DISCONNECT` |
| 130 | 0x82 | Protocol Error | `CONNACK`, `DISCONNECT` |
| 131 | 0x83 | Implementation specific error | `CONNACK`, `PUBACK`, `PUBREC`, `SUBACK`, `UNSUBACK`, `DISCONNECT` |
| 132 | 0x84 | Unsupported Protocol Version | `CONNACK` |
| 133 | 0x85 | Client Identifier not valid | `CONNACK` |
| 134 | 0x86 | Bad User Name or Password | `CONNACK` |
| 135 | 0x87 | Not authorized | `CONNACK`, `PUBACK`, `PUBREC`, `SUBACK`, `UNSUBACK`, `DISCONNECT` |
| 136 | 0x88 | Server unavailable | `CONNACK` |
| 137 | 0x89 | Server busy | `CONNACK`, `DISCONNECT` |
| 138 | 0x8A | Banned | `CONNACK` |
| 139 | 0x8B | Server shutting down | `DISCONNECT` |
| 140 | 0x8C | Bad authentication method | `CONNACK`, `DISCONNECT` |
| 141 | 0x8D | Keep Alive timeout | `DISCONNECT` |
| 142 | 0x8E | `Session` taken over | `DISCONNECT` |
| 143 | 0x8F | Topic Filter invalid | `SUBACK`, `UNSUBACK`, `DISCONNECT` |
| 144 | 0x90 | Topic Name invalid | `CONNACK`, `PUBACK`, `PUBREC`, `DISCONNECT` |
| 145 | 0x91 | `Packet Identifier` in use | `PUBACK`, `PUBREC`, `SUBACK`, `UNSUBACK` |
| 146 | 0x92 | `Packet Identifier` not found | `PUBREL`, `PUBCOMP` |
| 147 | 0x93 | Receive Maximum exceeded | `DISCONNECT` |
| 148 | 0x94 | Topic Alias invalid | `DISCONNECT` |
| 149 | 0x95 | Packet too large | `CONNACK`, `DISCONNECT` |
| 150 | 0x96 | Message rate too high | `DISCONNECT` |
| 151 | 0x97 | Quota exceeded | `CONNACK`, `PUBACK`, `PUBREC`, `SUBACK`, `DISCONNECT` |
| 152 | 0x98 | Administrative action | `DISCONNECT` |
| 153 | 0x99 | `Payload` format invalid | `CONNACK`, `PUBACK`, `PUBREC`, `DISCONNECT` |
| 154 | 0x9A | Retain not supported | `CONNACK`, `DISCONNECT` |
| 155 | 0x9B | `QoS` not supported | `CONNACK`, `DISCONNECT` |
| 156 | 0x9C | Use another server | `CONNACK`, `DISCONNECT` |
| 157 | 0x9D | Server moved | `CONNACK`, `DISCONNECT` |
| 158 | 0x9E | Shared Subscriptions not supported | `SUBACK`, `DISCONNECT` |
| 159 | 0x9F | Connection rate exceeded | `CONNACK`, `DISCONNECT` |
| 160 | 0xA0 | Maximum connect time | `DISCONNECT` |
| 161 | 0xA1 | Subscription Identifiers not supported | `SUBACK`, `DISCONNECT` |
| 162 | 0xA2 | Wildcard Subscriptions not supported | `SUBACK`, `DISCONNECT` |

> **非规范性说明**
>
> 对于 `Reason Code` 0x91（`Packet Identifier` in use），响应是尝试修复状态，或者通过使用 Clean Start 设置为 1 进行连接来重置 `Session` 状态，或者确定 `Client` 或 `Server` 实现是否存在缺陷。