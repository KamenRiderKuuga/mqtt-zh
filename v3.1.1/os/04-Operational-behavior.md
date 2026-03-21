# 4 操作行为

## 4.1 存储状态

`Client` 和 `Server` 必须存储 `Session state` 以提供 `Quality of Service` 保证。**`Client` 和 `Server` 必须在 `Session` 的整个持续期间存储 `Session state`** [MQTT-4.1.0-1]。**`Session` 必须持续至少与其活跃的 `Network Connection` 一样长** [MQTT-4.1.0-2]。

`Retained messages` 不属于 `Server` 上 `Session state` 的一部分。`Server` 应该保留此类消息直到被 `Client` 删除。

> **非规范性说明**

`Client` 和 `Server` 实现的存储能力当然会在容量方面受到限制，并且可能受到管理策略的限制，例如 `Network Connection` 之间存储 `Session state` 的最长时间。存储的 `Session state` 可能由于管理员操作而被丢弃，包括对定义条件的自动响应。这具有终止 `Session` 的效果。这些操作可能是由于资源限制或其他操作原因而触发的。评估 `Client` 和 `Server` 的存储能力以确保它们足够是明智的。

> **非规范性说明**

硬件或软件故障可能导致 `Client` 或 `Server` 存储的 `Session state` 丢失或损坏。

> **非规范性说明**

`Client` 或 `Server` 的正常操作可能意味着由于管理员操作、硬件故障或软件故障而导致存储的状态丢失或损坏。管理员操作可能是对定义条件的自动响应。这些操作可能是由于资源限制或其他操作原因而触发的。例如，服务器可能根据外部知识确定一条或多条消息无法再传递给任何当前或未来的客户端。

> **非规范性说明**

MQTT 用户应该评估 MQTT `Client` 和 `Server` 实现的存储能力，以确保它们满足其需求。

### 4.1.1 非规范性示例

例如，希望收集电表读数的用户可能决定需要使用 `QoS` 1 消息，因为他们需要保护读数免受网络丢失，但是他们可能已经确定电源足够可靠，`Client` 和 `Server` 中的数据可以存储在易失性存储器中，而不会有太大的丢失风险。

相反，停车计费支付应用提供商可能决定没有任何情况下可以丢失支付消息，因此他们要求所有数据在通过网络传输之前强制写入非易失性存储器。

## 4.2 `Network Connection`

MQTT 协议需要一个底层传输，该传输提供从 `Client` 到 `Server` 以及从 `Server` 到 `Client` 的有序、无损的字节流。

> **非规范性说明**

用于承载 MQTT 3.1 的传输协议是 [RFC793] 中定义的 TCP/IP。TCP/IP 可用于 MQTT 3.1.1。以下也适用：

- TLS [RFC5246]
- WebSocket [RFC6455]

> **非规范性说明**

TCP 端口 8883 和 1883 已在 IANA 注册，分别用于 MQTT TLS 和非 TLS 通信。

无连接网络传输（如 User Datagram Protocol (UDP)）本身不适合，因为它们可能会丢失或重新排序数据。

## 4.3 `Quality of Service` levels 和协议流程

MQTT 根据这里定义的 `Quality of Service` (`QoS`) 级别传递 `Application Messages`。传递协议是对称的，在下面的描述中，`Client` 和 `Server` 都可以扮演 `Sender` 或 `Receiver` 的角色。传递协议仅涉及从单个 `Sender` 到单个 `Receiver` 的应用消息传递。当 `Server` 向多个 `Client` 传递 `Application Message` 时，每个 `Client` 被独立处理。用于向 `Client` 传递 `Application Message` 的 `QoS` 级别可能与入站 `Application Message` 的 `QoS` 级别不同。

以下章节中的非规范性流程图旨在展示可能的实现方法。

### 4.3.1 `QoS` 0: `At most once` delivery

消息根据底层网络的能力进行传递。接收方不发送响应，发送方不进行重试。消息要么到达接收方一次，要么根本不到达。

在 `QoS` 0 传递协议中，`Sender`

- **必须发送 QoS=0, DUP=0 的 `PUBLISH` packet** [MQTT-4.3.1-1]。

在 `QoS` 0 传递协议中，`Receiver`

- 在收到 `PUBLISH` packet 时接受消息的所有权。

**图 4.1 - QoS 0 协议流程图，非规范性示例**

