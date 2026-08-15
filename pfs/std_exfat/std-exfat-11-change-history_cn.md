> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: `portfs/doc/std_exfat/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；变更表为摘要翻译，细节以英文原文为准
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 11 文档变更历史 (Documentation Change History)

表 39 描述本文件各次发布、更正、增补、删除与澄清。

**表 39 文档变更历史**

| **日期** | **变更说明** |
| --- | --- |
| 08-Jan-2008 | Basic Specification 首次发布，含第 1–10 节（引言、卷结构、主/备份引导区、FAT 区、数据区、目录结构、目录项定义、实现说明、限制、附录）。 |
| 08-Jun-2008 | 第二次发布：新增第 11 节；新增 Vendor Extension / Vendor Allocation（7.8/7.9）；新增推荐 Up-case Table（7.2.5）；新增 UtcOffset（7.4）与 UTC 缩写（1.3）；修正 CustomDefined 大小、NameLength 范围、Timestamp/10msIncrement；澄清 Null Parameters、NoFatChain、DataLength、VolumeDirty/写序、MediaFailure 等。 |
| 01-Oct-2008 | 第三次发布：字段说明增加 SHALL/SHOULD/MAY；表 2 增加 UTC；修改 1.5 以对齐 TexFAT；澄清仅 Microsoft 可定义 Directory Entry 布局（6.2）；澄清 FirstCluster/DataLength/NoFatChain（6.3.5/6.4.3）；澄清有效 File 目录项（7.4）；增加文件/目录名唯一性要求（7.7）；增加 ASCII 实现说明（7.7.3）。 |
| 01-Jan-2009 | 第四次发布：移除 Windows CE Access Control 相关引用；澄清 7.2.5.1 须提供完整 Up-case Table。 |
| 02-Sep-2009 | 第五次发布：文档格式调整以便生成 PDF。 |
| 24-Feb-2010 | 第六次发布：修正 NoFatChain/FirstCluster/DataLength 相关错误表述（6.3.5/6.4.3/6.3.6/6.4.4）；更新版权至 2010。 |
| 26-Aug-2019 | 第七次发布：更新法律条款（移除 Confidential 与 Technical Documentation License Agreement 等）；更新版权至 2019。 |
