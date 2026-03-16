# MQTT 协议中文翻译项目

本项目旨在对 MQTT 协议的两个主要版本（v3.1.1、v5.0）的 OASIS Standard (OS) 正式版本进行中文翻译。

**为什么不翻译 MQTT 3.1 协议？：**
由于早期物联网的历史遗留、单片机设备更新固件的高昂成本（如早期的 ESP8266 模块可能长达十年不通过 OTA 升级）以及各主流云服务的极佳向下兼容性，**MQTT 3.1.1** 作为一套极简成熟的协议，足以支持当前绝大多数（不需要 MQTT 5.0 的部分高级特性如属性、原因码等即可运转的）IoT 设备需求。
**注：MQTT 3.1.1 是对早期 MQTT 3.1 的标准化升级，目前市面上绝大多数不支持 MQTT 5.0 的设备，运行的均是本协议（3.1.1）。因为差异极小，开发者了解两者的区别看博客即可，不必要去翻阅最古老的 3.1 文档。**

## 目录结构

项目按协议版本划分为两个主要文件夹，每个文件夹下包含 `os/` 目录，并将协议原文的章节拆分为单独的 Markdown 文件：

- `v3.1.1/os/` - MQTT 3.1.1 版本的翻译
- `v5.0/os/` - MQTT 5.0 版本的翻译

每个 `os` 文件夹下的章节结构如下：
1. `01-Introduction.md`
2. `02-MQTT-Control-Packet-format.md`
3. `03-MQTT-Control-Packets.md`
4. `04-Operational-behavior.md`
5. `05-Security.md`
6. `06-Using-WebSocket-as-a-network-transport.md`
7. `07-Conformance.md`

## 关于 OASIS 官方目录说明

如 OASIS 官方文档仓库（例如 `Index of /mqtt/mqtt/v5.0/`）所示，不同目录代表标准制定过程的不同推进阶段：
- **csprdXX**：Committee Specification Public Review Draft（委员会规范公开审查草案）
- **csXX**：Committee Specification（委员会规范）
- **cosXX**：Committee Specification OASIS Standard（成为 OASIS 标准的委员会规范）
- **os**：OASIS Standard（最终正式出台的 OASIS 标准）

## 📖 扩展阅读：MQTT 版本的历史冷知识

很多接触 MQTT 的开发者会好奇：为什么第一个进入大众视野的协议版本是 3.1？V1 和 V2 去了哪里？V4 为什么不存在？

*   **V1 / V2 的闭源时代**：MQTT 诞生于 1999 年，主要用于极其昂贵、低带宽的卫星网络来监控沙漠里的石油管道。一直到 2010 年长达十多年的时间里，MQTT 早期版本（V1、V2 及部分 V3.0）完全是 IBM 内部的私有商业协议，并未对外公开。
*   **V3.1 与 V3.1.1 的开源**：随着物联网（IoT）在 2010 年代爆发，IBM 决定免费向公众开放 MQTT，而当时内部正好迭代到的版本便是 **V3.1**。随后，IBM 将协议移交给 OASIS，这便有了后来标准化、被大规模使用的 **V3.1.1**。
*   **不存在的 V4.0**：客户端在连接时发送的 `CONNECT` 报文中，会用一个字节来表示“协议级别”（Protocol Level）。对于 MQTT 3.1 这个值是 `3`，由于 3.1.1 是其修改版，该值就变成了 `4`。如果 OASIS 将下一个大版本称作 V4.0，那么 `CONNECT` 报文的级别标识只能填 `5`。为了消除“规范叫 4，底层填 5”的割裂感，OASIS 决定让规范版本号直接跟底层协议级别对齐，于是便直接定名为 **MQTT 5.0**。

本项目主要围绕最终的 `os` （正式标准）版本进行日常翻译与对齐工作。
