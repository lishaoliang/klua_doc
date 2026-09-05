> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## FAT 目录相关其他说明 (Other Notes Relating to FAT Directories)

**LFN 目录项** 在所有 FAT 类型上 **结构相同**. 细节见前文各节.

`DIR_FileSize` 为 32 位字段. 对 FAT32 卷, FAT 驱动 **must not** 创建长度超过 `0x100000000` 字节的簇链, 且该链最后一簇的 **最后一字节** 不得分配给文件 — 从而保证无文件大小 `> 0xFFFFFFFF` 字节. 这是 **所有 FAT 类型** 的根本限制; FAT 卷上允许的最大文件大小为 `0xFFFFFFFF` (4,294,967,295) 字节.

同样, FAT 驱动 **must not** 允许 **目录** (作为其他文件容器的文件) 大于 `65536 * 32` (2,097,152) 字节 — 对齐 `pfs_fat_raw.h` 中 `FAT_MAX_DIR_SIZE` / `FAT_MAX_DIR_ENTRIES`.

**注**: 该限制针对 **目录本身大小**, **与目录内文件个数无关**. 原因有二:

1. FAT 目录 **无排序或索引**; 巨型目录会使创建新项等操作 (须扫描每个已分配目录项以确认名不存在) **极慢**.
2. 许多 FAT 驱动与磁盘工具 (含 Microsoft 自家) 用 **16 位 WORD** 计数目录项数, 故目录项数不得超过 16 位可表示范围 (`65536` 项).

---

*FAT: General Overview of On-Disk Format — Microsoft Corporation. All rights reserved.*
