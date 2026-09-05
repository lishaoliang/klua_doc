# exFAT File System Specification (local mirror)

> **Local mirror**: chapter split under [portfs/doc/std_exfat/std-exfat-*.md](https://gitee.com/klua/portfs/blob/trunk/doc/std_exfat/std-exfat-*.md) for pfs offline reference; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 拷贝来源

| 项 | 地址 |
|----|------|
| **拷贝来源** | https://docs.microsoft.com/en-us/windows/win32/fileio/exfat-specification |
| **现行 Learn**（同上，docs 重定向） | https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification |

## Chapters

- [std-exfat-01-introduction.md](std-exfat-01-introduction.md) / [中文](std-exfat-01-introduction_cn.md) -- section 1 Introduction
- [std-exfat-02-volume-structure.md](std-exfat-02-volume-structure.md) / [中文](std-exfat-02-volume-structure_cn.md) -- section 2 Volume Structure
- [std-exfat-03-boot-regions.md](std-exfat-03-boot-regions.md) / [中文](std-exfat-03-boot-regions_cn.md) -- section 3 Main and Backup Boot Regions
- [std-exfat-04-fat-region.md](std-exfat-04-fat-region.md) / [中文](std-exfat-04-fat-region_cn.md) -- section 4 File Allocation Table Region
- [std-exfat-05-data-region.md](std-exfat-05-data-region.md) / [中文](std-exfat-05-data-region_cn.md) -- section 5 Data Region
- [std-exfat-06-directory-structure.md](std-exfat-06-directory-structure.md) / [中文](std-exfat-06-directory-structure_cn.md) -- section 6 Directory Structure
- [std-exfat-07-directory-entry-definitions.md](std-exfat-07-directory-entry-definitions.md) / [中文](std-exfat-07-directory-entry-definitions_cn.md) -- section 7 Directory Entry Definitions
- [std-exfat-08-implementation-notes.md](std-exfat-08-implementation-notes.md) / [中文](std-exfat-08-implementation-notes_cn.md) -- section 8 Implementation Notes
- [std-exfat-09-file-system-limits.md](std-exfat-09-file-system-limits.md) / [中文](std-exfat-09-file-system-limits_cn.md) -- section 9 File System Limits
- [std-exfat-10-appendix.md](std-exfat-10-appendix.md) / [中文](std-exfat-10-appendix_cn.md) -- section 10 Appendix
- [std-exfat-11-change-history.md](std-exfat-11-change-history.md) / [中文](std-exfat-11-change-history_cn.md) -- section 11 Documentation Change History

## Naming

| Pattern | Meaning |
|---------|---------|
| `std-exfat-NN-*.md` | Microsoft exFAT specification chapter NN (English) |
| `std-exfat-NN-*_cn.md` | Chinese translation of chapter NN |
| `std-exfat-specification.md` | This English index |
| `std-exfat-specification_cn.md` | Chinese index |

## Related

- Chinese index: [std-exfat-specification_cn.md](std-exfat-specification_cn.md)
- Implementation: [portfs/src/exfat/](https://gitee.com/klua/portfs/tree/trunk/src/exfat/)
- Kernel reference: `../linux-7.1.2/fs/exfat/`
