# 1 简介（Introduction）

## 1.1 MQTT 章节结构

本规范共分为七章：

- 第 1 章 - 简介
- 第 2 章 - MQTT 控制报文格式
- 第 3 章 - MQTT 控制报文
- 第 4 章 - 操作行为
- 第 5 章 - 安全性
- 第 6 章 - 使用 WebSocket 作为网络传输
- 第 7 章 - 一致性目标

## 1.2 术语

本规范中的关键词 `MUST`、`MUST NOT`、`REQUIRED`、`SHALL`、`SHALL NOT`、`SHOULD`、`SHOULD NOT`、`RECOMMENDED`、`MAY` 和 `OPTIONAL` 均按照 IETF RFC 2119 [RFC2119] 的定义进行解释。

### 网络连接（Network Connection）

`MQTT` 所使用的底层传输协议提供的连接构造。

- 它将客户端（`Client`）与服务器（`Server`）连接起来。
- 它提供双向有序、无丢失的字节流传输。

示例见 4.2 节。

### 应用消息（Application Message）

通过 `MQTT` 协议在网络上传输的应用数据。当应用消息通过 `MQTT` 传输时，它们关联有服务质量（`QoS`）和主题名称（`Topic Name`）。

### 客户端（Client）

使用 `MQTT` 的程序或设备。客户端总是主动与服务器建立网络连接。它可以：

- 发布应用消息供其他客户端订阅。
- 订阅以请求接收感兴趣的应用消息。
- 取消订阅以移除接收应用消息的请求。
- 与服务器断开连接。

### 服务器（Server）

作为中介的程序或设备，负责在发布应用消息的客户端和订阅消息的客户端之间转发消息。服务器可以：

- 接受客户端的网络连接。
- 接收客户端发布的应用消息。
- 处理客户端的订阅和取消订阅请求。
- 转发匹配客户端订阅的应用消息。

### 订阅（Subscription）

订阅由主题过滤器（`Topic Filter`）和最大 `QoS` 组成。一个订阅关联一个会话（`Session`）。一个会话可以包含多个订阅。每个会话中的订阅具有不同的主题过滤器。

### 主题名称（Topic Name）

应用消息附带的标签，用于与服务器已知的订阅进行匹配。服务器会将应用消息的副本发送给每个匹配订阅的客户端。

### 主题过滤器（Topic Filter）

订阅中包含的表达式，用于表示对一个或多个主题的兴趣。主题过滤器可以包含通配符。

### 会话（Session）

客户端与服务器之间的有状态交互。有些会话只持续网络连接期间，有些则可以跨越客户端与服务器之间的多个连续网络连接。

### MQTT 控制报文（MQTT Control Packet）

通过网络连接发送的信息包。`MQTT` 规范定义了 14 种不同类型的控制报文，其中一种（`PUBLISH` 报文）用于传递应用消息。

## 1.3 规范性引用（Normative references）

- **[RFC2119]** Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
  http://www.ietf.org/rfc/rfc2119.txt

- **[RFC3629]** Yergeau, F., "UTF-8, a transformation format of ISO 10646", STD 63, RFC 3629, November 2003.
  http://www.ietf.org/rfc/rfc3629.txt

- **[RFC5246]** Dierks, T. and E. Rescorla, "The Transport Layer Security (TLS) Protocol Version 1.2", RFC 5246, August 2008.
  http://www.ietf.org/rfc/rfc5246.txt

- **[RFC6455]** Fette, I. and A. Melnikov, "The WebSocket Protocol", RFC 6455, December 2011.
  http://www.ietf.org/rfc/rfc6455.txt

- **[Unicode]** The Unicode Consortium. The Unicode Standard.
  http://www.unicode.org/versions/latest/

## 1.4 非规范性引用（Non normative references）

- **[RFC793]** Postel, J. Transmission Control Protocol. STD 7, IETF RFC 793, September 1981.
  http://www.ietf.org/rfc/rfc793.txt

