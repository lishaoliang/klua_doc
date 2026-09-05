> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## 本文档记法约定 (Notational Conventions in this Document)

以 `0x` 开头的数字为十六进制 (base 16).

不以 `0x` 开头的数字为十进制 (base 10).

本文档中的代码片段采用 **C** 语言风格. 为便于阅读, 未严格遵循类型与语法.

若干代码片段混用 32 位与 16 位数据. 假定读者能正确标注类型, 避免将 32 位值截断为 16 位而丢失数据. 另请注意: **所有数据类型均为无符号 (UNSIGNED)**. 勿用有符号整型做 FAT 运算, 否则在部分 FAT 卷上结果会错误.
