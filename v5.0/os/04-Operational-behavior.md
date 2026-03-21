# 4 Operational behavior 操作行为

## 4.1 Session State 会话状态

为了实现 QoS 1 和 QoS 2 协议流程，Client 和 Server 需要将状态与 Client Identifier 关联起来，这称为 Session State。Server 还将 Subscription 作为 Session State 的一部分进行存储。

Session 可以跨越一系列 Network Connection 存在。它的持续时间等于最近的 Network Connection 加上 Session Expiry Interval。

Client 中的 Session State 包括：

- 已发送到 Server 但尚未完全确认的 QoS 1 和 QoS 2 消息。
- 已从 Server 接收但尚未完全确认的 QoS 2 消息。

Server 中的 Session State 包括：

- Session 的存在，即使 Session State 的其余部分为空。
- Client 的 Subscription，包括任何 Subscription Identifier。
- 已发送到 Client 但尚未完全确认的 QoS 1 和 QoS 2 消息。
- 等待传输给 Client 的 QoS 1 和 QoS 2 消息，以及可选的等待传输给 Client 的 QoS 0 消息。
- 已从 Client 接收但尚未完全确认的 QoS 2 消息。
- Will Message 和 Will Delay Interval。
- 如果 Session 当前未连接，则包括 Session 结束和 Session State 被丢弃的时间。

Retained Message 不构成 Server 中 Session State 的一部分，它们不会因为 Session 结束而被删除。

### 4.1.1 Storing Session State 存储 Session State

Client 和 Server **禁止**在 Network Connection 打开时丢弃 Session State [MQTT-4.1.0-1]。当 Network Connection 关闭且 Session Expiry Interval 已过时，Server **必须**丢弃 Session State [MQTT-4.1.0-2]。

**非规范性评论**

Client 和 Server 实现的存储能力在容量方面当然会有所限制，并可能受到管理策略的约束。存储的 Session State 可能因管理员操作而被丢弃，包括对定义条件的自动响应。这具有终止 Session 的效果。这些操作可能是由资源限制或其他操作原因触发的。硬件或软件故障可能导致 Client 或 Server 存储的 Session State 丢失或损坏。评估 Client 和 Server 的存储能力以确保它们足够是明智的做法。

### 4.1.2 Session State non-normative examples Session State 非规范性示例

例如，一个电表读数解决方案可能使用 QoS 1 消息来保护读数在网络上的丢失。解决方案开发者可能已确定电源足够可靠，在这种情况下，Client 和 Server 中的数据可以存储在易失性存储器中，而不会有太大的丢失风险。

相反，停车收费表支付应用提供商可能决定支付消息不应因网络或 Client 故障而丢失。因此，他们要求所有数据在通过网络传输之前写入非易失性存储器。

## 4.2 Network Connections 网络连接

MQTT 协议需要一个底层传输，该传输提供从 Client 到 Server 和从 Server 到 Client 的有序、无损的字节流。本规范不要求支持任何特定的传输协议。Client 或 Server **可以**支持此处列出的任何传输协议，或满足本节要求的任何其他传输协议。

Client 或 Server **必须**支持使用一个或多个底层传输协议，这些协议提供从 Client 到 Server 和从 Server 到 Client 的有序、无损的字节流 [MQTT-4.2-1]。

**非规范性评论**

[RFC0793] 中定义的 TCP/IP 可用于 MQTT v5.0。以下传输协议也适用：

- TLS [RFC5246]
- WebSocket [RFC6455]

**非规范性评论**

TCP 端口 8883 和 1883 已在 IANA 注册，分别用于 MQTT TLS 和非 TLS 通信。

**非规范性评论**

无连接网络传输（如用户数据报协议 (UDP)）本身不适用，因为它们可能会丢失或重新排序数据。

## 4.3 Quality of Service levels and protocol flows 服务质量等级和协议流程

MQTT 根据以下章节中定义的服务质量 (QoS) 等级传递 Application Message。传递协议是对称的，在下面的描述中，Client 和 Server 都可以担任发送者或接收者的角色。传递协议仅关注从单个发送者到单个接收者的 Application Message 的传递。当 Server 将 Application Message 传递给多个 Client 时，每个 Client 被独立处理。用于向 Client 传出传递 Application Message 的 QoS 等级可能与传入的 Application Message 的 QoS 等级不同。

### 4.3.1 QoS 0: At most once delivery 最多一次传递

消息根据底层网络的能力进行传递。接收者不发送响应，发送者不进行重试。消息要么到达接收者一次，要么根本不到达。

在 QoS 0 传递协议中，发送者：

- **必须**发送 QoS 为 0 且 DUP 标志设置为 0 的 PUBLISH 报文 [MQTT-4.3.1-1]。

在 QoS 0 传递协议中，接收者：

- 在收到 PUBLISH 报文时接受消息的所有权。

**图 4.1 - QoS 0 协议流程图，非规范性示例**

| Sender Action 发送者动作 | Control Packet 控制报文 | Receiver Action 接收者动作 |
|--------------------------|-------------------------|---------------------------|
| PUBLISH QoS 0, DUP=0 | | |
| | ----------> | |
| | | Deliver Application Message to appropriate onward recipient(s) 将 Application Message 传递给适当的后续接收者 |

### 4.3.2 QoS 1: At least once delivery 至少一次传递

此服务质量等级确保消息至少到达接收者一次。QoS 1 PUBLISH 报文在其 Variable Header 中有一个 Packet Identifier，并由 PUBACK 报文确认。第 2.2.1 节提供了有关 Packet Identifier 的更多信息。

在 QoS 1 传递协议中，发送者：

- 每次有新的 Application Message 要发布时，**必须**分配一个未使用的 Packet Identifier [MQTT-4.3.2-1]。
- **必须**发送包含此 Packet Identifier 且 QoS 为 1、DUP 标志设置为 0 的 PUBLISH 报文 [MQTT-4.3.2-2]。
- **必须**将 PUBLISH 报文视为"未确认"，直到它从接收者收到相应的 PUBACK 报文。有关未确认消息的讨论，请参阅第 4.4 节 [MQTT-4.3.2-3]。

一旦发送者收到 PUBACK 报文，Packet Identifier 就可被重用。

