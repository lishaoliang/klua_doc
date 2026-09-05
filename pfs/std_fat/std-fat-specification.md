# FAT File System Specification (local mirror)

> **Local mirror**: chapter split under [portfs/doc/std_fat/std-fat-*.md](https://gitee.com/klua/portfs/blob/trunk/doc/std_fat/std-fat-*.md) for pfs offline reference; official sources are authoritative.
> **Fetched**: 2026-07-25

## 拷贝来源

| 项 | 地址 |
|----|------|
| **拷贝来源（FAT32 / fatgen103）** | https://download.microsoft.com/download/1/6/1/161ba512-40e2-4cc9-843a-923143f3456c/fatgen103.doc |
| **文档全称** | Microsoft Extensible Firmware Initiative FAT32 File System Specification — FAT: General Overview of On-Disk Format, Version 1.03, December 6, 2000 |
| **历史介绍页（参考）** | https://www.microsoft.com/whdc/system/platform/firmware/fatgen.mspx （旧 WHDC；可能失效） |
| **拷贝来源（FAT12/16 交换）** | https://www.ecma-international.org/publications-and-standards/standards/ecma-107/ |
| **ECMA-107 PDF** | https://ecma-international.org/wp-content/uploads/ECMA-107_2nd_edition_june_1995.pdf |
| **等同国际标准** | ISO/IEC 9293（与 ECMA-107 等同；**非** FAT32 全书、**非** exFAT） |

## Chapters（Microsoft fatgen103）

- [std-fat-00-front-matter.md](std-fat-00-front-matter.md) / [中文](std-fat-00-front-matter_cn.md) -- front matter / legal / history
- [std-fat-01-notational-conventions.md](std-fat-01-notational-conventions.md) / [中文](std-fat-01-notational-conventions_cn.md) -- Notational Conventions
- [std-fat-02-general-comments.md](std-fat-02-general-comments.md) / [中文](std-fat-02-general-comments_cn.md) -- General Comments
- [std-fat-03-boot-sector-and-bpb.md](std-fat-03-boot-sector-and-bpb.md) / [中文](std-fat-03-boot-sector-and-bpb_cn.md) -- Boot Sector and BPB
- [std-fat-04-fat-data-structure.md](std-fat-04-fat-data-structure.md) / [中文](std-fat-04-fat-data-structure_cn.md) -- FAT Data Structure
- [std-fat-05-fat-type-determination.md](std-fat-05-fat-type-determination.md) / [中文](std-fat-05-fat-type-determination_cn.md) -- FAT Type Determination
- [std-fat-06-fat-volume-initialization.md](std-fat-06-fat-volume-initialization.md) / [中文](std-fat-06-fat-volume-initialization_cn.md) -- FAT Volume Initialization
- [std-fat-07-fat32-fsinfo-and-backup-boot.md](std-fat-07-fat32-fsinfo-and-backup-boot.md) / [中文](std-fat-07-fat32-fsinfo-and-backup-boot_cn.md) -- FAT32 FSInfo and Backup Boot
- [std-fat-08-fat-directory-structure.md](std-fat-08-fat-directory-structure.md) / [中文](std-fat-08-fat-directory-structure_cn.md) -- FAT Directory Structure
- [std-fat-09-fat-long-directory-entries.md](std-fat-09-fat-long-directory-entries.md) / [中文](std-fat-09-fat-long-directory-entries_cn.md) -- FAT Long Directory Entries
- [std-fat-10-name-limits-and-character-sets.md](std-fat-10-name-limits-and-character-sets.md) / [中文](std-fat-10-name-limits-and-character-sets_cn.md) -- Name Limits and Character Sets
- [std-fat-11-name-matching.md](std-fat-11-name-matching.md) / [中文](std-fat-11-name-matching_cn.md) -- Name Matching
- [std-fat-12-naming-conventions.md](std-fat-12-naming-conventions.md) / [中文](std-fat-12-naming-conventions_cn.md) -- Naming Conventions
- [std-fat-13-long-entries-down-level.md](std-fat-13-long-entries-down-level.md) / [中文](std-fat-13-long-entries-down-level_cn.md) -- Long Entries Down Level
- [std-fat-14-validating-directory.md](std-fat-14-validating-directory.md) / [中文](std-fat-14-validating-directory_cn.md) -- Validating Directory
- [std-fat-15-other-notes-directories.md](std-fat-15-other-notes-directories.md) / [中文](std-fat-15-other-notes-directories_cn.md) -- Other Notes

## Naming

| Pattern | Meaning |
|---------|---------|
| `std-fat-NN-*.md` | Microsoft fatgen103 chapter / section NN (English) |
| `std-fat-NN-*_cn.md` | Chinese translation |
| `std-fat-specification.md` | This English index |
| `std-fat-specification_cn.md` | Chinese index |
| `std-fat-00-front-matter.md` | Title, legal agreement, document history |

## Notes

- **pfs 主路径**仍是 **FAT32**（fatgen103）；FAT12/16 交换细节另见 ECMA-107 / ISO 9293（本目录暂未拆章镜像）。
- 本地章文由 `fatgen103.doc` OLE 抽取转 Markdown，排版可能有噪声；冲突以官方 `.doc` 为准。
- 勿与 exFAT 规范混淆（见 `../std_exfat/`）。

## Related

- Chinese index: [std-fat-specification_cn.md](std-fat-specification_cn.md)
- Implementation: [portfs/src/fat/](https://gitee.com/klua/portfs/tree/trunk/src/fat/)
- Kernel: `../linux-7.1.2/fs/fat/`
- exFAT mirror: `../std_exfat/std-exfat-specification.md`
