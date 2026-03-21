# 3 MQTT 控制报文（MQTT Control Packets）

## 3.1 `CONNECT` - 客户端请求连接到服务器（Client requests a connection to a Server）

在 `Client` 与 `Server` 建立网络连接后，`Client` 发送给 `Server` 的第一个报文必须是 `CONNECT` 报文 [MQTT-3.1.0-1]。

`Client` 在一个网络连接上只能发送一次 `CONNECT` 报文。`Server` 必须将 `Client` 发送的第二个 `CONNECT` 报文视为协议违规并断开 `Client` 连接 [MQTT-3.1.0-2]。有关错误处理的信息，请参见第 4.8 节。

有效载荷包含一个或多个编码字段。它们指定了 `Client` 的唯一 `Client Identifier`、`Will Topic`、`Will Message`、`User Name` 和 `Password`。除 `Client Identifier` 外，其他字段都是可选的，其是否存在取决于可变报头中的标志位。

### 3.1.1 固定报头（Fixed Header）

**图 3.1 – `CONNECT` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (1) | Reserved |
| | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

**`Remaining Length` 字段**

`Remaining Length` 是可变报头（10 字节）加上有效载荷的长度。它按照第 2.2.3 节描述的方式进行编码。

### 3.1.2 可变报头（Variable Header）

`CONNECT` 报文的可变报头按顺序包含四个字段：`Protocol Name`、`Protocol Level`、`Connect Flags` 和 `Keep Alive`。

#### 3.1.2.1 协议名称（Protocol Name）

**图 3.2 - Protocol Name 字节**

| | Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|-------------|---|---|---|---|---|---|---|---|
| Protocol Name | | | | | | | | | |
| byte 1 | Length MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | Length LSB (4) | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| byte 3 | 'M' | 0 | 1 | 0 | 0 | 1 | 1 | 0 | 1 |
| byte 4 | 'Q' | 0 | 1 | 0 | 1 | 0 | 0 | 0 | 1 |
| byte 5 | 'T' | 0 | 1 | 0 | 1 | 0 | 1 | 0 | 0 |
| byte 6 | 'T' | 0 | 1 | 0 | 1 | 0 | 1 | 0 | 0 |

`Protocol Name` 是一个 UTF-8 编码字符串，表示协议名称 "MQTT"，如上所示大写。该字符串、其偏移量和长度不会在 MQTT 规范的未来版本中更改。

如果协议名称不正确，`Server` 可以断开 `Client` 连接，也可以按照其他规范继续处理 `CONNECT` 报文。在后一种情况下，`Server` 禁止按照本规范继续处理 `CONNECT` 报文 [MQTT-3.1.2-1]。

**非规范性说明**

报文检测器（如防火墙）可以使用 `Protocol Name` 来识别 MQTT 流量。

#### 3.1.2.2 协议级别（Protocol Level）

**图 3.3 - `Protocol Level` 字节**

| | Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|-------------|---|---|---|---|---|---|---|---|
| Protocol Level | | | | | | | | | |
| byte 7 | Level (4) | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |

表示 `Client` 使用的协议版本级别的 8 位无符号值。协议版本 3.1.1 的 `Protocol Level` 字段值为 4 (0x04)。如果 `Server` 不支持该 `Protocol Level`，必须以 `CONNACK` 返回码 0x01（不可接受的协议级别）响应 `CONNECT` 报文，然后断开 `Client` 连接 [MQTT-3.1.2-2]。

#### 3.1.2.3 连接标志（Connect Flags）

`Connect Flags` 字节包含多个指定 `MQTT` 连接行为的参数。它还指示有效载荷中字段的存在与否。

**图 3.4 - 连接标志位（Connect Flag 位）**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| | User Name Flag | Password Flag | Will Retain | Will QoS | | Will Flag | Clean Session | Reserved |
| byte 8 | X | X | X | X | X | X | X | 0 |

`Server` 必须验证 `CONNECT` 控制报文中的保留标志是否设置为零，如果不为零则断开 `Client` 连接 [MQTT-3.1.2-3]。

#### 3.1.2.4 清除会话（Clean Session）

**位置：** `Connect Flags` 字节的第 1 位。

此位指定 `Session` 状态的处理方式。

`Client` 和 `Server` 可以存储 `Session` 状态，以便在一系列网络连接之间实现可靠的消息传递继续。此位用于控制 `Session` 状态的生命周期。

如果 `CleanSession` 设置为 0，`Server` 必须基于当前 `Session`（由 `Client Identifier` 标识）的状态恢复与 `Client` 的通信。如果没有与 `Client Identifier` 关联的 `Session`，`Server` 必须创建一个新的 `Session`。`Client` 和 `Server` 必须在断开连接后存储 `Session` [MQTT-3.1.2-4]。对于 `CleanSession` 设置为 0 的 `Session` 断开后，`Server` 必须存储与 `Client` 断开时拥有的任何订阅匹配的更多 `QoS` 1 和 `QoS` 2 消息作为 `Session` 状态的一部分 [MQTT-3.1.2-5]。它也可以存储满足相同条件的 `QoS` 0 消息。

如果 `CleanSession` 设置为 1，`Client` 和 `Server` 必须丢弃任何先前的 `Session` 并开始一个新 `Session`。此 `Session` 持续时间与网络连接相同。与此 `Session` 关联的状态数据禁止在任何后续 `Session` 中重用 [MQTT-3.1.2-6]。

`Client` 中的 `Session` 状态包括：
- 已发送给 `Server` 但尚未完全确认的 `QoS` 1 和 `QoS` 2 消息。
- 已从 `Server` 接收但尚未完全确认的 `QoS` 2 消息。

`Server` 中的 `Session` 状态包括：
- `Session` 的存在，即使其余 `Session` 状态为空。
- `Client` 的订阅。
- 已发送给 `Client` 但尚未完全确认的 `QoS` 1 和 `QoS` 2 消息。
- 待传输给 `Client` 的 `QoS` 1 和 `QoS` 2 消息。
- 已从 `Client` 接收但尚未完全确认的 `QoS` 2 消息。
- 可选地，待传输给 `Client` 的 `QoS` 0 消息。

保留消息不构成 `Server` 中 `Session` 状态的一部分，它们禁止在 `Session` 结束时被删除 [MQTT-3.1.2-7]。

有关存储状态的详细信息和限制，请参见第 4.1 节。

当 `CleanSession` 设置为 1 时，`Client` 和 `Server` 不需要原子地处理状态删除。

**非规范性说明**

为确保在发生故障时状态一致，`Client` 应该重复尝试以 `CleanSession` 设置为 1 进行连接，直到成功连接。

