> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

## NTFS 物理结构

### NTFS 卷上的簇与扇区 (Clusters and Sectors)

簇 (cluster, 亦称 allocation unit / 分配单元) 是可用于存放文件的最小磁盘空间单位. Windows Server 2003 使用的所有文件系统均按簇大小组织硬盘, 簇大小由簇所含扇区 (sector, 硬盘上的存储单位) 数量决定. 例如, 在使用 512 字节扇区的磁盘上, 512 字节簇含 1 个扇区, 而 4 KB 簇含 8 个扇区.

计算机在启动时会访问硬盘上的特定扇区, 以确定启动哪个操作系统以及分区位于何处. 这些扇区上存储的数据因计算机平台而异.

#### NTFS 卷上的簇序列

NTFS 卷上的簇从分区起始处顺序编号, 形成逻辑簇号 (Logical Cluster Number, LCN). NTFS 使用称为主文件表 (Master File Table, MFT) 的记录存储文件系统中的所有对象, 其结构类似数据库.

在 NTFS 卷上, 簇从扇区 0 开始; 因此每个簇都对齐于簇边界. 用于文件存储的连续簇可加快文件处理速度.

**注意**

- 软盘不使用 NTFS, 始终格式化为 FAT.

#### NTFS 卷上簇大小的限制

由于 NTFS 依卷大小使用不同簇大小, 每种文件系统可支持的簇数有上限. 簇越小, 磁盘存储信息可能越高效, 因为簇内未用空间无法被其他文件使用. 文件系统支持的簇越多, 用该文件系统可创建与格式化的卷就越大. NTFS 使用较小簇大小, 因此是更高效的文件组织结构.

表 Default NTFS Cluster Sizes 列出 NTFS 卷与默认簇大小.

**Default NTFS Cluster Sizes (默认 NTFS 簇大小)**

| 卷大小 | NTFS 簇大小 |
| --- | --- |
| 7 MB–512 MB | 512 字节 |
| 513 MB–1,024 MB | 1 KB |
| 1,025 MB–2 GB | 2 KB |
| 2 GB–2 TB | 4 KB |

#### NTFS 卷上的最大尺寸

在格式化 NTFS 卷之前, 应评估将存储的文件类型, 以决定是否使用默认簇大小.

格式化 NTFS 卷时, 可使用 Disk Management 管理单元指定最大 64 KB 的簇大小. 若格式化卷时未指定簇大小, 则使用默认值. 若要在格式化后更改簇大小, 必须重新格式化卷.

在选择非默认簇大小之前, 请注意以下重要限制:

- 对 Microsoft Windows NT、Windows 2000、Windows XP 与 Windows Server 2003, 2 GB 至 4 GB 的 FAT16 卷簇大小为 64 KB, 可能与某些应用程序产生兼容性问题. 例如, 安装程序在 64 KB 簇的卷上无法正确计算可用空间, 并可能因误以为可用空间不足而无法运行. 因此, 大于 2 GB 的卷可使用 NTFS 或 FAT32 格式化.
- 因为文件压缩不支持大于 4 KB 的簇大小, Windows Server 2003 的默认 NTFS 簇大小不超过 4 KB.

理论上, NTFS 卷最大大小为 2^64 簇减 1 簇. 然而, Windows Server 2003 实现的 NTFS 卷最大大小为 2^32 簇减 1 簇. 例如, 使用 64 KB 簇时, NTFS 卷最大为 256 TB 减 64 KB. 使用默认 4 KB 簇时, NTFS 卷最大为 16 TB 减 4 KB.

**注意**

- 若在 NTFS 文件夹中使用大量文件 (30 万或更多), 应禁用短文件名生成以获得更好性能, 尤其当长文件名的前 6 个字符相似时.

表 NTFS Size Limits 列出 NTFS 尺寸限制.

**NTFS Size Limits (NTFS 尺寸限制)**

| 说明 | 限制 |
| --- | --- |
| 最大文件大小 | 架构上: 16 EB 减 1 KB (2^64 字节减 1 KB) 实现上: 16 TB 减 64 KB (2^44 字节减 64 KB) |
| 最大卷大小 | 架构上: 2^64 簇减 1 簇 实现上: 256 TB 减 64 KB (2^32 簇减 1 簇) |
| 每卷文件数 | 4,294,967,295 (2^32 减 1 个文件) |

#### MBR 与 GUID 磁盘上的分区表

主引导记录 (MBR) 磁盘同时使用基本卷与动态卷. 因为 MBR 磁盘分区表仅支持最大 2 TB 的分区, 要创建超过 2 TB 的 NTFS 卷必须使用动态卷. Windows Server 2003 在特殊数据库中管理动态卷, 而非分区表; 因此动态卷不受分区表 2 TB 物理限制. 动态 NTFS 卷可大到 NTFS 支持的最大卷大小. 使用 GUID 分区表 (GPT) 磁盘的 Itanium 计算机也支持大于 2 TB 的 NTFS 卷.