- **[AES]** Advanced Encryption Standard (AES) (FIPS PUB 197).
  http://csrc.nist.gov/publications/fips/fips197/fips-197.pdf

- **[DES]** Data Encryption Standard (DES).
  http://csrc.nist.gov/publications/fips/fips46-3/fips46-3.pdf

- **[FIPS1402]** Security Requirements for Cryptographic Modules (FIPS PUB 140-2).
  http://csrc.nist.gov/publications/fips/fips140-2/fips1402.pdf

- **[IEEE 802.1AR]** IEEE Standard for Local and metropolitan area networks - Secure Device Identity.
  http://standards.ieee.org/findstds/standard/802.1AR-2009.html

- **[ISO29192]** ISO/IEC 29192-1:2012 Information technology -- Security techniques -- Lightweight cryptography -- Part 1: General.
  http://www.iso.org/iso/home/store/catalogue_tc/catalogue_detail.htm?csnumber=56425

- **[MQTT NIST]** MQTT supplemental publication, MQTT and the NIST Framework for Improving Critical Infrastructure Cybersecurity.
  http://docs.oasis-open.org/mqtt/mqtt-nist-cybersecurity/v1.0/mqtt-nist-cybersecurity-v1.0.html

- **[MQTTV31]** MQTT V3.1 Protocol Specification.
  http://public.dhe.ibm.com/software/dw/webservices/ws-mqtt/mqtt-v3r1.html

- **[NISTCSF]** Improving Critical Infrastructure Cybersecurity Executive Order 13636.
  http://www.nist.gov/itl/upload/preliminary-cybersecurity-framework.pdf

- **[NIST7628]** NISTIR 7628 Guidelines for Smart Grid Cyber Security.
  http://www.nist.gov/smartgrid/upload/nistir-7628_total.pdf

- **[NSAB]** NSA Suite B Cryptography.
  http://www.nsa.gov/ia/programs/suiteb_cryptography/

- **[PCIDSS]** PCI-DSS Payment Card Industry Data Security Standard.
  https://www.pcisecuritystandards.org/security_standards/

- **[RFC1928]** Leech, M., Ganis, M., Lee, Y., Kuris, R., Koblas, D., and L. Jones, "SOCKS Protocol Version 5", RFC 1928, March 1996.
  http://www.ietf.org/rfc/rfc1928.txt

- **[RFC4511]** Sermersheim, J., Ed., "Lightweight Directory Access Protocol (LDAP): The Protocol", RFC 4511, June 2006.
  http://www.ietf.org/rfc/rfc4511.txt

- **[RFC5077]** Salowey, J., Zhou, H., Eronen, P., and H. Tschofenig, "Transport Layer Security (TLS) Session Resumption without Server-Side State", RFC 5077, January 2008.
  http://www.ietf.org/rfc/rfc5077.txt

- **[RFC5280]** Cooper, D., Santesson, S., Farrell, S., Boeyen, S., Housley, R., and W. Polk, "Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile", RFC 5280, May 2008.
  http://www.ietf.org/rfc/rfc5280.txt

- **[RFC6066]** Eastlake 3rd, D., "Transport Layer Security (TLS) Extensions: Extension Definitions", RFC 6066, January 2011.
  http://www.ietf.org/rfc/rfc6066.txt

- **[RFC6749]** Hardt, D., Ed., "The OAuth 2.0 Authorization Framework", RFC 6749, October 2012.
  http://www.ietf.org/rfc/rfc6749.txt

- **[RFC6960]** Santesson, S., Myers, M., Ankney, R., Malpani, A., Galperin, S., and C. Adams, "X.509 Internet Public Key Infrastructure Online Certificate Status Protocol - OCSP", RFC 6960, June 2013.
  http://www.ietf.org/rfc/rfc6960.txt

- **[SARBANES]** Sarbanes-Oxley Act of 2002.
  http://www.gpo.gov/fdsys/pkg/PLAW-107publ204/html/PLAW-107publ204.htm

- **[USEUSAFEHARB]** U.S.-EU Safe Harbor.
  http://export.gov/safeharbor/eu/eg_main_018365.asp