请注意，发送者在等待接收确认时，允许发送具有不同 Packet Identifier 的更多 PUBLISH 报文。

在 QoS 1 传递协议中，接收者：

- **必须**响应包含来自传入 PUBLISH 报文的 Packet Identifier 的 PUBACK 报文，并已接受 Application Message 的所有权 [MQTT-4.3.2-4]。
- 在发送 PUBACK 报文后，接收者**必须**将任何包含相同 Packet Identifier 的传入 PUBLISH 报文视为新的 Application Message，无论其 DUP 标志设置如何 [MQTT-4.3.2-5]。

**图 4.2 - QoS 1 协议流程图，非规范性示例**

| Sender Action 发送者动作 | MQTT Control Packet MQTT 控制报文 | Receiver Action 接收者动作 |
|--------------------------|-----------------------------------|---------------------------|
| Store message 存储消息 | | |
| Send PUBLISH QoS 1, DUP=0, &lt;Packet Identifier&gt; 发送 PUBLISH QoS 1, DUP=0, &lt;Packet Identifier&gt; | ----------> | |
| | | Initiate onward delivery of the Application Message<sup>1</sup> 启动 Application Message 的后续传递 |
| | <---------- | Send PUBACK &lt;Packet Identifier&gt; 发送 PUBACK &lt;Packet Identifier&gt; |
| Discard message 丢弃消息 | | |

<sup>1</sup> 接收者不需要在发送 PUBACK 之前完成 Application Message 的传递。当其原始发送者收到 PUBACK 报文时，Application Message 的所有权转移到接收者。

### 4.3.3 QoS 2: Exactly once delivery 恰好一次传递

这是最高服务质量等级，用于既不能接受消息丢失也不能接受消息重复的情况。QoS 2 相关的开销增加。

QoS 2 消息在其 Variable Header 中有一个 Packet Identifier。第 2.2.1 节提供了有关 Packet Identifier 的更多信息。QoS 2 PUBLISH 报文的接收者使用两步确认过程来确认接收。

在 QoS 2 传递协议中，发送者：

- 当有新的 Application Message 要发布时，**必须**分配一个未使用的 Packet Identifier [MQTT-4.3.3-1]。
- **必须**发送包含此 Packet Identifier 且 QoS 为 2、DUP 标志设置为 0 的 PUBLISH 报文 [MQTT-4.3.3-2]。
- **必须**将 PUBLISH 报文视为"未确认"，直到它从接收者收到相应的 PUBREC 报文 [MQTT-4.3.3-3]。有关未确认消息的讨论，请参阅第 4.4 节。
- 当收到接收者的 Reason Code 值小于 0x80 的 PUBREC 报文时，**必须**发送 PUBREL 报文。此 PUBREL 报文**必须**包含与原始 PUBLISH 报文相同的 Packet Identifier [MQTT-4.3.3-4]。
- **必须**将 PUBREL 报文视为"未确认"，直到它从接收者收到相应的 PUBCOMP 报文 [MQTT-4.3.3-5]。
- 一旦发送了相应的 PUBREL 报文，就**禁止**重新发送 PUBLISH [MQTT-4.3.3-6]。
- 如果已发送 PUBLISH 报文，就**禁止**应用 Message Expiry [MQTT-4.3.3-7]。

一旦发送者收到 PUBCOMP 报文或 Reason Code 为 0x80 或更大的 PUBREC，Packet Identifier 就可被重用。

请注意，发送者在等待接收确认时，允许发送具有不同 Packet Identifier 的更多 PUBLISH 报文，但须遵守第 4.9 节中描述的流量控制。

在 QoS 2 传递协议中，接收者：

- **必须**响应包含来自传入 PUBLISH 报文的 Packet Identifier 的 PUBREC，并已接受 Application Message 的所有权 [MQTT-4.3.3-8]。
- 如果已发送 Reason Code 为 0x80 或更大的 PUBREC，接收者**必须**将任何后续包含该 Packet Identifier 的 PUBLISH 报文视为新的 Application Message [MQTT-4.3.3-9]。
- 直到收到相应的 PUBREL 报文之前，接收者**必须**通过发送 PUBREC 来确认任何后续具有相同 Packet Identifier 的 PUBLISH 报文。在这种情况下，它**禁止**导致将重复消息传递给任何后续接收者 [MQTT-4.3.3-10]。
- **必须**通过发送包含与 PUBREL 相同的 Packet Identifier 的 PUBCOMP 报文来响应 PUBREL 报文 [MQTT-4.3.3-11]。
- 发送 PUBCOMP 后，接收者**必须**将任何后续包含该 Packet Identifier 的 PUBLISH 报文视为新的 Application Message [MQTT-4.3.3-12]。
- 即使已应用 Message Expiry，也**必须**继续 QoS 2 确认序列 [MQTT-4.3.3-13]。

## 4.4 Message delivery retry 消息传递重试

当 Client 以 Clean Start 设置为 0 重新连接且 Session 存在时，Client 和 Server 都**必须**使用其原始 Packet Identifier 重发任何未确认的 PUBLISH 报文（其中 QoS > 0）和 PUBREL 报文。这是唯一要求 Client 或 Server 重发消息的情况。Client 和 Server **禁止**在任何其他时间重发消息 [MQTT-4.4.0-1]。

如果收到包含 Reason Code 为 0x80 或更大的 PUBACK 或 PUBREC，则相应的 PUBLISH 报文被视为已确认，且**禁止**重传 [MQTT-4.4.0-2]。

**图 4.3 - QoS 2 协议流程图，非规范性示例**

| Sender Action 发送者动作 | MQTT Control Packet MQTT 控制报文 | Receiver Action 接收者动作 |
|--------------------------|-----------------------------------|---------------------------|
| Store message 存储消息 | | |
| PUBLISH QoS 2, DUP=0 &lt;Packet Identifier&gt; | | |
| | ----------> | |
| | | Store &lt;Packet Identifier&gt; then Initiate onward delivery of the Application Message<sup>1</sup> 存储 &lt;Packet Identifier&gt; 然后启动 Application Message 的后续传递 |
| | | PUBREC &lt;Packet Identifier&gt;&lt;Reason Code&gt; |
| | <---------- | |
| Discard message, Store PUBREC received &lt;Packet Identifier&gt; 丢弃消息，存储收到的 PUBREC &lt;Packet Identifier&gt; | | |
| PUBREL &lt;Packet Identifier&gt; | | |
| | ----------> | |
| | | Discard &lt;Packet Identifier&gt; 丢弃 &lt;Packet Identifier&gt; |
| | | Send PUBCOMP &lt;Packet Identifier&gt; 发送 PUBCOMP &lt;Packet Identifier&gt; |
| | <---------- | |
| Discard stored state 丢弃存储的状态 | | |

