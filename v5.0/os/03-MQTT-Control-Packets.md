# 3 MQTT Control Packets

## 3.1 CONNECT - Connection Request

在 `Client` 与 `Server` 建立网络连接后，`Client` 发送给 `Server` 的第一个报文必须是 `CONNECT` 报文 [MQTT-3.1.0-1]。

`Client` 只能通过网络连接发送一次 `CONNECT` 报文。`Server` 必须将 `Client` 发送的第二个 `CONNECT` 报文视为协议错误并关闭网络连接 [MQTT-3.1.2-2]。有关错误处理的信息，请参阅 [第 4.13 节](#4-13-错误处理)。

`Payload` 包含一个或多个编码字段。它们指定了 `Client` 的唯一 `Client Identifier`、`Will Topic`、Will Payload、`User Name` 和 `Password`。除了 `Client Identifier` 之外，其他字段都可以省略，它们是否存在由 `Variable Header` 中的标志决定。

### 3.1.1 CONNECT `Fixed Header`

**图 3-1 - `CONNECT` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (1) | Reserved |
| | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | Remaining Length |

**`Remaining Length` 字段**

这是 `Variable Header` 的长度加上 `Payload` 的长度。它被编码为 `Variable Byte Integer`。

### 3.1.2 CONNECT `Variable Header`

`CONNECT` 报文的 `Variable Header` 按顺序包含以下字段：Protocol Name、Protocol Level、Connect Flags、`Keep Alive` 和 Properties。属性编码规则在 [第 2.2.2 节](#222-properties) 中描述。

#### 3.1.2.1 Protocol Name

**图 3-2 - Protocol Name 字节**

| | Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|---|
| Protocol Name |
| byte 1 | Length MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | Length LSB (4) | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| byte 3 | 'M' | 0 | 1 | 0 | 0 | 1 | 1 | 0 | 1 |
| byte 4 | 'Q' | 0 | 1 | 0 | 1 | 0 | 0 | 0 | 1 |
| byte 5 | 'T' | 0 | 1 | 0 | 1 | 0 | 1 | 0 | 0 |
| byte 6 | 'T' | 0 | 1 | 0 | 1 | 0 | 1 | 0 | 0 |

Protocol Name 是一个 `UTF-8 Encoded String`，表示协议名称 "MQTT"，按所示大写。该字符串、其偏移量和长度不会因 MQTT 规范的未来版本而改变。

支持多种协议的 `Server` 使用 Protocol Name 来确定数据是否为 MQTT。协议名称必须是 UTF-8 字符串 "MQTT"。如果 `Server` 不想接受 `CONNECT`，并希望表明它是一个 MQTT `Server`，它可以发送一个 `Reason Code` 为 0x84（不支持的协议版本）的 `CONNACK` 报文，然后必须关闭网络连接 [MQTT-3.1.2-1]。

> **非规范性注释**
>
> 报文检查器（如防火墙）可以使用 Protocol Name 来识别 MQTT 流量。

#### 3.1.2.2 Protocol Version

**图 3-3 - Protocol Version 字节**

| | Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|---|
| Protocol Level |
| byte 7 | Version (5) | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 1 |

表示 `Client` 使用的协议修订级别的单字节无符号值。协议版本 5.0 的 Protocol Version 字段值为 5 (0x05)。

支持多个 MQTT 协议版本的 `Server` 使用 Protocol Version 来确定 `Client` 使用的 MQTT 版本。如果 Protocol Version 不是 5，并且 `Server` 不想接受 `CONNECT` 报文，`Server` 可以发送一个 `Reason Code` 为 0x84（不支持的协议版本）的 `CONNACK` 报文，然后必须关闭网络连接 [MQTT-3.1.2-2]。

#### 3.1.2.3 Connect Flags

Connect Flags 字节包含多个指定 MQTT 连接行为的参数。它还指示 `Payload` 中字段的存在或缺失。

**图 3-4 - Connect Flag 位**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| | User Name Flag | Password Flag | Will Retain | Will QoS | Will Flag | Clean Start | Reserved |
| byte 8 | X | X | X | X | X | X | X | 0 |

`Server` 必须验证 `CONNECT` 报文中的保留标志设置为 0 [MQTT-3.1.2-3]。如果保留标志不为 0，则为格式错误报文（Malformed Packet）。有关错误处理的信息，请参阅 [第 4.13 节](#4-13-错误处理)。

#### 3.1.2.4 Clean Start

**位置：** Connect Flags 字节的第 1 位。

此位指定连接是开始新会话还是继续现有会话。有关 `Session` State 的定义，请参阅 [第 4.1 节](#41-session-state)。

如果收到 `Clean Start` 设置为 1 的 `CONNECT` 报文，`Client` 和 `Server` 必须丢弃任何现有会话并开始新会话 [MQTT-3.1.2-4]。因此，如果 `Clean Start` 设置为 1，`CONNACK` 中的 `Session` Present 标志始终设置为 0。

如果收到 `Clean Start` 设置为 0 的 `CONNECT` 报文，并且存在与 `Client Identifier` 关联的会话，`Server` 必须基于现有会话的状态与 `Client` 恢复通信 [MQTT-3.1.2-5]。如果收到 `Clean Start` 设置为 0 的 `CONNECT` 报文，并且不存在与 `Client Identifier` 关联的会话，`Server` 必须创建新会话 [MQTT-3.1.2-6]。

#### 3.1.2.5 Will Flag

**位置：** Connect Flags 的第 2 位。

如果 Will Flag 设置为 1，表示必须在 `Server` 上存储 `Will Message` 并将其与会话关联 [MQTT-3.1.2-7]。`Will Message` 由 `CONNECT` `Payload` 中的 Will Properties、`Will Topic` 和 Will Payload 字段组成。必须在网络连接随后关闭且 `Will Delay Interval` 已过或会话结束后发布 `Will Message`，除非 `Server` 在收到 `Reason Code` 为 0x00（正常断开连接）的 `DISCONNECT` 报文时删除了 `Will Message`，或者在 `Will Delay Interval` 过期之前为 ClientID 打开了新的网络连接 [MQTT-3.1.2-8]。

`Will Message` 发布的情况包括但不限于：

- `Server` 检测到的 I/O 错误或网络故障。
- `Client` 未能在 `Keep Alive` 时间内通信。
- `Client` 关闭网络连接而未先发送 `Reason Code` 为 0x00（正常断开连接）的 `DISCONNECT` 报文。
- `Server` 关闭网络连接而未先收到 `Reason Code` 为 0x00（正常断开连接）的 `DISCONNECT` 报文。

如果 Will Flag 设置为 1，`Payload` 中必须包含 Will Properties、`Will Topic` 和 Will Payload 字段 [MQTT-3.1.2-9]。一旦 `Will Message` 被发布或 `Server` 从 `Client` 收到 `Reason Code` 为 0x00（正常断开连接）的 `DISCONNECT` 报文，必须从 `Server` 存储的 `Session` State 中删除 `Will Message` [MQTT-3.1.2-10]。

`Server` 应该在网络连接关闭且 `Will Delay Interval` 过去后，或者在会话结束时（以先发生者为准），及时发布 Will Messages。在 `Server` 关闭或故障的情况下，`Server` 可以推迟 Will Messages 的发布直到后续重新启动。如果发生这种情况，从 `Server` 故障到 `Will Message` 发布之间可能会有延迟。

有关 Will Delay Interval 的信息，请参阅 [第 3.1.3.2 节](#3132-will-delay-interval)。

> **非规范性注释**
>
> `Client` 可以通过将 `Will Delay Interval` 设置为长于 `Session Expiry Interval` 并发送 `Reason Code` 为 0x04（带 `Will Message` 断开连接）的 `DISCONNECT` 来安排 `Will Message` 通知会话过期已发生。

#### 3.1.2.6 Will QoS

**位置：** Connect Flags 的第 4 位和第 3 位。

这两位指定发布 `Will Message` 时使用的 `QoS` 级别。

如果 Will Flag 设置为 0，则 Will `QoS` 必须设置为 0 (0x00) [MQTT-3.1.2-11]。

如果 Will Flag 设置为 1，Will `QoS` 的值可以是 0 (0x00)、1 (0x01) 或 2 (0x02) [MQTT-3.1.2-12]。值为 3 (0x03) 是格式错误报文（Malformed Packet）。有关错误处理的信息，请参阅 [第 4.13 节](#4-13-错误处理)。

#### 3.1.2.7 Will Retain

**位置：** Connect Flags 的第 5 位。

此位指定 `Will Message` 发布时是否保留。

如果 Will Flag 设置为 0，则 Will Retain 必须设置为 0 [MQTT-3.1.2-13]。如果 Will Flag 设置为 1 且 Will Retain 设置为 0，`Server` 必须将 `Will Message` 作为非保留消息发布 [MQTT-3.1.2-14]。如果 Will Flag 设置为 1 且 Will Retain 设置为 1，`Server` 必须将 `Will Message` 作为保留消息发布 [MQTT-3.1.2-15]。

#### 3.1.2.8 User Name Flag

**位置：** Connect Flags 的第 7 位。

如果 User Name Flag 设置为 0，`Payload` 中禁止出现 `User Name` [MQTT-3.1.2-16]。如果 User Name Flag 设置为 1，`Payload` 中必须存在 `User Name` [MQTT-3.1.2-17]。

#### 3.1.2.9 Password Flag

**位置：** Connect Flags 的第 6 位。

如果 Password Flag 设置为 0，`Payload` 中禁止出现 `Password` [MQTT-3.1.2-18]。如果 Password Flag 设置为 1，`Payload` 中必须存在 `Password` [MQTT-3.1.2-19]。

> **非规范性注释**
>
> 此版本的协议允许在没有 `User Name` 的情况下发送 `Password`，而 MQTT v3.1.1 不允许。这反映了 `Password` 通常用于密码以外的凭据的常见用法。

#### 3.1.2.10 `Keep Alive`

**图 3-5 - `Keep Alive` 字节**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 9 | `Keep Alive` MSB |
| byte 10 | `Keep Alive` LSB |

`Keep Alive` 是一个双字节整数，是以秒为单位的时间间隔。它是允许从 `Client` 完成传输一个 MQTT Control Packet 到开始发送下一个 MQTT Control Packet 之间经过的最大时间间隔。`Client` 有责任确保发送 MQTT Control Packets 之间的间隔不超过 `Keep Alive` 值。如果 `Keep Alive` 非零且没有发送任何其他 MQTT Control Packets，`Client` 必须发送 `PINGREQ` 报文 [MQTT-3.1.2-20]。

如果 `Server` 在 `CONNACK` 报文上返回 Server Keep Alive，`Client` 必须使用该值而不是它作为 `Keep Alive` 发送的值 [MQTT-3.1.2-21]。

`Client` 可以在任何时间发送 `PINGREQ`，无论 `Keep Alive` 值如何，并检查相应的 `PINGRESP` 以确定网络和 `Server` 可用。

如果 `Keep Alive` 值非零，并且 `Server` 在 `Keep Alive` 时间段的一倍半时间内没有从 `Client` 收到 MQTT Control Packet，它必须关闭与 `Client` 的网络连接，就像网络故障一样 [MQTT-3.1.2-22]。

如果 `Client` 在发送 `PINGREQ` 后在合理时间内没有收到 `PINGRESP` 报文，它应该关闭与 `Server` 的网络连接。

`Keep Alive` 值为 0 具有关闭 `Keep Alive` 机制的效果。如果 `Keep Alive` 为 0，`Client` 不必按任何特定时间表发送 MQTT Control Packets。

> **非规范性注释**
>
> `Server` 可能有其他原因断开 `Client` 连接，例如因为它正在关闭。设置 `Keep Alive` 并不保证 `Client` 将保持连接。

> **非规范性注释**
>
> `Keep Alive` 的实际值是特定于应用程序的；通常，这是几分钟。最大值 65,535 是 18 小时 12 分钟 15 秒。

#### 3.1.2.11 CONNECT Properties

##### 3.1.2.11.1 Property Length

`CONNECT` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.1.2.11.2 `Session Expiry Interval`

**17 (0x11) 字节**，`Session Expiry Interval` 的标识符。

后跟表示 `Session Expiry Interval`（以秒为单位）的四字节整数。多次包含 `Session Expiry Interval` 是协议错误。

如果 `Session Expiry Interval` 缺失，则使用值 0。如果设置为 0 或缺失，会话在网络连接关闭时结束。

如果 `Session Expiry Interval` 为 0xFFFFFFFF (UINT_MAX)，会话不会过期。

如果 `Session Expiry Interval` 大于 0，`Client` 和 `Server` 必须在网络连接关闭后存储 `Session` State [MQTT-3.1.2-23]。

> **非规范性注释**
>
> `Client` 或 `Server` 中的时钟可能在部分时间间隔内没有运行，例如因为 `Client` 或 `Server` 没有运行。这可能导致状态删除被延迟。

有关会话的更多信息，请参阅 [第 4.1 节](#41-session-state)。有关存储状态的详细信息和限制，请参阅 [第 4.1.1 节](#411-storing-session-state)。

当会话过期时，`Client` 和 `Server` 不需要原子地处理状态删除。

> **非规范性注释**
>
> 将 `Clean Start` 设置为 1 并将 `Session Expiry Interval` 设置为 0，相当于在 MQTT 规范版本 3.1.1 中将 CleanSession 设置为 1。将 `Clean Start` 设置为 0 且没有 `Session Expiry Interval`，相当于在 MQTT 规范版本 3.1.1 中将 CleanSession 设置为 0。

> **非规范性注释**
>
> 只想在连接时处理消息的 `Client` 将把 `Clean Start` 设置为 1 并将 `Session Expiry Interval` 设置为 0。它不会收到在连接之前发布的 Application Messages，并且每次连接时必须重新订阅它感兴趣的任何主题。

> **非规范性注释**
>
> `Client` 可能使用提供间歇连接的网络连接到 `Server`。此 `Client` 可以使用较短的 `Session Expiry Interval`，以便在网络再次可用时可以重新连接并继续可靠的消息传递。如果 `Client` 没有重新连接，允许会话过期，则 Application Messages 将丢失。

> **非规范性注释**
>
> 当 `Client` 以较长的 `Session Expiry Interval` 连接时，它请求 `Server` 在断开连接后长时间维护其 MQTT 会话状态。`Client` 只有打算在稍后的某个时间点重新连接到 `Server` 时，才应该以较长的 `Session Expiry Interval` 连接。当 `Client` 确定它不再使用该会话时，它应该以 `Session Expiry Interval` 设置为 0 断开连接。

> **非规范性注释**
>
> `Client` 应该始终使用 `CONNACK` 中的 `Session` Present 标志来确定 `Server` 是否具有此 `Client` 的 `Session` State。

> **非规范性注释**
>
> `Client` 可以避免实现自己的会话过期，而是依赖从 `Server` 返回的 `Session` Present 标志来确定会话是否已过期。如果 `Client` 确实实现了自己的会话过期，它需要将会话状态将被删除的时间作为其 `Session` State 的一部分存储。

##### 3.1.2.11.3 `Receive Maximum`

**33 (0x21) 字节**，`Receive Maximum` 的标识符。

后跟表示 `Receive Maximum` 值的双字节整数。多次包含 `Receive Maximum` 值或其值为 0 是协议错误。

`Client` 使用此值来限制它愿意同时处理的 `QoS` 1 和 `QoS` 2 发布的数量。没有机制来限制 `Server` 可能尝试发送的 `QoS` 0 发布。

`Receive Maximum` 的值仅适用于当前网络连接。如果 `Receive Maximum` 值缺失，则其值默认为 65,535。

有关如何使用 `Receive Maximum` 的详细信息，请参阅 [第 4.9 节](#49-flow-control)流控制。

##### 3.1.2.11.4 `Maximum Packet Size`

**39 (0x27) 字节**，`Maximum Packet Size` 的标识符。

后跟表示 `Client` 愿意接受的最大报文大小的四字节整数。如果 `Maximum Packet Size` 不存在，则除了由于剩余长度编码和协议头大小而在协议中产生的限制外，不对报文大小施加限制。

多次包含 `Maximum Packet Size` 或值设置为零是协议错误。

> **非规范性注释**
>
> 如果应用程序选择限制 `Maximum Packet Size`，则由应用程序负责选择合适的 `Maximum Packet Size` 值。

报文大小是 MQTT Control Packet 中的总字节数，如 [第 2.1.4 节](#214-remaining-length) 中所定义。`Client` 使用 `Maximum Packet Size` 通知 `Server` 它不会处理超过此限制的报文。

`Server` 禁止向 `Client` 发送超过 `Maximum Packet Size` 的报文 [MQTT-3.1.2-24]。如果 `Client` 收到超过此限制的报文，这是协议错误，`Client` 使用 `Reason Code` 为 0x95（报文过大）的 `DISCONNECT`，如 [第 4.13 节](#4-13-错误处理) 所述。

当报文太大无法发送时，`Server` 必须丢弃它而不发送，然后表现得好像它已经完成发送该 Application Message [MQTT-3.1.2-25]。

在 Shared Subscription 的情况下，如果消息太大无法发送给一个或多个 `Client`，但其他 `Client` 可以接收它，`Server` 可以选择丢弃消息而不向任何 `Client` 发送消息，或者向可以接收它的 `Client` 之一发送消息。

> **非规范性注释**
>
> 当报文被丢弃而不发送时，`Server` 可以将丢弃的报文放在"死信队列"上或执行其他诊断操作。此类操作不在本规范的范围内。

##### 3.1.2.11.5 `Topic Alias Maximum`

**34 (0x22) 字节**，`Topic Alias Maximum` 的标识符。

后跟表示 `Topic Alias Maximum` 值的双字节整数。多次包含 `Topic Alias Maximum` 值是协议错误。如果 `Topic Alias Maximum` 属性缺失，默认值为 0。

此值表示 `Client` 将接受的 `Server` 发送的 `Topic Alias` 的最大值。`Client` 使用此值来限制它愿意在此连接上持有的 `Topic Alias` 的数量。`Server` 禁止在 `PUBLISH` 报文中向 `Client` 发送大于 `Topic Alias Maximum` 的 `Topic Alias` [MQTT-3.1.2-26]。值为 0 表示 `Client` 不接受此连接上的任何 `Topic Alias`。如果 `Topic Alias Maximum` 缺失或为零，`Server` 禁止向 `Client` 发送任何 `Topic Alias` [MQTT-3.1.2-27]。

##### 3.1.2.11.6 Request Response Information

**25 (0x19) 字节**，Request Response Information 的标识符。

后跟值为 0 或 1 的字节。多次包含 Request Response Information 或值不是 0 或 1 是协议错误。如果 Request Response Information 缺失，则使用值 0。

`Client` 使用此值请求 `Server` 在 `CONNACK` 中返回 `Response Information`。值为 0 表示 `Server` 禁止返回 `Response Information` [MQTT-3.1.2-28]。如果值为 1，`Server` 可以在 `CONNACK` 报文中返回 `Response Information`。

> **非规范性注释**
>
> 即使 `Client` 请求，`Server` 也可以选择不在 `CONNACK` 中包含 `Response Information`。

有关请求/响应的更多信息，请参阅 [第 4.10 节](#410-requestresponse)。

##### 3.1.2.11.7 Request Problem Information

**23 (0x17) 字节**，Request Problem Information 的标识符。

后跟值为 0 或 1 的字节。多次包含 Request Problem Information 或值不是 0 或 1 是协议错误。如果 Request Problem Information 缺失，则使用值 1。

`Client` 使用此值指示在失败情况下是否发送 Reason String 或 `User Properties`。

如果 Request Problem Information 的值为 0，`Server` 可以在 `CONNACK` 或 `DISCONNECT` 报文上返回 Reason String 或 `User Properties`，但禁止在除 `PUBLISH`、`CONNACK` 或 `DISCONNECT` 以外的任何报文上发送 Reason String 或 `User Properties` [MQTT-3.1.2-29]。如果值为 0 且 `Client` 在除 `PUBLISH`、`CONNACK` 或 `DISCONNECT` 以外的报文中收到 Reason String 或 `User Properties`，它使用 `Reason Code` 为 0x82（协议错误）的 `DISCONNECT` 报文，如 [第 4.13 节](#4-13-错误处理)处理错误中所述。

如果此值为 1，`Server` 可以在允许的任何报文上返回 Reason String 或 `User Properties`。

##### 3.1.2.11.8 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。

允许 `User Property` 多次出现以表示多个名称-值对。允许同一名称多次出现。

> **非规范性注释**
>
> `CONNECT` 报文上的 `User Properties` 可用于从 `Client` 向 `Server` 发送与连接相关的属性。这些属性的含义不由本规范定义。

##### 3.1.2.11.9 `Authentication Method`

**21 (0x15) 字节**，`Authentication Method` 的标识符。

后跟包含用于扩展认证的认证方法名称的 `UTF-8 Encoded String`。多次包含 `Authentication Method` 是协议错误。

如果 `Authentication Method` 缺失，则不执行扩展认证。请参阅 [第 4.12 节](#412-enhanced-authentication)。

如果 `Client` 在 `CONNECT` 中设置了 `Authentication Method`，`Client` 禁止发送除 `AUTH` 或 `DISCONNECT` 报文以外的任何报文，直到它收到 `CONNACK` 报文 [MQTT-3.1.2-30]。

##### 3.1.2.11.10 `Authentication Data`

**22 (0x16) 字节**，`Authentication Data` 的标识符。

后跟包含认证数据的 `Binary Data`。如果没有 `Authentication Method`，包含 `Authentication Data` 是协议错误。多次包含 `Authentication Data` 是协议错误。

此数据的内容由认证方法定义。有关扩展认证的更多信息，请参阅 [第 4.12 节](#412-enhanced-authentication)。

#### 3.1.2.12 `Variable Header` 非规范性示例

**图 3-6 - `Variable Header` 示例**

（此示例展示了包含 Protocol Name、Protocol Version、Connect Flags、`Keep Alive`、Properties 等字段的 `CONNECT` 报文 `Variable Header`）

### 3.1.3 CONNECT `Payload`

`CONNECT` 报文的 `Payload` 包含一个或多个长度前缀字段，它们的存在由 `Variable Header` 中的标志决定。如果存在，这些字段必须按以下顺序出现：`Client Identifier`、Will Properties、`Will Topic`、Will Payload、`User Name`、`Password` [MQTT-3.1.3-1]。

#### 3.1.3.1 `Client Identifier` (ClientID)

`Client Identifier` (ClientID) 向 `Server` 标识 `Client`。每个连接到 `Server` 的 `Client` 都有一个唯一的 ClientID。`Client` 和 `Server` 必须使用 ClientID 来标识它们持有的与 `Client` 和 `Server` 之间此 MQTT 会话相关的状态 [MQTT-3.1.3-2]。有关 `Session` State 的更多信息，请参阅 [第 4.1 节](#41-session-state)。

ClientID 必须存在，并且是 `CONNECT` 报文 `Payload` 中的第一个字段 [MQTT-3.1.3-3]。

ClientID 必须是 [第 1.5.4 节](#154-utf-8-encoded-string) 中定义的 `UTF-8 Encoded String` [MQTT-3.1.3-4]。

`Server` 必须允许长度在 1 到 23 个 UTF-8 编码字节之间且仅包含字符 "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ" 的 ClientID [MQTT-3.1.3-5]。

`Server` 可以允许包含超过 23 个编码字节的 ClientID。`Server` 可以允许包含上述列表中未包含的字符的 ClientID。

`Server` 可以允许 `Client` 提供长度为零字节的 ClientID，但如果这样做，`Server` 必须将其视为特殊情况并为该 `Client` 分配唯一的 ClientID [MQTT-3.1.3-6]。然后它必须像 `Client` 提供了该唯一 ClientID 一样处理 `CONNECT` 报文，并且必须在 `CONNACK` 报文中返回 Assigned Client Identifier [MQTT-3.1.3-7]。

如果 `Server` 拒绝 ClientID，它可以响应 `CONNECT` 报文，发送 `Reason Code` 为 0x85（`Client Identifier` 无效）的 `CONNACK`，如 [第 4.13 节](#4-13-错误处理) 中所述，然后必须关闭网络连接 [MQTT-3.1.3-8]。

> **非规范性注释**
>
> `Client` 实现可以提供一种便捷方法来生成随机 ClientID。使用此方法的 `Client` 应该注意避免创建长期存在的孤立会话。

#### 3.1.3.2 Will Properties

如果 Will Flag 设置为 1，Will Properties 是 `Payload` 中的下一个字段。Will Properties 字段定义了 `Will Message` 发布时要发送的 Application Message 属性，以及定义何时发布 `Will Message` 的属性。Will Properties 由 Property Length 和 Properties 组成。

##### 3.1.3.2.1 Property Length

Will Properties 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.1.3.2.2 `Will Delay Interval`

**24 (0x18) 字节**，`Will Delay Interval` 的标识符。

后跟表示 `Will Delay Interval`（以秒为单位）的四字节整数。多次包含 `Will Delay Interval` 是协议错误。如果 `Will Delay Interval` 缺失，默认值为 0，发布 `Will Message` 之前没有延迟。

`Server` 延迟发布 `Client` 的 `Will Message`，直到 `Will Delay Interval` 已过或会话结束（以先发生者为准）。如果在此 `Will Delay Interval` 过期之前建立了到此会话的新网络连接，`Server` 禁止发送 `Will Message` [MQTT-3.1.3-9]。

> **非规范性注释**
>
> 此用途之一是避免在临时网络断开连接且 `Client` 在 `Will Message` 发布之前成功重新连接并继续其会话时发布 Will Messages。

> **非规范性注释**
>
> 如果网络连接使用 `Server` 上现有网络连接的 `Client Identifier`，则现有连接的 `Will Message` 将被发送，除非新连接指定 `Clean Start` 为 0 且 Will Delay 大于零。如果 Will Delay 为 0，则 `Will Message` 在现有网络连接关闭时发送，如果 `Clean Start` 为 1，则因为会话结束而发送 `Will Message`。

##### 3.1.3.2.3 Payload Format Indicator

**1 (0x01) 字节**，Payload Format Indicator 的标识符。

后跟 Payload Format Indicator 的值，可以是：

- 0 (0x00) 字节表示 `Will Message` 是未指定的字节，相当于不发送 Payload Format Indicator。
- 1 (0x01) 字节表示 `Will Message` 是 UTF-8 Encoded Character Data。`Payload` 中的 UTF-8 数据必须是 Unicode 规范 [Unicode] 定义的有效 UTF-8，并在 RFC 3629 [RFC3629] 中重述。

多次包含 Payload Format Indicator 是协议错误。`Server` 可以验证 `Will Message` 是否为指示的格式，如果不是，则发送 `Reason Code` 为 0x99（`Payload` 格式无效）的 `CONNACK`，如第 4.13 节所述。

##### 3.1.3.2.4 Message Expiry Interval

**2 (0x02) 字节**，Message Expiry Interval 的标识符。

后跟表示 Message Expiry Interval 的四字节整数。多次包含 Message Expiry Interval 是协议错误。

如果存在，四字节值是 `Will Message` 的生命周期（以秒为单位），并在 `Server` 发布 `Will Message` 时作为 Publication Expiry Interval 发送。

如果缺失，`Server` 发布 `Will Message` 时不发送 Message Expiry Interval。

##### 3.1.3.2.5 Content Type

**3 (0x03)** Content Type 的标识符。

后跟描述 `Will Message` 内容的 `UTF-8 Encoded String`。多次包含 Content Type 是协议错误。Content Type 的值由发送和接收应用程序定义。

##### 3.1.3.2.6 Response Topic

**8 (0x08) 字节**，Response Topic 的标识符。

后跟用作响应消息 `Topic Name` 的 `UTF-8 Encoded String`。多次包含 Response Topic 是协议错误。Response Topic 的存在将 `Will Message` 标识为请求。

有关请求/响应的更多信息，请参阅 [第 4.10 节](#410-requestresponse)。

##### 3.1.3.2.7 Correlation Data

**9 (0x09) 字节**，Correlation Data 的标识符。

后跟 `Binary Data`。Correlation Data 由请求消息的发送者在收到响应消息时用于标识响应消息对应的请求。多次包含 Correlation Data 是协议错误。如果 Correlation Data 不存在，则请求者不需要任何关联数据。

Correlation Data 的值仅对请求消息的发送者和响应消息的接收者有意义。

有关请求/响应的更多信息，请参阅 [第 4.10 节](#410-requestresponse)。

##### 3.1.3.2.8 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。允许 `User Property` 多次出现以表示多个名称-值对。允许同一名称多次出现。

`Server` 必须在发布 `Will Message` 时保持 `User Properties` 的顺序 [MQTT-3.1.3-10]。

> **非规范性注释**
>
> 此属性旨在提供传输应用程序层名称-值标签的方法，其含义和解释仅由负责发送和接收它们的应用程序知道。

#### 3.1.3.3 `Will Topic`

如果 Will Flag 设置为 1，`Will Topic` 是 `Payload` 中的下一个字段。`Will Topic` 必须是 [第 1.5.4 节](#154-utf-8-encoded-string) 中定义的 `UTF-8 Encoded String` [MQTT-3.1.3-11]。

#### 3.1.3.4 Will Payload

如果 Will Flag 设置为 1，Will Payload 是 `Payload` 中的下一个字段。Will Payload 定义要发布到 `Will Topic` 的 Application Message `Payload`，如 [第 3.1.2.5 节](#3125-will-flag) 中所述。此字段由 `Binary Data` 组成。

#### 3.1.3.5 `User Name`

如果 User Name Flag 设置为 1，`User Name` 是 `Payload` 中的下一个字段。`User Name` 必须是 [第 1.5.4 节](#154-utf-8-encoded-string) 中定义的 `UTF-8 Encoded String` [MQTT-3.1.3-12]。它可以被 `Server` 用于认证和授权。

#### 3.1.3.6 `Password`

如果 Password Flag 设置为 1，`Password` 是 `Payload` 中的下一个字段。`Password` 字段是 `Binary Data`。尽管此字段称为 `Password`，但它可以用于携带任何凭据信息。

### 3.1.4 CONNECT Actions

请注意，`Server` 可以在同一 TCP 端口或其他网络端点上支持多种协议（包括 MQTT 协议的其他版本）。如果 `Server` 确定协议是 MQTT v5.0，则它按如下方式验证连接尝试。

1. 如果 `Server` 在网络连接建立后的合理时间内没有收到 `CONNECT` 报文，`Server` 应该关闭网络连接。

2. `Server` 必须验证 `CONNECT` 报文是否符合 [第 3.1 节](#31-connect---connection-request) 中描述的格式，如果不符合则关闭网络连接 [MQTT-3.1.4-1]。`Server` 可以在关闭网络连接之前发送 `Reason Code` 为 0x80 或更大的 `CONNACK`，如第 4.13 节所述。

3. `Server` 可以检查 `CONNECT` 报文的内容是否满足任何进一步限制，并应该执行认证和授权检查。如果任何这些检查失败，它必须关闭网络连接 [MQTT-3.1.4-2]。在关闭网络连接之前，它可以发送适当的 `CONNACK` 响应，`Reason Code` 为 0x80 或更大，如 [第 3.2 节](#32-connack---connect-acknowledgement) 和 [第 4.13 节](#4-13-错误处理) 中所述。

如果验证成功，`Server` 执行以下步骤。

1. 如果 ClientID 表示已经连接到 `Server` 的 `Client`，`Server` 向现有 `Client` 发送 `Reason Code` 为 0x8E（会话被接管）的 `DISCONNECT` 报文，如 [第 4.13 节](#4-13-错误处理) 中所述，并且必须关闭现有 `Client` 的网络连接 [MQTT-3.1.4-3]。如果现有 `Client` 有 `Will Message`，则该 `Will Message` 按第 3.1.2.5 节中所述发布。

> **非规范性注释**
>
> 如果现有网络连接的 `Will Delay Interval` 为 0 且存在 `Will Message`，它将被发送，因为网络连接已关闭。如果现有网络连接的 `Session Expiry Interval` 为 0，或新网络连接的 `Clean Start` 设置为 1，那么如果现有网络连接有 `Will Message`，它将被发送，因为原始会话在接管时结束。

2. `Server` 必须执行 [第 3.1.2.4 节](#3124-clean-start) 中描述的 `Clean Start` 处理 [MQTT-3.1.4-4]。

3. `Server` 必须使用包含 0x00（成功）`Reason Code` 的 `CONNACK` 报文确认 `CONNECT` 报文 [MQTT-3.1.4-5]。

> **非规范性注释**
>
> 如果 `Server` 用于处理任何形式的关键业务数据，建议执行认证和授权检查。如果这些检查成功，`Server` 通过发送 `Reason Code` 为 0x00（成功）的 `CONNACK` 来响应。如果失败，建议 `Server` 根本不发送 `CONNACK`，因为这可能会提醒潜在攻击者 MQTT `Server` 的存在，并鼓励此类攻击者发起拒绝服务或密码猜测攻击。

4. 开始消息传递和 `Keep Alive` 监控。

`Client` 被允许在发送 `CONNECT` 报文后立即发送更多 MQTT Control Packets；`Client` 不需要等待从 `Server` 发来的 `CONNACK` 报文。如果 `Server` 拒绝 `CONNECT`，它禁止处理 `Client` 在 `CONNECT` 报文之后发送的任何数据，除了 `AUTH` 报文 [MQTT-3.1.4-6]。

> **非规范性注释**
>
> `Client` 通常等待 `CONNACK` 报文，但是，如果 `Client` 利用其在收到 `CONNACK` 之前发送 MQTT Control Packets 的自由，它可能会简化 `Client` 实现，因为它不必监管连接状态。`Client` 接受在收到 `Server` 的 `CONNACK` 报文之前发送的任何数据如果 `Server` 拒绝连接将不会被处理。

> **非规范性注释**
>
> 在收到 `CONNACK` 之前发送 MQTT Control Packets 的 `Client` 将不知道 `Server` 约束以及是否正在使用任何现有会话。

> **非规范性注释**
>
> 如果 `Client` 在认证完成之前发送过多数据，`Server` 可以限制从网络连接读取或关闭网络连接。这被建议作为避免拒绝服务攻击的一种方法。

## 3.2 CONNACK - Connect acknowledgement

`CONNACK` 报文是 `Server` 为响应从 `Client` 收到的 `CONNECT` 报文而发送的报文。`Server` 必须在发送除 `AUTH` 以外的任何报文之前发送 `Reason Code` 为 0x00（成功）的 `CONNACK` [MQTT-3.2.0-1]。`Server` 禁止在网络连接中发送多个 `CONNACK` [MQTT-3.2.0-2]。

如果 `Client` 在合理时间内没有从 `Server` 收到 `CONNACK` 报文，`Client` 应该关闭网络连接。"合理"的时间量取决于应用程序类型和通信基础设施。

### 3.2.1 CONNACK `Fixed Header`

`Fixed Header` 格式如图 3-7 所示。

**图 3-7 - `CONNACK` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet Type (2) | Reserved |
| | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

**`Remaining Length` 字段**

这是编码为 `Variable Byte Integer` 的 `Variable Header` 的长度。

### 3.2.2 CONNACK `Variable Header`

`CONNACK` 报文的 `Variable Header` 按顺序包含以下字段：Connect Acknowledge Flags、Connect Reason Code 和 Properties。属性编码规则在 [第 2.2.2 节](#222-properties) 中描述。

#### 3.2.2.1 Connect Acknowledge Flags

字节 1 是"Connect Acknowledge Flags"。第 7-1 位是保留的，必须设置为 0 [MQTT-3.2.2-1]。

第 0 位是 Session Present 标志。

##### 3.2.2.1.1 `Session` Present

**位置：** Connect Acknowledge Flags 的第 0 位。

`Session` Present 标志通知 `Client` `Server` 是否为此 ClientID 使用来自先前连接的 `Session` State。这允许 `Client` 和 `Server` 对 `Session` State 有一致的视图。

如果 `Server` 接受 `Clean Start` 设置为 1 的连接，`Server` 必须在 `CONNACK` 报文中将 `Session` Present 设置为 0，此外还要在 `CONNACK` 报文中设置 0x00（成功）`Reason Code` [MQTT-3.2.2-2]。

如果 `Server` 接受 `Clean Start` 设置为 0 的连接，并且 `Server` 具有 ClientID 的 `Session` State，它必须在 `CONNACK` 报文中将 `Session` Present 设置为 1，否则它必须在 `CONNACK` 报文中将 `Session` Present 设置为 0。在这两种情况下，它必须在 `CONNACK` 报文中设置 0x00（成功）`Reason Code` [MQTT-3.2.2-3]。

如果 `Client` 从 `Server` 收到的 `Session` Present 的值不符合预期，`Client` 按如下方式进行：

- 如果 `Client` 没有 `Session` State 并收到 `Session` Present 设置为 1，它必须关闭网络连接 [MQTT-3.2.2-4]。如果它希望以新会话重新开始，`Client` 可以使用 `Clean Start` 设置为 1 重新连接。

- 如果 `Client` 有 `Session` State 并收到 `Session` Present 设置为 0，如果它继续网络连接，它必须丢弃其 `Session` State [MQTT-3.2.2-5]。

如果 `Server` 发送包含非零 `Reason Code` 的 `CONNACK` 报文，它必须将 `Session` Present 设置为 0 [MQTT-3.2.2-6]。

#### 3.2.2.2 Connect Reason Code

`Variable Header` 中的字节 2 是 Connect Reason Code。

Connect Reason Code 的值如下所示。如果 `Server` 收到格式正确的 `CONNECT` 报文，但 `Server` 无法完成连接，`Server` 可以发送包含此表中相应 Connect Reason Code 的 `CONNACK` 报文。如果 `Server` 发送包含 128 或更大的 `Reason Code` 的 `CONNACK` 报文，它必须随后关闭网络连接 [MQTT-3.2.2-7]。

**表 3-1 - Connect Reason Code 值**

| Value | Hex | Reason Code name | Description |
|-------|-----|------------------|-------------|
| 0 | 0x00 | Success | 连接被接受。 |
| 128 | 0x80 | Unspecified error | Server 不希望透露失败原因，或其他 Reason Code 都不适用。 |
| 129 | 0x81 | Malformed Packet | CONNECT 报文中的数据无法正确解析。 |
| 130 | 0x82 | Protocol Error | CONNECT 报文中的数据不符合此规范。 |
| 131 | 0x83 | Implementation specific error | CONNECT 有效但未被此 Server 接受。 |
| 132 | 0x84 | Unsupported Protocol Version | `Server` 不支持 `Client` 请求的 MQTT 协议版本。 |
| 133 | 0x85 | Client Identifier not valid | `Client Identifier` 是有效的字符串但不被 `Server` 允许。 |
| 134 | 0x86 | Bad User Name or Password | `Server` 不接受 `Client` 指定的 `User Name` 或 `Password`。 |
| 135 | 0x87 | Not authorized | `Client` 未被授权连接。 |
| 136 | 0x88 | Server unavailable | MQTT `Server` 不可用。 |
| 137 | 0x89 | Server busy | `Server` 忙。请稍后重试。 |
| 138 | 0x8A | Banned | 此 `Client` 已被管理操作禁止。联系服务器管理员。 |
| 140 | 0x8C | Bad authentication method | 不支持认证方法或与当前使用的认证方法不匹配。 |
| 144 | 0x90 | Topic Name invalid | Will `Topic Name` 没有格式错误，但未被此 `Server` 接受。 |
| 149 | 0x95 | Packet too large | `CONNECT` 报文超过了最大允许大小。 |
| 151 | 0x97 | Quota exceeded | 已超过实现或管理设定的限制。 |
| 153 | 0x99 | Payload format invalid | Will `Payload` 与指定的 Payload Format Indicator 不匹配。 |
| 154 | 0x9A | Retain not supported | `Server` 不支持保留消息，且 Will Retain 设置为 1。 |
| 155 | 0x9B | QoS not supported | `Server` 不支持 Will `QoS` 中设置的 `QoS`。 |
| 156 | 0x9C | Use another server | `Client` 应该临时使用另一台服务器。 |
| 157 | 0x9D | Server moved | `Client` 应该永久使用另一台服务器。 |
| 159 | 0x9F | Connection rate exceeded | 已超过连接速率限制。 |

发送 `CONNACK` 报文的 `Server` 必须使用 Connect Reason Code 值之一 [MQTT-3.2.2-8]。

> **非规范性注释**
>
> `Reason Code` 0x80（未指定错误）可用于 `Server` 知道失败原因但不想向 `Client` 透露的情况，或当其他 `Reason Code` 值都不适用时。

`Server` 可以选择在 `CONNECT` 上发现错误时不发送 `CONNACK` 就关闭网络连接，以增强安全性。例如，在公共网络上且连接未被授权时，表明这是 MQTT `Server` 可能是不明智的。

#### 3.2.2.3 CONNACK Properties

##### 3.2.2.3.1 Property Length

这是 `CONNACK` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.2.2.3.2 `Session Expiry Interval`

**17 (0x11) 字节**，`Session Expiry Interval` 的标识符。

后跟表示 `Session Expiry Interval`（以秒为单位）的四字节整数。多次包含 `Session Expiry Interval` 是协议错误。

如果 `Session Expiry Interval` 缺失，则使用 `CONNECT` 报文中的值。`Server` 使用此属性通知 `Client` 它在 `CONNACK` 中使用的值与 `Client` 发送的值不同。有关 `Session Expiry Interval` 的使用说明，请参阅第 3.1.2.11.2 节。

##### 3.2.2.3.3 `Receive Maximum`

**33 (0x21) 字节**，`Receive Maximum` 的标识符。

后跟表示 `Receive Maximum` 值的双字节整数。多次包含 `Receive Maximum` 值或其值为 0 是协议错误。

`Server` 使用此值来限制它愿意为 `Client` 同时处理的 `QoS` 1 和 `QoS` 2 发布的数量。它不提供机制来限制 `Client` 可能尝试发送的 `QoS` 0 发布。

如果 `Receive Maximum` 值缺失，则其值默认为 65,535。

有关如何使用 `Receive Maximum` 的详细信息，请参阅 [第 4.9 节](#49-flow-control)流控制。

##### 3.2.2.3.4 `Maximum QoS`

**36 (0x24) 字节**，`Maximum QoS` 的标识符。

后跟值为 0 或 1 的字节。多次包含 `Maximum QoS` 或值不是 0 或 1 是协议错误。如果 `Maximum QoS` 缺失，`Client` 使用 `Maximum QoS` 为 2。

如果 `Server` 不支持 `QoS` 1 或 `QoS` 2 `PUBLISH` 报文，它必须在 `CONNACK` 报文中发送 `Maximum QoS`，指定它支持的最高 `QoS` [MQTT-3.2.2-9]。不支持 `QoS` 1 或 `QoS` 2 `PUBLISH` 报文的 `Server` 必须仍然接受包含请求 `QoS` 为 0、1 或 2 的 `SUBSCRIBE` 报文 [MQTT-3.2.2-10]。

如果 `Client` 从 `Server` 收到 `Maximum QoS`，它禁止发送超过指定 `Maximum QoS` 级别的 `QoS` 级别的 `PUBLISH` 报文 [MQTT-3.2.2-11]。如果 `Server` 收到 `QoS` 大于它指定的 `Maximum QoS` 的 `PUBLISH` 报文，这是协议错误。在这种情况下，使用 `Reason Code` 为 0x9B（不支持 `QoS`）的 `DISCONNECT`，如 [第 4.13 节](#4-13-错误处理)处理错误中所述。

如果 `Server` 收到包含超过其能力的 Will `QoS` 的 `CONNECT` 报文，它必须拒绝连接。它应该使用 `Reason Code` 为 0x9B（不支持 `QoS`）的 `CONNACK` 报文，如 [第 4.13 节](#4-13-错误处理)处理错误中所述，并且必须关闭网络连接 [MQTT-3.2.2-12]。

> **非规范性注释**
>
> `Client` 不需要支持 `QoS` 1 或 `QoS` 2 `PUBLISH` 报文。如果是这种情况，`Client` 只需将其发送的任何 `SUBSCRIBE` 命令中的最大 `QoS` 字段限制为它可以支持的值。

##### 3.2.2.3.5 Retain Available

**37 (0x25) 字节**，Retain Available 的标识符。

后跟字节字段。如果存在，此字节声明 `Server` 是否支持保留消息。值为 0 表示不支持保留消息。值为 1 表示支持保留消息。如果不存在，则支持保留消息。多次包含 Retain Available 或使用 0 或 1 以外的值是协议错误。

如果 `Server` 收到包含 Will Retain 设置为 1 的 `Will Message` 的 `CONNECT` 报文，并且它不支持保留消息，`Server` 必须拒绝连接请求。它应该发送 `Reason Code` 为 0x9A（不支持保留）的 `CONNACK`，然后必须关闭网络连接 [MQTT-3.2.2-13]。

从 `Server` 收到 Retain Available 设置为 0 的 `Client` 禁止发送 `RETAIN` 标志设置为 1 的 `PUBLISH` 报文 [MQTT-3.2.2-14]。如果 `Server` 收到这样的报文，这是协议错误。`Server` 应该发送 `Reason Code` 为 0x9A（不支持保留）的 `DISCONNECT`，如 [第 4.13 节](#4-13-错误处理)中所述。

##### 3.2.2.3.6 `Maximum Packet Size`

**39 (0x27) 字节**，`Maximum Packet Size` 的标识符。

后跟表示 `Server` 愿意接受的最大报文大小的四字节整数。如果 `Maximum Packet Size` 不存在，除了由于剩余长度编码和协议头大小而在协议中产生的限制外，不对报文大小施加限制。

多次包含 `Maximum Packet Size` 或值设置为零是协议错误。

报文大小是 MQTT Control Packet 中的总字节数，如 [第 2.1.4 节](#214-remaining-length) 中所定义。`Server` 使用 `Maximum Packet Size` 通知 `Client` 它不会处理超过此限制的报文。

`Client` 禁止向 `Server` 发送超过 `Maximum Packet Size` 的报文 [MQTT-3.2.2-15]。如果 `Server` 收到超过此限制的报文，这是协议错误，`Server` 使用 `Reason Code` 为 0x95（报文过大）的 `DISCONNECT`，如 [第 4.13 节](#4-13-错误处理)中所述。

##### 3.2.2.3.7 Assigned Client Identifier

**18 (0x12) 字节**，Assigned Client Identifier 的标识符。

后跟作为 Assigned Client Identifier 的 UTF-8 字符串。多次包含 Assigned Client Identifier 是协议错误。

由于 `CONNECT` 报文中发现零长度 `Client Identifier` 而由 `Server` 分配的 `Client Identifier`。

如果 `Client` 使用零长度 `Client Identifier` 连接，`Server` 必须响应包含 Assigned Client Identifier 的 `CONNACK`。Assigned Client Identifier 必须是 `Server` 中当前任何其他会话未使用的新 `Client Identifier` [MQTT-3.2.2-16]。

##### 3.2.2.3.8 `Topic Alias Maximum`

**34 (0x22) 字节**，`Topic Alias Maximum` 的标识符。

后跟表示 `Topic Alias Maximum` 值的双字节整数。多次包含 `Topic Alias Maximum` 值是协议错误。如果 `Topic Alias Maximum` 属性缺失，默认值为 0。

此值表示 `Server` 将接受的 `Client` 发送的 `Topic Alias` 的最大值。`Server` 使用此值来限制它愿意在此连接上持有的 `Topic Alias` 的数量。`Client` 禁止向 `Server` 发送大于此值的 `PUBLISH` 报文中的 `Topic Alias` [MQTT-3.2.2-17]。值为 0 表示 `Server` 不接受此连接上的任何 `Topic Alias`。如果 `Topic Alias Maximum` 缺失或为 0，`Client` 禁止向 `Server` 发送任何 `Topic Alias` [MQTT-3.2.2-18]。

##### 3.2.2.3.9 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示与此响应关联的原因的 `UTF-8 Encoded String`。此 Reason String 是为诊断设计的人类可读字符串，不应该由 `Client` 解析。

`Server` 使用此值向 `Client` 提供额外信息。如果这会使 `CONNACK` 报文的大小超过 `Client` 指定的 `Maximum Packet Size`，`Server` 禁止发送此属性 [MQTT-3.2.2-19]。多次包含 Reason String 是协议错误。

> **非规范性注释**
>
> `Client` 中 Reason String 的正确用途包括在 `Client` 代码抛出的异常中使用此信息，或将此字符串写入日志。

##### 3.2.2.3.10 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此属性可用于向 `Client` 提供额外信息，包括诊断信息。如果这会使 `CONNACK` 报文的大小超过 `Client` 指定的 `Maximum Packet Size`，`Server` 禁止发送此属性 [MQTT-3.2.2-20]。允许 `User Property` 多次出现以表示多个名称-值对。允许同一名称多次出现。

此属性的内容和含义不由本规范定义。收到包含此属性的 `CONNACK` 的接收者可以忽略它。

##### 3.2.2.3.11 Wildcard Subscription Available

**40 (0x28) 字节**，Wildcard Subscription Available 的标识符。

后跟字节字段。如果存在，此字节声明 `Server` 是否支持通配符订阅。值为 0 表示不支持通配符订阅。值为 1 表示支持通配符订阅。如果不存在，则支持通配符订阅。多次包含 Wildcard Subscription Available 或发送 0 或 1 以外的值是协议错误。

如果 `Server` 收到包含通配符订阅的 `SUBSCRIBE` 报文，并且它不支持通配符订阅，这是协议错误。`Server` 使用 `Reason Code` 为 0xA2（不支持通配符订阅）的 `DISCONNECT`，如 [第 4.13 节](#4-13-错误处理)中所述。

如果 `Server` 支持通配符订阅，它仍然可以拒绝包含通配符订阅的特定订阅请求。在这种情况下，`Server` 可以发送 `Reason Code` 为 0xA2（不支持通配符订阅）的 `SUBACK` Control Packet。

##### 3.2.2.3.12 Subscription Identifiers Available

**41 (0x29) 字节**，Subscription Identifier Available 的标识符。

后跟字节字段。如果存在，此字节声明 `Server` 是否支持 `Subscription Identifier`。值为 0 表示不支持 `Subscription Identifier`。值为 1 表示支持 `Subscription Identifier`。如果不存在，则支持 `Subscription Identifier`。多次包含 Subscription Identifier Available 或发送 0 或 1 以外的值是协议错误。

如果 `Server` 收到包含 `Subscription Identifier` 的 `SUBSCRIBE` 报文，并且它不支持 `Subscription Identifier`，这是协议错误。`Server` 使用 `Reason Code` 为 0xA1（不支持 `Subscription Identifier`）的 `DISCONNECT`，如 [第 4.13 节](#4-13-错误处理)中所述。

##### 3.2.2.3.13 Shared Subscription Available

**42 (0x2A) 字节**，Shared Subscription Available 的标识符。

后跟字节字段。如果存在，此字节声明 `Server` 是否支持共享订阅。值为 0 表示不支持共享订阅。值为 1 表示支持共享订阅。如果不存在，则支持共享订阅。多次包含 Shared Subscription Available 或发送 0 或 1 以外的值是协议错误。

如果 `Server` 收到包含共享订阅的 `SUBSCRIBE` 报文，并且它不支持共享订阅，这是协议错误。`Server` 使用 `Reason Code` 为 0x9E（不支持共享订阅）的 `DISCONNECT`，如 [第 4.13 节](#4-13-错误处理)中所述。

##### 3.2.2.3.14 Server Keep Alive

**19 (0x13) 字节**，Server Keep Alive 的标识符。

后跟具有 `Server` 分配的 `Keep Alive` 时间的双字节整数。如果 `Server` 在 `CONNACK` 报文上发送 Server Keep Alive，`Client` 必须使用此值而不是 `Client` 在 `CONNECT` 上发送的 `Keep Alive` 值 [MQTT-3.2.2-21]。如果 `Server` 不发送 Server Keep Alive，`Server` 必须使用 `Client` 在 `CONNECT` 上设置的 `Keep Alive` 值 [MQTT-3.2.2-22]。多次包含 Server Keep Alive 是协议错误。

> **非规范性注释**
>
> Server Keep Alive 的主要用途是让 `Server` 通知 `Client` 它将因不活动而断开 `Client` 连接的时间比 `Client` 指定的 `Keep Alive` 更早。

##### 3.2.2.3.15 `Response Information`

**26 (0x1A) 字节**，`Response Information` 的标识符。

后跟用作创建 Response Topic 基础的 `UTF-8 Encoded String`。`Client` 从 `Response Information` 创建 Response Topic 的方式不由本规范定义。多次包含 `Response Information` 是协议错误。

如果 `Client` 发送值为 1 的 Request Response Information，`Server` 在 `CONNACK` 中发送 `Response Information` 是可选的。

> **非规范性注释**
>
> 此属性的常见用途是传递主题树的全局唯一部分，该部分至少在 `Client` 会话的生命周期内为此 `Client` 保留。这通常不能只是随机名称，因为请求 `Client` 和响应 `Client` 都需要被授权使用它。通常将其用作特定 `Client` 的主题树的根。对于 `Server` 返回此信息，通常需要正确配置。使用此机制允许在 `Server` 中完成一次此配置，而不是在每个 `Client` 中。

有关请求/响应的更多信息，请参阅 [第 4.10 节](#410-requestresponse)。

##### 3.2.2.3.16 `Server Reference`

**28 (0x1C) 字节**，`Server Reference` 的标识符。

后跟 `Client` 可用于标识要使用的另一台 `Server` 的 `UTF-8 Encoded String`。多次包含 `Server Reference` 是协议错误。

`Server` 在 `CONNACK` 或 `DISCONNECT` 报文中使用 `Server Reference`，`Reason Code` 为 0x9C（使用另一台服务器）或 `Reason Code` 0x9D（服务器已移动），如 [第 4.13 节](#4-13-错误处理)中所述。

有关如何使用 `Server Reference` 的信息，请参阅 [第 4.11 节](#411-server-redirection)服务器重定向。

##### 3.2.2.3.17 `Authentication Method`

**21 (0x15) 字节**，`Authentication Method` 的标识符。

后跟包含认证方法名称的 `UTF-8 Encoded String`。多次包含 `Authentication Method` 是协议错误。有关扩展认证的更多信息，请参阅 [第 4.12 节](#412-enhanced-authentication)。

##### 3.2.2.3.18 `Authentication Data`

**22 (0x16) 字节**，`Authentication Data` 的标识符。

后跟包含认证数据的 `Binary Data`。此数据的内容由认证方法和已交换的认证数据的状态定义。多次包含 `Authentication Data` 是协议错误。有关扩展认证的更多信息，请参阅 [第 4.12 节](#412-enhanced-authentication)。

### 3.2.3 CONNACK `Payload`

`CONNACK` 报文没有 `Payload`。

## 3.3 PUBLISH - 发布消息

`PUBLISH` 报文从 `Client` 发送到 `Server` 或者从 `Server` 发送到 `Client`，用于传输 Application Message。

### 3.3.1 PUBLISH `Fixed Header`

**图 3-8 - `PUBLISH` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (3) | `DUP` flag | `QoS` level | `RETAIN` |
| | 0 | 0 | 1 | 1 | X | X | X | X |
| byte 2 | `Remaining Length` |

#### 3.3.1.1 `DUP`

**位置：** byte 1, bit 3。

如果 `DUP` flag 设置为 0，表示这是 `Client` 或 `Server` 首次尝试发送此 `PUBLISH` 报文。如果 `DUP` flag 设置为 1，表示这可能是之前发送该报文尝试的重传。

`Client` 或 `Server` 在尝试重传 `PUBLISH` 报文时必须将 `DUP` flag 设置为 1 [MQTT-3.3.1-1]。对于所有 `QoS` 0 消息，`DUP` flag 必须设置为 0 [MQTT-3.3.1-2]。

来自传入 `PUBLISH` 报文的 `DUP` flag 值不会在 `Server` 将 `PUBLISH` 报文发送给订阅者时传播。传出 `PUBLISH` 报文中的 `DUP` flag 独立于传入 `PUBLISH` 报文设置，其值必须仅由传出 `PUBLISH` 报文是否为重传来确定 [MQTT-3.3.1-3]。

**非规范性说明**

接收包含 `DUP` flag 设置为 1 的 MQTT Control Packet 的接收方不能假设它之前已经看到过该报文的副本。

**非规范性说明**

需要注意的是，`DUP` flag 指的是 MQTT Control Packet 本身，而不是它所包含的 Application Message。当使用 `QoS` 1 时，`Client` 可能收到 `DUP` flag 设置为 0 的 `PUBLISH` 报文，其中包含它之前接收过的 Application Message 的重复，但具有不同的 `Packet Identifier`。[第 2.2.1 节](#221-packet-identifier)提供了关于 `Packet Identifier` 的更多信息。

#### 3.3.1.2 `QoS`

**位置：** byte 1, bits 2-1。

此字段指示 Application Message 的传递保证级别。`QoS` 级别如下所示。

**表 3-2 - `QoS` 定义**

| `QoS` 值 | Bit 2 | Bit 1 | 描述 |
|--------|-------|-------|------|
| 0 | 0 | 0 | At most once delivery（最多一次传递） |
| 1 | 0 | 1 | At least once delivery（至少一次传递） |
| 2 | 1 | 0 | Exactly once delivery（恰好一次传递） |
| - | 1 | 1 | 保留 - 禁止使用 |

如果 `Server` 在其对 `Client` 的 `CONNACK` 响应中包含了 `Maximum QoS`，并且收到 `QoS` 大于此值的 `PUBLISH` 报文，则它使用 `DISCONNECT` 并带有 `Reason Code` 0x9B (`QoS` not supported)，如 [第 4.13 节](#413-error-handling)错误处理中所述。

`PUBLISH` 报文禁止将两个 `QoS` 位都设置为 1 [MQTT-3.3.1-4]。如果 `Server` 或 `Client` 收到两个 `QoS` 位都设置为 1 的 `PUBLISH` 报文，这是一个 Malformed Packet。使用 `DISCONNECT` 并带有 `Reason Code` 0x81 (Malformed Packet)，如 [第 4.13 节](#413-error-handling)中所述。

#### 3.3.1.3 `RETAIN`

**位置：** byte 1, bit 0。

如果 `Client` 发送给 `Server` 的 `PUBLISH` 报文中 `RETAIN` flag 设置为 1，`Server` 必须替换此 topic 的任何现有保留消息并存储 Application Message [MQTT-3.3.1-5]，以便将其传递给未来订阅匹配其 `Topic Name` 的订阅者。如果 `Payload` 包含零字节，`Server` 将正常处理它，但必须删除具有相同 topic name 的任何保留消息，并且该 topic 的任何未来订阅者不会收到保留消息 [MQTT-3.3.1-6]。包含零字节 `Payload` 的保留消息禁止作为保留消息存储在 `Server` 上 [MQTT-3.3.1-7]。

如果 `Client` 发送给 `Server` 的 `PUBLISH` 报文中 `RETAIN` flag 为 0，`Server` 禁止将消息存储为保留消息，且禁止删除或替换任何现有的保留消息 [MQTT-3.3.1-8]。

如果 `Server` 在其对 `Client` 的 `CONNACK` 响应中包含 Retain Available 且其值设置为 0，并且它收到 `RETAIN` flag 设置为 1 的 `PUBLISH` 报文，则它使用 `DISCONNECT` `Reason Code` 0x9A (Retain not supported)，如 [第 4.13 节](#413-error-handling)中所述。

当进行新的非共享订阅时，每个匹配 topic name 上的最后保留消息（如果有）将根据 Retain Handling Subscription Option 发送给 `Client`。这些消息在发送时 `RETAIN` flag 设置为 1。发送哪些保留消息由 Retain Handling Subscription Option 控制。在订阅时：

- 如果 Retain Handling 设置为 0，`Server` 必须将与订阅的 `Topic Filter` 匹配的保留消息发送给 `Client` [MQTT-3.3.1-9]。
- 如果 Retain Handling 设置为 1，如果订阅不存在，`Server` 必须将与订阅的 `Topic Filter` 匹配的所有保留消息发送给 `Client`；如果订阅存在，`Server` 禁止发送保留消息 [MQTT-3.3.1-10]。
- 如果 Retain Handling 设置为 2，`Server` 禁止发送保留消息 [MQTT-3.3.1-11]。

有关 Subscription Options 的定义，请参阅 [第 3.8.3.1 节](#3831-subscription-options)。

如果 `Server` 收到 `RETAIN` flag 设置为 1 且 `QoS` 为 0 的 `PUBLISH` 报文，它应该将该 `QoS` 0 消息存储为该 topic 的新保留消息，但可以随时选择丢弃它。如果发生这种情况，该 topic 将没有保留消息。

如果 Topic 的当前保留消息过期，它将被丢弃，该 topic 将没有保留消息。

`Server` 从已建立的连接转发的 Application Message 中 `RETAIN` flag 的设置由 Retain As Published subscription option 控制。有关 Subscription Options 的定义，请参阅 [第 3.8.3.1 节](#3831-subscription-options)。

- 如果 Retain As Published subscription option 的值设置为 0，`Server` 在转发 Application Message 时必须将 `RETAIN` flag 设置为 0，无论接收到的 `PUBLISH` 报文中 `RETAIN` flag 如何设置 [MQTT-3.3.1-12]。
- 如果 Retain As Published subscription option 的值设置为 1，`Server` 必须将 `RETAIN` flag 设置为与接收到的 `PUBLISH` 报文中的 `RETAIN` flag 相等 [MQTT-3.3.1-13]。

**非规范性说明**

保留消息对于发布者以不规则方式发送状态消息的情况很有用。新的非共享订阅者将收到最新状态。

#### 3.3.1.4 `Remaining Length`

这是 `Variable Header` 的长度加上 `Payload` 的长度，编码为 `Variable Byte Integer`。

### 3.3.2 PUBLISH `Variable Header`

`PUBLISH` 报文的 `Variable Header` 按顺序包含以下字段：`Topic Name`、`Packet Identifier` 和 Properties。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

#### 3.3.2.1 `Topic Name`

`Topic Name` 标识发布 `Payload` 数据的信息通道。

`Topic Name` 必须作为 `PUBLISH` 报文 `Variable Header` 中的第一个字段存在。它必须是 [第 1.5.4 节](#154-utf-8-encoded-string)中定义的 `UTF-8 Encoded String` [MQTT-3.3.2-1]。

`PUBLISH` 报文中的 `Topic Name` 禁止包含通配符 [MQTT-3.3.2-2]。

`Server` 发送给订阅 `Client` 的 `PUBLISH` 报文中的 `Topic Name` 必须根据 [第 4.7 节](#47-topic-names-and-topic-filters)中定义的匹配过程与订阅的 `Topic Filter` 匹配 [MQTT-3.3.2-3]。但是，由于 `Server` 被允许将 `Topic Name` 映射到另一个名称，它可能与原始 `PUBLISH` 报文中的 `Topic Name` 不同。

为了减小 `PUBLISH` 报文的大小，发送方可以使用 `Topic Alias`。`Topic Alias` 在 [第 3.3.2.3.4 节](#33234-topic-alias)中描述。如果 `Topic Name` 长度为零且没有 `Topic Alias`，则为协议错误。

#### 3.3.2.2 `Packet Identifier`

`Packet Identifier` 字段仅存在于 `QoS` 级别为 1 或 2 的 `PUBLISH` 报文中。[第 2.2.1 节](#221-packet-identifier)提供了关于 `Packet Identifier` 的更多信息。

#### 3.3.2.3 PUBLISH Properties

##### 3.3.2.3.1 Property Length

`PUBLISH` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.3.2.3.2 Payload Format Indicator

**1 (0x01) 字节**，Payload Format Indicator 的标识符。

后跟 Payload Format Indicator 的值，可以是：

- 0 (0x00) 字节 表示 `Payload` 是未指定的字节，相当于不发送 Payload Format Indicator。
- 1 (0x01) 字节 表示 `Payload` 是 UTF-8 Encoded Character Data。`Payload` 中的 UTF-8 数据必须是 Unicode 规范 [Unicode] 定义的有效 UTF-8，并在 RFC 3629 [RFC3629] 中重述。

`Server` 必须将 Payload Format Indicator 不做修改地发送给所有接收 Application Message 的订阅者 [MQTT-3.3.2-4]。接收方可以验证 `Payload` 是否为指示的格式，如果不是，则发送带有 `Reason Code` 0x99 (`Payload` format invalid) 的 `PUBACK`、`PUBREC` 或 `DISCONNECT`，如 [第 4.13 节](#413-error-handling)中所述。有关验证 payload 格式的安全问题，请参阅 [第 5.4.9 节](#549-payload-validation)。

##### 3.3.2.3.3 Message Expiry Interval

**2 (0x02) 字节**，Message Expiry Interval 的标识符。

后跟表示 Message Expiry Interval 的 Four Byte Integer。

如果存在，该 Four Byte 值是 Application Message 的生存时间（以秒为单位）。如果 Message Expiry Interval 已过且 `Server` 尚未开始向匹配的订阅者进行后续传递，则它必须删除该订阅者的消息副本 [MQTT-3.3.2-5]。

如果不存在，则 Application Message 不会过期。

`Server` 发送给 `Client` 的 `PUBLISH` 报文必须包含设置为接收值减去 Application Message 在 `Server` 中等待时间的 Message Expiry Interval [MQTT-3.3.2-6]。有关存储状态的详细信息和限制，请参阅 [第 4.1 节](#41-session-state)。

##### 3.3.2.3.4 `Topic Alias`

**35 (0x23) 字节**，`Topic Alias` 的标识符。

后跟表示 `Topic Alias` 值的 Two Byte integer。多次包含 `Topic Alias` 值是协议错误。

`Topic Alias` 是一个整数值，用于标识 Topic 而不是使用 `Topic Name`。这减少了 `PUBLISH` 报文的大小，当 `Topic Name` 较长且在同一 Network Connection 中重复使用相同的 `Topic Name` 时非常有用。

发送方决定是否使用 `Topic Alias` 并选择其值。它通过在 `PUBLISH` 报文中包含非零长度的 `Topic Name` 和 `Topic Alias` 来设置 `Topic Alias` 映射。接收方正常处理 `PUBLISH`，但也将指定的 `Topic Alias` 映射设置为此 `Topic Name`。

如果在接收方已设置了 `Topic Alias` 映射，发送方可以发送包含该 `Topic Alias` 和零长度 `Topic Name` 的 `PUBLISH` 报文。然后接收方将传入的 `PUBLISH` 视为包含该 `Topic Alias` 的 `Topic Name`。

发送方可以通过在同一 Network Connection 中发送具有相同 `Topic Alias` 值和不同非零长度 `Topic Name` 的另一个 `PUBLISH` 来修改 `Topic Alias` 映射。

`Topic Alias` 映射仅存在于 Network Connection 内，并且仅在该 Network Connection 的生命周期内持续存在。接收方禁止将任何 `Topic Alias` 映射从一个 Network Connection 转移到另一个 [MQTT-3.3.2-7]。

`Topic Alias` 值为 0 是不允许的。发送方禁止发送包含值为 0 的 `Topic Alias` 的 `PUBLISH` 报文 [MQTT-3.3.2-8]。

`Client` 禁止发送 `Topic Alias` 大于 `Server` 在 `CONNACK` 报文中返回的 `Topic Alias Maximum` 值的 `PUBLISH` 报文 [MQTT-3.3.2-9]。`Client` 必须接受所有大于 0 且小于或等于它在 `CONNECT` 报文中发送的 `Topic Alias Maximum` 值的 `Topic Alias` 值 [MQTT-3.3.2-10]。

`Server` 禁止发送 `Topic Alias` 大于 `Client` 在 `CONNECT` 报文中发送的 `Topic Alias Maximum` 值的 `PUBLISH` 报文 [MQTT-3.3.2-11]。`Server` 必须接受所有大于 0 且小于或等于它在 `CONNACK` 报文中返回的 `Topic Alias Maximum` 值的 `Topic Alias` 值 [MQTT-3.3.2-12]。

`Client` 和 `Server` 使用的 `Topic Alias` 映射彼此独立。因此，当 `Client` 发送包含 `Topic Alias` 值为 1 的 `PUBLISH` 给 `Server`，并且 `Server` 发送 `Topic Alias` 值为 1 的 `PUBLISH` 给该 `Client` 时，它们通常引用不同的 Topic。

##### 3.3.2.3.5 Response Topic

**8 (0x08) 字节**，Response Topic 的标识符。

后跟一个 `UTF-8 Encoded String`，用作响应消息的 `Topic Name`。Response Topic 必须是 [第 1.5.4 节](#154-utf-8-encoded-string)中定义的 `UTF-8 Encoded String` [MQTT-3.3.2-13]。Response Topic 禁止包含通配符 [MQTT-3.3.2-14]。多次包含 Response Topic 是协议错误。Response Topic 的存在将消息标识为请求。

有关 Request / Response 的更多信息，请参阅 [第 4.10 节](#410-requestresponse)。

`Server` 必须将 Response Topic 不做修改地发送给所有接收 Application Message 的订阅者 [MQTT-3.3.2-15]。

**非规范性说明：**

收到带有 Response Topic 的 Application Message 的接收方使用 Response Topic 作为 `PUBLISH` 的 `Topic Name` 发送响应。如果请求消息包含 Correlation Data，请求消息的接收方还应该将此 Correlation Data 作为响应消息的 `PUBLISH` 报文中的 property 包含在内。

##### 3.3.2.3.6 Correlation Data

**9 (0x09) 字节**，Correlation Data 的标识符。

后跟 `Binary Data`。Correlation Data 由请求消息的发送方在收到响应消息时用于标识该响应对应哪个请求。多次包含 Correlation Data 是协议错误。如果不存在 Correlation Data，则请求者不需要任何关联数据。

`Server` 必须将 Correlation Data 不做修改地发送给所有接收 Application Message 的订阅者 [MQTT-3.3.2-16]。Correlation Data 的值仅对请求消息的发送方和响应消息的接收方有意义。

**非规范性说明**

收到包含 Response Topic 和 Correlation Data 的 Application Message 的接收方使用 Response Topic 作为 `PUBLISH` 的 `Topic Name` 发送响应。`Client` 还应该将 Correlation Data 不做修改地作为响应 `PUBLISH` 的一部分发送。

**非规范性说明**

如果 Correlation Data 包含可能因响应请求的 `Client` 修改而导致应用程序失败的信息，则应该对其进行加密和/或哈希处理，以便检测任何篡改。

有关 Request / Response 的更多信息，请参阅 [第 4.10 节](#410-requestresponse)。

##### 3.3.2.3.7 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

`Server` 在将 Application Message 转发给 `Client` 时，必须在 `PUBLISH` 报文中不做修改地发送所有 `User Property` [MQTT-3.3.2-17]。`Server` 在转发 Application Message 时必须保持 `User Property` 的顺序 [MQTT-3.3.2-18]。

**非规范性说明**

此 property 旨在提供一种传输应用层名称-值标签的方法，其含义和解释仅由负责发送和接收它们的应用程序知道。

##### 3.3.2.3.8 `Subscription Identifier`

**11 (0x0B)**，`Subscription Identifier` 的标识符。

后跟表示订阅标识符的 `Variable Byte Integer`。

`Subscription Identifier` 的值可以是 1 到 268,435,455。如果 `Subscription Identifier` 的值为 0，则为协议错误。如果发布是多个订阅匹配的结果，将包含多个 `Subscription Identifier`，在这种情况下它们的顺序不重要。

##### 3.3.2.3.9 Content Type

**3 (0x03)**，Content Type 的标识符。

后跟描述 Application Message 内容的 `UTF-8 Encoded String`。Content Type 必须是 [第 1.5.4 节](#154-utf-8-encoded-string)中定义的 `UTF-8 Encoded String` [MQTT-3.3.2-19]。

多次包含 Content Type 是协议错误。Content Type 的值由发送和接收应用程序定义。

`Server` 必须将 Content Type 不做修改地发送给所有接收 Application Message 的订阅者 [MQTT-3.3.2-20]。

**非规范性说明**

`UTF-8 Encoded String` 可以使用 MIME content type 字符串来描述 Application message 的内容。但是，由于发送和接收应用程序负责该字符串的定义和解释，MQTT 不对该字符串进行任何验证，只确保它是有效的 `UTF-8 Encoded String`。

**非规范性示例**

图 3-9 显示了一个 `PUBLISH` 报文的示例，其 `Topic Name` 设置为"a/b"，`Packet Identifier` 设置为 10，且没有 properties。

**图 3-9 - `PUBLISH` 报文 `Variable Header` 非规范性示例**

| | 描述 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|------|---|---|---|---|---|---|---|---|
| **Topic Name** |
| byte 1 | Length MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | Length LSB (3) | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 1 |
| byte 3 | 'a' (0x61) | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 1 |
| byte 4 | '/' (0x2F) | 0 | 0 | 1 | 0 | 1 | 1 | 1 | 1 |
| byte 5 | 'b' (0x62) | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 |
| **Packet Identifier** |
| byte 6 | Packet Identifier MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 7 | Packet Identifier LSB (10) | 0 | 0 | 0 | 0 | 1 | 0 | 1 | 0 |
| **Property Length** |
| byte 8 | No Properties | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

### 3.3.3 PUBLISH `Payload`

`Payload` 包含正在发布的 Application Message。数据的内容和格式是特定于应用程序的。可以通过从 `Fixed Header` 中的 `Remaining Length` 字段减去 `Variable Header` 的长度来计算 `Payload` 的长度。`PUBLISH` 报文包含零长度 `Payload` 是有效的。

### 3.3.4 PUBLISH Actions

`PUBLISH` 报文的接收方必须根据 `PUBLISH` 报文中的 `QoS` 确定的报文进行响应 [MQTT-3.3.4-1]。

**表 3-3 预期的 `PUBLISH` 报文响应**

| `QoS` 级别 | 预期响应 |
|----------|----------|
| `QoS` 0 | 无 |
| `QoS` 1 | `PUBACK` 报文 |
| `QoS` 2 | `PUBREC` 报文 |

`Client` 使用 `PUBLISH` 报文将 Application Message 发送给 `Server`，以便分发给具有匹配订阅的 `Client`。

`Server` 使用 `PUBLISH` 报文将 Application Message 发送给每个具有匹配订阅的 `Client`。`PUBLISH` 报文包含 `SUBSCRIBE` 报文中携带的 `Subscription Identifier`（如果有）。

当 `Client` 使用包含通配符的 `Topic Filter` 进行订阅时，`Client` 的订阅可能会重叠，因此发布的消息可能会匹配多个 filter。在这种情况下，`Server` 必须在遵守所有匹配订阅的最大 `QoS` 的同时将消息传递给 `Client` [MQTT-3.3.4-2]。此外，`Server` 可以传递消息的更多副本，每个额外匹配的订阅一个，并在每种情况下遵守订阅的 `QoS`。

如果 `Client` 收到非请求的 Application Message（不是订阅的结果），其 `QoS` 大于 `Maximum QoS`，它使用带有 `Reason Code` 0x9B (`QoS` not supported) 的 `DISCONNECT` 报文，如 [第 4.13 节](#413-error-handling)错误处理中所述。

如果 `Client` 为任何重叠订阅指定了 `Subscription Identifier`，`Server` 必须在由于订阅而发布的消息中发送这些 `Subscription Identifier` [MQTT-3.3.4-3]。如果 `Server` 发送消息的单个副本，它必须在 `PUBLISH` 报文中包含所有具有 `Subscription Identifier` 的匹配订阅的 `Subscription Identifier`，它们的顺序不重要 [MQTT-3.3.4-4]。如果 `Server` 发送多个 `PUBLISH` 报文，它必须在每个报文中发送匹配订阅的 `Subscription Identifier`（如果它有 `Subscription Identifier`）[MQTT-3.3.4-5]。

可能 `Client` 进行了多个匹配某个发布的订阅，并且它为其中多个订阅使用了相同的标识符。在这种情况下，`PUBLISH` 报文将携带多个相同的 `Subscription Identifier`。

`PUBLISH` 报文包含任何导致其流动的 `SUBSCRIBE` 报文中收到的 `Subscription Identifier` 以外的 `Subscription Identifier` 是协议错误。从 `Client` 发送到 `Server` 的 `PUBLISH` 报文禁止包含 `Subscription Identifier` [MQTT-3.3.4-6]。

如果订阅是共享的，则只有来自正在接收消息的 `Client` 的 `SUBSCRIBE` 报文中存在的 `Subscription Identifier` 才会在 `PUBLISH` 报文中返回。

接收方收到 `PUBLISH` 报文时的操作取决于 `QoS` 级别，如 [第 4.3 节](#43-quality-of-service)中所述。

如果 `PUBLISH` 报文包含 `Topic Alias`，接收方按如下方式处理：

1. `Topic Alias` 值为 0 或大于 Maximum Topic Alias 是协议错误，接收方使用带有 `Reason Code` 0x94 (`Topic Alias` invalid) 的 `DISCONNECT`，如 [第 4.13 节](#413-error-handling)中所述。

2. 如果接收方已经为 `Topic Alias` 建立了映射，则：
   a. 如果报文具有零长度的 `Topic Name`，接收方使用对应于 `Topic Alias` 的 `Topic Name` 处理它
   b. 如果报文包含非零长度的 `Topic Name`，接收方使用该 `Topic Name` 处理报文，并将其对 `Topic Alias` 的映射更新为传入报文中的 `Topic Name`

3. 如果接收方尚未为此 `Topic Alias` 建立映射：
   a. 如果报文具有零长度的 `Topic Name` 字段，则为协议错误，接收方使用带有 `Reason Code` 0x82 (Protocol Error) 的 `DISCONNECT`，如 [第 4.13 节](#413-error-handling)中所述。
   b. 如果报文包含非零长度的 `Topic Name`，接收方使用该 `Topic Name` 处理报文，并将其对 `Topic Alias` 的映射设置为传入报文中的 `Topic Name`。

**非规范性说明**

如果 `Server` 将 Application Message 分发给使用不支持本规范提供的 properties 或其他功能的不同协议级别（如 MQTT V3.1.1）的 `Client`，Application Message 中的某些信息可能会丢失，依赖此信息的应用程序可能无法正常工作。

`Client` 禁止发送超过 `Receive Maximum` 个尚未从 `Server` 收到 `PUBACK`、`PUBCOMP` 或带有 `Reason Code` 128 或更高的 `PUBREC` 的 `QoS` 1 和 `QoS` 2 `PUBLISH` 报文 [MQTT-3.3.4-7]。如果收到超过 `Receive Maximum` 个尚未发送 `PUBACK` 或 `PUBCOMP` 作为响应的 `QoS` 1 和 `QoS` 2 `PUBLISH` 报文，`Server` 使用带有 `Reason Code` 0x93 (`Receive Maximum` exceeded) 的 `DISCONNECT` 报文，如 [第 4.13 节](#413-error-handling)错误处理中所述。有关流量控制的更多信息，请参阅 [第 4.9 节](#49-flow-control)。

`Client` 禁止由于已发送 `Receive Maximum` 个 `PUBLISH` 报文而未收到确认就延迟发送除 `PUBLISH` 报文以外的任何报文 [MQTT-3.3.4-8]。`Receive Maximum` 的值仅适用于当前 Network Connection。

**非规范性说明**

`Client` 可能选择在未收到确认的情况下向 `Server` 发送少于 `Receive Maximum` 条消息，即使它有超过此数量的消息可用于发送。

**非规范性说明**

`Client` 可能选择在暂停发送 `QoS` 1 和 `QoS` 2 `PUBLISH` 报文时暂停发送 `QoS` 0 `PUBLISH` 报文。

**非规范性说明**

如果 `Client` 在收到 `CONNACK` 报文之前发送 `QoS` 1 或 `QoS` 2 `PUBLISH` 报文，它可能会因为发送了超过 `Receive Maximum` 条发布而被断开连接。

`Server` 禁止发送超过 `Receive Maximum` 个尚未从 `Client` 收到 `PUBACK`、`PUBCOMP` 或带有 `Reason Code` 128 或更高的 `PUBREC` 的 `QoS` 1 和 `QoS` 2 `PUBLISH` 报文 [MQTT-3.3.4-9]。如果收到超过 `Receive Maximum` 个尚未发送 `PUBACK` 或 `PUBCOMP` 作为响应的 `QoS` 1 和 `QoS` 2 `PUBLISH` 报文，`Client` 使用带有 `Reason Code` 0x93 (`Receive Maximum` exceeded) 的 `DISCONNECT` 报文，如 [第 4.13 节](#413-error-handling)错误处理中所述。有关流量控制的更多信息，请参阅 [第 4.9 节](#49-flow-control)。

`Server` 禁止由于已发送 `Receive Maximum` 个 `PUBLISH` 报文而未收到确认就延迟发送除 `PUBLISH` 报文以外的任何报文 [MQTT-3.3.4-10]。

**非规范性说明**

`Server` 可能选择在未收到确认的情况下向 `Client` 发送少于 `Receive Maximum` 条消息，即使它有超过此数量的消息可用于发送。

**非规范性说明**

`Server` 可能选择在暂停发送 `QoS` 1 和 `QoS` 2 `PUBLISH` 报文时暂停发送 `QoS` 0 `PUBLISH` 报文。

## 3.4 PUBACK - 发布确认

`PUBACK` 报文是对 `QoS` 1 的 `PUBLISH` 报文的响应。

### 3.4.1 PUBACK `Fixed Header`

**图 3-10 - `PUBACK` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (4) | Reserved |
| | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

**`Remaining Length` 字段**

这是 `Variable Header` 的长度，编码为 `Variable Byte Integer`。

### 3.4.2 PUBACK `Variable Header`

`PUBACK` 报文的 `Variable Header` 按顺序包含以下字段：来自正在被确认的 `PUBLISH` 报文的 `Packet Identifier`、PUBACK `Reason Code`、Property Length 和 Properties。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

**图 3-11 - `PUBACK` 报文 `Variable Header`**

| | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | `Packet Identifier` MSB |
| byte 2 | `Packet Identifier` LSB |
| byte 3 | PUBACK `Reason Code` |
| byte 4 | Property Length |

#### 3.4.2.1 PUBACK `Reason Code`

`Variable Header` 的 byte 3 是 PUBACK `Reason Code`。如果 `Remaining Length` 为 2，则没有 `Reason Code`，使用值 0x00 (Success)。

**表 3-4 - PUBACK `Reason Code`s**

| 值 | Hex | `Reason Code` 名称 | 描述 |
|----|-----|------------------|------|
| 0 | 0x00 | Success | 消息被接受。`QoS` 1 消息的发布继续进行。 |
| 16 | 0x10 | No matching subscribers | 消息被接受但没有订阅者。这仅由 `Server` 发送。如果 `Server` 知道没有匹配的订阅者，它可以使用此 `Reason Code` 代替 0x00 (Success)。 |
| 128 | 0x80 | Unspecified error | 接收方不接受该发布，但要么不想透露原因，要么与其他值都不匹配。 |
| 131 | 0x83 | Implementation specific error | `PUBLISH` 有效但接收方不愿意接受。 |
| 135 | 0x87 | Not authorized | `PUBLISH` 未被授权。 |
| 144 | 0x90 | `Topic Name` invalid | `Topic Name` 格式正确，但不被此 `Client` 或 `Server` 接受。 |
| 145 | 0x91 | `Packet Identifier` in use | `Packet Identifier` 已在使用中。这可能表示 `Client` 和 `Server` 之间的 `Session` State 不匹配。 |
| 151 | 0x97 | Quota exceeded | 已超出实现或管理设定的限制。 |
| 153 | 0x99 | `Payload` format invalid | `Payload` 格式与指定的 Payload Format Indicator 不匹配。 |

发送 `PUBACK` 报文的 `Client` 或 `Server` 必须使用其中一个 PUBACK `Reason Code` [MQTT-3.4.2-1]。如果 `Reason Code` 为 0x00 (Success) 且没有 Properties，则可以省略 `Reason Code` 和 Property Length。在这种情况下，`PUBACK` 的 `Remaining Length` 为 2。

#### 3.4.2.2 PUBACK Properties

##### 3.4.2.2.1 Property Length

`PUBACK` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。如果 `Remaining Length` 小于 4，则没有 Property Length，使用值 0。

##### 3.4.2.2.2 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示与此响应关联的原因的 `UTF-8 Encoded String`。此 Reason String 是人类可读的字符串，设计用于诊断，不打算被接收方解析。

发送方使用此值向接收方提供额外信息。如果这会使 `PUBACK` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.4.2-2]。多次包含 Reason String 是协议错误。

##### 3.4.2.2.3 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此 property 可用于提供额外的诊断或其他信息。如果这会使 `PUBACK` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.4.2-3]。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

### 3.4.3 PUBACK `Payload`

`PUBACK` 报文没有 `Payload`。

### 3.4.4 PUBACK Actions

这在 [第 4.3.2 节](#432-qos-1-at-least-once-delivery)中描述。

## 3.5 PUBREC - 发布已接收（`QoS` 2 传递第 1 部分）

`PUBREC` 报文是对 `QoS` 2 的 `PUBLISH` 报文的响应。它是 `QoS` 2 协议交换的第二个报文。

### 3.5.1 PUBREC `Fixed Header`

**图 3-12 - `PUBREC` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (5) | Reserved |
| | 0 | 1 | 0 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

**`Remaining Length` 字段**

这是 `Variable Header` 的长度，编码为 `Variable Byte Integer`。

### 3.5.2 PUBREC `Variable Header`

`PUBREC` 报文的 `Variable Header` 按顺序包含以下字段：来自正在被确认的 `PUBLISH` 报文的 `Packet Identifier`、PUBREC `Reason Code` 和 Properties。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

**图 3-13 - `PUBREC` 报文 `Variable Header`**

| | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | `Packet Identifier` MSB |
| byte 2 | `Packet Identifier` LSB |
| byte 3 | PUBREC `Reason Code` |
| byte 4 | Property Length |

#### 3.5.2.1 PUBREC `Reason Code`

`Variable Header` 的 byte 3 是 PUBREC `Reason Code`。如果 `Remaining Length` 为 2，则 Publish `Reason Code` 的值为 0x00 (Success)。

**表 3-5 - PUBREC `Reason Code`s**

| 值 | Hex | `Reason Code` 名称 | 描述 |
|----|-----|------------------|------|
| 0 | 0x00 | Success | 消息被接受。`QoS` 2 消息的发布继续进行。 |
| 16 | 0x10 | No matching subscribers | 消息被接受但没有订阅者。这仅由 `Server` 发送。如果 `Server` 知道没有匹配的订阅者，它可以使用此 `Reason Code` 代替 0x00 (Success)。 |
| 128 | 0x80 | Unspecified error | 接收方不接受该发布，但要么不想透露原因，要么与其他值都不匹配。 |
| 131 | 0x83 | Implementation specific error | `PUBLISH` 有效但接收方不愿意接受。 |
| 135 | 0x87 | Not authorized | `PUBLISH` 未被授权。 |
| 144 | 0x90 | `Topic Name` invalid | `Topic Name` 格式正确，但不被此 `Client` 或 `Server` 接受。 |
| 145 | 0x91 | `Packet Identifier` in use | `Packet Identifier` 已在使用中。这可能表示 `Client` 和 `Server` 之间的 `Session` State 不匹配。 |
| 151 | 0x97 | Quota exceeded | 已超出实现或管理设定的限制。 |
| 153 | 0x99 | `Payload` format invalid | `Payload` 格式与 Payload Format Indicator 中指定的格式不匹配。 |

发送 `PUBREC` 报文的 `Client` 或 `Server` 必须使用其中一个 PUBREC `Reason Code` 值 [MQTT-3.5.2-1]。如果 `Reason Code` 为 0x00 (Success) 且没有 Properties，则可以省略 `Reason Code` 和 Property Length。在这种情况下，`PUBREC` 的 `Remaining Length` 为 2。

#### 3.5.2.2 PUBREC Properties

##### 3.5.2.2.1 Property Length

`PUBREC` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。如果 `Remaining Length` 小于 4，则没有 Property Length，使用值 0。

##### 3.5.2.2.2 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示与此响应关联的原因的 `UTF-8 Encoded String`。此 Reason String 是人类可读的，设计用于诊断，不应该被接收方解析。

发送方使用此值向接收方提供额外信息。如果这会使 `PUBREC` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.5.2-2]。多次包含 Reason String 是协议错误。

##### 3.5.2.2.3 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此 property 可用于提供额外的诊断或其他信息。如果这会使 `PUBREC` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.5.2-3]。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

### 3.5.3 PUBREC `Payload`

`PUBREC` 报文没有 `Payload`。

### 3.5.4 PUBREC Actions

这在 [第 4.3.3 节](#433-qos-2-exactly-once-delivery)中描述。

## 3.6 PUBREL - 发布释放（`QoS` 2 传递第 2 部分）

`PUBREL` 报文是对 `PUBREC` 报文的响应。它是 `QoS` 2 协议交换的第三个报文。

### 3.6.1 PUBREL `Fixed Header`

**图 3-14 - `PUBREL` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (6) | Reserved |
| | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 |
| byte 2 | `Remaining Length` |

`PUBREL` 报文 `Fixed Header` 的 bits 3,2,1 和 0 是保留的，必须分别设置为 0,0,1 和 0。`Server` 必须将任何其他值视为格式错误并关闭 Network Connection [MQTT-3.6.1-1]。

**`Remaining Length` 字段**

这是 `Variable Header` 的长度，编码为 `Variable Byte Integer`。

### 3.6.2 PUBREL `Variable Header`

`PUBREL` 报文的 `Variable Header` 按顺序包含以下字段：来自正在被确认的 `PUBREC` 报文的 `Packet Identifier`、PUBREL `Reason Code` 和 Properties。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

**图 3-15 - `PUBREL` 报文 `Variable Header`**

| | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | `Packet Identifier` MSB |
| byte 2 | `Packet Identifier` LSB |
| byte 3 | PUBREL `Reason Code` |
| byte 4 | Property Length |

#### 3.6.2.1 PUBREL `Reason Code`

`Variable Header` 的 byte 3 是 PUBREL `Reason Code`。如果 `Remaining Length` 为 2，则使用值 0x00 (Success)。

**表 3-6 - PUBREL `Reason Code`s**

| 值 | Hex | `Reason Code` 名称 | 描述 |
|----|-----|------------------|------|
| 0 | 0x00 | Success | 消息已释放。 |
| 146 | 0x92 | `Packet Identifier` not found | `Packet Identifier` 未知。这在恢复期间不是错误，但在其他时候表示 `Client` 和 `Server` 之间的 `Session` State 不匹配。 |

发送 `PUBREL` 报文的 `Client` 或 `Server` 必须使用其中一个 PUBREL `Reason Code` 值 [MQTT-3.6.2-1]。如果 `Reason Code` 为 0x00 (Success) 且没有 Properties，则可以省略 `Reason Code` 和 Property Length。在这种情况下，`PUBREL` 的 `Remaining Length` 为 2。

#### 3.6.2.2 PUBREL Properties

##### 3.6.2.2.1 Property Length

`PUBREL` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。如果 `Remaining Length` 小于 4，则没有 Property Length，使用值 0。

##### 3.6.2.2.2 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示与此响应关联的原因的 `UTF-8 Encoded String`。此 Reason String 是人类可读的，设计用于诊断，不应该被接收方解析。

发送方使用此值向接收方提供额外信息。如果这会使 `PUBREL` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 Property [MQTT-3.6.2-2]。多次包含 Reason String 是协议错误。

##### 3.6.2.2.3 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此 property 可用于为 `PUBREL` 提供额外的诊断或其他信息。如果这会使 `PUBREL` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.6.2-3]。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

### 3.6.3 PUBREL `Payload`

`PUBREL` 报文没有 `Payload`。

### 3.6.4 PUBREL Actions

这在 [第 4.3.3 节](#433-qos-2-exactly-once-delivery)中描述。

## 3.7 PUBCOMP - 发布完成（`QoS` 2 传递第 3 部分）

`PUBCOMP` 报文是对 `PUBREL` 报文的响应。它是 `QoS` 2 协议交换的第四个也是最后一个报文。

### 3.7.1 PUBCOMP `Fixed Header`

**图 3-16 - `PUBCOMP` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (7) | Reserved |
| | 0 | 1 | 1 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

**`Remaining Length` 字段**

这是 `Variable Header` 的长度，编码为 `Variable Byte Integer`。

### 3.7.2 PUBCOMP `Variable Header`

`PUBCOMP` 报文的 `Variable Header` 按顺序包含以下字段：来自正在被确认的 `PUBREL` 报文的 `Packet Identifier`、PUBCOMP `Reason Code` 和 Properties。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

**图 3-17 - `PUBCOMP` 报文 `Variable Header`**

| | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | `Packet Identifier` MSB |
| byte 2 | `Packet Identifier` LSB |
| byte 3 | PUBCOMP `Reason Code` |
| byte 4 | Property Length |

#### 3.7.2.1 PUBCOMP `Reason Code`

`Variable Header` 的 byte 3 是 PUBCOMP `Reason Code`。如果 `Remaining Length` 为 2，则使用值 0x00 (Success)。

**表 3-7 - PUBCOMP `Reason Code`s**

| 值 | Hex | `Reason Code` 名称 | 描述 |
|----|-----|------------------|------|
| 0 | 0x00 | Success | `Packet Identifier` 已释放。`QoS` 2 消息的发布完成。 |
| 146 | 0x92 | `Packet Identifier` not found | `Packet Identifier` 未知。这在恢复期间不是错误，但在其他时候表示 `Client` 和 `Server` 之间的 `Session` State 不匹配。 |

发送 `PUBCOMP` 报文的 `Client` 或 `Server` 必须使用其中一个 PUBCOMP `Reason Code` 值 [MQTT-3.7.2-1]。如果 `Reason Code` 为 0x00 (Success) 且没有 Properties，则可以省略 `Reason Code` 和 Property Length。在这种情况下，`PUBCOMP` 的 `Remaining Length` 为 2。

#### 3.7.2.2 PUBCOMP Properties

##### 3.7.2.2.1 Property Length

`PUBCOMP` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。如果 `Remaining Length` 小于 4，则没有 Property Length，使用值 0。

##### 3.7.2.2.2 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示与此响应关联的原因的 `UTF-8 Encoded String`。此 Reason String 是人类可读的字符串，设计用于诊断，不应该被接收方解析。

发送方使用此值向接收方提供额外信息。如果这会使 `PUBCOMP` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 Property [MQTT-3.7.2-2]。多次包含 Reason String 是协议错误。

##### 3.7.2.2.3 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此 property 可用于提供额外的诊断或其他信息。如果这会使 `PUBCOMP` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.7.2-3]。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

### 3.7.3 PUBCOMP `Payload`

`PUBCOMP` 报文没有 `Payload`。

### 3.7.4 PUBCOMP Actions

这在 [第 4.3.3 节](#433-qos-2-exactly-once-delivery)中描述。

## 3.8 SUBSCRIBE - 订阅请求

`SUBSCRIBE` 报文从 `Client` 发送到 `Server`，用于创建一个或多个 Subscription。每个 Subscription 注册 `Client` 对一个或多个 Topic 的兴趣。`Server` 向 `Client` 发送 `PUBLISH` 报文，以转发发布到与这些 Subscription 匹配的 Topic 的 Application Message。`SUBSCRIBE` 报文还（为每个 Subscription）指定了 `Server` 可以向 `Client` 发送 Application Message 的最大 `QoS`。

### 3.8.1 SUBSCRIBE `Fixed Header`

**图 3-18 - `SUBSCRIBE` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (8) | Reserved |
| | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| byte 2 | `Remaining Length` |

`SUBSCRIBE` 报文 `Fixed Header` 的 Bits 3,2,1 和 0 是保留的，必须分别设置为 0,0,1 和 0。`Server` 必须将任何其他值视为格式错误并关闭 Network Connection [MQTT-3.8.1-1]。

**`Remaining Length` field**

这是 `Variable Header` 加上 `Payload` 的长度，编码为 `Variable Byte Integer`。

### 3.8.2 SUBSCRIBE `Variable Header`

`SUBSCRIBE` 报文的 `Variable Header` 按顺序包含以下字段：`Packet Identifier` 和 Properties。[第 2.2.1 节](#221-packet-identifier)提供了有关 `Packet Identifier` 的更多信息。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

**非规范性示例**

图 3-19 显示了一个 `Packet Identifier` 为 10 且没有 properties 的 `SUBSCRIBE` variable header 示例。

**图 3-19 - SUBSCRIBE `Variable Header` 示例**

| | Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|-------------|---|---|---|---|---|---|---|---|
| `Packet Identifier` |
| byte 1 | `Packet Identifier` MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Packet Identifier` LSB (10) | 0 | 0 | 0 | 0 | 1 | 0 | 1 | 0 |
| byte 3 | Property Length (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

#### 3.8.2.1 SUBSCRIBE Properties

##### 3.8.2.1.1 Property Length

`SUBSCRIBE` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.8.2.1.2 `Subscription Identifier`

**11 (0x0B) 字节**，`Subscription Identifier` 的标识符。

后跟表示订阅标识符的 `Variable Byte Integer`。`Subscription Identifier` 的值可以是 1 到 268,435,455。如果 `Subscription Identifier` 的值为 0，则是协议错误。多次包含 `Subscription Identifier` 是协议错误。

`Subscription Identifier` 与此 `SUBSCRIBE` 报文创建或修改的任何订阅相关联。如果有 `Subscription Identifier`，则将其与订阅一起存储。如果未指定此 property，则将 `Subscription Identifier` 的缺失与订阅一起存储。

有关 `Subscription Identifier` 处理的更多信息，请参阅 [第 3.8.3.1 节](#3831-subscription-options)。

##### 3.8.2.1.3 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。

`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

**非规范性评论**

`SUBSCRIBE` 报文上的 `User Properties` 可用于从 `Client` 向 `Server` 发送与订阅相关的 properties。这些 properties 的含义未由本规范定义。

### 3.8.3 SUBSCRIBE `Payload`

`SUBSCRIBE` 报文的 `Payload` 包含一个 `Topic Filter` 列表，指示 `Client` 希望订阅的 Topic。`Topic Filter` 必须是 `UTF-8 Encoded String` [MQTT-3.8.3-1]。每个 `Topic Filter` 后跟一个 Subscription Options 字节。

`Payload` 必须包含至少一个 `Topic Filter` 和 Subscription Options 对 [MQTT-3.8.3-2]。没有 `Payload` 的 `SUBSCRIBE` 报文是协议错误。有关处理错误的信息，请参阅 [第 4.13 节](#413-errors)。

#### 3.8.3.1 Subscription Options

Subscription Options 的 Bits 0 和 1 表示 Maximum `QoS` 字段。这给出了 `Server` 可以向 `Client` 发送 Application Message 的最大 `QoS` 级别。如果 Maximum `QoS` 字段的值为 3，则是协议错误。

Subscription Options 的 Bit 2 表示 No Local 选项。如果值为 1，禁止将 Application Messages 转发给 ClientID 等于发布连接的 ClientID 的连接 [MQTT-3.8.3-3]。在 Shared Subscription 上将 No Local 位设置为 1 是协议错误 [MQTT-3.8.3-4]。

Subscription Options 的 Bit 3 表示 Retain As Published 选项。如果为 1，使用此订阅转发的 Application Messages 保持它们发布时的 `RETAIN` 标志。如果为 0，使用此订阅转发的 Application Messages 将 `RETAIN` 标志设置为 0。建立订阅时发送的保留消息将 `RETAIN` 标志设置为 1。

Subscription Options 的 Bits 4 和 5 表示 Retain Handling 选项。此选项指定在建立订阅时是否发送保留消息。这不影响订阅后任何时候保留消息的发送。如果没有与 `Topic Filter` 匹配的保留消息，所有这些值的作用相同。这些值是：

- 0 = 在订阅时发送保留消息
- 1 = 仅当订阅当前不存在时，在订阅时发送保留消息
- 2 = 在订阅时不发送保留消息

发送 Retain Handling 值为 3 是协议错误。

Subscription Options 字节的 Bits 6 和 7 保留供将来使用。如果 `Payload` 中的任何 Reserved 位非零，`Server` 必须将 `SUBSCRIBE` 报文视为格式错误 [MQTT-3.8.3-5]。

**非规范性评论**

No Local 和 Retain As Published 订阅选项可用于实现桥接，其中 `Client` 将消息转发到另一个 `Server`。

**非规范性评论**

对于现有订阅不发送保留消息在重新连接时很有用，此时 `Client` 不确定在上一次连接到 `Session` 时订阅是否已完成。

**非规范性评论**

由于新订阅而不发送存储的保留消息在 `Client` 希望接收更改通知且不需要知道初始状态时很有用。

**非规范性评论**

对于表示不支持保留消息的 `Server`，Retain As Published 和 Retain Handling 的所有有效值都给出相同的结果，即在订阅时不发送任何保留消息，并将所有消息的 `RETAIN` 标志设置为 0。

**图 3-20 - `SUBSCRIBE` 报文 `Payload` 格式**

| Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-------------|---|---|---|---|---|---|---|---|
| Topic Filter |
| byte 1 | Length MSB |
| byte 2 | Length LSB |
| bytes 3..N | `Topic Filter` |
| Subscription Options |
| | Reserved | Retain Handling | RAP | NL | `QoS` |
| byte N+1 | 0 | 0 | X | X | X | X | X | X |

RAP 表示 Retain as Published。

NL 表示 No Local。

**非规范性示例**

图 3-21 显示了带有两个 `Topic Filter` 的 `SUBSCRIBE` `Payload` 示例。第一个是 "a/b"，`QoS` 为 1，第二个是 "c/d"，`QoS` 为 2。

**图 3-21 - `Payload` 字节格式非规范性示例**

（详细字节表示略，参见原文）

### 3.8.4 SUBSCRIBE Actions

当 `Server` 从 `Client` 收到 `SUBSCRIBE` 报文时，`Server` 必须以 `SUBACK` 报文响应 [MQTT-3.8.4-1]。`SUBACK` 报文必须具有与其确认的 `SUBSCRIBE` 报文相同的 `Packet Identifier` [MQTT-3.8.4-2]。

`Server` 可以在发送 `SUBACK` 报文之前开始发送与 Subscription 匹配的 `PUBLISH` 报文。

如果 `Server` 收到的 `SUBSCRIBE` 报文包含与当前 `Session` 的 Non-shared Subscription 的 `Topic Filter` 相同的 `Topic Filter`，则它必须用新的 Subscription 替换该现有 Subscription [MQTT-3.8.4-3]。新 Subscription 中的 `Topic Filter` 将与前一个 Subscription 中的相同，尽管其 Subscription Options 可能不同。如果 Retain Handling 选项为 0，必须重新发送与 `Topic Filter` 匹配的任何现有保留消息，但由于替换 Subscription，禁止丢失 Application Messages [MQTT-3.8.4-4]。

如果 `Server` 收到与当前 `Session` 的任何 `Topic Filter` 不相同的 Non-shared `Topic Filter`，则会创建新的 Non-shared Subscription。如果 Retain Handling 选项不是 2，则向 `Client` 发送所有匹配的保留消息。

如果 `Server` 收到与 `Server` 上已存在的 Shared Subscription 的 `Topic Filter` 相同的 `Topic Filter`，则将该 `Session` 添加为该 Shared Subscription 的订阅者。不发送保留消息。

如果 `Server` 收到与任何现有 Shared Subscription 的 `Topic Filter` 不相同的 Shared Subscription `Topic Filter`，则会创建新的 Shared Subscription。将该 `Session` 添加为该 Shared Subscription 的订阅者。不发送保留消息。

有关 Shared Subscription 的更多详细信息，请参阅 [第 4.8 节](#48-shared-subscriptions)。

如果 `Server` 收到包含多个 `Topic Filter` 的 `SUBSCRIBE` 报文，它必须处理该报文，就像收到了一系列多个 `SUBSCRIBE` 报文一样，只是它将它们的响应组合成单个 `SUBACK` 响应 [MQTT-3.8.4-5]。

`Server` 发送给 `Client` 的 `SUBACK` 报文必须包含每个 `Topic Filter`/Subscription Option 对的 `Reason Code` [MQTT-3.8.4-6]。此 `Reason Code` 必须显示为该 Subscription 授予的最大 `QoS` 或指示订阅失败 [MQTT-3.8.4-7]。`Server` 可能会授予比订阅者请求的更低的 Maximum `QoS`。响应 Subscription 发送的 Application Messages 的 `QoS` 必须是原始发布消息的 `QoS` 与 `Server` 授予的 Maximum `QoS` 中的较小值 [MQTT-3.8.4-8]。如果原始消息以 `QoS` 1 发布且授予的最大 `QoS` 为 `QoS` 0，`Server` 允许向订阅者发送消息的重复副本。

**非规范性评论**

如果订阅 `Client` 已被授予特定 `Topic Filter` 的最大 `QoS` 1，则以 `QoS` 0 向 `Client` 交付匹配该过滤器的 Application Message。这意味着 `Client` 最多收到消息的一个副本。另一方面，发布到同一 Topic 的 `QoS` 2 Message 被 `Server` 降级为 `QoS` 1 以交付给 `Client`，因此 `Client` 可能会收到该消息的重复副本。

**非规范性评论**

如果订阅 `Client` 已被授予最大 `QoS` 0，则最初以 `QoS` 2 发布的 Application Message 可能在到 `Client` 的跃点上丢失，但 `Server` 永远不应该发送该消息的副本。发布到同一 Topic 的 `QoS` 1 Message 可能在传输到该 `Client` 时丢失或重复。

**非规范性评论**

以 `QoS` 2 订阅 `Topic Filter` 等同于说"我希望以发布时的 `QoS` 接收匹配此过滤器的消息"。这意味着发布者负责确定消息可以交付的最大 `QoS`，但订阅者能够要求 `Server` 将 `QoS` 降级为更适合其用途的级别。

`Subscription Identifier` 是 `Server` 中 `Session` State 的一部分，并返回给接收匹配 `PUBLISH` 报文的 `Client`。当 `Server` 收到 `UNSUBSCRIBE` 报文、当 `Server` 从 `Client` 收到相同 `Topic Filter` 但具有不同 `Subscription Identifier` 或没有 `Subscription Identifier` 的 `SUBSCRIBE` 报文时，或者当 `Server` 在 `CONNACK` 报文中发送 `Session` Present 0 时，它们会从 `Server` 的 `Session` State 中删除。

`Subscription Identifier` 不构成 `Client` 中 `Client` 的 `Session` State 的一部分。在有用的实现中，`Client` 会将 `Subscription Identifier` 与其他 `Client` 端状态关联，此状态通常在 `Client` 取消订阅时、当 `Client` 使用不同的标识符或没有标识符订阅同一 `Topic Filter` 时、或者当 `Client` 在 `CONNACK` 报文中收到 `Session` Present 0 时删除。

`Server` 不需要在重新传输的 `PUBLISH` 报文中使用相同的 `Subscription Identifier` 集合。`Client` 可以通过发送包含与当前 `Session` 中现有 Subscription 的 `Topic Filter` 相同的 `Topic Filter` 的 `SUBSCRIBE` 报文来重建 Subscription。如果 `Client` 在 `PUBLISH` 报文的初始传输之后重建了订阅，并使用了不同的 `Subscription Identifier`，则 `Server` 允许在任何重新传输中使用来自第一次传输的标识符。或者，`Server` 允许在重新传输期间使用新标识符。`Server` 不允许在发送包含新标识符的 `PUBLISH` 报文后恢复使用旧标识符。

**非规范性评论**

`Subscription Identifier` 的使用场景，用于说明。

- `Client` 实现通过其编程接口指示发布匹配了多个订阅。`Client` 实现每次进行订阅时生成新标识符。如果返回的发布携带多个 `Subscription Identifier`，则发布匹配了多个订阅。

- `Client` 实现允许订阅者将消息定向到与订阅关联的回调。`Client` 实现生成唯一映射标识符到回调的标识符。当收到发布时，它使用 `Subscription Identifier` 确定驱动哪个回调。

- `Client` 实现在交付发布消息时将用于进行订阅的 Topic 字符串返回给应用程序。为此，`Client` 生成唯一标识 `Topic Filter` 的标识符。当收到发布时，`Client` 实现使用标识符查找原始 `Topic Filter` 并将它们返回给 `Client` 应用程序。

- 网关将从 `Server` 收到的发布转发给已订阅网关的 `Client`。网关实现维护它转发的每个唯一 `Topic Filter` 到它也收到的 ClientID, `Subscription Identifier` 对集合的映射。它为转发给 `Server` 的每个 `Topic Filter` 生成唯一标识符。当收到发布时，网关使用从 `Server` 收到的 `Subscription Identifier` 查找与它们关联的 `Client Identifier`, `Subscription Identifier` 对。它将这些添加到发送给 `Client` 的 `PUBLISH` 报文中。如果上游 `Server` 发送了多个 `PUBLISH` 报文，因为消息匹配了多个订阅，则此行为会镜像到 `Client`。

## 3.9 SUBACK - 订阅确认

`SUBACK` 报文由 `Server` 发送给 `Client`，用于确认收到并处理 `SUBSCRIBE` 报文。

`SUBACK` 报文包含一个 `Reason Code` 列表，指定为每个 Subscription 授予的最大 `QoS` 级别或发现的错误。

### 3.9.1 SUBACK `Fixed Header`

**图 3-22 - `SUBACK` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (9) | Reserved |
| | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

**`Remaining Length` field**

这是 `Variable Header` 加上 `Payload` 的长度，编码为 `Variable Byte Integer`。

### 3.9.2 SUBACK `Variable Header`

`SUBACK` 报文的 `Variable Header` 按顺序包含以下字段：来自被确认的 `SUBSCRIBE` 报文的 `Packet Identifier` 和 Properties。

#### 3.9.2.1 SUBACK Properties

##### 3.9.2.1.1 Property Length

`SUBACK` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.9.2.1.2 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示与此响应关联的原因的 `UTF-8 Encoded String`。此 Reason String 是人类可读的字符串，设计用于诊断，不应该被 `Client` 解析。

`Server` 使用此值向 `Client` 提供额外信息。如果这会使 `SUBACK` 报文的大小超过 `Client` 指定的 `Maximum Packet Size`，`Server` 禁止发送此 Property [MQTT-3.9.2-1]。多次包含 Reason String 是协议错误。

##### 3.9.2.1.3 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此 property 可用于提供额外的诊断或其他信息。如果这会使 `SUBACK` 报文的大小超过 `Client` 指定的 `Maximum Packet Size`，`Server` 禁止发送此 property [MQTT-3.9.2-2]。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

**图 3-23 - `SUBACK` 报文 `Variable Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | `Packet Identifier` MSB |
| byte 2 | `Packet Identifier` LSB |

### 3.9.3 SUBACK `Payload`

`Payload` 包含一个 `Reason Code` 列表。每个 `Reason Code` 对应于被确认的 `SUBSCRIBE` 报文中的一个 `Topic Filter`。`SUBACK` 报文中 `Reason Code` 的顺序必须与 `SUBSCRIBE` 报文中 `Topic Filter` 的顺序匹配 [MQTT-3.9.3-1]。

**表 3-8 - Subscribe `Reason Code`s**

| 值 | Hex | `Reason Code` 名称 | 描述 |
|----|-----|------------------|------|
| 0 | 0x00 | Granted `QoS` 0 | 订阅被接受，发送的最大 `QoS` 将为 `QoS` 0。这可能是比请求的更低的 `QoS`。 |
| 1 | 0x01 | Granted `QoS` 1 | 订阅被接受，发送的最大 `QoS` 将为 `QoS` 1。这可能是比请求的更低的 `QoS`。 |
| 2 | 0x02 | Granted `QoS` 2 | 订阅被接受，任何接收到的 `QoS` 都将发送到此订阅。 |
| 128 | 0x80 | Unspecified error | 订阅未被接受，`Server` 不希望透露原因或没有其他 `Reason Code` 适用。 |
| 131 | 0x83 | Implementation specific error | `SUBSCRIBE` 有效但 `Server` 不接受它。 |
| 135 | 0x87 | Not authorized | `Client` 未被授权进行此订阅。 |
| 143 | 0x8F | `Topic Filter` invalid | `Topic Filter` 格式正确但不允许此 `Client` 使用。 |
| 145 | 0x91 | `Packet Identifier` in use | 指定的 `Packet Identifier` 已在使用中。 |
| 151 | 0x97 | Quota exceeded | 已超过实现或管理限制。 |
| 158 | 0x9E | Shared Subscriptions not supported | `Server` 不支持此 `Client` 的 Shared Subscriptions。 |
| 161 | 0xA1 | Subscription Identifiers not supported | `Server` 不支持 `Subscription Identifiers`；订阅未被接受。 |
| 162 | 0xA2 | Wildcard Subscriptions not supported | `Server` 不支持 Wildcard Subscriptions；订阅未被接受。 |

发送 `SUBACK` 报文的 `Server` 必须为收到的每个 `Topic Filter` 使用其中一个 Subscribe `Reason Code`s [MQTT-3.9.3-2]。

**非规范性评论**

对于对应的 `SUBSCRIBE` 报文中的每个 `Topic Filter`，总是有一个 `Reason Code`。如果 `Reason Code` 不是特定于 `Topic Filter`（例如 0x91 (`Packet Identifier` in use)），则为每个 `Topic Filter` 设置它。

## 3.10 UNSUBSCRIBE - 取消订阅请求

`UNSUBSCRIBE` 报文由 `Client` 发送给 `Server`，用于取消订阅 Topic。

### 3.10.1 UNSUBSCRIBE `Fixed Header`

**图 3-28 - `UNSUBSCRIBE` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (10) | Reserved |
| | 1 | 0 | 1 | 0 | 0 | 0 | 1 | 0 |
| byte 2 | `Remaining Length` |

`UNSUBSCRIBE` 报文 `Fixed Header` 的 Bits 3,2,1 和 0 是保留的，必须分别设置为 0,0,1 和 0。`Server` 必须将任何其他值视为格式错误并关闭 Network Connection [MQTT-3.10.1-1]。

**`Remaining Length` field**

这是 `Variable Header`（2 字节）加上 `Payload` 的长度，编码为 `Variable Byte Integer`。

### 3.10.2 UNSUBSCRIBE `Variable Header`

`UNSUBSCRIBE` 报文的 `Variable Header` 按顺序包含以下字段：`Packet Identifier` 和 Properties。[第 2.2.1 节](#221-packet-identifier)提供了有关 `Packet Identifier` 的更多信息。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

#### 3.10.2.1 UNSUBSCRIBE Properties

##### 3.10.2.1.1 Property Length

`UNSUBSCRIBE` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.10.2.1.2 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。

`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

**非规范性评论**

`UNSUBSCRIBE` 报文上的 `User Properties` 可用于从 `Client` 向 `Server` 发送与订阅相关的 properties。这些 properties 的含义未由本规范定义。

### 3.10.3 UNSUBSCRIBE `Payload`

`UNSUBSCRIBE` 报文的 `Payload` 包含 `Client` 希望取消订阅的 `Topic Filter` 列表。`UNSUBSCRIBE` 报文中的 `Topic Filter` 必须是 `UTF-8 Encoded String`s [MQTT-3.10.3-1]，如 [第 1.5.4 节](#154-utf-8-encoded-string)所定义，连续打包。

`UNSUBSCRIBE` 报文的 `Payload` 必须包含至少一个 `Topic Filter` [MQTT-3.10.3-2]。没有 `Payload` 的 `UNSUBSCRIBE` 报文是协议错误。有关处理错误的信息，请参阅 [第 4.13 节](#413-errors)。

**非规范性示例**

图 3-30 显示了带有两个 `Topic Filter` "a/b" 和 "c/d" 的 `UNSUBSCRIBE` 报文的 `Payload`。

**图 3-30 - `Payload` 字节格式非规范性示例**

（详细字节表示略，参见原文）

### 3.10.4 UNSUBSCRIBE Actions

`UNSUBSCRIBE` 报文中提供的 `Topic Filter`（无论它们是否包含通配符）必须与 `Server` 为 `Client` 持有的当前 `Topic Filter` 集合逐字符比较。如果任何过滤器完全匹配，则必须删除其拥有的 Subscription [MQTT-3.10.4-1]，否则不会进行额外的处理。

当 `Server` 收到 `UNSUBSCRIBE` 时：

- 必须停止为 `Client` 交付添加任何与 `Topic Filter` 匹配的新消息 [MQTT-3.10.4-2]。
- 必须完成已经开始发送给 `Client` 的任何与 `Topic Filter` 匹配的 `QoS` 1 或 `QoS` 2 消息的交付 [MQTT-3.10.4-3]。
- 可以继续交付任何为 `Client` 交付缓冲的现有消息。

`Server` 必须通过发送 `UNSUBACK` 报文来响应 `UNSUBSCRIBE` 请求 [MQTT-3.10.4-4]。`UNSUBACK` 报文必须具有与 `UNSUBSCRIBE` 报文相同的 `Packet Identifier`。即使没有删除任何 Topic Subscription，`Server` 也必须响应 `UNSUBACK` [MQTT-3.10.4-5]。

如果 `Server` 收到包含多个 `Topic Filter` 的 `UNSUBSCRIBE` 报文，它必须处理该报文，就像收到了一系列多个 `UNSUBSCRIBE` 报文一样，只是它只发送一个 `UNSUBACK` 响应 [MQTT-3.10.4-6]。

如果 `Topic Filter` 表示 Shared Subscription，则此 `Session` 将从 Shared Subscription 分离。如果此 `Session` 是 Shared Subscription 关联的唯一 `Session`，则删除 Shared Subscription。有关 Shared Subscription 处理的描述，请参阅 [第 4.8.2 节](#482-shared-subscriptions)。

## 3.11 UNSUBACK - 取消订阅确认

`UNSUBACK` 报文由 `Server` 发送给 `Client`，用于确认收到 `UNSUBSCRIBE` 报文。

### 3.11.1 UNSUBACK `Fixed Header`

**图 3-31 - `UNSUBACK` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (11) | Reserved |
| | 1 | 0 | 1 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

**`Remaining Length` field**

这是 `Variable Header` 加上 `Payload` 的长度，编码为 `Variable Byte Integer`。

### 3.11.2 UNSUBACK `Variable Header`

`UNSUBACK` 报文的 `Variable Header` 按顺序包含以下字段：来自被确认的 `UNSUBSCRIBE` 报文的 `Packet Identifier` 和 Properties。

#### 3.11.2.1 UNSUBACK Properties

##### 3.11.2.1.1 Property Length

`UNSUBACK` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.11.2.1.2 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示与此响应关联的原因的 `UTF-8 Encoded String`。此 Reason String 是人类可读的字符串，设计用于诊断，不应该被 `Client` 解析。

`Server` 使用此值向 `Client` 提供额外信息。如果这会使 `UNSUBACK` 报文的大小超过 `Client` 指定的 `Maximum Packet Size`，`Server` 禁止发送此 Property [MQTT-3.11.2-1]。多次包含 Reason String 是协议错误。

##### 3.11.2.1.3 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此 property 可用于提供额外的诊断或其他信息。如果这会使 `UNSUBACK` 报文的大小超过 `Client` 指定的 `Maximum Packet Size`，`Server` 禁止发送此 property [MQTT-3.11.2-2]。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

### 3.11.3 UNSUBACK `Payload`

`Payload` 包含一个 `Reason Code` 列表。每个 `Reason Code` 对应于被确认的 `UNSUBSCRIBE` 报文中的一个 `Topic Filter`。`UNSUBACK` 报文中 `Reason Code` 的顺序必须与 `UNSUBSCRIBE` 报文中 `Topic Filter` 的顺序匹配 [MQTT-3.11.3-1]。

**表 3-9 - Unsubscribe `Reason Code`s**

| 值 | Hex | `Reason Code` 名称 | 描述 |
|----|-----|------------------|------|
| 0 | 0x00 | Success | 订阅已删除。 |
| 17 | 0x11 | No subscription existed | 没有与 `Topic Filter` 匹配的订阅。 |
| 128 | 0x80 | Unspecified error | 取消订阅未被接受，`Server` 不希望透露原因或没有其他 `Reason Code` 适用。 |
| 131 | 0x83 | Implementation specific error | `UNSUBSCRIBE` 有效但 `Server` 不接受它。 |
| 135 | 0x87 | Not authorized | `Client` 未被授权进行取消订阅。 |
| 143 | 0x8F | `Topic Filter` invalid | `Topic Filter` 格式正确但不允许此 `Client` 使用。 |
| 145 | 0x91 | `Packet Identifier` in use | 指定的 `Packet Identifier` 已在使用中。 |

发送 `UNSUBACK` 报文的 `Server` 必须为收到的每个 `Topic Filter` 使用其中一个 Unsubscribe `Reason Code`s [MQTT-3.11.3-2]。

**非规范性评论**

对于对应的 `UNSUBSCRIBE` 报文中的每个 `Topic Filter`，总是有一个 `Reason Code`。如果 `Reason Code` 不是特定于 `Topic Filter`（例如 0x91 (`Packet Identifier` in use)），则为每个 `Topic Filter` 设置它。

## 3.12 PINGREQ - PING 请求

`PINGREQ` 报文从 `Client` 发送到 `Server`。它可以用于：

- 在没有从 `Client` 向 `Server` 发送任何其他 MQTT Control Packets 的情况下，向 `Server` 指示 `Client` 是活跃的。
- 请求 `Server` 响应以确认它是活跃的。
- 测试网络以指示 Network Connection 是活跃的。

此报文用于 `Keep Alive` 处理。有关更多详细信息，请参阅 [第 3.1.2.10 节](#31210-keep-alive)。

### 3.12.1 PINGREQ `Fixed Header`

**图 3-33 - `PINGREQ` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (12) | Reserved |
| | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (0) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

### 3.12.2 PINGREQ `Variable Header`

`PINGREQ` 报文没有 `Variable Header`。

### 3.12.3 PINGREQ `Payload`

`PINGREQ` 报文没有 `Payload`。

### 3.12.4 PINGREQ Actions

`Server` 必须发送 `PINGRESP` 报文以响应 `PINGREQ` 报文 [MQTT-3.12.4-1]。

## 3.13 PINGRESP - PING 响应

`PINGRESP` 报文由 `Server` 发送给 `Client` 以响应 `PINGREQ` 报文。它指示 `Server` 是活跃的。

此报文用于 `Keep Alive` 处理。有关更多详细信息，请参阅 [第 3.1.2.10 节](#31210-keep-alive)。

### 3.13.1 PINGRESP `Fixed Header`

**图 3-34 - `PINGRESP` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (13) | Reserved |
| | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (0) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

### 3.13.2 PINGRESP `Variable Header`

`PINGRESP` 报文没有 `Variable Header`。

### 3.13.3 PINGRESP `Payload`

`PINGRESP` 报文没有 `Payload`。

### 3.13.4 PINGRESP Actions

`Client` 在收到此报文时不采取任何操作。

## 3.14 DISCONNECT - 断开连接通知

`DISCONNECT` 报文是从 `Client` 或 `Server` 发送的最后一个 MQTT Control Packet。它指示关闭 Network Connection 的原因。`Client` 或 `Server` 可以在关闭 Network Connection 之前发送 `DISCONNECT` 报文。如果在 `Client` 首先发送带有 `Reason Code` 0x00（正常断开连接）的 `DISCONNECT` 报文之前关闭 Network Connection，并且 Connection 有 `Will Message`，则会发布 `Will Message`。有关更多详细信息，请参阅 [第 3.1.2.5 节](#3125-will-message)。

`Server` 禁止在发送 `Reason Code` 小于 0x80 的 `CONNACK` 之后发送 `DISCONNECT` [MQTT-3.14.0-1]。

### 3.14.1 DISCONNECT `Fixed Header`

**图 3-35 - `DISCONNECT` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (14) | Reserved |
| | 1 | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

`Client` 或 `Server` 必须验证保留位设置为 0。如果它们不为零，则按照 [第 4.13 节](#413-errors)中的描述发送带有 `Reason Code` 0x81（格式错误）的 `DISCONNECT` 报文 [MQTT-3.14.1-1]。

**`Remaining Length` field**

这是 `Variable Header` 的长度，编码为 `Variable Byte Integer`。

### 3.14.2 DISCONNECT `Variable Header`

`DISCONNECT` 报文的 `Variable Header` 按顺序包含以下字段：Disconnect `Reason Code` 和 Properties。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

#### 3.14.2.1 Disconnect `Reason Code`

`Variable Header` 中的字节 1 是 Disconnect `Reason Code`。如果 `Remaining Length` 小于 1，则使用值 0x00（正常断开连接）。

单字节无符号 Disconnect `Reason Code` 字段的值如下所示。

**表 3-10 - Disconnect `Reason Code` 值**

| 值 | Hex | `Reason Code` 名称 | 发送方 | 描述 |
|----|-----|------------------|--------|------|
| 0 | 0x00 | Normal disconnection | `Client` 或 `Server` | 正常关闭连接。不发送 `Will Message`。 |
| 4 | 0x04 | Disconnect with Will Message | `Client` | `Client` 希望断开连接但要求 `Server` 也发布其 `Will Message`。 |
| 128 | 0x80 | Unspecified error | `Client` 或 `Server` | 连接已关闭，但发送方要么不希望透露原因，要么没有其他 `Reason Code` 适用。 |
| 129 | 0x81 | Malformed Packet | `Client` 或 `Server` | 收到的报文不符合此规范。 |
| 130 | 0x82 | Protocol Error | `Client` 或 `Server` | 收到意外或乱序的报文。 |
| 131 | 0x83 | Implementation specific error | `Client` 或 `Server` | 收到的报文有效但无法由此实现处理。 |
| 135 | 0x87 | Not authorized | `Server` | 请求未授权。 |
| 137 | 0x89 | Server busy | `Server` | `Server` 忙碌，无法继续处理来自此 `Client` 的请求。 |
| 139 | 0x8B | Server shutting down | `Server` | `Server` 正在关闭。 |
| 141 | 0x8D | `Keep Alive` timeout | `Server` | 连接已关闭，因为在 Keepalive 时间的 1.5 倍时间内没有收到任何报文。 |
| 142 | 0x8E | `Session` taken over | `Server` | 另一个使用相同 ClientID 的连接已连接，导致此连接被关闭。 |
| 143 | 0x8F | `Topic Filter` invalid | `Server` | `Topic Filter` 格式正确，但不被此 `Server` 接受。 |
| 144 | 0x90 | `Topic Name` invalid | `Client` 或 `Server` | `Topic Name` 格式正确，但不被此 `Client` 或 `Server` 接受。 |
| 147 | 0x93 | `Receive Maximum` exceeded | `Client` 或 `Server` | `Client` 或 `Server` 收到的尚未发送 `PUBACK` 或 `PUBCOMP` 的发布数量超过 `Receive Maximum`。 |
| 148 | 0x94 | `Topic Alias` invalid | `Client` 或 `Server` | `Client` 或 `Server` 收到的 `PUBLISH` 报文包含的 `Topic Alias` 大于它在 `CONNECT` 或 `CONNACK` 报文中发送的 Maximum Topic Alias。 |
| 149 | 0x95 | Packet too large | `Client` 或 `Server` | 报文大小大于此 `Client` 或 `Server` 的 `Maximum Packet Size`。 |
| 150 | 0x96 | Message rate too high | `Client` 或 `Server` | 接收数据速率过高。 |
| 151 | 0x97 | Quota exceeded | `Client` 或 `Server` | 已超过实现或管理限制。 |
| 152 | 0x98 | Administrative action | `Client` 或 `Server` | 连接由于管理操作而关闭。 |
| 153 | 0x99 | `Payload` format invalid | `Client` 或 `Server` | `Payload` 格式与 Payload Format Indicator 指定的格式不匹配。 |
| 154 | 0x9A | Retain not supported | `Server` | `Server` 不支持保留消息。 |
| 155 | 0x9B | `QoS` not supported | `Server` | `Client` 指定的 `QoS` 大于 `CONNACK` 中 `Maximum QoS` 指定的 `QoS`。 |
| 156 | 0x9C | Use another server | `Server` | `Client` 应该临时更换其 `Server`。 |
| 157 | 0x9D | Server moved | `Server` | `Server` 已移动，`Client` 应该永久更换其服务器位置。 |
| 158 | 0x9E | Shared Subscriptions not supported | `Server` | `Server` 不支持 Shared Subscriptions。 |
| 159 | 0x9F | Connection rate exceeded | `Server` | 此连接已关闭，因为连接速率过高。 |
| 160 | 0xA0 | Maximum connect time | `Server` | 已超过此连接授权的最大连接时间。 |
| 161 | 0xA1 | Subscription Identifiers not supported | `Server` | `Server` 不支持 `Subscription Identifiers`；订阅未被接受。 |
| 162 | 0xA2 | Wildcard Subscriptions not supported | `Server` | `Server` 不支持 Wildcard Subscriptions；订阅未被接受。 |

发送 `DISCONNECT` 报文的 `Client` 或 `Server` 必须使用其中一个 DISCONNECT `Reason Code` 值 [MQTT-3.14.2-1]。如果 `Reason Code` 为 0x00（正常断开连接）且没有 Properties，则可以省略 `Reason Code` 和 Property Length。在这种情况下，`DISCONNECT` 的 `Remaining Length` 为 0。

**非规范性评论**

`DISCONNECT` 报文用于在没有确认报文（如 `QoS` 0 发布）的情况下或当 `Client` 或 `Server` 无法继续处理 Connection 时指示断开连接的原因。

**非规范性评论**

此信息可由 `Client` 用于决定是否重试连接以及重试连接之前应该等待多长时间。

#### 3.14.2.2 DISCONNECT Properties

##### 3.14.2.2.1 Property Length

`DISCONNECT` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。如果 `Remaining Length` 小于 2，则使用值 0。

##### 3.14.2.2.2 `Session Expiry Interval`

**17 (0x11) 字节**，`Session Expiry Interval` 的标识符。

后跟表示 `Session Expiry Interval`（以秒为单位）的 Four Byte Integer。多次包含 `Session Expiry Interval` 是协议错误。

如果不存在 `Session Expiry Interval`，则使用 `CONNECT` 报文中的 `Session Expiry Interval`。

`Server` 禁止在 `DISCONNECT` 上发送 `Session Expiry Interval` [MQTT-3.14.2-2]。

如果 `CONNECT` 报文中的 `Session Expiry Interval` 为零，则在 `Client` 发送的 `DISCONNECT` 报文中设置非零 `Session Expiry Interval` 是协议错误。如果 `Server` 收到此非零 `Session Expiry Interval`，则不会将其视为有效的 `DISCONNECT` 报文。`Server` 使用 [第 4.13 节](#413-errors)中描述的带有 `Reason Code` 0x82（协议错误）的 `DISCONNECT`。

##### 3.14.2.2.3 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示断开连接原因的 `UTF-8 Encoded String`。此 Reason String 是人类可读的，设计用于诊断，不应该被接收方解析。

如果这会使 `DISCONNECT` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 Property [MQTT-3.14.2-3]。多次包含 Reason String 是协议错误。

##### 3.14.2.2.4 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此 property 可用于提供额外的诊断或其他信息。如果这会使 `DISCONNECT` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.14.2-4]。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

##### 3.14.2.2.5 `Server Reference`

**28 (0x1C) 字节**，`Server Reference` 的标识符。

后跟 `Client` 可用于识别另一个要使用的 `Server` 的 `UTF-8 Encoded String`。多次包含 `Server Reference` 是协议错误。

`Server` 发送包含 `Server Reference` 和 `Reason Code` 0x9C（使用另一个服务器）或 0x9D（服务器已移动）的 `DISCONNECT`，如 [第 4.13 节](#413-errors)中所述。

有关如何使用 `Server Reference` 的信息，请参阅 [第 4.11 节](#411-server-redirection) Server Redirection。

**图 3-24 - `DISCONNECT` 报文 `Variable Header` 非规范性示例**

（详细字节表示略，参见原文）

### 3.14.3 DISCONNECT `Payload`

`DISCONNECT` 报文没有 `Payload`。

### 3.14.4 DISCONNECT Actions

发送 `DISCONNECT` 报文后，发送方：

- 禁止在该 Network Connection 上发送任何更多的 MQTT Control Packets [MQTT-3.14.4-1]。
- 必须关闭 Network Connection [MQTT-3.14.4-2]。

收到 `Reason Code` 为 0x00（成功）的 `DISCONNECT` 后，`Server`：

- 必须丢弃与当前 Connection 关联的任何 `Will Message` 而不发布它 [MQTT-3.14.4-3]，如 [第 3.1.2.5 节](#3125-will-message)中所述。

收到 `DISCONNECT` 后，接收方：

- 应该关闭 Network Connection。

## 3.15 AUTH - 认证交换

`AUTH` 报文从 `Client` 发送到 `Server` 或从 `Server` 发送到 `Client`，作为扩展认证交换的一部分，例如挑战/响应认证。如果 `CONNECT` 报文不包含相同的 `Authentication Method`，则 `Client` 或 `Server` 发送 `AUTH` 报文是协议错误。

### 3.15.1 AUTH `Fixed Header`

**图 3-35 - `AUTH` 报文 `Fixed Header`**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (15) | Reserved |
| | 1 | 1 | 1 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

`AUTH` 报文 `Fixed Header` 的 Bits 3,2,1 和 0 是保留的，必须全部设置为 0。`Client` 或 `Server` 必须将任何其他值视为格式错误并关闭 Network Connection [MQTT-3.15.1-1]。

**`Remaining Length` field**

这是 `Variable Header` 的长度，编码为 `Variable Byte Integer`。

### 3.15.2 AUTH `Variable Header`

`AUTH` 报文的 `Variable Header` 按顺序包含以下字段：Authenticate `Reason Code` 和 Properties。Properties 的编码规则在 [第 2.2.2 节](#222-properties)中描述。

#### 3.15.2.1 Authenticate `Reason Code`

`Variable Header` 中的字节 0 是 Authenticate `Reason Code`。单字节无符号 Authenticate `Reason Code` 字段的值如下所示。`AUTH` 报文的发送方必须使用其中一个 Authenticate `Reason Code`s [MQTT-3.15.2-1]。

**表 3-11 - Authenticate `Reason Code`s**

| 值 | Hex | `Reason Code` 名称 | 发送方 | 描述 |
|----|-----|------------------|--------|------|
| 0 | 0x00 | Success | `Server` | 认证成功。 |
| 24 | 0x18 | Continue authentication | `Client` 或 `Server` | 使用另一个步骤继续认证。 |
| 25 | 0x19 | Re-authenticate | `Client` | 发起重新认证。 |

如果 `Reason Code` 为 0x00（成功）且没有 Properties，则可以省略 `Reason Code` 和 Property Length。在这种情况下，`AUTH` 的 `Remaining Length` 为 0。

#### 3.15.2.2 AUTH Properties

##### 3.15.2.2.1 Property Length

`AUTH` 报文 `Variable Header` 中 Properties 的长度，编码为 `Variable Byte Integer`。

##### 3.15.2.2.2 `Authentication Method`

**21 (0x15) 字节**，`Authentication Method` 的标识符。

后跟包含认证方法名称的 `UTF-8 Encoded String`。省略 `Authentication Method` 或多次包含它是协议错误。有关扩展认证的更多信息，请参阅 [第 4.12 节](#412-enhanced-authentication)。

##### 3.15.2.2.3 `Authentication Data`

**22 (0x16) 字节**，`Authentication Data` 的标识符。

后跟包含认证数据的 `Binary Data`。多次包含 `Authentication Data` 是协议错误。此数据的内容由认证方法定义。有关扩展认证的更多信息，请参阅 [第 4.12 节](#412-enhanced-authentication)。

##### 3.15.2.2.4 Reason String

**31 (0x1F) 字节**，Reason String 的标识符。

后跟表示断开连接原因的 `UTF-8 Encoded String`。此 Reason String 是人类可读的，设计用于诊断，不应该被接收方解析。

如果这会使 `AUTH` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.15.2-2]。多次包含 Reason String 是协议错误。

##### 3.15.2.2.5 `User Property`

**38 (0x26) 字节**，`User Property` 的标识符。

后跟 UTF-8 String Pair。此 property 可用于提供额外的诊断或其他信息。如果这会使 `AUTH` 报文的大小超过接收方指定的 `Maximum Packet Size`，发送方禁止发送此 property [MQTT-3.15.2-3]。`User Property` 允许多次出现以表示多个名称-值对。允许相同的名称多次出现。

### 3.15.3 AUTH `Payload`

`AUTH` 报文没有 `Payload`。

### 3.15.4 AUTH Actions

有关扩展认证的更多信息，请参阅 [第 4.12 节](#412-enhanced-authentication)。