**非规范性说明**

通常，`Client` 会始终使用 `CleanSession` 设置为 0 或 `CleanSession` 设置为 1 进行连接，而不会在两个值之间切换。选择取决于应用程序。使用 `CleanSession` 设置为 1 的 `Client` 不会收到旧的应用消息，并且每次连接时必须重新订阅其感兴趣的任何主题。使用 `CleanSession` 设置为 0 的 `Client` 将收到在断开连接期间发布的所有 `QoS` 1 或 `QoS` 2 消息。因此，为确保在断开连接期间不会丢失消息，请使用 `QoS` 1 或 `QoS` 2 并将 `CleanSession` 设置为 0。

**非规范性说明**

当 `Client` 以 `CleanSession` 设置为 0 连接时，它请求 `Server` 在断开连接后维护其 MQTT 会话状态。只有在打算在稍后某个时间点重新连接到 `Server` 时，`Client` 才应该以 `CleanSession` 设置为 0 进行连接。当 `Client` 确定它不再需要该会话时，应该以 `CleanSession` 设置为 1 进行最终连接，然后断开连接。

#### 3.1.2.5 遗嘱标志（Will Flag）

**位置：** `Connect Flags` 的第 2 位。

如果 `Will Flag` 设置为 1，这表示如果连接请求被接受，`Will Message` 必须存储在 `Server` 上并与网络连接关联。除非 `Server` 在收到 `DISCONNECT` 报文时删除了 `Will Message`，否则当网络连接随后关闭时必须发布 `Will Message` [MQTT-3.1.2-8]。

`Will Message` 被发布的情况包括但不限于：
- `Server` 检测到的 I/O 错误或网络故障。
- `Client` 未能在 `Keep Alive` 时间内通信。
- `Client` 关闭网络连接而未首先发送 `DISCONNECT` 报文。
- `Server` 因协议错误关闭网络连接。

如果 `Will Flag` 设置为 1，`Server` 将使用 `Connect Flags` 中的 `Will QoS` 和 `Will Retain` 字段，并且有效载荷中必须存在 `Will Topic` 和 `Will Message` 字段 [MQTT-3.1.2-9]。

一旦 `Will Message` 被发布或 `Server` 收到来自 `Client` 的 `DISCONNECT` 报文，必须从 `Server` 中存储的 `Session` 状态中删除 `Will Message` [MQTT-3.1.2-10]。

如果 `Will Flag` 设置为 0，`Connect Flags` 中的 `Will QoS` 和 `Will Retain` 字段必须设置为零，并且有效载荷中禁止存在 `Will Topic` 和 `Will Message` 字段 [MQTT-3.1.2-11]。

如果 `Will Flag` 设置为 0，当此网络连接结束时禁止发布 `Will Message` [MQTT-3.1.2-12]。

`Server` 应该及时发布 Will Messages。在 `Server` 关闭或故障的情况下，服务器可以将 Will Messages 的发布推迟到后续重新启动。如果发生这种情况，从服务器发生故障到发布 `Will Message` 之间可能会有延迟。

#### 3.1.2.6 遗嘱服务质量（Will QoS）

**位置：** `Connect Flags` 的第 4 和第 3 位。

这两位指定发布 `Will Message` 时使用的 `QoS` 级别。

如果 `Will Flag` 设置为 0，则 `Will QoS` 必须设置为 0 (0x00) [MQTT-3.1.2-13]。

如果 `Will Flag` 设置为 1，`Will QoS` 的值可以是 0 (0x00)、1 (0x01) 或 2 (0x02)。它禁止是 3 (0x03) [MQTT-3.1.2-14]。

#### 3.1.2.7 遗嘱保留（Will Retain）

**位置：** `Connect Flags` 的第 5 位。

此位指定 `Will Message` 在发布时是否被保留。

如果 `Will Flag` 设置为 0，则 Will Retain Flag 必须设置为 0 [MQTT-3.1.2-15]。

如果 `Will Flag` 设置为 1：
- 如果 `Will Retain` 设置为 0，`Server` 必须将 `Will Message` 作为非保留消息发布 [MQTT-3.1.2-16]。
- 如果 `Will Retain` 设置为 1，`Server` 必须将 `Will Message` 作为保留消息发布 [MQTT-3.1.2-17]。

#### 3.1.2.8 用户名标志（User Name Flag）

**位置：** `Connect Flags` 的第 7 位。

如果 `User Name Flag` 设置为 0，有效载荷中禁止存在用户名 [MQTT-3.1.2-18]。

如果 `User Name Flag` 设置为 1，有效载荷中必须存在用户名 [MQTT-3.1.2-19]。

#### 3.1.2.9 密码标志（Password Flag）

**位置：** `Connect Flags` 字节的第 6 位。

如果 `Password Flag` 设置为 0，有效载荷中禁止存在密码 [MQTT-3.1.2-20]。

如果 `Password Flag` 设置为 1，有效载荷中必须存在密码 [MQTT-3.1.2-21]。

如果 `User Name Flag` 设置为 0，`Password Flag` 必须设置为 0 [MQTT-3.1.2-22]。

#### 3.1.2.10 保活时间（Keep Alive）

**图 3.5 `Keep Alive` 字节**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 9 | Keep Alive MSB |
| byte 10 | Keep Alive LSB |

`Keep Alive` 是一个以秒为单位的时间间隔。它表示为一个 16 位字，是允许在 `Client` 完成传输一个控制报文到开始发送下一个控制报文之间经过的最大时间间隔。`Client` 负责确保发送控制报文之间的间隔不超过 `Keep Alive` 值。在没有发送任何其他控制报文的情况下，`Client` 必须发送 `PINGREQ` 报文 [MQTT-3.1.2-23]。

`Client` 可以在任何时间发送 `PINGREQ`，无论 `Keep Alive` 值如何，并使用 `PINGRESP` 来确定网络和 `Server` 正在工作。

如果 `Keep Alive` 值非零，并且 `Server` 在一个半倍的 `Keep Alive` 时间段内没有从 `Client` 收到控制报文，它必须断开与 `Client` 的网络连接，就像网络故障一样 [MQTT-3.1.2-24]。

如果 `Client` 在发送 `PINGREQ` 后一段合理的时间内没有收到 `PINGRESP` 报文，它应该关闭与 `Server` 的网络连接。

`Keep Alive` 值为零 (0) 的效果是关闭 keep alive 机制。这意味着在这种情况下，`Server` 不需要因不活动而断开 `Client` 连接。请注意，无论 `Client` 提供的 `Keep Alive` 值如何，`Server` 都被允许在任何时候断开它认为不活动或无响应的 `Client` 连接。