<sup>1</sup> 接收者不需要在发送 PUBREC 或 PUBCOMP 之前完成 Application Message 的传递。当其原始发送者收到 PUBREC 报文时，Application Message 的所有权转移到接收者。但是，接收者需要在接受所有权之前执行所有可能导致转发失败的条件检查（例如配额超限、授权等）。接收者使用 PUBREC 中适当的 Reason Code 表示成功或失败。

## 4.5 Message receipt 消息接收

当 Server 取得传入 Application Message 的所有权时，它**必须**将其添加到具有匹配 Subscription 的那些 Client 的 Session State 中 [MQTT-4.5.0-1]。匹配规则在第 4.7 节中定义。

在正常情况下，Client 收到消息是响应它们创建的 Subscription。Client 也可能收到与其任何显式 Subscription 不匹配的消息。如果 Server 自动将 Subscription 分配给 Client，就会发生这种情况。Client 也可能在 UNSUBSCRIBE 操作进行时收到消息。Client **必须**根据适用的 QoS 规则确认其收到的任何 Publish 报文，无论它是否选择处理其中包含的 Application Message [MQTT-4.5.0-2]。

## 4.6 Message ordering 消息排序

在实现第 4.3 节中定义的协议流程时，以下规则适用于 Client：

- 当 Client 重发任何 PUBLISH 报文时，它**必须**按照原始 PUBLISH 报文发送的顺序重发它们（这适用于 QoS 1 和 QoS 2 消息）[MQTT-4.6.0-1]
- Client **必须**按照收到相应 PUBLISH 报文的顺序发送 PUBACK 报文（QoS 1 消息）[MQTT-4.6.0-2]
- Client **必须**按照收到相应 PUBLISH 报文的顺序发送 PUBREC 报文（QoS 2 消息）[MQTT-4.6.0-3]
- Client **必须**按照收到相应 PUBREC 报文的顺序发送 PUBREL 报文（QoS 2 消息）[MQTT-4.6.0-4]

有序 Topic（Ordered Topic）是一个 Topic，其中 Client 可以确定来自同一 Client 且具有相同 QoS 的该 Topic 中的 Application Message 按其发布的顺序接收。当 Server 处理已发布到有序 Topic 的消息时，它**必须**按照从任何给定 Client 接收的顺序向消费者发送 PUBLISH 报文（对于同一 Topic 和 QoS）[MQTT-4.6.0-5]。这是上述规则的补充。

默认情况下，Server 在 Non-shared Subscription 上转发消息时，**必须**将每个 Topic 视为有序 Topic [MQTT-4.6.0-6]。Server **可以**提供管理或其他机制来允许一个或多个 Topic 不被视为有序 Topic。

**非规范性评论**

上面列出的规则确保当消息流发布并订阅到具有 QoS 1 的有序 Topic 时，订阅者收到的每条消息的最终副本将按照它们发布的顺序。如果重发消息，重复消息可能在收到较早消息之一后被收到。例如，发布者可能按顺序 1、2、3、4 发送消息，但如果在发送消息 3 后发生网络断开连接，订阅者可能按顺序 1、2、3、2、3、4 收到它们。

如果 Client 和 Server 都将 Receive Maximum 设置为 1，它们确保在任何时候不超过一条消息"在传输中"。在这种情况下，即使在重新连接时，也不会在任何较晚的消息之后收到 QoS 1 消息。例如，订阅者可能按顺序 1、2、3、3、4 收到它们，但不会是 1、2、3、2、3、4。有关如何使用 Receive Maximum 的详细信息，请参阅第 4.9 节流量控制。

## 4.7 Topic Names and Topic Filters Topic 名称和 Topic 过滤器

### 4.7.1 Topic wildcards Topic 通配符

Topic 级别分隔符用于在 Topic Name 中引入结构。如果存在，它将 Topic Name 分为多个"Topic 级别"。

Subscription 的 Topic Filter 可以包含特殊的通配符字符，允许 Client 一次订阅多个 Topic。

通配符字符可以在 Topic Filter 中使用，但**禁止**在 Topic Name 中使用 [MQTT-4.7.0-1]。

#### 4.7.1.1 Topic level separator Topic 级别分隔符

正斜杠（"/" U+002F）用于分隔 Topic 树中的每个级别，并为 Topic Name 提供层次结构。当在订阅 Client 指定的 Topic Filter 中遇到两个通配符字符中的任何一个时，Topic 级别分隔符的使用具有重要意义。Topic 级别分隔符可以出现在 Topic Filter 或 Topic Name 的任何位置。相邻的 Topic 级别分隔符表示零长度的 Topic 级别。

#### 4.7.1.2 Multi-level wildcard 多级通配符

井号（"#" U+0023）是一个通配符字符，匹配 Topic 中的任意数量的级别。多级通配符表示父级和任意数量的子级别。多级通配符字符**必须**单独指定或跟在 Topic 级别分隔符之后。在任一情况下，它都**必须**是 Topic Filter 中指定的最后一个字符 [MQTT-4.7.1-1]。

**非规范性评论**

例如，如果 Client 订阅 "sport/tennis/player1/#"，它将收到使用以下 Topic Name 发布的消息：

- "sport/tennis/player1"
- "sport/tennis/player1/ranking"
- "sport/tennis/player1/score/wimbledon"

**非规范性评论**

- "sport/#" 也匹配单个 "sport"，因为 # 包括父级别。
- "#" 是有效的，将收到每个 Application Message
- "sport/tennis/#" 是有效的
- "sport/tennis#" 是无效的
- "sport/tennis/#/ranking" 是无效的

#### 4.7.1.3 Single-level wildcard 单级通配符

