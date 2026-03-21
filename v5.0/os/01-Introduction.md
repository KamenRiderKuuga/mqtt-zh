# 1 Introduction（简介）

## 1.0 Intellectual property rights policy（知识产权政策）

本规范根据 OASIS IPR Policy 的 [Non-Assertion](https://www.oasis-open.org/policies-guidelines/ipr#Non-Assertion-Mode) 模式提供，该模式是在技术委员会成立时选择的。有关是否已披露可能对本规范实施至关重要的任何专利，以及任何专利许可条款的提供信息，请参阅 TC 网页的知识产权部分 (<https://www.oasis-open.org/committees/mqtt/ipr.php>)。

## 1.1 Organization of the MQTT specification（MQTT 规范的组织结构）

本规范分为七章：

- [Chapter 1 - Introduction](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Introduction)（第 1 章 - 简介）
- [Chapter 2 - MQTT Control Packet format](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packet)（第 2 章 - MQTT 控制报文格式）
- [Chapter 3 - MQTT Control Packets](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_MQTT_Control_Packets)（第 3 章 - MQTT 控制报文）
- [Chapter 4 - Operational behavior](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Operational_behavior)（第 4 章 - 操作行为）
- [Chapter 5 - Security](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Security)（第 5 章 - 安全）
- [Chapter 6 - Using WebSocket as a network transport](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Using_WebSocket_as)（第 6 章 - 使用 WebSocket 作为网络传输）
- [Chapter 7 - Conformance Targets](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Conformance)（第 7 章 - 一致性目标）

## 1.2 Terminology（术语）

本规范中的关键词 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"MAY" 和 "OPTIONAL" 应按照 IETF RFC 2119 [RFC2119] 中的描述进行解释，除非它们出现在标记为非规范性的文本中。

**Network Connection（网络连接）：**

`MQTT` 使用的底层传输协议提供的构造。

- 它将 `Client` 连接到 `Server`。
- 它提供了在两个方向上发送有序、无损字节流的手段。

有关非规范性示例，请参阅 [4.2 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Network_Connections) `Network Connection`。

**Application Message（应用消息）：**

`MQTT` 协议通过网络为应用程序传输的数据。当 `Application Message` 通过 `MQTT` 传输时，它包含负载数据、`Quality of Service` (`QoS`)、一组 `Properties` 和一个 `Topic Name`。

**Client（客户端）：**

使用 `MQTT` 的程序或设备。`Client`：

- 建立到 `Server` 的 `Network Connection`
- 发布其他 `Client` 可能感兴趣的 `Application Message`
- 订阅以请求接收它感兴趣的 `Application Message`
- 取消订阅以移除对 `Application Message` 的请求
- 关闭到 `Server` 的 `Network Connection`

**Server（服务端）：**

作为发布 `Application Message` 的 `Client` 与已创建 `Subscription` 的 `Client` 之间中介的程序或设备。`Server`：

- 接受来自 `Client` 的 `Network Connection`
- 接受 `Client` 发布的 `Application Message`
- 处理来自 `Client` 的 `SUBSCRIBE` 和 `UNSUBSCRIBE` 请求
- 转发与 `Client` `Subscription` 匹配的 `Application Message`
- 关闭来自 `Client` 的 `Network Connection`

**Session（会话）：**

`Client` 与 `Server` 之间的有状态交互。某些 `Session` 仅持续 `Network Connection` 的持续时间，其他 `Session` 可以跨越 `Client` 与 `Server` 之间多个连续的 `Network Connection`。

**Subscription（订阅）：**

`Subscription` 包含一个 `Topic Filter` 和一个最大 `QoS`。`Subscription` 与单个 `Session` 关联。一个 `Session` 可以包含多个 `Subscription`。`Session` 内的每个 `Subscription` 具有不同的 `Topic Filter`。

**Shared Subscription（共享订阅）：**

`Shared Subscription` 包含一个 `Topic Filter` 和一个最大 `QoS`。`Shared Subscription` 可以与多个 `Session` 关联以支持更广泛的消息交换模式。与 `Shared Subscription` 匹配的 `Application Message` 仅发送给与这些 `Session` 之一关联的 `Client`。一个 `Session` 可以订阅多个 `Shared Subscription`，并且可以同时包含 `Shared Subscription` 和非共享的 `Subscription`。

**Wildcard Subscription（通配符订阅）：**

`Wildcard Subscription` 是 `Topic Filter` 包含一个或多个通配符字符的 `Subscription`。这允许订阅匹配多个 `Topic Name`。有关 `Topic Filter` 中通配符字符的描述，请参阅 [4.7 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Topic_Names_and)。

**Topic Name（主题名称）：**

附加到 `Application Message` 的标签，与 `Server` 已知的 `Subscription` 进行匹配。

**Topic Filter（主题过滤器）：**

`Subscription` 中包含的表达式，用于指示对一个或多个主题的兴趣。`Topic Filter` 可以包含通配符字符。

**MQTT Control Packet（MQTT 控制报文）：**

通过网络连接发送的信息包。`MQTT` 规范定义了 15 种不同类型的 `MQTT Control Packet`，例如 PUBLISH 报文用于传递 `Application Message`。

**Malformed Packet（格式错误报文）：**

无法根据本规范解析的控制报文。有关错误处理的信息，请参阅 [4.13 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Handling_errors)。

**Protocol Error（协议错误）：**

在报文解析后检测到的错误，发现报文包含协议不允许的数据或与 `Client` 或 `Server` 的状态不一致。有关错误处理的信息，请参阅 [4.13 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#S4_13_Errors)。

**Will Message（遗嘱消息）：**

在 `Network Connection` 非正常关闭的情况下，`Server` 在 `Network Connection` 关闭后发布的 `Application Message`。有关 `Will Message` 的信息，请参阅 [3.1.2.5 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_Toc479576982)。

**Disallowed Unicode code point（不允许的 Unicode 码点）：**

不应包含在 `UTF-8 Encoded String` 中的 `Unicode Control Codes` 和 `Unicode Noncharacters` 集合。有关 `Disallowed Unicode code points` 的更多信息，请参阅 [1.5.4 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#_UTF-8_Encoded_String)。

## 1.3 Normative references（规范性引用文件）

本规范引用了以下文件：

- **[RFC2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, DOI 10.17487/RFC2119, March 1997, <http://www.rfc-editor.org/info/rfc2119>

- **[RFC3629]** Yergeau, F., "UTF-8, a transformation format of ISO 10646", STD 63, RFC 3629, DOI 10.17487/RFC3629, November 2003, <http://www.rfc-editor.org/info/rfc3629>

- **[RFC6455]** Fette, I. and A. Melnikov, "The WebSocket Protocol", RFC 6455, DOI 10.17487/RFC6455, December 2011, <http://www.rfc-editor.org/info/rfc6455>

- **[Unicode]** The Unicode Consortium. The Unicode Standard, <http://www.unicode.org/versions/latest/>

## 1.4 Non-normative references（非规范性引用文件）

以下文件作为参考资料提供：

- **[RFC0793]** Postel, J., "Transmission Control Protocol", STD 7, RFC 793, DOI 10.17487/RFC0793, September 1981, <http://www.rfc-editor.org/info/rfc793>

- **[RFC5246]** Dierks, T. and E. Rescorla, "The Transport Layer Security (TLS) Protocol Version 1.2", RFC 5246, DOI 10.17487/RFC5246, August 2008, <http://www.rfc-editor.org/info/rfc5246>

- **[AES]** Advanced Encryption Standard (AES) (FIPS PUB 197). <https://csrc.nist.gov/csrc/media/publications/fips/197/final/documents/fips-197.pdf>

- **[CHACHA20]** ChaCha20 and Poly1305 for IETF Protocols <https://tools.ietf.org/html/rfc7539>

- **[FIPS1402]** Security Requirements for Cryptographic Modules (FIPS PUB 140-2) <https://csrc.nist.gov/csrc/media/publications/fips/140/2/final/documents/fips1402.pdf>

- **[IEEE 802.1AR]** IEEE Standard for Local and metropolitan area networks - Secure Device Identity <http://standards.ieee.org/findstds/standard/802.1AR-2009.html>

- **[ISO29192]** ISO/IEC 29192-1:2012 Information technology -- Security techniques -- Lightweight cryptography -- Part 1: General <https://www.iso.org/standard/56425.html>

- **[MQTT NIST]** MQTT supplemental publication, MQTT and the NIST Framework for Improving Critical Infrastructure Cybersecurity <http://docs.oasis-open.org/mqtt/mqtt-nist-cybersecurity/v1.0/mqtt-nist-cybersecurity-v1.0.html>

- **[MQTTV311]** MQTT V3.1.1 Protocol Specification <http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html>

- **[ISO20922]** MQTT V3.1.1 ISO Standard (ISO/IEC 20922:2016) <https://www.iso.org/standard/69466.html>

- **[NISTCSF]** Improving Critical Infrastructure Cybersecurity Executive Order 13636 <https://www.nist.gov/sites/default/files/documents/itl/preliminary-cybersecurity-framework.pdf>

- **[NIST7628]** NISTIR 7628 Guidelines for Smart Grid Cyber Security Catalogue <https://www.nist.gov/sites/default/files/documents/smartgrid/nistir-7628_total.pdf>

- **[NSAB]** NSA Suite B Cryptography <http://www.nsa.gov/ia/programs/suiteb_cryptography/>

- **[PCIDSS]** PCI-DSS Payment Card Industry Data Security Standard <https://www.pcisecuritystandards.org/pci_security/>

- **[RFC1928]** Leech, M., Ganis, M., Lee, Y., Kuris, R., Koblas, D., and L. Jones, "SOCKS Protocol Version 5", RFC 1928, DOI 10.17487/RFC1928, March 1996, <http://www.rfc-editor.org/info/rfc1928>

- **[RFC4511]** Sermersheim, J., Ed., "Lightweight Directory Access Protocol (LDAP): The Protocol", RFC 4511, DOI 10.17487/RFC4511, June 2006, <http://www.rfc-editor.org/info/rfc4511>

- **[RFC5280]** Cooper, D., Santesson, S., Farrell, S., Boeyen, S., Housley, R., and W. Polk, "Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile", RFC 5280, DOI 10.17487/RFC5280, May 2008, <http://www.rfc-editor.org/info/rfc5280>

- **[RFC6066]** Eastlake 3rd, D., "Transport Layer Security (TLS) Extensions: Extension Definitions", RFC 6066, DOI 10.17487/RFC6066, January 2011, <http://www.rfc-editor.org/info/rfc6066>

- **[RFC6749]** Hardt, D., Ed., "The OAuth 2.0 Authorization Framework", RFC 6749, DOI 10.17487/RFC6749, October 2012, <http://www.rfc-editor.org/info/rfc6749>

- **[RFC6960]** Santesson, S., Myers, M., Ankney, R., Malpani, A., Galperin, S., and C. Adams, "X.509 Internet Public Key Infrastructure Online Certificate Status Protocol - OCSP", RFC 6960, DOI 10.17487/RFC6960, June 2013, <http://www.rfc-editor.org/info/rfc6960>

- **[SARBANES]** Sarbanes-Oxley Act of 2002. <http://www.gpo.gov/fdsys/pkg/PLAW-107publ204/html/PLAW-107publ204.htm>

- **[USEUPRIVSH]** U.S.-EU Privacy Shield Framework <https://www.privacyshield.gov>

- **[RFC3986]** Berners-Lee, T., Fielding, R., and L. Masinter, "Uniform Resource Identifier (URI): Generic Syntax", STD 66, RFC 3986, DOI 10.17487/RFC3986, January 2005, <http://www.rfc-editor.org/info/rfc3986>

- **[RFC1035]** Mockapetris, P., "Domain names - implementation and specification", STD 13, RFC 1035, DOI 10.17487/RFC1035, November 1987, <http://www.rfc-editor.org/info/rfc1035>

- **[RFC2782]** Gulbrandsen, A., Vixie, P., and L. Esibov, "A DNS RR for specifying the location of services (DNS SRV)", RFC 2782, DOI 10.17487/RFC2782, February 2000, <http://www.rfc-editor.org/info/rfc2782>

## 1.5 Data representation（数据表示）

### 1.5.1 Bits（位）

字节中的位编号为 7 到 0。第 7 位是最高有效位，最低有效位被分配为第 0 位。

### 1.5.2 Two Byte Integer（双字节整数）

Two Byte Integer 数据值是按大端序排列的 16 位无符号整数：高序字节在前，低序字节在后。这意味着 16 位字在网络上的表示为最高有效字节（MSB）在前，随后是最低有效字节（LSB）。

### 1.5.3 Four Byte Integer（四字节整数）

Four Byte Integer 数据值是按大端序排列的 32 位无符号整数：高序字节在前，其后依次是较低序字节。这意味着 32 位字在网络上的表示为最高有效字节（MSB）在前，随后是次高有效字节（MSB），再随后是下一个次高有效字节（MSB），最后是最低有效字节（LSB）。

### 1.5.4 UTF-8 Encoded String（UTF-8 编码字符串）

`MQTT Control Packet` 中的文本字段编码为 `UTF-8` 字符串。`UTF-8` [RFC3629] 是 `Unicode` [Unicode] 字符的高效编码，优化了 ASCII 字符的编码以支持基于文本的通信。

每个字符串前面都有一个 Two Byte Integer 长度字段，给出 `UTF-8` 编码字符串本身的字节数，如下图 1-1 `UTF-8 Encoded String` 的结构所示。因此，`UTF-8 Encoded String` 的最大大小为 65,535 字节。

除非另有说明，所有 `UTF-8` 编码字符串的长度可以在 0 到 65,535 字节范围内。

**图 1-1 UTF-8 Encoded String 的结构**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | String length MSB ||||||||
| byte 2 | String length LSB ||||||||
| byte 3 ... | UTF-8 encoded character data, if length > 0. ||||||||

`UTF-8 Encoded String` 中的字符数据 **必须** 是 Unicode 规范 [Unicode] 定义并在 RFC 3629 [RFC3629] 中重述的格式正确的 `UTF-8`。特别是，字符数据 **禁止** 包含 U+D800 和 U+DFFF 之间码点的编码 [MQTT-1.5.4-1]。如果 `Client` 或 `Server` 收到包含格式错误 `UTF-8` 的 `MQTT Control Packet`，则它是 `Malformed Packet`。有关处理错误的信息，请参阅 [4.13 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#S4_13_Errors)。

`UTF-8 Encoded String` **禁止** 包含空字符 U+0000 的编码 [MQTT-1.5.4-2]。如果接收方（`Server` 或 `Client`）收到包含 U+0000 的 `MQTT Control Packet`，则它是 `Malformed Packet`。有关处理错误的信息，请参阅 [4.13 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#S4_13_Errors)。

数据 **不应该** 包含下面列出的 `Unicode` [Unicode] 码点的编码。如果接收方（`Server` 或 `Client`）收到包含其中任何一个的 `MQTT Control Packet`，它 **可以** 将其视为 `Malformed Packet`。这些是 Disallowed Unicode code points。有关处理 Disallowed Unicode code points 的更多信息，请参阅 [5.4.9 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#S4_13_Errors)。

- U+0001..U+001F 控制字符
- U+007F..U+009F 控制字符
- Unicode 规范 [Unicode] 中定义为非字符的码点（例如 U+0FFFF）

`UTF-8` 编码序列 0xEF 0xBB 0xBF 无论出现在字符串中的何处，总是被解释为 U+FEFF（"ZERO WIDTH NO-BREAK SPACE"），报文接收方 **禁止** 跳过或剥离它 [MQTT-1.5.4-3]。

**非规范性示例**

例如，字符串 A𪛔，即 LATIN CAPITAL Letter A 后跟码点 U+2A6D4（表示一个 CJK IDEOGRAPH EXTENSION B 字符），编码如下：

**图 1-2 UTF-8 Encoded String 非规范性示例**

| Bit | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|-----|---|---|---|---|---|---|---|---|
| byte 1 | String Length MSB (0x00) ||||||||
| | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | String Length LSB (0x05) ||||||||
| | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 1 |
| byte 3 | 'A' (0x41) ||||||||
| | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 1 |
| byte 4 | (0xF0) ||||||||
| | 1 | 1 | 1 | 1 | 0 | 0 | 0 | 0 |
| byte 5 | (0xAA) ||||||||
| | 1 | 0 | 1 | 0 | 1 | 0 | 1 | 0 |
| byte 6 | (0x9B) ||||||||
| | 1 | 0 | 0 | 1 | 1 | 0 | 1 | 1 |
| byte 7 | (0x94) ||||||||
| | 1 | 0 | 0 | 1 | 0 | 1 | 0 | 0 |

### 1.5.5 Variable Byte Integer（可变字节整数）

`Variable Byte Integer` 使用一种编码方案进行编码，该方案对于最大 127 的值使用单个字节。较大的值处理如下。每个字节的低七位编码数据，最高有效位用于指示表示中是否有后续字节。因此，每个字节编码 128 个值和一个"延续位"。`Variable Byte Integer` 字段的最大字节数为四个。编码值 **必须** 使用表示该值所需的最小字节数 [MQTT-1.5.5-1]。如下表 1-1 `Variable Byte Integer` 的大小所示。

**表 1-1 Variable Byte Integer 的大小**

| Digits | From | To |
|--------|------|-----|
| 1 | 0 (0x00) | 127 (0x7F) |
| 2 | 128 (0x80, 0x01) | 16,383 (0xFF, 0x7F) |
| 3 | 16,384 (0x80, 0x80, 0x01) | 2,097,151 (0xFF, 0xFF, 0x7F) |
| 4 | 2,097,152 (0x80, 0x80, 0x80, 0x01) | 268,435,455 (0xFF, 0xFF, 0xFF, 0x7F) |

**非规范性说明**

将非负整数（X）编码为 `Variable Byte Integer` 编码方案的算法如下：

```
do
    encodedByte = X MOD 128
    X = X DIV 128
    // if there are more data to encode, set the top bit of this byte
    if (X > 0)
        encodedByte = encodedByte OR 128
    endif
    'output' encodedByte
while (X > 0)
```

其中 MOD 是取模运算符（C 语言中的 %），DIV 是整数除法（C 语言中的 /），OR 是按位或运算（C 语言中的 |）。

**非规范性说明**

解码 `Variable Byte Integer` 类型的算法如下：

```
multiplier = 1
value = 0
do
    encodedByte = 'next byte from stream'
    value += (encodedByte AND 127) * multiplier
    if (multiplier > 128*128*128)
        throw Error(Malformed Variable Byte Integer)
    multiplier *= 128
while ((encodedByte AND 128) != 0)
```

其中 AND 是按位与运算符（C 语言中的 &）。

当此算法终止时，`value` 包含 `Variable Byte Integer` 值。

### 1.5.6 Binary Data（二进制数据）

`Binary Data` 由一个 Two Byte Integer 长度表示，该长度指示数据字节数，后跟该数量的字节。因此，`Binary Data` 的长度限制在 0 到 65,535 字节范围内。

### 1.5.7 UTF-8 String Pair（UTF-8 字符串对）

`UTF-8 String Pair` 由两个 `UTF-8 Encoded String` 组成。此数据类型用于保存名称-值对。第一个字符串用作名称，第二个字符串包含值。

两个字符串 **必须** 符合 `UTF-8 Encoded String` 的要求 [MQTT-1.5.7-1]。如果接收方（`Client` 或 `Server`）收到不符合这些要求的字符串对，则它是 `Malformed Packet`。有关处理错误的信息，请参阅 [4.13 节](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#S4_13_Errors)。

## 1.6 Security（安全）

`MQTT Client` 和 `MQTT Server` 实现 **应该** 提供身份验证、授权和安全通信选项，例如第 5 章中讨论的那些。关注关键基础设施、个人身份信息或其他个人或敏感信息的应用程序强烈建议使用这些安全功能。

## 1.7 Editing convention（编辑约定）

本规范中以黄色高亮显示的文本标识一致性声明。每个一致性声明都分配了一个格式为 [MQTT-x.x.x-y] 的引用，其中 x.x.x 是章节号，y 是该章节内的声明计数器。

## 1.8 Change history（变更历史）

### 1.8.1 MQTT v3.1.1

`MQTT` v3.1.1 是 `MQTT` 的第一个 OASIS 标准版本 [MQTTV311]。

`MQTT` v3.1.1 也被标准化为 ISO/IEC 20922:2016 [ISO20922]。

### 1.8.2 MQTT v5.0

`MQTT` v5.0 在保持大部分核心内容不变的情况下，为 `MQTT` 添加了大量新功能。主要功能目标是：

- 增强可扩展性和大规模系统支持
- 改进错误报告
- 规范化常见模式，包括能力发现和请求响应
- 扩展性机制，包括用户属性
- 性能改进和对小型客户端的支持

有关 `MQTT` v5.0 变更的摘要，请参阅 [附录 C](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html#AppendixC)。