**非规范性说明**

`Keep Alive` 的实际值取决于应用程序；通常是几分钟。最大值是 18 小时 12 分钟 15 秒。

#### 3.1.2.11 可变报头非规范性示例（Variable Header 非规范性示例）

**图 3.6 - `Variable Header` 非规范性示例**

| | Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|-------------|---|---|---|---|---|---|---|---|
| Protocol Name | | | | | | | | | |
| byte 1 | Length MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | Length LSB (4) | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| byte 3 | 'M' | 0 | 1 | 0 | 0 | 1 | 1 | 0 | 1 |
| byte 4 | 'Q' | 0 | 1 | 0 | 1 | 0 | 0 | 0 | 1 |
| byte 5 | 'T' | 0 | 1 | 0 | 1 | 0 | 1 | 0 | 0 |
| byte 6 | 'T' | 0 | 1 | 0 | 1 | 0 | 1 | 0 | 0 |
| Protocol Level | | | | | | | | | |
| byte 7 | Level (4) | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 |
| Connect Flags | | | | | | | | | |
| byte 8 | User Name Flag (1)<br>Password Flag (1)<br>Will Retain (0)<br>Will QoS (01)<br>Will Flag (1)<br>Clean Session (1)<br>Reserved (0) | 1 | 1 | 0 | 0 | 1 | 1 | 1 | 0 |
| Keep Alive | | | | | | | | | |
| byte 9 | Keep Alive MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 10 | Keep Alive LSB (10) | 0 | 0 | 0 | 0 | 1 | 0 | 1 | 0 |

### 3.1.3 有效载荷（Payload）

`CONNECT` 报文的有效载荷包含一个或多个以长度为前缀的字段，其存在由可变报头中的标志确定。如果存在，这些字段必须按 `Client Identifier`、`Will Topic`、`Will Message`、`User Name`、`Password` 的顺序出现 [MQTT-3.1.3-1]。

#### 3.1.3.1 客户端标识符（Client Identifier）

`Client Identifier` (ClientId) 向 `Server` 标识 `Client`。每个连接到 `Server` 的 `Client` 都有一个唯一的 ClientId。`Client` 和 `Server` 必须使用 ClientId 来标识它们持有的与该 `Client` 和 `Server` 之间的 MQTT `Session` 相关的状态 [MQTT-3.1.3-2]。

`Client Identifier` (ClientId) 必须存在，并且必须是 `CONNECT` 报文有效载荷中的第一个字段 [MQTT-3.1.3-3]。

ClientId 必须是第 1.5.3 节中定义的 UTF-8 编码字符串 [MQTT-3.1.3-4]。

`Server` 必须允许长度在 1 到 23 个 UTF-8 编码字节之间且仅包含字符 "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ" 的 ClientId [MQTT-3.1.3-5]。

`Server` 可以允许包含超过 23 个编码字节的 ClientId。`Server` 可以允许包含上述列表中未包含的字符的 ClientId。

`Server` 可以允许 `Client` 提供长度为零字节的 ClientId，但如果这样做，`Server` 必须将此视为特殊情况并为该 `Client` 分配一个唯一的 ClientId。然后它必须像 `Client` 提供了该唯一 ClientId 一样处理 `CONNECT` 报文 [MQTT-3.1.3-6]。

如果 `Client` 提供零字节 ClientId，`Client` 也必须将 `CleanSession` 设置为 1 [MQTT-3.1.3-7]。

如果 `Client` 提供零字节 ClientId 且 `CleanSession` 设置为 0，`Server` 必须以 `CONNACK` 返回码 0x02（标识符被拒绝）响应 `CONNECT` 报文，然后关闭网络连接 [MQTT-3.1.3-8]。

如果 `Server` 拒绝 ClientId，它必须以 `CONNACK` 返回码 0x02（标识符被拒绝）响应 `CONNECT` 报文，然后关闭网络连接 [MQTT-3.1.3-9]。

**非规范性说明**

`Client` 实现可以提供一个方便的方法来生成随机 ClientId。当 `CleanSession` 设置为 0 时，应该积极阻止使用这种方法。

#### 3.1.3.2 遗嘱主题（Will Topic）

如果 `Will Flag` 设置为 1，`Will Topic` 是有效载荷中的下一个字段。`Will Topic` 必须是第 1.5.3 节中定义的 `UTF-8` 编码字符串 [MQTT-3.1.3-10]。

#### 3.1.3.3 遗嘱消息（Will Message）

如果 `Will Flag` 设置为 1，`Will Message` 是有效载荷中的下一个字段。`Will Message` 定义要发布到 `Will Topic` 的应用消息，如第 3.1.2.5 节所述。此字段由两字节长度后跟 `Will Message` 的有效载荷（表示为零个或多个字节的序列）组成。长度给出后续数据中的字节数，不包括长度本身占用的 2 个字节。

当 `Will Message` 发布到 `Will Topic` 时，其有效载荷仅包含此字段的数据部分，而不是前两个长度字节。

#### 3.1.3.4 用户名（User Name）

如果 `User Name Flag` 设置为 1，这是有效载荷中的下一个字段。`User Name` 必须是第 1.5.3 节中定义的 `UTF-8` 编码字符串 [MQTT-3.1.3-11]。它可以被 `Server` 用于身份验证和授权。

#### 3.1.3.5 密码（Password）

如果 `Password Flag` 设置为 1，这是有效载荷中的下一个字段。`Password` 字段包含 0 到 65535 字节的二进制数据，前缀为两字节长度字段，指示二进制数据使用的字节数（不包括长度字段本身占用的两个字节）。

**图 3.7 - `Password` 字节**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Data length MSB |
| byte 2 | Data length LSB |
| byte 3 ... | Data, if length > 0. |

### 3.1.4 响应（Response）

请注意，`Server` 可以在同一个 TCP 端口或其他网络端点上支持多种协议（包括此协议的早期版本）。如果 `Server` 确定协议是 MQTT 3.1.1，则它按如下方式验证连接尝试：

1. 如果 `Server` 在网络连接建立后的合理时间内没有收到 `CONNECT` 报文，`Server` 应该关闭连接。

2. `Server` 必须验证 `CONNECT` 报文是否符合第 3.1 节，如果不符合则关闭网络连接而不发送 `CONNACK` [MQTT-3.1.4-1]。

3. `Server` 可以检查 `CONNECT` 报文的内容是否满足任何进一步的限制，并可以执行身份验证和授权检查。如果这些检查中有任何失败，它应该发送带有非零返回码的适当 `CONNACK` 响应（如第 3.2 节所述），并且必须关闭网络连接。