加号（"+" U+002B）是一个通配符字符，仅匹配一个 Topic 级别。

单级通配符可以在 Topic Filter 的任何级别使用，包括第一级和最后一级。在使用时，它**必须**占据过滤器的整个级别 [MQTT-4.7.1-2]。它可以在 Topic Filter 的多个级别使用，并可以与多级通配符一起使用。

**非规范性评论**

例如，"sport/tennis/+" 匹配 "sport/tennis/player1" 和 "sport/tennis/player2"，但不匹配 "sport/tennis/player1/ranking"。此外，由于单级通配符仅匹配单个级别，"sport/+" 不匹配 "sport" 但它匹配 "sport/"。

- "+" 是有效的
- "+/tennis/#" 是有效的
- "sport+" 是无效的
- "sport/+/player1" 是有效的
- "/finance" 匹配 "+/+" 和 "/+"，但不匹配 "+"

### 4.7.2 Topics beginning with $ 以 $ 开头的 Topic

Server **禁止**将以通配符字符（# 或 +）开头的 Topic Filter 与以 $ 字符开头的 Topic Name 匹配 [MQTT-4.7.2-1]。Server **应该**阻止 Client 使用此类 Topic Name 与其他 Client 交换消息。Server 实现**可以**将以 $ 字符开头的 Topic Name 用于其他目的。

**非规范性评论**

- $SYS/ 已被广泛采用作为包含 Server 特定信息或控制 API 的 Topic 的前缀
- 应用程序不能将以 $ 字符开头的 Topic 用于自己的目的

**非规范性评论**

- 订阅 "#" 将不会收到任何发布到以 $ 开头的 Topic 的消息
- 订阅 "+/monitor/Clients" 将不会收到任何发布到 "$SYS/monitor/Clients" 的消息
- 订阅 "$SYS/#" 将收到发布到以 "$SYS/" 开头的 Topic 的消息
- 订阅 "$SYS/monitor/+" 将收到发布到 "$SYS/monitor/Clients" 的消息
- 要使 Client 收到来自以 $SYS/ 开头的 Topic 和不以 $ 开头的 Topic 的消息，它必须同时订阅 "#" 和 "$SYS/#"

### 4.7.3 Topic semantic and usage Topic 语义和用法

以下规则适用于 Topic Name 和 Topic Filter：

- 所有 Topic Name 和 Topic Filter **必须**至少有一个字符长 [MQTT-4.7.3-1]
- Topic Name 和 Topic Filter 区分大小写
- Topic Name 和 Topic Filter 可以包含空格字符
- 前导或尾随的 "/" 创建一个不同的 Topic Name 或 Topic Filter
- 仅由 "/" 字符组成的 Topic Name 或 Topic Filter 是有效的
- Topic Name 和 Topic Filter **禁止**包含空字符（Unicode U+0000）[Unicode] [MQTT-4.7.3-2]
- Topic Name 和 Topic Filter 是 UTF-8 编码字符串；它们**禁止**编码超过 65,535 字节 [MQTT-4.7.3-3]。请参阅第 1.5.4 节。

除了 UTF-8 编码字符串的整体长度所施加的限制外，Topic Name 或 Topic Filter 中的级别数量没有限制。

当执行 Subscription 匹配时，Server **禁止**对 Topic Name 或 Topic Filter 执行任何规范化，或对无法识别的字符进行任何修改或替换 [MQTT-4.7.3-4]。Topic Filter 中的每个非通配级别必须逐字符匹配 Topic Name 中的相应级别，匹配才能成功。

**非规范性评论**

UTF-8 编码规则意味着可以通过比较编码的 UTF-8 字节或比较解码的 Unicode 字符来执行 Topic Filter 和 Topic Name 的比较。

**非规范性评论**

- "ACCOUNTS" 和 "Accounts" 是两个不同的 Topic Name
- "Accounts payable" 是一个有效的 Topic Name
- "/finance" 不同于 "finance"

Application Message 被发送到每个 Topic Filter 与附加到 Application Message 的 Topic Name 匹配的 Client Subscription。Topic 资源**可以**由管理员在 Server 中预定义，或者**可以**在 Server 收到第一个 Subscription 或具有该 Topic Name 的 Application Message 时由 Server 动态创建。Server 还**可以**使用安全组件来授权给定 Client 对 Topic 资源的特定操作。

## 4.8 Subscriptions 订阅

MQTT 提供两种 Subscription：Shared 和 Non-shared。

**非规范性评论**

在早期版本的 MQTT 中，所有 Subscription 都是 Non-shared。

### 4.8.1 Non-shared Subscriptions 非共享订阅

Non-shared Subscription 仅与创建它的 MQTT Session 相关联。每个 Subscription 包含一个 Topic Filter，指示要在该 Session 上传递消息的 Topic，以及 Subscription Options。Server 负责收集与过滤器匹配的消息，并在该 Session 的 MQTT 连接处于活动状态时（如果处于活动状态）在 Session 的 MQTT 连接上传输它们。

Session 不能有多个具有相同 Topic Filter 的 Non-shared Subscription，因此 Topic Filter 可以用作在该 Session 中标识 Subscription 的键。

如果有多个 Client，每个都有自己的 Non-shared Subscription 到同一 Topic，每个 Client 都会获得在该 Topic 上发布的 Application Message 的自己的副本。这意味着 Non-shared Subscription 不能用于在多个消费 Client 之间负载均衡 Application Message，因为在这种情况下，每条消息都会传递给每个订阅的 Client。

### 4.8.2 Shared Subscriptions 共享订阅

Shared Subscription 可以与多个订阅的 MQTT Session 相关联。与 Non-shared Subscription 一样，它有一个 Topic Filter 和 Subscription Options；但是，与其 Topic Filter 匹配的发布仅发送到其订阅 Session 之一。Shared Subscription 在多个消费 Client 并行共享发布处理的情况下很有用。

Shared Subscription 使用特殊风格的 Topic Filter 进行标识。此过滤器的格式为：

```
$share/{ShareName}/{filter}
```

- $share 是一个字面字符串，将 Topic Filter 标记为 Shared Subscription Topic Filter。
- {ShareName} 是一个不包含 "/"、"+" 或 "#" 的字符串
- {filter} 字符串的其余部分具有与非共享订阅中的 Topic Filter 相同的语法和语义。请参阅第 4.7 节。

