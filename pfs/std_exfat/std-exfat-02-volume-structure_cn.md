> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: `portfs/doc/std_exfat/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名保留英文；区域名对齐内核 `fs/exfat`（Main/Backup Boot、FAT、Cluster Heap）
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 2 卷结构 (Volume Structure)

卷 (volume) 是存储与检索用户数据所需的全部文件系统结构与数据空间的集合。所有 exFAT 卷包含四个区域（见表 3）。

**表 3 卷结构**

| **子区域名** | **偏移** **(扇区)** | **大小** **(扇区)** | **说明** |
| --- | --- | --- | --- |
| **Main Boot Region（主引导区）** | | | |
| Main Boot Sector | 0 | 1 | 强制；内容见第 3.1 节。 |
| Main Extended Boot Sectors | 1 | 8 | 强制；内容见第 3.2 节。 |
| Main OEM Parameters | 9 | 1 | 强制；内容见第 3.3 节。 |
| Main Reserved | 10 | 1 | 强制；内容保留 (Reserved)。 |
| Main Boot Checksum | 11 | 1 | 强制；内容见第 3.4 节。 |
| **Backup Boot Region（备份引导区）** | | | |
| Backup Boot Sector | 12 | 1 | 强制；内容见第 3.1 节。 |
| Backup Extended Boot Sectors | 13 | 8 | 强制；内容见第 3.2 节。 |
| Backup OEM Parameters | 21 | 1 | 强制；内容见第 3.3 节。 |
| Backup Reserved | 22 | 1 | 强制；内容保留。 |
| Backup Boot Checksum | 23 | 1 | 强制；内容见第 3.4 节。 |
| **FAT Region（FAT 区）** | | | |
| FAT Alignment | 24 | FatOffset – 24 | 强制；若有内容则为 Undefined。注：Main/Backup Boot Sector 均含 FatOffset。 |
| First FAT | FatOffset | FatLength | 强制；内容见第 4.1 节。注：含 FatOffset 与 FatLength。 |
| Second FAT | FatOffset + FatLength | FatLength \* (NumberOfFats – 1) | 强制；若有内容见第 4.1 节。NumberOfFats 仅可为 1 或 2。 |
| **Data Region（数据区）** | | | |
| Cluster Heap Alignment | FatOffset + FatLength \* NumberOfFats | ClusterHeapOffset – (FatOffset + FatLength \* NumberOfFats) | 强制；若有内容则为 Undefined。 |
| Cluster Heap（簇堆） | ClusterHeapOffset | ClusterCount \* 2^SectorsPerClusterShift^ | 强制；内容见第 5.1 节。对齐内核中的 cluster heap / 数据簇区。 |
| Excess Space | ClusterHeapOffset + ClusterCount \* 2^SectorsPerClusterShift^ | VolumeLength – (ClusterHeapOffset + ClusterCount \* 2^SectorsPerClusterShift^) | 强制；若有内容则为 Undefined。 |
