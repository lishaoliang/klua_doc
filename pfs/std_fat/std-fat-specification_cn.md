# FAT 文件系统规范（本地镜像 · 中文索引）

> **本地镜像**: [portfs/doc/std_fat/std-fat-*_cn.md](https://gitee.com/klua/portfs/blob/trunk/doc/std_fat/std-fat-*_cn.md)；权威以官方 fatgen103.doc / ECMA-107 为准
> **Fetched**: 2026-07-25
> **译文说明**: 字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`

## 拷贝来源

| 项 | 地址 |
|----|------|
| **拷贝来源（FAT32 / fatgen103）** | https://download.microsoft.com/download/1/6/1/161ba512-40e2-4cc9-843a-923143f3456c/fatgen103.doc |
| **文档全称** | Microsoft Extensible Firmware Initiative FAT32 File System Specification — FAT: General Overview of On-Disk Format, Version 1.03, December 6, 2000 |
| **拷贝来源（FAT12/16 交换）** | https://www.ecma-international.org/publications-and-standards/standards/ecma-107/ |
| **ECMA-107 PDF** | https://ecma-international.org/wp-content/uploads/ECMA-107_2nd_edition_june_1995.pdf |
| **英文索引** | [std-fat-specification.md](std-fat-specification.md) |

## 中文章节（Microsoft fatgen103）

- [std-fat-00-front-matter_cn.md](std-fat-00-front-matter_cn.md) -- 前言 / 法律 / 修订历史
- [std-fat-01-notational-conventions_cn.md](std-fat-01-notational-conventions_cn.md) -- 记号约定
- [std-fat-02-general-comments_cn.md](std-fat-02-general-comments_cn.md) -- 通用说明
- [std-fat-03-boot-sector-and-bpb_cn.md](std-fat-03-boot-sector-and-bpb_cn.md) -- 引导扇区与 BPB
- [std-fat-04-fat-data-structure_cn.md](std-fat-04-fat-data-structure_cn.md) -- FAT 数据结构
- [std-fat-05-fat-type-determination_cn.md](std-fat-05-fat-type-determination_cn.md) -- FAT 类型判定
- [std-fat-06-fat-volume-initialization_cn.md](std-fat-06-fat-volume-initialization_cn.md) -- 卷初始化
- [std-fat-07-fat32-fsinfo-and-backup-boot_cn.md](std-fat-07-fat32-fsinfo-and-backup-boot_cn.md) -- FAT32 FSInfo 与备份引导扇区
- [std-fat-08-fat-directory-structure_cn.md](std-fat-08-fat-directory-structure_cn.md) -- 目录结构
- [std-fat-09-fat-long-directory-entries_cn.md](std-fat-09-fat-long-directory-entries_cn.md) -- 长文件名目录项
- [std-fat-10-name-limits-and-character-sets_cn.md](std-fat-10-name-limits-and-character-sets_cn.md) -- 名称限制与字符集
- [std-fat-11-name-matching_cn.md](std-fat-11-name-matching_cn.md) -- 短名与长名匹配
- [std-fat-12-naming-conventions_cn.md](std-fat-12-naming-conventions_cn.md) -- 命名约定与长名
- [std-fat-13-long-entries-down-level_cn.md](std-fat-13-long-entries-down-level_cn.md) -- 长名项对旧版 FAT 的影响
- [std-fat-14-validating-directory_cn.md](std-fat-14-validating-directory_cn.md) -- 校验目录内容
- [std-fat-15-other-notes-directories_cn.md](std-fat-15-other-notes-directories_cn.md) -- 目录相关其它说明

## 命名

| Pattern | Meaning |
|---------|---------|
| `std-fat-NN-*.md` | fatgen103 第 NN 节（英文） |
| `std-fat-NN-*_cn.md` | 对应中文译本 |
| `std-fat-specification.md` | 英文索引 |
| `std-fat-specification_cn.md` | 本中文索引 |

## Notes

- **pfs 主路径**仍是 **FAT32**；FAT12/16 交换见 ECMA-107（本目录暂未拆章镜像）。
- 冲突以官方 `.doc` 为准；本地章文由 OLE 抽取，中文译本已适度清理噪声。
- 勿与 exFAT 混淆（见 `../std_exfat/`）。

## Related

- 实现：[portfs/src/fat/](https://gitee.com/klua/portfs/tree/trunk/src/fat/)
- 内核：`../linux-7.1.2/fs/fat/`