Shared Subscription 的 Topic Filter **必须**以 $share/ 开头，并且**必须**包含至少一个字符长的 ShareName [MQTT-4.8.2-1]。ShareName **禁止**包含字符 "/"、"+" 或 "#"，但**必须**后跟一个 "/" 字符。这个 "/" 字符**必须**后跟一个 Topic Filter [MQTT-4.8.2-2]，如第 4.7 节所述。

**非规范性评论**

Shared Subscription 是在 MQTT Server 的范围内定义的，而不是 Session 的范围。Shared Subscription 的 Topic Filter 中包含 ShareName，以便 Server 上可以有多个具有相同 {filter} 组件的 Shared Subscription。通常，应用程序使用 ShareName 来表示正在共享 Subscription 的 Session 组。

示例：

- Shared Subscription "$share/consumer1/sport/tennis/+" 和 "$share/consumer2/sport/tennis/+" 是不同的共享订阅，因此可以与不同的 Session 组关联。它们都与 Non-shared Subscription 到 sport/tennis/+ 匹配相同的 Topic。

如果发布一条匹配 sport/tennis/+ 的消息，则副本将发送到订阅 $share/consumer1/sport/tennis/+ 的 Session 之一，消息的单独副本将发送到订阅 $share/consumer2/sport/tennis/+ 的 Session 之一，更多副本将发送到任何具有 Non-shared Subscription 到 sport/tennis/+ 的 Client。

- Shared Subscription "$share/consumer1//finance" 与 Non-shared Subscription 到 /finance 匹配相同的 Topic。

请注意，"$share/consumer1//finance" 和 "$share/consumer1/sport/tennis/+" 是不同的共享订阅，即使它们具有相同的 ShareName。虽然它们可能以某种方式相关，但它们具有相同的 ShareName 并不意味着它们之间存在特定的关系。

Shared Subscription 是通过在 SUBSCRIBE 请求中使用 Shared Subscription Topic Filter 创建的。只要只有一个 Session 订阅特定的 Shared Subscription，该共享 Subscription 的行为就像 Non-shared Subscription，除了：

- Topic Filter 的 $share 和 {ShareName} 部分在与发布匹配时不被考虑。
- 当 Session 首次订阅时，不会向其发送 Retained Message。它将在发布时收到其他匹配的消息。

一旦 Shared Subscription 存在，其他 Session 可以订阅具有相同 Shared Subscription Topic Filter。新 Session 作为额外的订阅者与 Shared Subscription 关联。Retained Message 不会发送给这个新订阅者。每个匹配 Shared Subscription 的后续 Application Message 现在发送到订阅 Shared Subscription 的 Session 之一且仅一个。

Session 可以通过发送包含完整 Shared Subscription Topic Filter 的 UNSUBSCRIBE 报文来显式地从 Shared Subscription 分离。当 Session 终止时，它们也会从 Shared Subscription 分离。

Shared Subscription 在与至少一个 Session 关联时持续存在（即已向其 Topic Filter 发出成功 SUBSCRIBE 请求且尚未完成相应 UNSUBSCRIBE 的 Session）。当最初创建它的 Session 取消订阅时，Shared Subscription 会继续存在，除非此时没有其他 Session。当不再有任何订阅它的 Session 时，Shared Subscription 结束，并删除与其关联的任何未传递消息。

**Shared Subscription 注意事项**

- 如果有多个 Session 订阅 Shared Subscription，Server 实现可以逐条消息自由选择使用哪个 Session 以及使用什么标准进行此选择。

- 不同的订阅 Client 被允许在其 SUBSCRIBE 报文中请求不同的 Requested QoS 级别。Server 决定向每个 Client 授予哪个 Maximum QoS，并且允许向不同的订阅者授予不同的 Maximum QoS 级别。当向 Client 发送 Application Message 时，Server **必须**遵守 Client Subscription 的授予 QoS [MQTT-4.8.2-3]，就像向非共享订阅者发送消息一样。

- 如果 Server 正在向其选择的订阅 Client 发送 QoS 2 消息，并且在传递完成之前与 Client 的连接断开，Server **必须**在重新连接时完成向该 Client 的消息传递 [MQTT-4.8.2-4]，如第 4.3.3 节所述。如果 Client 的 Session 在 Client 重新连接之前终止，Server **禁止**将 Application Message 发送给任何其他订阅的 Client [MQTT-4.8.2-5]。

- 如果 Server 正在向其选择的订阅 Client 发送 QoS 1 消息，并且在 Server 收到来自 Client 的确认之前与该 Client 的连接断开，Server **可以**等待 Client 重新连接并将消息重传给该 Client。如果 Client 的 Session 在 Client 重新连接之前终止，Server **应该**将 Application Message 发送给订阅同一 Shared Subscription 的另一个 Client。它**可以**在失去与第一个 Client 的连接后立即尝试将消息发送给另一个 Client。

- 如果 Client 以 Reason Code 为 0x80 或更大的 PUBACK 或 PUBREC 响应来自 Server 的 PUBLISH 报文，Server **必须**丢弃 Application Message 且不尝试将其发送给任何其他 Subscriber [MQTT-4.8.2-6]。

- Client 被允许在已订阅该 Shared Subscription 的 Session 上向 Shared Subscription 提交第二个 SUBSCRIBE 请求。例如，它可能这样做以更改其 Subscription 的 Requested QoS，或者因为它不确定上一个订阅在上一个连接关闭之前是否完成。这不会增加 Session 与 Shared Subscription 关联的次数，因此 Session 将在第一次 UNSUBSCRIBE 时离开 Shared Subscription。

- 每个 Shared Subscription 独立于任何其他 Subscription。可以有具有重叠过滤器的两个 Shared Subscription。在这种情况下，匹配两个 Shared Subscription 的消息将由它们两者分别处理。如果 Client 有 Shared Subscription 和 Non-shared Subscription，并且消息匹配它们两者，则 Client 将凭借具有 Non-shared Subscription 收到消息的副本。消息的第二份副本将发送给 Shared Subscription 的订阅者之一，这可能导致向该 Client 发送第二份副本。

