> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 10 附录 (Appendix)

### 10.1 全局唯一标识符 (Globally Unique Identifiers, GUIDs)

GUID 是 Microsoft 对通用唯一标识符的实现。GUID 为 128 位值，由一组 8 位十六进制数字、三组各 4 位十六进制数字、再加一组 12 位十六进制数字组成，例如 {6B29FC40-CA47-1067-B31D-00DD010662DA}（见表 38）。

**表 38 GUID 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| Data1 | 0 | 4 | 强制；含 GUID 第一组四字节（例：6B29FC40h）。 |
| Data2 | 4 | 2 | 强制；含第二组两字节（例：CA47h）。 |
| Data3 | 6 | 2 | 强制；含第三组两字节（例：1067h）。 |
| Data4[0] | 8 | 1 | 强制；含第四组最高有效字节（例：B3h）。 |
| Data4[1] | 9 | 1 | 强制；含第四组最低有效字节（例：1Dh）。 |
| Data4[2] | 10 | 1 | 强制；含第五组第一字节（例：00h）。 |
| Data4[3] | 11 | 1 | 强制；含第五组第二字节（例：DDh）。 |
| Data4[4] | 12 | 1 | 强制；含第五组第三字节（例：01h）。 |
| Data4[5] | 13 | 1 | 强制；含第五组第四字节（例：06h）。 |
| Data4[6] | 14 | 1 | 强制；含第五组第五字节（例：62h）。 |
| Data4[7] | 15 | 1 | 强制；含第五组第六字节（例：DAh）。 |

### 10.2 分区表 (Partition Tables)

为在广泛场景下保证 exFAT 卷互操作，实现方宜对 MBR 分区存储使用分区类型 07h，对 GPT 分区存储使用分区 GUID {EBD0A0A2-B9E5-4433-87C0-68B6B72699C7}。
