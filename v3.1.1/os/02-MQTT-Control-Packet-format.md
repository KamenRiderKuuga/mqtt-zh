# 2 MQTT Control Packet format

## 2.1 Structure of an MQTT Control Packet

MQTT 协议通过以定义的方式交换一系列 MQTT Control Packet 来工作。本节描述这些报文的格式。

一个 MQTT Control Packet 最多由三部分组成，始终按以下顺序排列，如 **图 2.1 - MQTT Control Packet 的结构** 所示。

**图 2.1 - MQTT Control Packet 的结构**

| 固定报头（Fixed header），存在于所有 MQTT Control Packet 中 |
|--------------------------------------------------------|
| 可变报头（Variable header），存在于部分 MQTT Control Packet 中 |
| 有效载荷（Payload），存在于部分 MQTT Control Packet 中 |

## 2.2 Fixed header

每个 MQTT Control Packet 都包含一个固定报头。**图 2.2 - 固定报头格式** 展示了固定报头的格式。

**图 2.2 - 固定报头格式**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet 类型 | | | | 每种 MQTT Control Packet 类型特定的标志 |
| byte 2 | 剩余长度（Remaining Length） | | | | | | |

### 2.2.1 MQTT Control Packet type

**位置：** 第 1 字节，第 7-4 位。

表示为一个 4 位无符号值，这些值列于 **表 2.1 - 控制报文类型** 中。

**表 2.1 - 控制报文类型**

| 名称 | 值 | 流向 | 描述 |
|------|----|-----------|------|
| Reserved | 0 | 禁止 | 保留 |
| CONNECT | 1 | Client to Server | 客户端请求连接到服务端 |
| CONNACK | 2 | Server to Client | 连接确认 |
| PUBLISH | 3 | Client to Server 或 Server to Client | 发布消息 |
| PUBACK | 4 | Client to Server 或 Server to Client | 发布确认 |
| PUBREC | 5 | Client to Server 或 Server to Client | 发布已接收（可靠投递第 1 部分） |
| PUBREL | 6 | Client to Server 或 Server to Client | 发布释放（可靠投递第 2 部分） |
| PUBCOMP | 7 | Client to Server 或 Server to Client | 发布完成（可靠投递第 3 部分） |
| SUBSCRIBE | 8 | Client to Server | 客户端订阅请求 |
| SUBACK | 9 | Server to Client | 订阅确认 |
| UNSUBSCRIBE | 10 | Client to Server | 取消订阅请求 |
| UNSUBACK | 11 | Server to Client | 取消订阅确认 |
| PINGREQ | 12 | Client to Server | PING 请求 |
| PINGRESP | 13 | Server to Client | PING 响应 |
| DISCONNECT | 14 | Client to Server | 客户端正在断开连接 |
| Reserved | 15 | 禁止 | 保留 |

### 2.2.2 Flags

固定报头第 1 字节的剩余位 [3-0] 包含每种 MQTT Control Packet 类型特定的标志，如下面 **表 2.2 - 标志位** 所列。当表 2.2 中的某个标志位被标记为"保留"时，它保留供将来使用，**必须设置为该表中列出的值** [MQTT-2.2.2-1]。**如果收到无效的标志，接收方必须关闭网络连接** [MQTT-2.2.2-2]。有关处理错误的详细信息，请参阅第 4.8 节。

**表 2.2 - 标志位**

| 控制报文 | 固定报头标志 | Bit 3 | Bit 2 | Bit 1 | Bit 0 |
|----------|--------------|-------|-------|-------|-------|
| CONNECT | Reserved | 0 | 0 | 0 | 0 |
| CONNACK | Reserved | 0 | 0 | 0 | 0 |
| PUBLISH | Used in MQTT 3.1.1 | DUP | QoS | QoS | RETAIN |
| PUBACK | Reserved | 0 | 0 | 0 | 0 |
| PUBREC | Reserved | 0 | 0 | 0 | 0 |
| PUBREL | Reserved | 0 | 0 | 1 | 0 |
| PUBCOMP | Reserved | 0 | 0 | 0 | 0 |
| SUBSCRIBE | Reserved | 0 | 0 | 1 | 0 |
| SUBACK | Reserved | 0 | 0 | 0 | 0 |
| UNSUBSCRIBE | Reserved | 0 | 0 | 1 | 0 |
| UNSUBACK | Reserved | 0 | 0 | 0 | 0 |
| PINGREQ | Reserved | 0 | 0 | 0 | 0 |
| PINGRESP | Reserved | 0 | 0 | 0 | 0 |
| DISCONNECT | Reserved | 0 | 0 | 0 | 0 |

