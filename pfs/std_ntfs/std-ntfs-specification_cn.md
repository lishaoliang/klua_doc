# NTFS 文件系统规范（本地镜像 · 中文索引）

> **本地镜像**: `portfs/doc/std_ntfs/std-ntfs-*_cn.md`；权威以官方 Learn 英文页为准
> **Fetched**: 2026-07-25
> **译文说明**: 字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`

## 拷贝来源

| 项 | 地址 |
|----|------|
| **拷贝来源（主文）** | https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10) |
| **文档全称** | How NTFS Works: Local File Systems (Windows Server 2003 TechNet archive) |
| **拷贝来源（Boot Sector）** | https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc976796(v=technet.10) |
| **拷贝来源（MFT Developer Notes）** | https://learn.microsoft.com/en-us/windows/win32/devnotes/master-file-table |
| **本仓 on-disk 对齐** | `portfs/src/ntfs/pfs_ntfs_raw.h` ← linux-7.1.2 `fs/ntfs/layout.h` |
| **英文索引** | [std-ntfs-specification.md](std-ntfs-specification.md) |

## Important

- Microsoft **没有**发布类似 exFAT / fatgen103 的 **单一公开完整 NTFS on-disk 规范**。
- 本目录为公开 Learn/TechNet 说明的中文译本；改布局仍以 **锁定 raw** 与 **内核 layout.h** 为准。

## 中文章节

- [std-ntfs-01-introduction_cn.md](std-ntfs-01-introduction_cn.md) -- 引言
- [std-ntfs-02-architecture_cn.md](std-ntfs-02-architecture_cn.md) -- 架构
- [std-ntfs-03-physical-structure_cn.md](std-ntfs-03-physical-structure_cn.md) -- 物理结构
- [std-ntfs-04-processes-and-interactions_cn.md](std-ntfs-04-processes-and-interactions_cn.md) -- 过程与交互
- [std-ntfs-05-related-information_cn.md](std-ntfs-05-related-information_cn.md) -- 相关信息
- [std-ntfs-31-clusters-and-sectors_cn.md](std-ntfs-31-clusters-and-sectors_cn.md) -- 簇与扇区
- [std-ntfs-32-volume-organization_cn.md](std-ntfs-32-volume-organization_cn.md) -- 卷组织
- [std-ntfs-33-boot-sectors_cn.md](std-ntfs-33-boot-sectors_cn.md) -- 引导扇区
- [std-ntfs-34-master-file-table_cn.md](std-ntfs-34-master-file-table_cn.md) -- 主文件表 (MFT)
- [std-ntfs-35-file-record-attributes_cn.md](std-ntfs-35-file-record-attributes_cn.md) -- 文件记录属性
- [std-ntfs-40-boot-sector-technet_cn.md](std-ntfs-40-boot-sector-technet_cn.md) -- Boot Sector（TechNet）
- [std-ntfs-41-mft-developer-notes_cn.md](std-ntfs-41-mft-developer-notes_cn.md) -- MFT Developer Notes

## 命名

| Pattern | Meaning |
|---------|---------|
| `std-ntfs-NN-*.md` | 英文镜像 |
| `std-ntfs-NN-*_cn.md` | 对应中文译本 |
| `std-ntfs-specification.md` | 英文索引 |
| `std-ntfs-specification_cn.md` | 本中文索引 |

## Notes

- 勿与 FAT / exFAT 混淆（见 `../std_fat/`、`../std_exfat/`）。
- `fs/ntfs3` **不作** on-disk raw 真源。

## Related

- 实现：`portfs/src/ntfs/`
- 内核：`../linux-7.1.2/fs/ntfs/`