## 4.9 Flow Control 流量控制

Client 和 Server 通过使用第 3.1.2.11.4 节和第 3.2.2.3.2 节中描述的 Receive Maximum 值来控制它们收到的未确认 PUBLISH 报文的数量。Receive Maximum 建立一个发送配额，用于限制可以在不收到 PUBACK（对于 QoS 1）或 PUBCOMP（对于 QoS 2）的情况下发送的 PUBLISH QOS > 0 报文的数量。PUBACK 和 PUBCOMP 以下面描述的方式补充配额。

Client 或 Server **必须**将其初始发送配额设置为不超过 Receive Maximum 的非零值 [MQTT-4.9.0-1]。

每次 Client 或 Server 以 QoS > 0 发送 PUBLISH 报文时，它会递减发送配额。如果发送配额达到零，Client 或 Server **禁止**发送任何更多 QoS > 0 的 PUBLISH 报文 [MQTT-4.9.0-2]。它**可以**继续发送 QoS 0 的 PUBLISH 报文，或者**可以**选择也暂停发送这些报文。即使配额为零，Client 和 Server 也**必须**继续处理和响应所有其他 MQTT 控制报文 [MQTT-4.9.0-3]。

发送配额增加 1：

- 每次收到 PUBACK 或 PUBCOMP 报文时，无论 PUBACK 或 PUBCOMP 是否携带错误代码。
- 每次收到 Return Code 为 0x80 或更大的 PUBREC 报文时。

如果发送配额已经等于初始发送配额，则不会增加发送配额。尝试增加到初始发送配额以上可能是由在建立新 Network Connection 后重传 PUBREL 报文引起的。

有关如果 Client 和 Server 收到超过 Receive Maximum 允许的 PUBLISH 报文时如何反应的描述，请参阅第 3.3.4 节。

发送配额和 Receive Maximum 值不会跨 Network Connection 保留，并且每个新 Network Connection 都会按上述方式重新初始化。它们不是 Session State 的一部分。

## 4.10 Request / Response 请求/响应

某些应用程序或标准可能希望在 MQTT 上运行请求/响应交互。此版本的 MQTT 包含三个可用于此目的的属性：

- Response Topic，在第 3.3.2.3.5 节中描述
- Correlation Data，在第 3.3.2.3.6 节中描述
- Request Response Information，在第 3.1.2.11.7 节中描述
- Response Information，在第 3.2.2.3.14 节中描述

以下非规范性章节描述了如何使用这些属性。

Client 通过发布设置了 Response Topic 的 Application Message 来发送请求消息，如第 3.3.2.3.5 节所述。请求可以包含 Correlation Data 属性，如第 3.3.2.3.6 节所述。

### 4.10.1 Basic Request Response (non-normative) 基本请求响应（非规范性）

请求/响应交互按如下方式进行：

1. MQTT Client（请求者）向 Topic 发布请求消息。请求消息是具有 Response Topic 的 Application Message。
2. 另一个 MQTT Client（响应者）已订阅与发布请求消息时使用的 Topic Name 匹配的 Topic Filter。因此，它收到请求消息。可能有多个响应者订阅此 Topic Name，也可能没有。
3. 响应者根据请求消息采取适当的操作，然后将响应消息发布到请求消息中携带的 Response Topic 属性中的 Topic Name。
4. 在典型用法中，请求者已订阅 Response Topic，从而收到响应消息。但是，可能有其他 Client 订阅了 Response Topic，在这种情况下，响应消息也将被该 Client 接收和处理。与请求消息一样，发送响应消息的 Topic 可能被多个 Client 订阅，也可能没有被任何 Client 订阅。

如果请求消息包含 Correlation Data 属性，响应者将此属性复制到响应消息中，响应消息的接收者使用它将响应消息与原始请求关联起来。响应消息不包含 Response Topic 属性。

MQTT Server 在请求消息中转发 Response Topic 和 Correlation Data 属性，以及在响应消息中转发 Correlation Data。Server 像处理任何其他 Application Message 一样处理请求消息和响应消息。

请求者通常在发布请求消息之前订阅 Response Topic。如果在发送响应消息时 Response Topic 没有订阅者，响应消息将不会传递给任何 Client。

请求消息和响应消息可以是任何 QoS，响应者可以使用具有非零 Session Expiry Interval 的 Session。通常以 QoS 0 发送请求消息，并且仅在预期响应者连接时发送。但这并不是必需的。

响应者可以使用 Shared Subscription 来允许多个响应 Client 池。但请注意，在使用 Shared Subscription 时，多个 Client 之间的消息传递顺序不能保证。

请求者有责任确保其具有发布到请求 Topic 的必要权限，并订阅其在 Response Topic 属性中设置的 Topic Name。响应者有责任确保其具有订阅请求 Topic 和发布到 Response Topic 的权限。虽然 Topic 授权超出了本规范的范围，但建议 Server 实施此类授权。

### 4.10.2 Determining a Response Topic value (non-normative) 确定 Response Topic 值（非规范性）

请求者可以以他们选择的任何方式确定用作其 Response Topic 的 Topic Name，包括通过本地配置。为了避免不同请求者之间的冲突，请求者 Client 使用的 Response Topic 最好是该 Client 唯一的。由于请求者和响应者通常需要被授权到这些 Topic，使用随机 Topic Name 可能是一个授权挑战。

为了帮助解决这个问题，本规范在 CONNACK 报文中定义了一个名为 Response Information 的属性。Server 可以使用此属性来指导 Client 选择要使用的 Response Topic。此机制对 Client 和 Server 都是可选的。在连接时，Client 通过在 CONNECT 报文中设置 Request Response Information 属性来请求 Server 发送 Response Information。这导致 Server 在 CONNACK 报文中发送 Response Information 属性（UTF-8 编码字符串）。

本规范没有定义 Response Information 的内容，但它可以用来传递 Topic 树的全局唯一部分，该部分至少在其 Session 的生命周期内为该 Client 保留。使用此机制允许此配置在 Server 中完成一次，而不是在每个 Client 中完成。

有关 Response Information 的定义，请参阅第 3.1.2.11.7 节。

## 4.11 Server redirection Server 重定向