DUP = PUBLISH Control Packet 的重复投递

QoS = PUBLISH 服务质量

RETAIN = PUBLISH 保留标志

有关 PUBLISH Control Packet 中 DUP、QoS 和 RETAIN 标志的描述，请参阅第 3.3.1 节。

### 2.2.3 Remaining Length

**位置：** 从第 2 字节开始。

剩余长度表示当前报文中剩余的字节数，包括可变报头和有效载荷中的数据。剩余长度不包括用于编码剩余长度本身的字节数。

剩余长度使用变长编码方案进行编码，对于不超过 127 的值使用单个字节。较大的值按以下方式处理：每个字节的低 7 位编码数据，最高位用于指示表示中是否还有后续字节。因此，每个字节编码 128 个值和一个"继续位"。剩余长度字段的最大字节数为 4。

> **非规范性说明**
>
> 例如，十进制数 64 编码为单个字节，十进制值 64，十六进制 0x40。十进制数 321（= 65 + 2*128）编码为两个字节，最低有效字节在前。第一个字节是 65+128 = 193。请注意，设置了最高位以指示至少有一个后续字节。第二个字节是 2。

> **非规范性说明**
>
> 这允许应用程序发送大小最大为 268,435,455（256 MB）的控制报文。此数字在线上的表示为：0xFF, 0xFF, 0xFF, 0x7F。

表 2.4 显示了由递增字节数表示的剩余长度值。

**表 2.4 剩余长度字段的大小**

| 字节数 | 从 | 到 |
|--------|----|----|
| 1 | 0 (0x00) | 127 (0x7F) |
| 2 | 128 (0x80, 0x01) | 16 383 (0xFF, 0x7F) |
| 3 | 16 384 (0x80, 0x80, 0x01) | 2 097 151 (0xFF, 0xFF, 0x7F) |
| 4 | 2 097 152 (0x80, 0x80, 0x80, 0x01) | 268 435 455 (0xFF, 0xFF, 0xFF, 0x7F) |

> **非规范性说明**
>
> 将非负整数（X）编码为变长编码方案的算法如下：
>
> ```
> do
>     encodedByte = X MOD 128
>     X = X DIV 128
>     // if there are more data to encode, set the top bit of this byte
>     if ( X > 0 )
>         encodedByte = encodedByte OR 128
>     endif
>     'output' encodedByte
> while ( X > 0 )
> ```
>
> 其中 `MOD` 是取模运算符（C 语言中的 `%`），`DIV` 是整数除法（C 语言中的 `/`），`OR` 是按位或运算符（C 语言中的 `|`）。

> **非规范性说明**
>
> 解码剩余长度字段的算法如下：
>
> ```
> multiplier = 1
> value = 0
> do
>     encodedByte = 'next byte from stream'
>     value += (encodedByte AND 127) * multiplier
>     if (multiplier > 128*128*128)
>         throw Error(Malformed Remaining Length)
>     multiplier *= 128
> while ((encodedByte AND 128) != 0)
> ```
>
> 其中 `AND` 是按位与运算符（C 语言中的 `&`）。

当此算法终止时，`value` 包含剩余长度值。

## 2.3 Variable header

某些类型的 MQTT Control Packet 包含一个可变报头组件。它位于固定报头和有效载荷之间。可变报头的内容因报文类型而异。可变报头的报文标识符字段在多种报文类型中都很常见。

### 2.3.1 Packet Identifier

**图 2.3 - 报文标识符字节**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | 报文标识符 MSB | | | | | | |
| byte 2 | 报文标识符 LSB | | | | | | |