| Sender Action | Control Packet | Receiver Action |
|---------------|----------------|-----------------|
| PUBLISH QoS 0, DUP=0 | | |
| | ----------> | |
| | | Deliver Application Message to appropriate onward recipient(s) |

### 4.3.2 `QoS` 1: `At least once` delivery

此 `Quality of Service` 确保消息至少到达接收方一次。`QoS` 1 `PUBLISH` `Packet` 在其可变头中包含 `Packet Identifier`，并由 `PUBACK` `Packet` 确认。第 2.3.1 节提供了有关 `Packet Identifier` 的更多信息。

在 `QoS` 1 传递协议中，`Sender`

- **每次有新的 `Application Message` 要发布时，必须分配一个未使用的 `Packet Identifier`。**
- **必须发送包含此 `Packet Identifier` 且 QoS=1, DUP=0 的 `PUBLISH` `Packet`。**
- **必须将 `PUBLISH` `Packet` 视为"未确认"状态，直到它收到来自接收方的相应 `PUBACK` packet。参见第 4.4 节关于未确认消息的讨论。**

[MQTT-4.3.2-1]。

一旦 `Sender` 收到 `PUBACK` `Packet`，`Packet Identifier` 就可以重用。

请注意，`Sender` 被允许在等待接收确认时发送具有不同 `Packet Identifier` 的更多 `PUBLISH` `Packets`。

在 `QoS` 1 传递协议中，`Receiver`

- **必须响应包含来自传入 `PUBLISH` `Packet` 的 `Packet Identifier` 的 `PUBACK` `Packet`，已接受 `Application Message` 的所有权**
- **发送 `PUBACK` `Packet` 后，`Receiver` 必须将任何包含相同 `Packet Identifier` 的传入 `PUBLISH` packet 视为新的发布，无论其 `DUP` flag 的设置如何。**

[MQTT-4.3.2-2]。

**图 4.2 - QoS 1 协议流程图，非规范性示例**

| Sender Action | Control Packet | Receiver action |
|---------------|----------------|-----------------|
| Store message | | |
| Send PUBLISH QoS 1, DUP 0, &lt;Packet Identifier&gt; | ----------> | |
| | | Initiate onward delivery of the Application Message<sup>1</sup> |
| | &lt;---------- | Send PUBACK &lt;Packet Identifier&gt; |
| Discard message | | |

<sup>1</sup> 接收方无需在发送 `PUBACK` 之前完成 `Application Message` 的传递。当其原始发送方收到 `PUBACK` packet 时，`Application Message` 的所有权转移给接收方。

### 4.3.3 `QoS` 2: `Exactly once` delivery

这是最高级别的 `Quality of Service`，用于既不接受消息丢失也不接受消息重复的情况。与此 `Quality of Service` 相关的开销增加。

`QoS` 2 消息在其可变头中包含 `Packet Identifier`。第 2.3.1 节提供了有关 `Packet Identifier` 的更多信息。`QoS` 2 `PUBLISH` `Packet` 的接收方使用两步确认过程确认接收。

在 `QoS` 2 传递协议中，`Sender`

- **当有新的 `Application Message` 要发布时，必须分配一个未使用的 `Packet Identifier`。**
- **必须发送包含此 `Packet Identifier` 且 QoS=2, DUP=0 的 `PUBLISH` packet。**
- **必须将 `PUBLISH` packet 视为"未确认"状态，直到它收到来自接收方的相应 `PUBREC` packet。参见第 4.4 节关于未确认消息的讨论。**
- **当它收到来自接收方的 `PUBREC` packet 时，必须发送 `PUBREL` packet。此 `PUBREL` packet 必须包含与原始 `PUBLISH` packet 相同的 `Packet Identifier`。**
- **必须将 `PUBREL` packet 视为"未确认"状态，直到它收到来自接收方的相应 `PUBCOMP` packet。**
- **一旦发送了相应的 `PUBREL` packet，禁止重新发送 `PUBLISH`。**

[MQTT-4.3.3-1]。

一旦 `Sender` 收到 `PUBCOMP` `Packet`，`Packet Identifier` 就可以重用。

请注意，`Sender` 被允许在等待接收确认时发送具有不同 `Packet Identifier` 的更多 `PUBLISH` `Packets`。

在 `QoS` 2 传递协议中，`Receiver`

