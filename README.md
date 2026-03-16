# MQTT 协议中文翻译项目

本项目旨在对 MQTT 协议的三个主要版本（v3.1、v3.1.1、v5.0）的 OASIS Standard (OS) 正式版本进行中文翻译。

## 目录结构

项目按协议版本划分为三个主要文件夹，每个文件夹下包含 `os/` 目录，并将协议原文的章节拆分为单独的 Markdown 文件：

- `v3.1/os/` - MQTT 3.1 版本的翻译
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

本项目主要围绕最终的 `os` （正式标准）版本进行日常翻译与对齐工作。