许多 Control Packet 类型的可变报头组件包含一个 2 字节的报文标识符字段。这些 Control Packet 包括：PUBLISH（当 QoS > 0 时）、PUBACK、PUBREC、PUBREL、PUBCOMP、SUBSCRIBE、SUBACK、UNSUBSCRIBE、UNSUBACK。

**SUBSCRIBE、UNSUBSCRIBE 和 PUBLISH（在 QoS > 0 的情况下）Control Packet 必须包含一个非零的 16 位报文标识符** [MQTT-2.3.1-1]。**每次客户端发送这些类型的新报文时，必须为其分配一个当前未使用的报文标识符** [MQTT-2.3.1-2]。**如果客户端重新发送特定的 Control Packet，则必须在后续重发中使用相同的报文标识符。在客户端处理完相应的确认报文后，报文标识符变为可重用。对于 QoS 1 PUBLISH，这是相应的 PUBACK；对于 QoS 2，是 PUBCOMP。对于 SUBSCRIBE 或 UNSUBSCRIBE，是相应的 SUBACK 或 UNSUBACK** [MQTT-2.3.1-3]。**当服务端发送 QoS > 0 的 PUBLISH 时，同样的条件适用** [MQTT-2.3.1-4]。

**如果 PUBLISH 报文的 QoS 值设置为 0，则该报文不得包含报文标识符** [MQTT-2.3.1-5]。

**PUBACK、PUBREC 或 PUBREL 报文必须包含与最初发送的 PUBLISH 报文相同的报文标识符** [MQTT-2.3.1-6]。**同样，SUBACK 和 UNSUBACK 必须分别包含相应的 SUBSCRIBE 和 UNSUBSCRIBE 报文中使用的报文标识符** [MQTT-2.3.1-7]。

需要报文标识符的 Control Packet 列于 **表 2.5 - 包含报文标识符的控制报文** 中。

**表 2.5 - 包含报文标识符的控制报文**

| 控制报文 | 报文标识符字段 |
|----------|----------------|
| CONNECT | NO |
| CONNACK | NO |
| PUBLISH | YES（如果 QoS > 0） |
| PUBACK | YES |
| PUBREC | YES |
| PUBREL | YES |
| PUBCOMP | YES |
| SUBSCRIBE | YES |
| SUBACK | YES |
| UNSUBSCRIBE | YES |
| UNSUBACK | YES |
| PINGREQ | NO |
| PINGRESP | NO |
| DISCONNECT | NO |

客户端和服务端独立分配报文标识符。因此，客户端-服务端对可以使用相同的报文标识符参与并发的消息交换。

> **非规范性说明**
>
> 客户端可能会发送一个报文标识符为 0x1234 的 PUBLISH 报文，然后在收到其发送的 PUBLISH 的 PUBACK 之前，从服务端接收到另一个报文标识符为 0x1234 的 PUBLISH。
>
> ```
>   Client                      Server
>   PUBLISH Packet Identifier=0x1234--->
>   <---PUBLISH Packet Identifier=0x1234
>   PUBACK Packet Identifier=0x1234--->
>   <---PUBACK Packet Identifier=0x1234
> ```

## 2.4 Payload

某些 MQTT Control Packet 包含一个有效载荷作为报文的最后部分，如第 3 章所述。对于 PUBLISH 报文，这就是应用消息。**表 2.6 - 包含有效载荷的控制报文** 列出了需要有效载荷的控制报文。

**表 2.6 - 包含有效载荷的控制报文**

| 控制报文 | 有效载荷 |
|----------|----------|
| CONNECT | 必需 |
| CONNACK | 无 |
| PUBLISH | 可选 |
| PUBACK | 无 |
| PUBREC | 无 |
| PUBREL | 无 |
| PUBCOMP | 无 |
| SUBSCRIBE | 必需 |
| SUBACK | 必需 |
| UNSUBSCRIBE | 必需 |
| UNSUBACK | 无 |
| PINGREQ | 无 |
| PINGRESP | 无 |
| DISCONNECT | 无 |