如果验证成功，`Server` 执行以下步骤：

1. 如果 ClientId 表示已经连接到 `Server` 的 `Client`，则 `Server` 必须断开现有 `Client` 的连接 [MQTT-3.1.4-2]。

2. `Server` 必须执行第 3.1.2.4 节中描述的 `CleanSession` 处理 [MQTT-3.1.4-3]。

3. `Server` 必须使用包含零返回码的 `CONNACK` 报文确认 `CONNECT` 报文 [MQTT-3.1.4-4]。

4. 开始消息传递和 keep alive 监控。

`Client` 被允许在发送 `CONNECT` 报文后立即发送进一步的控制报文；`Client` 不需要等待来自 `Server` 的 `CONNACK` 报文到达。如果 `Server` 拒绝 `CONNECT`，它禁止处理 `Client` 在 `CONNECT` 报文之后发送的任何数据 [MQTT-3.1.4-5]。

**非规范性说明**

`Client` 通常会等待 `CONNACK` 报文。但是，如果 `Client` 利用其在收到 `CONNACK` 之前发送控制报文的自由，它可能会简化 `Client` 实现，因为它不必监管连接状态。`Client` 接受如果在 `Server` 拒绝连接的情况下，它在收到来自 `Server` 的 `CONNACK` 报文之前发送的任何数据都不会被处理。

## 3.2 `CONNACK` - 确认连接请求（Acknowledge connection request）

`CONNACK` 报文是 `Server` 为响应来自 `Client` 的 `CONNECT` 报文而发送的报文。`Server` 发送给 `Client` 的第一个报文必须是 `CONNACK` 报文 [MQTT-3.2.0-1]。

如果 `Client` 在合理的时间内没有收到来自 `Server` 的 `CONNACK` 报文，`Client` 应该关闭网络连接。"合理"的时间量取决于应用程序类型和通信基础设施。

### 3.2.1 固定报头（Fixed Header）

**图 3.8 – `CONNACK` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet Type (2) | Reserved |
| | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (2) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |

**`Remaining Length` 字段**

这是可变报头的长度。对于 `CONNACK` 报文，此值为 2。

### 3.2.2 可变报头（Variable Header）

**图 3.9 – `CONNACK` 报文可变报头**

| | Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|-------------|---|---|---|---|---|---|---|---|
| Connect Acknowledge Flags | Reserved | SP¹ |
| byte 1 | | 0 | 0 | 0 | 0 | 0 | 0 | 0 | X |
| Connect Return code | | | | | | | | | |
| byte 2 | | X | X | X | X | X | X | X | X |

#### 3.2.2.1 连接确认标志（Connect Acknowledge Flags）

字节 1 是 "Connect Acknowledge Flags"。第 7-1 位是保留位，必须设置为 0。

第 0 位 (SP¹) 是会话存在（Session Present）标志。

#### 3.2.2.2 会话存在（Session Present）

位置：Connect Acknowledge Flags 的第 0 位。

如果 `Server` 接受 `CleanSession` 设置为 1 的连接，`Server` 必须在 `CONNACK` 报文中将 `Session Present` 设置为 0，除了在 `CONNACK` 报文中设置零返回码之外 [MQTT-3.2.2-1]。

如果 `Server` 接受 `CleanSession` 设置为 0 的连接，`Session Present` 中设置的值取决于 `Server` 是否已经存储了所提供 `Client ID` 的 `Session` 状态。如果 `Server` 存储了 `Session` 状态，它必须在 `CONNACK` 报文中将 `Session Present` 设置为 1 [MQTT-3.2.2-2]。如果 `Server` 没有存储 `Session` 状态，它必须在 `CONNACK` 报文中将 `Session Present` 设置为 0。除了在 `CONNACK` 报文中设置零返回码之外 [MQTT-3.2.2-3]。

`Session Present` 标志使 `Client` 能够确定 `Client` 和 `Server` 是否对是否已存在存储的 `Session` 状态有一致的看法。

一旦 `Session` 的初始设置完成，具有存储的 `Session` 状态的 `Client` 将期望 `Server` 维护其存储的 `Session` 状态。如果 `Client` 从 `Server` 接收到的 `Session Present` 值不符合预期，`Client` 可以选择继续 `Session` 还是断开连接。`Client` 可以通过断开连接、以 `CleanSession` 设置为 1 连接然后再次断开连接来丢弃 `Client` 和 `Server` 上的 `Session` 状态。

如果 `Server` 发送包含非零返回码的 `CONNACK` 报文，它必须将 `Session Present` 设置为 0 [MQTT-3.2.2-4]。

#### 3.2.2.3 连接返回码（Connect Return code）

可变报头中的字节 2。

一字节无符号 Connect Return code 字段的值列于表 3.1 中。如果 `Server` 收到格式正确的 `CONNECT` 报文，但由于某种原因无法处理它，则 `Server` 应该尝试发送包含此表中适当非零 Connect 返回码的 `CONNACK` 报文。如果 `Server` 发送包含非零返回码的 `CONNACK` 报文，它必须随后关闭网络连接 [MQTT-3.2.2-5]。

**表 3.1 – Connect Return code 值**

| Value | Return Code Response | Description |
|-------|---------------------|-------------|
| 0 | 0x00 Connection Accepted | 连接被接受 |
| 1 | 0x01 Connection Refused, unacceptable protocol version | `Server` 不支持 `Client` 请求的 MQTT 协议级别 |
| 2 | 0x02 Connection Refused, identifier rejected | `Client Identifier` 是正确的 UTF-8 但 `Server` 不允许 |
| 3 | 0x03 Connection Refused, Server unavailable | 网络连接已建立但 MQTT 服务不可用 |
| 4 | 0x04 Connection Refused, bad user name or password | 用户名或密码中的数据格式不正确 |
| 5 | 0x05 Connection Refused, not authorized | `Client` 未被授权连接 |
| 6-255 | | 保留供将来使用 |

如果表 3.1 中列出的返回码都不被认为适用，则 `Server` 必须关闭网络连接而不发送 `CONNACK` [MQTT-3.2.2-6]。

### 3.2.3 有效载荷（Payload）

`CONNACK` 报文没有有效载荷。

## 3.3 `PUBLISH` - 发布消息（Publish message）

`PUBLISH` 控制报文从 `Client` 发送到 `Server` 或从 `Server` 发送到 `Client`，用于传输应用消息。

### 3.3.1 固定报头（Fixed Header）

**图 3.10 – `PUBLISH` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (3) | DUP flag | QoS level | RETAIN |
| | 0 | 0 | 1 | 1 | X | X | X | X |
| byte 2 | `Remaining Length` |