- **必须响应包含来自传入 `PUBLISH` `Packet` 的 `Packet Identifier` 的 `PUBREC`，已接受 `Application Message` 的所有权。**
- **在收到相应的 `PUBREL` packet 之前，`Receiver` 必须通过发送 `PUBREC` 来确认任何后续具有相同 `Packet Identifier` 的 `PUBLISH` packet。在这种情况下，禁止导致向任何后续接收方传递重复消息。**
- **必须通过发送包含与 `PUBREL` 相同的 `Packet Identifier` 的 `PUBCOMP` packet 来响应 `PUBREL` packet。**
- **发送 `PUBCOMP` 后，接收方必须将任何包含该 `Packet Identifier` 的后续 `PUBLISH` packet 视为新的发布。**

[MQTT-4.3.3-2]。

**图 4.3 - QoS 2 协议流程图，非规范性示例**

| Sender Action | Control Packet | Receiver Action |
|---------------|----------------|-----------------|
| Store message | | |
| PUBLISH QoS 2, DUP 0 &lt;Packet Identifier&gt; | | |
| | ----------> | |
| | | Method A, Store message or Method B, Store &lt;Packet Identifier&gt; then Initiate onward delivery of the Application Message<sup>1</sup> |
| | | PUBREC &lt;Packet Identifier&gt; |
| | &lt;---------- | |
| Discard message, Store PUBREC received &lt;Packet Identifier&gt; | | |
| PUBREL &lt;Packet Identifier&gt; | | |
| | ----------> | |
| | | Method A, Initiate onward delivery of the Application Message<sup>1</sup> then discard message or Method B, Discard &lt;Packet Identifier&gt; |
| | | Send PUBCOMP &lt;Packet Identifier&gt; |
| | &lt;---------- | |
| Discard stored state | | |

<sup>1</sup> 接收方无需在发送 `PUBREC` 或 `PUBCOMP` 之前完成 `Application Message` 的传递。当其原始发送方收到 `PUBREC` packet 时，`Application Message` 的所有权转移给接收方。

图 4.3 显示接收方可以通过两种方法处理 `QoS` 2。它们的区别在于流程中使消息可用于后续传递的点。选择方法 A 还是方法 B 是特定于实现的。只要实现准确选择其中一种方法，这不会影响 `QoS` 2 流程的保证。

## 4.4 消息传递重试

**当 `Client` 以 `CleanSession` 设置为 0 重新连接时，`Client` 和 `Server` 必须使用其原始 `Packet Identifier` 重新发送任何未确认的 `PUBLISH` `Packets`（其中 QoS > 0）和 `PUBREL` `Packets`** [MQTT-4.4.0-1]。这是要求 `Client` 或 `Server` 重新传递消息的唯一情况。

> **非规范性说明**

历史上，`Control Packets` 的重传是为了克服某些旧 TCP 网络上的数据丢失而需要的。在要在此类环境中部署 MQTT 3.1.1 实现的情况下，这可能仍然是一个关注点。

## 4.5 消息接收

**当 `Server` 取得传入 `Application Message` 的所有权时，必须将其添加到具有匹配 `Subscriptions` 的那些 `Client` 的 `Session state` 中。匹配规则在第 4.7 节中定义** [MQTT-4.5.0-1]。

在正常情况下，`Client` 响应其创建的 `Subscriptions` 接收消息。`Client` 也可能收到与其任何显式 `Subscriptions` 不匹配的消息。如果 `Server` 自动为 `Client` 分配订阅，则可能发生这种情况。`Client` 也可能在 `UNSUBSCRIBE` 操作正在进行时收到消息。**`Client` 必须根据适用的 `QoS` 规则确认它收到的任何 `PUBLISH` `Packet`，无论它是否选择处理其中包含的 `Application Message`** [MQTT-4.5.0-2]。

## 4.6 消息排序

**`Client` 在实现本章其他地方定义的协议流程时必须遵循以下规则：**

- **当它重新发送任何 `PUBLISH` packets 时，必须按照原始 `PUBLISH` packets 发送的顺序重新发送它们（这适用于 `QoS` 1 和 `QoS` 2 消息）** [MQTT-4.6.0-1]
- **必须按照收到相应 `PUBLISH` packets 的顺序发送 `PUBACK` packets（`QoS` 1 消息）** [MQTT-4.6.0-2]
- **必须按照收到相应 `PUBLISH` packets 的顺序发送 `PUBREC` packets（`QoS` 2 消息）** [MQTT-4.6.0-3]
- **必须按照收到相应 `PUBREC` packets 的顺序发送 `PUBREL` packets（`QoS` 2 消息）** [MQTT-4.6.0-4]