Server 可以通过发送带有 Reason Code 0x9C（Use another server）或 0x9D（Server moved）的 CONNACK 或 DISCONNECT 来请求 Client 使用另一个 Server，如第 4.13 节所述。发送这些 Reason Code 之一时，Server **可以**还包含一个 Server Reference 属性，以指示 Client **应该**使用的 Server 或 Server 的位置。

Reason Code 0x9C（Use another server）指定 Client **应该**临时切换到使用另一个 Server。另一个 Server 已经为 Client 所知，或者使用 Server Reference 指定。

Reason Code 0x9D（Server moved）指定 Client **应该**永久切换到使用另一个 Server。另一个 Server 已经为 Client 所知，或者使用 Server Reference 指定。

Server Reference 是一个 UTF-8 编码字符串。此字符串的值是以空格分隔的引用列表。此处未指定引用的格式。

**非规范性评论**

建议每个引用由一个名称组成，可选地后跟冒号和端口号。如果名称包含冒号，则名称字符串可以用方括号（"[" 和 "]"）括起来。用方括号括起来的名称不能包含右方括号（"]"）字符。这用于表示使用冒号分隔符的 IPv6 字面地址。这是 [RFC3986] 中描述的 URI authority 的简化版本。

**非规范性评论**

Server Reference 中的名称通常表示主机名、DNS 名称 [RFC1035]、SRV 名称 [RFC2782] 或字面 IP 地址。冒号分隔符后的值通常是十进制的端口号。如果端口信息来自名称解析（如 SRV）或使用默认值，则不需要此值。

**非规范性评论**

如果给出多个引用，期望 Client 将选择其中之一。

**非规范性评论**

Server Reference 的示例包括：

```
myserver.xyz.org
myserver.xyz.org:8883
10.10.151.22:8883 [fe80::9610:3eff:fe1c]:1883
```

Server 被允许永远不发送 Server Reference，Client 被允许忽略 Server Reference。此功能可用于允许负载均衡、Server 重定位和 Client 配置到 Server。

## 4.12 Enhanced authentication 增强认证

MQTT CONNECT 报文支持使用 User Name 和 Password 字段对 Network Connection 进行基本认证。虽然这些字段是为简单的密码认证命名的，但它们可以用于携带其他形式的认证，例如将令牌作为 Password 传递。

增强认证扩展了这种基本认证，以包括质询/响应式认证。它可能涉及在 CONNECT 和 CONNACK 报文之后，Client 和 Server 之间交换 AUTH 报文。

要开始增强认证，Client 在 CONNECT 报文中包含 Authentication Method。这指定了要使用的认证方法。如果 Server 不支持 Client 提供的 Authentication Method，它**可以**发送带有 Reason Code 0x8C（Bad authentication method）或 0x87（Not Authorized）的 CONNACK，如第 4.13 节所述，并且**必须**关闭 Network Connection [MQTT-4.12.0-1]。

Authentication Method 是 Client 和 Server 之间关于 Authentication Data 中发送的数据的含义以及 CONNECT 中的任何其他字段，以及 Client 和 Server 完成认证所需的交换和处理的协议。

**非规范性评论**

Authentication Method 通常是 SASL 机制，使用此类注册名称有助于交换。但是，Authentication Method 不限于使用注册的 SASL 机制。

如果 Client 选择的 Authentication Method 指定 Client 首先发送数据，Client **应该**在 CONNECT 报文中包含 Authentication Data 属性。此属性可用于提供 Authentication Method 指定的数据。Authentication Data 的内容由认证方法定义。

如果 Server 需要额外信息来完成认证，它可以向 Client 发送 AUTH 报文。此报文**必须**包含 Reason Code 0x18（Continue authentication）[MQTT-4.12.0-2]。如果认证方法要求 Server 向 Client 发送认证数据，则在 Authentication Data 中发送。

Client 通过发送另一个 AUTH 报文来响应来自 Server 的 AUTH 报文。此报文**必须**包含 Reason Code 0x18（Continue authentication）[MQTT-4.12.0-3]。如果认证方法要求 Client 为 Server 发送认证数据，则在 Authentication Data 中发送。

Client 和 Server 根据需要交换 AUTH 报文，直到 Server 通过发送 Reason Code 为 0 的 CONNACK 接受认证。如果认证的接受需要向 Client 发送数据，则在 Authentication Data 中发送。

Client 可以在此过程中的任何时候关闭连接。它**可以**在这样做之前发送 DISCONNECT 报文。Server 可以在此过程中的任何时候拒绝认证。它**可以**发送 Reason Code 为 0x80 或以上的 CONNACK，如第 4.13 节所述，并且**必须**关闭 Network Connection [MQTT-4.12.0-4]。

如果初始 CONNECT 报文包含 Authentication Method 属性，那么所有 AUTH 报文和任何成功的 CONNACK 报文**必须**包含与 CONNECT 报文中相同值的 Authentication Method 属性 [MQTT-4.12.0-5]。

增强认证的实现对于 Client 和 Server 都是可选的。如果 Client 在 CONNECT 中不包含 Authentication Method，Server **禁止**发送 AUTH 报文，并且**禁止**在 CONNACK 报文中发送 Authentication Method [MQTT-4.12.0-6]。如果 Client 在 CONNECT 中不包含 Authentication Method，Client **禁止**向 Server 发送 AUTH 报文 [MQTT-4.12.0-7]。

如果 Client 在 CONNECT 报文中不包含 Authentication Method，Server **应该**使用 CONNECT 报文、TLS Session 和 Network Connection 中的部分或全部信息进行认证。

**非规范性示例 - SCRAM 质询**

- Client to Server: CONNECT Authentication Method="SCRAM-SHA-1" Authentication Data=client-first-data
- Server to Client: AUTH rc=0x18 Authentication Method="SCRAM-SHA-1" Authentication Data=server-first-data
- Client to Server: AUTH rc=0x18 Authentication Method="SCRAM-SHA-1" Authentication Data=client-final-data
- Server to Client: CONNACK rc=0 Authentication Method="SCRAM-SHA-1" Authentication Data=server-final-data

**非规范性示例 - Kerberos 质询**