#### 3.3.1.1 重发标志（DUP）

**位置：** 字节 1，第 3 位。

如果 `DUP` 标志设置为 0，表示这是 `Client` 或 `Server` 第一次尝试发送此 MQTT `PUBLISH` 报文。如果 `DUP` 标志设置为 1，表示这可能是早期尝试发送该报文的重传。

当 `Client` 或 `Server` 尝试重新传递 `PUBLISH` 报文时，必须将 `DUP` 标志设置为 1 [MQTT-3.3.1.-1]。对于所有 `QoS` 0 消息，`DUP` 标志必须设置为 0 [MQTT-3.3.1-2]。

来自传入 `PUBLISH` 报文的 `DUP` 标志值不会在 `Server` 将 `PUBLISH` 报文发送给订阅者时传播。传出 `PUBLISH` 报文中的 `DUP` 标志独立于传入 `PUBLISH` 报文设置，其值必须仅由传出 `PUBLISH` 报文是否为重传来确定 [MQTT-3.3.1-3]。

**非规范性说明**

接收到包含 `DUP` 标志设置为 1 的控制报文的接收方不能假设它已经看到过该报文的早期副本。

**非规范性说明**

需要注意的是，`DUP` 标志指的是控制报文本身，而不是它包含的应用消息。使用 `QoS` 1 时，`Client` 可能会收到 `DUP` 标志设置为 0 的 `PUBLISH` 报文，其中包含它之前收到的应用消息的重复，但具有不同的 `Packet Identifier`。第 2.3.1 节提供了有关 `Packet Identifier` 的更多信息。

#### 3.3.1.2 服务质量（QoS）

**位置：** 字节 1，第 2-1 位。

此字段指示应用消息传递的保证级别。`QoS` 级别列于下表 3.2 中。

**表 3.2 - `QoS` 定义**

| QoS value | Bit 2 | bit 1 | Description |
|-----------|-------|-------|-------------|
| 0 | 0 | 0 | At most once delivery |
| 1 | 0 | 1 | At least once delivery |
| 2 | 1 | 0 | Exactly once delivery |
| - | 1 | 1 | Reserved – must not be used |

`PUBLISH` 报文禁止将两个 `QoS` 位都设置为 1。如果 `Server` 或 `Client` 收到两个 `QoS` 位都设置为 1 的 `PUBLISH` 报文，它必须关闭网络连接 [MQTT-3.3.1-4]。

#### 3.3.1.3 保留标志（RETAIN）

**位置：** 字节 1，第 0 位。

如果 `RETAIN` 标志设置为 1，在 `Client` 发送给 `Server` 的 `PUBLISH` 报文中，`Server` 必须存储应用消息及其 `QoS`，以便可以将其传递给订阅与其主题名称匹配的未来订阅者 [MQTT-3.3.1-5]。当建立新订阅时，必须将每个匹配主题名称上的最后保留消息（如果有）发送给订阅者 [MQTT-3.3.1-6]。如果 `Server` 收到 `RETAIN` 标志设置为 1 的 `QoS` 0 消息，它必须丢弃该主题之前保留的任何消息。它应该将新的 `QoS` 0 消息存储为该主题的新保留消息，但可以选择随时丢弃它 - 如果发生这种情况，该主题将没有保留消息 [MQTT-3.3.1-7]。有关存储状态的更多信息，请参见第 4.1 节。

当 `Server` 向 `Client` 发送 `PUBLISH` 报文时，如果消息是由于 `Client` 建立新订阅而发送的，`Server` 必须将 `RETAIN` 标志设置为 1 [MQTT-3.3.1-8]。当 `PUBLISH` 报文发送给 `Client` 因为它匹配已建立的订阅时，它必须将 `RETAIN` 标志设置为 0，无论它在收到的消息中如何设置该标志 [MQTT-3.3.1-9]。

`RETAIN` 标志设置为 1 且有效载荷包含零字节的 `PUBLISH` 报文将由 `Server` 正常处理并发送给具有与主题名称匹配的订阅的 `Client`。此外，必须删除具有相同主题名称的任何现有保留消息，并且该主题的未来订阅者将不会收到保留消息 [MQTT-3.3.1-10]。"正常" 意味着现有 `Client` 收到的消息中未设置 `RETAIN` 标志。禁止将零字节保留消息存储为 `Server` 上的保留消息 [MQTT-3.3.1-11]。

如果 `RETAIN` 标志为 0，在 `Client` 发送给 `Server` 的 `PUBLISH` 报文中，`Server` 禁止存储该消息，并且禁止删除或替换任何现有的保留消息 [MQTT-3.3.1-12]。

**非规范性说明**

保留消息对于发布者不规则发送状态消息的情况很有用。新订阅者将收到最新状态。

**`Remaining Length` 字段**

这是可变报头的长度加上有效载荷的长度。

### 3.3.2 可变报头（Variable Header）

可变报头按顺序包含以下字段：`Topic Name`、`Packet Identifier`。

#### 3.3.2.1 主题名称（Topic Name）

`Topic Name` 标识有效载荷数据发布到的信息通道。

`Topic Name` 必须作为 `PUBLISH` 报文可变报头中的第一个字段存在。它必须是第 1.5.3 节中定义的 `UTF-8` 编码字符串 [MQTT-3.3.2-1]。

`PUBLISH` 报文中的 `Topic Name` 禁止包含通配符 [MQTT-3.3.2-2]。

`Server` 发送给订阅 `Client` 的 `PUBLISH` 报文中的 `Topic Name` 必须根据第 4.7 节中定义的匹配过程与订阅的 `Topic Filter` 匹配 [MQTT-3.3.2-3]。但是，由于 `Server` 被允许覆盖 `Topic Name`，它可能与原始 `PUBLISH` 报文中的 `Topic Name` 不同。

#### 3.3.2.2 报文标识符（Packet Identifier）

`Packet Identifier` 字段仅存在于 `QoS` 级别为 1 或 2 的 `PUBLISH` 报文中。第 2.3.1 节提供了有关 `Packet Identifier` 的更多信息。

#### 3.3.2.3 可变报头非规范性示例（Variable Header 非规范性示例）

**表 3.3 - `Publish` 报文非规范性示例**

| Field | Value |
|-------|-------|
| Topic Name | a/b |
| Packet Identifier | 10 |

**图 3.11 - `Publish` 报文可变报头非规范性示例**

