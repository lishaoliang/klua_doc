> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: `portfs/doc/std_exfat/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/类型名保留英文；术语尽量对齐 linux-7.1.2 `fs/exfat/` 与 `pfs_exfat_*`
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 1 引言 (Introduction)

exFAT 文件系统是 FAT 家族中 FAT32 的后继。本规范描述 exFAT，并给出实现 exFAT 所需的全部信息。

### 1.1 设计目标 (Design Goals)

exFAT 有三项核心设计目标（见下表）。

1. *保持基于 FAT 的文件系统的简洁性。*

>
> FAT 系文件系统的两项优势是相对简单、易于实现。沿袭前代，实现者应感到 exFAT 同样相对简单、易于实现。
2. *支持很大的文件与存储设备。*

>
> exFAT 用 64 位描述文件大小，从而支持依赖超大文件的应用。簇大小最大可达约 32MB，从而有效支持很大的存储设备。
3. *为未来创新保留可扩展性。*

>
> exFAT 在设计中纳入可扩展性，使文件系统能跟上存储创新与用法变化。

### 1.2 专用术语 (Specific Terminology)

在本规范语境下，若干术语（见表 1）对设计与实现具有特定含义。

**表 1 具有特定含义的术语定义**

| **术语** | **定义** |
| --- | --- |
| Shall（应当） | 本规范用 “shall” 描述**强制**行为。 |
| Should（宜） | 本规范用 “should” 描述强烈建议、但非强制的行为。 |
| May（可以） | 本规范用 “may” 描述可选行为。 |
| Mandatory（强制） | 描述实现方 **shall** 按本规范修改并解释的字段或结构。 |
| Optional（可选） | 描述实现方可支持或不支持的字段或结构。若支持，则 **shall** 按本规范修改并解释。 |
| Undefined（未定义） | 描述实现方可按需修改（例如在设置周边字段时清零）、且 **shall not** 解释为任何特定含义的字段或结构内容。 |
| Reserved（保留） | 描述实现方须遵守的字段或结构内容：1. **Shall** 初始化为 0，且不宜用于任何目的 2. 除计算校验和外，不宜解释 3. 在修改周边字段或结构的操作中 **shall** 予以保留 |

### 1.3 常用缩写全称 (Full Text of Common Acronyms)

本规范使用个人计算机业界常用缩写（见表 2）。

**表 2 常用缩写全称**

| **缩写** | **全称** |
| --- | --- |
| ASCII | American Standard Code for Information Interchange |
| BIOS | Basic Input Output System |
| CPU | Central Processing Unit |
| exFAT | extensible File Allocation Table（可扩展文件分配表） |
| FAT | File Allocation Table（文件分配表） |
| FAT12 | File Allocation Table, 12-bit cluster indices |
| FAT16 | File Allocation Table, 16-bit cluster indices |
| FAT32 | File Allocation Table, 32-bit cluster indices |
| GPT | GUID Partition Table |
| GUID | Globally Unique Identifier（见第 10.1 节） |
| INT | Interrupt |
| MBR | Master Boot Record |
| texFAT | Transaction-safe exFAT（事务安全 exFAT） |
| UTC | Coordinated Universal Time |

### 1.4 字段与结构的默认限定 (Default Field and Structure Qualifiers)

除非另有说明，本规范中的字段与结构具有下列限定：

1. 为无符号
2. 用十进制描述数值（另有说明除外）；十六进制用后缀字母 “h”；GUID 用花括号括起
3. 小端格式 (little-endian)；与内核/pfs 中 on-disk 布局一致
4. 字符串不要求以空字符结尾

### 1.5 Windows CE 与 TexFAT

TexFAT 是 exFAT 的扩展，在基础文件系统之上增加事务安全操作语义。Windows CE 使用 TexFAT。TexFAT 要求使用两份 FAT 与分配位图 (Allocation Bitmap) 以支持事务。它还定义若干附加结构，包括填充描述符 (padding descriptors) 与安全描述符 (security descriptors)。