- Client to Server: CONNECT Authentication Method="GS2-KRB5"
- Server to Client: AUTH rc=0x18 Authentication Method="GS2-KRB5"
- Client to Server: AUTH rc=0x18 Authentication Method="GS2-KRB5" Authentication Data=initial context token
- Server to Client: AUTH rc=0x18 Authentication Method="GS2-KRB5" Authentication Data=reply context token
- Client to Server: AUTH rc=0x18 Authentication Method="GS2-KRB5"
- Server to Client: CONNACK rc=0 Authentication Method="GS2-KRB5" Authentication Data=outcome of authentication

### 4.12.1 Re-authentication 重新认证

如果 Client 在 CONNECT 报文中提供了 Authentication Method，它可以在收到 CONNACK 后的任何时候启动重新认证。它通过发送 Reason Code 为 0x19（Re-authentication）的 AUTH 报文来实现。Client **必须**将 Authentication Method 设置为与最初用于认证 Network Connection 的 Authentication Method 相同的值 [MQTT-4.12.1-1]。如果认证方法要求 Client 首先发送数据，则此 AUTH 报文包含第一段认证数据作为 Authentication Data。

Server 通过向 Client 发送 Reason Code 为 0x00（Success）的 AUTH 报文来响应此重新认证请求，以指示重新认证已完成，或发送 Reason Code 为 0x18（Continue authentication）来指示需要更多认证数据。Client 可以通过发送 Reason Code 为 0x18（Continue authentication）的 AUTH 报文来响应额外的认证数据。此流程与原始认证一样继续，直到重新认证完成或重新认证失败。

如果重新认证失败，Client 或 Server **应该**发送带有适当 Reason Code 的 DISCONNECT，如第 4.13 节所述，并且**必须**关闭 Network Connection [MQTT-4.12.1-2]。

在此重新认证序列期间，Client 和 Server 之间的其他报文流可以使用先前的认证继续。

**非规范性评论**

Server 可能会通过拒绝重新认证来限制 Client 在重新认证中可以尝试的更改范围。例如，如果 Server 不允许更改 User Name，它可能会失败任何更改 User Name 的重新认证尝试。

## 4.13 Handling errors 处理错误

### 4.13.1 Malformed Packet and Protocol Errors 格式错误报文和协议错误

格式错误报文和协议错误的定义包含在第 1.2 节术语中，这些错误情况中的一部分（但不是全部）在整个规范中都有注明。Client 或 Server 检查其收到的 MQTT 控制报文的严格程度将是以下因素之间的折衷：

- Client 或 Server 实现的大小。
- 实现支持的功能。
- 接收者信任发送者发送正确 MQTT 控制报文的程度。
- 接收者信任网络正确传递 MQTT 控制报文的程度。
- 继续处理不正确报文的后果。

如果发送者符合本规范，它不会发送格式错误报文或导致协议错误。但是，如果 Client 在收到 CONNACK 之前发送 MQTT 控制报文，它可能会导致协议错误，因为它对 Server 能力做出了错误的假设。请参阅第 3.1.4 节 CONNECT Actions。

用于格式错误报文和协议错误的 Reason Code 包括：

- 0x81 Malformed Packet 格式错误报文
- 0x82 Protocol Error 协议错误
- 0x93 Receive Maximum exceeded 超过 Receive Maximum
- 0x95 Packet too large 报文过大
- 0x9A Retain not supported 不支持 Retain
- 0x9B QoS not supported 不支持 QoS
- 0x9E Shared Subscriptions not supported 不支持 Shared Subscription
- 0xA1 Subscription Identifiers not supported 不支持 Subscription Identifier
- 0xA2 Wildcard Subscriptions not supported 不支持通配符 Subscription

当 Client 检测到格式错误报文或协议错误，且规范中给出了 Reason Code 时，它**应该**关闭 Network Connection。在 AUTH 报文出错的情况下，它**可以**在关闭 Network Connection 之前发送包含 Reason Code 的 DISCONNECT 报文。在任何其他报文出错的情况下，它**应该**在关闭 Network Connection 之前发送包含 Reason Code 的 DISCONNECT 报文。使用 Reason Code 0x81（Malformed Packet）或 0x82（Protocol Error），除非在第 3.14.2.1 节 Disconnect Reason Code 中定义了更具体的 Reason Code。

当 Server 检测到格式错误报文或协议错误，且规范中给出了 Reason Code 时，它**必须**关闭 Network Connection [MQTT-4.13.1-1]。在 CONNECT 报文出错的情况下，它**可以**在关闭 Network Connection 之前发送包含 Reason Code 的 CONNACK 报文。在任何其他报文出错的情况下，它**应该**在关闭 Network Connection 之前发送包含 Reason Code 的 DISCONNECT 报文。使用 Reason Code 0x81（Malformed Packet）或 0x82（Protocol Error），除非在第 3.2.2.2 节 - Connect Reason Code 或第 3.14.2.1 节 - Disconnect Reason Code 中定义了更具体的 Reason Code。对其他 Session 没有影响。

如果 Server 或 Client 省略检查 MQTT 控制报文的某些功能，它可能无法检测到错误，因此可能允许数据被损坏。

### 4.13.2 Other errors 其他错误

格式错误报文和协议错误以外的错误无法被发送者预期，因为接收者可能有它没有传达给发送者的约束。接收 Client 或 Server 可能会遇到瞬态错误，例如内存不足，导致无法成功处理单个 MQTT 控制报文。

带有 Reason Code 0x80 或更大的确认报文 PUBACK、PUBREC、PUBREL、PUBCOMP、SUBACK、UNSUBACK 表示由 Packet Identifier 标识的已收到报文出错。对其他 Session 或同一 Session 上流动的其他报文没有影响。

CONNACK 和 DISCONNECT 报文允许使用 0x80 或更大的 Reason Code 来指示 Network Connection 将被关闭。如果指定了 0x80 或更大的 Reason Code，那么无论是否发送 CONNACK 或 DISCONNECT，Network Connection 都**必须**关闭 [MQTT-4.13.2-1]。发送这些 Reason Code 之一对任何其他 Session 没有影响。

如果控制报文包含多个错误，报文的接收者可以以任何顺序验证报文，并对发现的任何错误采取适当的操作。

有关处理不允许的 Unicode 代码点的信息，请参阅第 5.4.9 节。