| | Description | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|-------------|---|---|---|---|---|---|---|---|
| Topic Name | | | | | | | | | |
| byte 1 | Length MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | Length LSB (3) | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 1 |
| byte 3 | 'a' (0x61) | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 1 |
| byte 4 | '/' (0x2F) | 0 | 0 | 1 | 0 | 1 | 1 | 1 | 1 |
| byte 5 | 'b' (0x62) | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 |
| Packet Identifier | | | | | | | | | |
| byte 6 | Packet Identifier MSB (0) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 7 | Packet Identifier LSB (10) | 0 | 0 | 0 | 0 | 1 | 0 | 1 | 0 |

### 3.3.3 有效载荷（Payload）

有效载荷包含正在发布的应用消息。数据的内容和格式是特定于应用程序的。有效载荷的长度可以通过从固定报头中的 `Remaining Length` 字段减去可变报头的长度来计算。`PUBLISH` 报文包含零长度有效载荷是有效的。

### 3.3.4 响应（Response）

`PUBLISH` 报文的接收者必须根据 `PUBLISH` 报文中的 `QoS` 按照表 3.4 进行响应 [MQTT-3.3.4-1]。

**表 3.4 - 预期的 `Publish` 报文响应**

| QoS Level | Expected Response |
|-----------|-------------------|
| `QoS` 0 | None |
| `QoS` 1 | `PUBACK` Packet |
| `QoS` 2 | `PUBREC` Packet |

### 3.3.5 操作（Actions）

`Client` 使用 `PUBLISH` 报文将应用消息发送给 `Server`，以便分发给具有匹配订阅的 `Client`。

`Server` 使用 `PUBLISH` 报文将应用消息发送给每个具有匹配订阅的 `Client`。

当 `Client` 使用包含通配符的 `Topic Filter` 进行订阅时，`Client` 的订阅可能会重叠，因此发布的消息可能会匹配多个过滤器。在这种情况下，`Server` 必须在传递消息给 `Client` 时遵守所有匹配订阅的最大 `QoS` [MQTT-3.3.5-1]。此外，`Server` 可以传递消息的更多副本，每个额外的匹配订阅一个，并在每种情况下遵守订阅的 `QoS`。

接收方收到 `PUBLISH` 报文时的操作取决于 `QoS` 级别，如第 4.3 节所述。

如果 `Server` 实现未授权 `Client` 执行 `PUBLISH`；它无法通知该 `Client`。它必须根据正常的 `QoS` 规则进行肯定确认，或者关闭网络连接 [MQTT-3.3.5-2]。

## 3.4 `PUBACK` - 发布确认（Publish acknowledgement）

`PUBACK` 报文是对 `QoS` 级别 1 的 `PUBLISH` 报文的响应。

### 3.4.1 固定报头（Fixed Header）

**图 3.12 - `PUBACK` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (4) | Reserved |
| | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (2) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |

**`Remaining Length` 字段**

这是可变报头的长度。对于 `PUBACK` 报文，此值为 2。

### 3.4.2 可变报头（Variable Header）

这包含正在被确认的 `PUBLISH` 报文中的 `Packet Identifier`。

**图 3.13 – `PUBACK` 报文可变报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Packet Identifier MSB |
| byte 2 | Packet Identifier LSB |

### 3.4.3 有效载荷（Payload）

`PUBACK` 报文没有有效载荷。

### 3.4.4 操作（Actions）

这在第 4.3.2 节中有完整描述。

## 3.5 `PUBREC` - 发布已接收（QoS 2 发布已接收，第 1 部分）

`PUBREC` 报文是对 `QoS` 2 的 `PUBLISH` 报文的响应。它是 `QoS` 2 协议交换的第二个报文。

### 3.5.1 固定报头（Fixed Header）

**图 3.14 – `PUBREC` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (5) | Reserved |
| | 0 | 1 | 0 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (2) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |

**`Remaining Length` 字段**

这是可变报头的长度。对于 `PUBREC` 报文，此值为 2。

### 3.5.2 可变报头（Variable Header）

可变报头包含正在被确认的 `PUBLISH` 报文中的 `Packet Identifier`。

**图 3.15 – `PUBREC` 报文可变报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Packet Identifier MSB |
| byte 2 | Packet Identifier LSB |

### 3.5.3 有效载荷（Payload）

`PUBREC` 报文没有有效载荷。

### 3.5.4 操作（Actions）

这在第 4.3.3 节中有完整描述。

## 3.6 `PUBREL` - 发布释放（QoS 2 发布已接收，第 2 部分）

`PUBREL` 报文是对 `PUBREC` 报文的响应。它是 `QoS` 2 协议交换的第三个报文。

### 3.6.1 固定报头（Fixed Header）

**图 3.16 – `PUBREL` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (6) | Reserved |
| | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 |
| byte 2 | `Remaining Length` (2) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |

`PUBREL` 控制报文固定报头的第 3、2、1 和 0 位是保留位，必须分别设置为 0、0、1 和 0。`Server` 必须将任何其他值视为格式错误并关闭网络连接 [MQTT-3.6.1-1]。

**`Remaining Length` 字段**

这是可变报头的长度。对于 `PUBREL` 报文，此值为 2。

### 3.6.2 可变报头（Variable Header）

可变报头包含与正在被确认的 `PUBREC` 报文相同的 `Packet Identifier`。

**图 3.17 – `PUBREL` 报文可变报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Packet Identifier MSB |
| byte 2 | Packet Identifier LSB |

### 3.6.3 有效载荷（Payload）

`PUBREL` 报文没有有效载荷。

### 3.6.4 操作（Actions）

这在第 4.3.3 节中有完整描述。

## 3.7 `PUBCOMP` - 发布完成（QoS 2 发布已接收，第 3 部分）

`PUBCOMP` 报文是对 `PUBREL` 报文的响应。它是 `QoS` 2 协议交换的第四个也是最后一个报文。

### 3.7.1 固定报头（Fixed Header）

**图 3.18 – `PUBCOMP` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (7) | Reserved |
| | 0 | 1 | 1 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (2) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |

**`Remaining Length` 字段**

这是可变报头的长度。对于 `PUBCOMP` 报文，此值为 2。

### 3.7.2 可变报头（Variable Header）

可变报头包含与正在被确认的 `PUBREL` 报文相同的 `Packet Identifier`。

**图 3.19 – `PUBCOMP` 报文可变报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Packet Identifier MSB |
| byte 2 | Packet Identifier LSB |

### 3.7.3 有效载荷（Payload）

`PUBCOMP` 报文没有有效载荷。

### 3.7.4 操作（Actions）

这在第 4.3.3 节中有完整描述。

## 3.8 `SUBSCRIBE` - 订阅主题（Subscribe to topics）