**`Server` 必须默认将每个 `Topic` 视为"有序 `Topic`"。它可以提供管理或其他机制来允许将一个或多个 `Topic` 视为"无序 `Topic`"** [MQTT-4.6.0-5]。

**当 `Server` 处理已发布到有序 `Topic` 的消息时，在向其每个订阅者传递消息时必须遵循上面列出的规则。此外，必须按照从任何给定 `Client` 收到的顺序向消费者发送 `PUBLISH` packets（对于相同的 `Topic` 和 `QoS`）** [MQTT-4.6.0-6]。

> **非规范性说明**

上面列出的规则确保当以 `QoS` 1 发布和订阅消息流时，订阅者收到的每条消息的最终副本将按照它们最初发布的顺序排列，但消息重复的可能性可能导致较早消息的重发在其后继消息之一之后被接收。例如，发布者可能按顺序 1,2,3,4 发送消息，订阅者可能按顺序 1,2,3,2,3,4 接收它们。

如果 `Client` 和 `Server` 都确保任何时候最多只有一条消息"在途中"（通过在确认其前一条消息之前不发送消息），则不会在任何较晚的 `QoS` 1 消息之后收到任何较早的消息 - 例如，订阅者可能按顺序 1,2,3,3,4 接收它们，但不会是 1,2,3,2,3,4。设置在途窗口为 1 也意味着即使发布者在同一主题上发送具有不同 `QoS` 级别的消息序列，顺序也将被保留。

## 4.7 `Topic Names` 和 `Topic Filters`

### 4.7.1 Topic wildcards

`Topic level separator` 用于在 `Topic Name` 中引入结构。如果存在，它将 `Topic Name` 分为多个"topic levels"。

`Subscription` 的 `Topic Filter` 可以包含特殊通配符，允许您一次订阅多个主题。

**通配符可以在 `Topic Filters` 中使用，但禁止在 `Topic Name` 中使用** [MQTT-4.7.1-1]。

#### 4.7.1.1 `Topic level separator`

正斜杠（"/" U+002F）用于分隔主题树中的每个级别，并为 `Topic Names` 提供层次结构。当在订阅 `Client` 指定的 `Topic Filters` 中遇到两个通配符中的任何一个时，使用 `Topic level separator` 是有意义的。`Topic level separators` 可以出现在 `Topic Filter` 或 `Topic Name` 的任何位置。相邻的 `Topic level separators` 表示零长度的主题级别。

#### 4.7.1.2 `Multi-level wildcard`

数字符号（"#" U+0023）是匹配主题中任意数量级别的通配符。`Multi-level wildcard` 表示父级别和任意数量的子级别。**`Multi-level wildcard` 字符必须单独指定或跟在 `Topic level separator` 后面。在任何一种情况下，它必须是 `Topic Filter` 中指定的最后一个字符** [MQTT-4.7.1-2]。

> **非规范性说明**

例如，如果 `Client` 订阅 "sport/tennis/player1/#"，它将接收使用以下 `Topic Names` 发布的消息：

- "sport/tennis/player1"
- "sport/tennis/player1/ranking"
- "sport/tennis/player1/score/wimbledon"

> **非规范性说明**

- "sport/#" 也匹配单数 "sport"，因为 # 包括父级别。
- "#" 是有效的，将接收每个 `Application Message`
- "sport/tennis/#" 是有效的
- "sport/tennis#" 不是有效的
- "sport/tennis/#/ranking" 不是有效的

#### 4.7.1.3 `Single level wildcard`

加号（"+" U+002B）是仅匹配一个主题级别的通配符。

**`Single-level wildcard` 可以在 `Topic Filter` 的任何级别使用，包括第一级和最后一级。在使用它的地方，必须占据 `filter` 的整个级别** [MQTT-4.7.1-3]。它可以在 `Topic Filter` 的多个级别中使用，并且可以与 `multilevel wildcard` 一起使用。

> **非规范性说明**

例如，"sport/tennis/+" 匹配 "sport/tennis/player1" 和 "sport/tennis/player2"，但不匹配 "sport/tennis/player1/ranking"。此外，因为 `single-level wildcard` 仅匹配单个级别，"sport/+" 不匹配 "sport" 但它匹配 "sport/"。

> **非规范性说明**

