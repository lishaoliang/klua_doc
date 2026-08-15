# NTFS File System Specification (local mirror)

> **Local mirror**: chapter split under `portfs/doc/std_ntfs/std-ntfs-*.md` for pfs offline reference; official Learn pages are authoritative.
> **Fetched**: 2026-07-25

## 拷贝来源

| 项 | 地址 |
|----|------|
| **拷贝来源（主文）** | https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10) |
| **文档全称** | How NTFS Works: Local File Systems (Windows Server 2003 TechNet archive) |
| **docs 旧域（同上重定向/归档）** | https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10) |
| **拷贝来源（Boot Sector）** | https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc976796(v=technet.10) |
| **拷贝来源（MFT Developer Notes）** | https://learn.microsoft.com/en-us/windows/win32/devnotes/master-file-table |
| **docs 旧域（MFT）** | https://docs.microsoft.com/en-us/windows/win32/devnotes/master-file-table |
| **社区 on-disk 详表（非微软官方全文）** | https://flatcap.github.io/linux-ntfs/ntfs/ |
| **本仓 on-disk 对齐** | `portfs/src/ntfs/pfs_ntfs_raw.h` ← linux-7.1.2 `fs/ntfs/layout.h` |

## Important

- Microsoft **没有**发布类似 [exFAT specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) / fatgen103 的 **单一公开完整 NTFS on-disk 规范**。
- 本目录镜像的是 **公开 Learn / TechNet 说明文档** + Developer Notes；改布局时仍以 **锁定 raw** 与 **内核 layout.h** 为可核对真源（见 **pfs-fs-std-ntfs**）。

## Chapters

- [std-ntfs-01-introduction.md](std-ntfs-01-introduction.md) / [中文](std-ntfs-01-introduction_cn.md) -- Introduction
- [std-ntfs-02-architecture.md](std-ntfs-02-architecture.md) / [中文](std-ntfs-02-architecture_cn.md) -- NTFS Architecture
- [std-ntfs-03-physical-structure.md](std-ntfs-03-physical-structure.md) / [中文](std-ntfs-03-physical-structure_cn.md) -- NTFS Physical Structure
- [std-ntfs-04-processes-and-interactions.md](std-ntfs-04-processes-and-interactions.md) / [中文](std-ntfs-04-processes-and-interactions_cn.md) -- NTFS Processes and Interactions
- [std-ntfs-05-related-information.md](std-ntfs-05-related-information.md) / [中文](std-ntfs-05-related-information_cn.md) -- Related Information
- [std-ntfs-31-clusters-and-sectors.md](std-ntfs-31-clusters-and-sectors.md) / [中文](std-ntfs-31-clusters-and-sectors_cn.md) -- Clusters and Sectors
- [std-ntfs-32-volume-organization.md](std-ntfs-32-volume-organization.md) / [中文](std-ntfs-32-volume-organization_cn.md) -- Volume Organization
- [std-ntfs-33-boot-sectors.md](std-ntfs-33-boot-sectors.md) / [中文](std-ntfs-33-boot-sectors_cn.md) -- Boot Sectors
- [std-ntfs-34-master-file-table.md](std-ntfs-34-master-file-table.md) / [中文](std-ntfs-34-master-file-table_cn.md) -- Master File Table
- [std-ntfs-35-file-record-attributes.md](std-ntfs-35-file-record-attributes.md) / [中文](std-ntfs-35-file-record-attributes_cn.md) -- File Record Attributes
- [std-ntfs-40-boot-sector-technet.md](std-ntfs-40-boot-sector-technet.md) / [中文](std-ntfs-40-boot-sector-technet_cn.md) -- Boot Sector (TechNet)
- [std-ntfs-41-mft-developer-notes.md](std-ntfs-41-mft-developer-notes.md) / [中文](std-ntfs-41-mft-developer-notes_cn.md) -- MFT Developer Notes

## Naming

| Pattern | Meaning |
|---------|---------|
| `std-ntfs-01` … `std-ntfs-05` | How NTFS Works 主章节 (English) |
| `std-ntfs-31` … `std-ntfs-35` | Physical Structure 子节 |
| `std-ntfs-40-*` / `std-ntfs-41-*` | 其它 Microsoft 公开页镜像 |
| `std-ntfs-*_cn.md` | Chinese translation |
| `std-ntfs-specification.md` | This English index |
| `std-ntfs-specification_cn.md` | Chinese index |

## Notes

- 勿与 FAT / exFAT 规范混淆（见 `../std_fat/`、`../std_exfat/`）。
- `fs/ntfs3` 驱动头 **不作** on-disk raw 真源。

## Related

- Chinese index: [std-ntfs-specification_cn.md](std-ntfs-specification_cn.md)
- Skill index (key points, not full text): **pfs-fs-std-ntfs**
- Implementation: `portfs/src/ntfs/`
- Kernel: `../linux-7.1.2/fs/ntfs/`
- FAT mirror: `../std_fat/std-fat-specification.md`
- exFAT mirror: `../std_exfat/std-exfat-specification.md`