`SUBSCRIBE` 报文从 `Client` 发送到 `Server`，用于创建一个或多个订阅。每个订阅都注册 `Client` 对一个或多个主题的兴趣。`Server` 发送 `PUBLISH` 报文给 `Client`，以便转发与这些订阅匹配的应用消息。`SUBSCRIBE` 报文还（对于每个订阅）指定了 `Server` 在转发应用消息给 `Client` 时可以使用的最大 `QoS`。

### 3.8.1 固定报头（Fixed Header）

**图 3.20 – `SUBSCRIBE` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (8) | Reserved |
| | 1 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| byte 2 | `Remaining Length` |

`SUBSCRIBE` 控制报文固定报头的第 3、2、1 和 0 位是保留位，必须分别设置为 0、0、1 和 0。`Server` 必须将任何其他值视为格式错误并关闭网络连接 [MQTT-3.8.1-1]。

**`Remaining Length` 字段**

这是可变报头的长度加上有效载荷的长度。

### 3.8.2 可变报头（Variable Header）

可变报头包含 `Packet Identifier`。第 2.3.1 节提供了有关 `Packet Identifier` 的更多信息。

**图 3.21 – `SUBSCRIBE` 报文可变报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Packet Identifier MSB |
| byte 2 | Packet Identifier LSB |

### 3.8.3 有效载荷（Payload）

`SUBSCRIBE` 报文的有效载荷包含一个 `Topic Filter` 列表。每个 `Topic Filter` 后面跟着一个 Requested QoS 字节。`Topic Filter` 必须是第 1.5.3 节中定义的 `UTF-8` 编码字符串 [MQTT-3.8.3-1]。有效载荷中的 `Topic Filter` 列表必须是连续的，并且不允许有填充 [MQTT-3.8.3-2]。

**图 3.22 – `SUBSCRIBE` 报文有效载荷格式**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Topic Filter length MSB |
| byte 2 | Topic Filter length LSB |
| bytes 3..N | Topic Filter |
| byte N+1 | Requested QoS |

Requested QoS 字节的第 7-2 位是保留位，必须设置为 0 [MQTT-3.8.3-3]。第 1 和 0 位编码请求的 `QoS` 级别 [MQTT-3.8.3-4]。

**表 3.5 - Requested QoS 值**

| QoS value | Bit 1 | bit 0 | Description |
|-----------|-------|-------|-------------|
| 0 | 0 | 0 | At most once delivery |
| 1 | 0 | 1 | At least once delivery |
| 2 | 1 | 0 | Exactly once delivery |
| - | 1 | 1 | Reserved – must not be used |

有效载荷必须包含至少一个 `Topic Filter` / Requested QoS 对 [MQTT-3.8.3-5]。`SUBSCRIBE` 报文没有有效载荷是格式错误，`Server` 必须关闭网络连接 [MQTT-3.8.3-6]。

**非规范性示例**

下表显示了一个典型 `SUBSCRIBE` 报文的 `Topic Filter` 和 Requested QoS 字段。

**表 3.6 - `SUBSCRIBE` 报文非规范性示例**

| Topic Filter | Requested QoS |
|--------------|---------------|
| a/b | 0 |
| a/c | 1 |
| a/b/c/d | 2 |

### 3.8.4 响应（Response）

`Server` 必须发送带有 `Packet Identifier` 的 `SUBACK` 报文来确认收到 `SUBSCRIBE` 报文 [MQTT-3.8.4-1]。`SUBACK` 报文包含一个 Return Code 列表，每个 Return Code 对应一个 `Topic Filter`。这使 `Server` 能够验证订阅并返回不同的 Return Code。

`Server` 处理 `SUBSCRIBE` 报文时，必须按照 `Topic Filter` 在 `SUBSCRIBE` 报文中出现的顺序发送 `SUBACK` 报文中的 Return Codes [MQTT-3.8.4-2]。

如果 `Server` 收到的 `SUBSCRIBE` 报文包含多个 `Topic Filter`，它必须将该报文视为该报文中出现的所有 `Topic Filter` 的连续订阅请求 [MQTT-3.8.4-3]。`Server` 必须原子地处理此请求，即所有订阅必须同时创建或同时失败。如果没有失败，则在成功创建所有订阅后发送 `SUBACK` 报文。如果任何订阅创建失败，则发送带有失败原因代码的 `SUBACK` 报文，并且不创建任何订阅。

`Server` 接受 `SUBSCRIBE` 报文中的 `Topic Filter` 意味着 `Server` 接受来自 `Client` 的该 `Topic Filter` 的订阅。`Server` 可以接受最大 `QoS` 级别的订阅，该级别小于请求的级别。`Server` 必须以 `SUBACK` 报文响应，并在 Return Codes 列表中包含授予的 `QoS` 级别 [MQTT-3.8.4-4]。如果授予的 `QoS` 级别小于请求的 `QoS` 级别，`Server` 必须继续将该 `Client` 视为已订阅，就像它请求了授予的 `QoS` 级别一样 [MQTT-3.8.4-5]。

如果 `Server` 拒绝订阅，它必须以失败 Return Code 响应该 `Topic Filter` [MQTT-3.8.4-6]。

如果 `Server` 收到一个 `SUBSCRIBE` 报文，其中 `Topic Filter` 与现有订阅的 `Topic Filter` 相同，它必须用新的订阅覆盖该现有订阅 [MQTT-3.8.4-7]。新订阅使用来自 `SUBSCRIBE` 报文的 `QoS` 级别。新订阅保留与现有订阅相同的订阅标识符（如果有）。

## 3.9 `SUBACK` - 订阅确认（Subscribe acknowledgement）

`SUBACK` 报文由 `Server` 发送，用于确认收到 `SUBSCRIBE` 报文。

### 3.9.1 固定报头（Fixed Header）