- "+" 是有效的
- "+/tennis/#" 是有效的
- "sport+" 不是有效的
- "sport/+/player1" 是有效的
- "/finance" 匹配 "+/+" 和 "/+"，但不匹配 "+"

### 4.7.2 以 $ 开头的 Topics

**`Server` 禁止将以通配符（# 或 +）开头的 `Topic Filters` 与以 $ 字符开头的 `Topic Names` 匹配** [MQTT-4.7.2-1]。`Server` 应该阻止 `Client` 使用此类 `Topic Names` 与其他 `Client` 交换消息。`Server` 实现可以将以 $ 字符开头的 `Topic Names` 用于其他目的。

> **非规范性说明**

- $SYS/ 已被广泛采用作为包含 `Server` 特定信息或控制 API 的主题前缀
- 应用程序不能将以 $ 字符开头的主题用于自己的目的

> **非规范性说明**

- 订阅 "#" 将不会收到任何发布到以 $ 开头的主题的消息
- 订阅 "+/monitor/Clients" 将不会收到任何发布到 "$SYS/monitor/Clients" 的消息
- 订阅 "$SYS/#" 将收到发布到以 "$SYS/" 开头的主题的消息
- 订阅 "$SYS/monitor/+" 将收到发布到 "$SYS/monitor/Clients" 的消息
- 对于要从以 $SYS/ 开头的主题和不以 $ 开头的主题接收消息的 `Client`，它必须同时订阅 "#" 和 "$SYS/#"

### 4.7.3 Topic 语义和用法

以下规则适用于 `Topic Names` 和 `Topic Filters`：

- **所有 `Topic Names` 和 `Topic Filters` 必须至少为一个字符长** [MQTT-4.7.3-1]
- `Topic Names` 和 `Topic Filters` 区分大小写
- `Topic Names` 和 `Topic Filters` 可以包含空格字符
- 前导或尾随的 "/" 创建一个独特的 `Topic Name` 或 `Topic Filter`
- 仅由 "/" 字符组成的 `Topic Name` 或 `Topic Filter` 是有效的
- **`Topic Names` 和 `Topic Filters` 禁止包含空字符（Unicode U+0000）** [Unicode] [MQTT-4.7.3-2]
- **`Topic Names` 和 `Topic Filters` 是 UTF-8 编码字符串，它们的编码禁止超过 65535 字节** [MQTT-4.7.3-3]。参见第 1.5.3 节

除了 UTF-8 编码字符串的整体长度所施加的限制外，`Topic Name` 或 `Topic Filter` 中的级别数量没有限制。

**当执行订阅匹配时，`Server` 禁止对 `Topic Names` 或 `Topic Filters` 执行任何规范化，或对无法识别的字符进行任何修改或替换** [MQTT-4.7.3-4]。`Topic Filter` 中的每个非通配级别必须逐字符匹配 `Topic Name` 中的相应级别，匹配才能成功。

> **非规范性说明**

UTF-8 编码规则意味着可以通过比较编码的 UTF-8 字节或比较解码的 Unicode 字符来执行 `Topic Filter` 和 `Topic Name` 的比较

> **非规范性说明**

- "ACCOUNTS" 和 "Accounts" 是两个不同的主题名称
- "Accounts payable" 是一个有效的主题名称
- "/finance" 与 "finance" 不同

`Application Message` 被发送到每个 `Topic Filter` 与附加到 `Application Message` 的 `Topic Name` 匹配的 `Client` `Subscription`。`Topic` 资源可以由管理员在 `Server` 中预定义，或者可以在 `Server` 收到第一个订阅或具有该 `Topic Name` 的 `Application Message` 时由 `Server` 动态创建。`Server` 还可以使用安全组件有选择地授权给定 `Client` 对 `Topic` 资源的操作。

## 4.8 错误处理

**除非另有说明，如果 `Server` 或 `Client` 遇到协议违规，它必须关闭收到导致协议违规的 `Control Packet` 的 `Network Connection`** [MQTT-4.8.0-1]。

`Client` 或 `Server` 实现可能会遇到 `Transient Error`（例如内部缓冲区满条件），从而阻止成功处理 MQTT packet。

**如果 `Client` 或 `Server` 在处理入站 `Control Packet` 时遇到 `Transient Error`，它必须关闭收到该 `Control Packet` 的 `Network Connection`** [MQTT-4.8.0-2]。如果 `Server` 检测到 `Transient Error`，它不应该断开连接或对其与任何其他 `Client` 的交互产生任何其他影响。