## 1.5 数据表示

### 1.5.1 比特

字节中的比特编号为 7 到 0。比特 7 是最高有效位，比特 0 是最低有效位。

### 1.5.2 整数值

整数值为 16 位，采用大端序：高字节在前，低字节在后。这意味着 16 位字在网络上的表示顺序为：最高有效字节（MSB），后跟最低有效字节（LSB）。

### 1.5.3 UTF-8 编码字符串

控制报文中的文本字段编码为 `UTF-8` 字符串。`UTF-8` [RFC3629] 是 `Unicode` [Unicode] 字符的高效编码，针对文本通信中的 ASCII 字符进行了优化。

每个字符串前面都有一个两字节的长度字段，给出 `UTF-8` 编码字符串本身的字节数，如下面的图 1.1 所示。因此，在一个 `UTF-8` 编码字符串组件中传递的字符串大小有限制；不能使用编码后超过 65535 字节的字符串。

除非另有说明，所有 `UTF-8` 编码字符串的长度可以在 0 到 65535 字节范围内。

**图 1.1 UTF-8 编码字符串结构**

| 位 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| byte 1 | 字符串长度 MSB ||||||||
| byte 2 | 字符串长度 LSB ||||||||
| byte 3 ... | UTF-8 编码字符数据（如果长度 > 0）||||||||

`UTF-8` 编码字符串中的字符数据 **必须** 是 `Unicode` 规范 [Unicode] 定义的有效 `UTF-8`，并在 RFC 3629 [RFC3629] 中重申。特别是，此数据 **禁止** 包含 U+D800 和 U+DFFF 之间的码点编码。如果服务器或客户端接收到包含无效 `UTF-8` 的控制报文，它 **必须** 关闭网络连接 [MQTT-1.5.3-1]。

`UTF-8` 编码字符串 **禁止** 包含空字符 U+0000 的编码。如果接收者（服务器或客户端）接收到包含 U+0000 的控制报文，它 **必须** 关闭网络连接 [MQTT-1.5.3-2]。

数据 **不应该** 包含以下 `Unicode` [Unicode] 码点的编码。如果接收者（服务器或客户端）接收到包含任何这些码点的控制报文，它 **可以** 关闭网络连接：

- U+0001..U+001F 控制字符
- U+007F..U+009F 控制字符
- `Unicode` 规范 [Unicode] 中定义为非字符的码点（例如 U+0FFFF）

`UTF-8` 编码序列 0xEF 0xBB 0xBF 总是被解释为 U+FEFF（"零宽度非断空格"），无论它出现在字符串的何处，数据包接收者 **禁止** 跳过或删除它 [MQTT-1.5.3-3]。

#### 1.5.3.1 非规范性示例

例如，字符串 A𪛔（拉丁大写字母 A 后跟码点 U+2A6D4，表示一个 CJK 扩展 B 字符）编码如下：

**图 1.2 UTF-8 编码字符串非规范性示例**

| 位 | 7 | 6 | 5 | 4 | 3 | 2 | 1 | 0 |
|---|---|---|---|---|---|---|---|---|
| byte 1 | 字符串长度 MSB (0x00) |
| | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| byte 2 | 字符串长度 LSB (0x05) |
| | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 1 |
| byte 3 | 'A' (0x41) |
| | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 1 |
| byte 4 | (0xF0) |
| | 1 | 1 | 1 | 1 | 0 | 0 | 0 | 0 |
| byte 5 | (0xAA) |
| | 1 | 0 | 1 | 0 | 1 | 0 | 1 | 0 |
| byte 6 | (0x9B) |
| | 1 | 0 | 0 | 1 | 1 | 0 | 1 | 1 |
| byte 7 | (0x94) |
| | 1 | 0 | 0 | 1 | 0 | 1 | 0 | 0 |

## 1.6 编辑约定

本规范中以<span style="background:yellow">黄色</span>高亮显示的文本标识一致性声明。每个一致性声明都被分配了一个格式为 [MQTT-x.x.x-y] 的引用。