**图 3.23 – `SUBACK` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (9) | Reserved |
| | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` |

**`Remaining Length` 字段**

这是可变报头的长度加上有效载荷的长度。

### 3.9.2 可变报头（Variable Header）

可变报头包含 `Packet Identifier`。

**图 3.24 – `SUBACK` 报文可变报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Packet Identifier MSB |
| byte 2 | Packet Identifier LSB |

### 3.9.3 有效载荷（Payload）

有效载荷包含一个 Return Code 列表。每个 Return Code 对应 `SUBSCRIBE` 报文中的一个 `Topic Filter`。Return Codes 的顺序与 `SUBSCRIBE` 报文中 `Topic Filter` 的顺序相匹配。

**图 3.25 – `SUBACK` 报文有效载荷格式**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Return Code |

**表 3.7 - Return Code 值**

| Value | Bit 7 | Bit 6 | Bit 5 | Bit 4 | Bit 3 | Bit 2 | Bit 1 | Bit 0 | Description |
|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------------|
| 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Success - Maximum `QoS` 0 |
| 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | Success - Maximum `QoS` 1 |
| 2 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | Success - Maximum `QoS` 2 |
| 128 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | Failure |

返回码 0、1、2 表示已接受的订阅，以及 `Server` 授予的最大 `QoS` 级别。返回码 128 表示订阅失败。

## 3.10 `UNSUBSCRIBE` - 取消订阅主题（Unsubscribe from topics）

`UNSUBSCRIBE` 报文从 `Client` 发送到 `Server`，用于取消订阅。

### 3.10.1 固定报头（Fixed Header）

**图 3.26 – `UNSUBSCRIBE` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (10) | Reserved |
| | 1 | 0 | 1 | 0 | 0 | 0 | 1 | 0 |
| byte 2 | `Remaining Length` |

`UNSUBSCRIBE` 控制报文固定报头的第 3、2、1 和 0 位是保留位，必须分别设置为 0、0、1 和 0。`Server` 必须将任何其他值视为格式错误并关闭网络连接 [MQTT-3.10.1-1]。

**`Remaining Length` 字段**

这是可变报头的长度加上有效载荷的长度。

### 3.10.2 可变报头（Variable Header）

可变报头包含 `Packet Identifier`。第 2.3.1 节提供了有关 `Packet Identifier` 的更多信息。

**图 3.27 – `UNSUBSCRIBE` 报文可变报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Packet Identifier MSB |
| byte 2 | Packet Identifier LSB |

### 3.10.3 有效载荷（Payload）

`UNSUBSCRIBE` 报文的有效载荷包含一个 `Topic Filter` 列表。`Topic Filter` 必须是第 1.5.3 节中定义的 `UTF-8` 编码字符串 [MQTT-3.10.3-1]。有效载荷中的 `Topic Filter` 列表必须是连续的，并且不允许有填充 [MQTT-3.10.3-2]。

**图 3.28 – `UNSUBSCRIBE` 报文有效载荷格式**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Topic Filter length MSB |
| byte 2 | Topic Filter length LSB |
| bytes 3..N | Topic Filter |

有效载荷必须包含至少一个 `Topic Filter` [MQTT-3.10.3-3]。`UNSUBSCRIBE` 报文没有有效载荷是格式错误，`Server` 必须关闭网络连接 [MQTT-3.10.3-4]。

### 3.10.4 响应（Response）

`UNSUBSCRIBE` 报文被发送给 `Server` 以取消订阅。`Server` 必须发送带有 `Packet Identifier` 的 `UNSUBACK` 报文来确认收到 `UNSUBSCRIBE` 报文 [MQTT-3.10.4-1]。

`UNSUBSCRIBE` 报文中的 `Topic Filter`（无论它们是否代表现有订阅）必须在 `Server` 上删除，并且 `Server` 必须停止向 `Client` 传递与这些 `Topic Filter` 匹配的消息 [MQTT-3.10.4-2]。

## 3.11 `UNSUBACK` - 取消订阅确认（Unsubscribe acknowledgement）

`UNSUBACK` 报文由 `Server` 发送给 `Client`，用于确认收到 `UNSUBSCRIBE` 报文。

### 3.11.1 固定报头（Fixed Header）

**图 3.29 – `UNSUBACK` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (11) | Reserved |
| | 1 | 0 | 1 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (2) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |

**`Remaining Length` 字段**

这是可变报头的长度。对于 `UNSUBACK` 报文，此值为 2。

### 3.11.2 可变报头（Variable Header）

可变报头包含 `Packet Identifier`。

**图 3.30 – `UNSUBACK` 报文可变报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | Packet Identifier MSB |
| byte 2 | Packet Identifier LSB |

### 3.11.3 有效载荷（Payload）

`UNSUBACK` 报文没有有效载荷。

## 3.12 `PINGREQ` - PING 请求（PING request）

`PINGREQ` 报文从 `Client` 发送到 `Server`。它可用于：

1. 在没有其他控制报文发送时，指示 `Client` 处于活动状态。
2. 请求 `Server` 响应以确认它处于活动状态。
3. 确认网络连接未被关闭。

### 3.12.1 固定报头（Fixed Header）

**图 3.31 – `PINGREQ` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (12) | Reserved |
| | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (0) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

### 3.12.2 可变报头（Variable Header）

`PINGREQ` 报文没有可变报头。

### 3.12.3 有效载荷（Payload）

`PINGREQ` 报文没有有效载荷。

### 3.12.4 响应（Response）

`Server` 必须发送 `PINGRESP` 报文来响应 `PINGREQ` 报文 [MQTT-3.12.4-1]。

## 3.13 `PINGRESP` - PING 响应（PING response）

`PINGRESP` 报文由 `Server` 发送给 `Client`，以响应 `PINGREQ` 报文。它指示 `Server` 处于活动状态。

### 3.13.1 固定报头（Fixed Header）

**图 3.32 – `PINGRESP` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (13) | Reserved |
| | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (0) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

### 3.13.2 可变报头（Variable Header）

`PINGRESP` 报文没有可变报头。

### 3.13.3 有效载荷（Payload）

`PINGRESP` 报文没有有效载荷。

## 3.14 `DISCONNECT` - 断开连接通知（Disconnect notification）

`DISCONNECT` 报文是 `Client` 发送给 `Server` 的最后一个控制报文。它表示 `Client` 正常断开连接。

### 3.14.1 固定报头（Fixed Header）

**图 3.33 – `DISCONNECT` 报文固定报头**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | MQTT Control Packet type (14) | Reserved |
| | 1 | 1 | 1 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | `Remaining Length` (0) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |

### 3.14.2 可变报头（Variable Header）

`DISCONNECT` 报文没有可变报头。

### 3.14.3 有效载荷（Payload）

`DISCONNECT` 报文没有有效载荷。

### 3.14.4 操作（Actions）

`Client` 发送 `DISCONNECT` 报文后，必须关闭网络连接 [MQTT-3.14.4-1]。

`Server` 收到 `DISCONNECT` 报文后，必须关闭网络连接 [MQTT-3.14.4-2]。

如果 `Client` 在保持网络连接的同时发送 `DISCONNECT` 报文，`Server` 将处理该 `DISCONNECT` 报文，但不得发送 `CONNACK` 或任何其他控制报文来响应。

如果 `Client` 在没有首先发送包含零返回码的 `CONNECT` 报文的情况下发送 `DISCONNECT` 报文，`Server` 不得采取任何进一步操作。