# exFAT 文件系统规范（本地镜像 · 中文索引）

> **本地镜像**: `portfs/doc/std_exfat/std-exfat-*_cn.md`；权威以官方 Learn **英文**页为准
> **Fetched**: 2026-07-25; ms.date 2025-07-08
> **译文说明**: 字段名保留英文；术语对齐 linux-7.1.2 `fs/exfat/` 与 `pfs_exfat_*`

## 拷贝来源

| 项 | 地址 |
|----|------|
| **拷贝来源** | https://docs.microsoft.com/en-us/windows/win32/fileio/exfat-specification |
| **现行 Learn**（同上，docs 重定向） | https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification |
| **英文索引** | [std-exfat-specification.md](std-exfat-specification.md) |

## 中文章节

- [std-exfat-01-introduction_cn.md](std-exfat-01-introduction_cn.md) -- 第 1 章 引言
- [std-exfat-02-volume-structure_cn.md](std-exfat-02-volume-structure_cn.md) -- 第 2 章 卷结构
- [std-exfat-03-boot-regions_cn.md](std-exfat-03-boot-regions_cn.md) -- 第 3 章 主/备份引导区
- [std-exfat-04-fat-region_cn.md](std-exfat-04-fat-region_cn.md) -- 第 4 章 FAT 区
- [std-exfat-05-data-region_cn.md](std-exfat-05-data-region_cn.md) -- 第 5 章 数据区
- [std-exfat-06-directory-structure_cn.md](std-exfat-06-directory-structure_cn.md) -- 第 6 章 目录结构
- [std-exfat-07-directory-entry-definitions_cn.md](std-exfat-07-directory-entry-definitions_cn.md) -- 第 7 章 目录项定义
- [std-exfat-08-implementation-notes_cn.md](std-exfat-08-implementation-notes_cn.md) -- 第 8 章 实现说明
- [std-exfat-09-file-system-limits_cn.md](std-exfat-09-file-system-limits_cn.md) -- 第 9 章 文件系统限制
- [std-exfat-10-appendix_cn.md](std-exfat-10-appendix_cn.md) -- 第 10 章 附录
- [std-exfat-11-change-history_cn.md](std-exfat-11-change-history_cn.md) -- 第 11 章 文档变更历史

## 命名

| Pattern | Meaning |
|---------|---------|
| `std-exfat-NN-*.md` | Microsoft exFAT 规范第 NN 章（英文） |
| `std-exfat-NN-*_cn.md` | 对应中文译本 |
| `std-exfat-specification.md` | 英文索引 |
| `std-exfat-specification_cn.md` | 本中文索引 |

## Notes

- 冲突时以官方 Learn 英文原文为准；中文仅供离线阅读。
- §7.2 推荐 Up-case Table 的十六进制数据与英文保持一致（未改写）。
- 内核对照：`../linux-7.1.2/fs/exfat/`；实现：`portfs/src/exfat/`。

## Related

- Skill：**pfs-fs-std-exfat**
- 实现：`portfs/src/exfat/`
