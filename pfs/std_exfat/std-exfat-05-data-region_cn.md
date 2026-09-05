> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；Cluster Heap / Allocation Bitmap 对齐 `fs/exfat/balloc.c`
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 5 数据区 (Data Region)

数据区包含 Cluster Heap（簇堆），为文件系统结构、目录与文件提供受管理的空间。

### 5.1 Cluster Heap 子区

Cluster Heap 的结构非常简单（见表 12）；连续扇区序列描述一个簇，大小由 SectorsPerClusterShift 定义。重要的是：Cluster Heap 的第一个簇索引为 **2**，直接对应 FatEntry[2] 的索引。

在 exFAT 卷中，由 Allocation Bitmap（分配位图，见第 7.1.5 节）维护所有簇的分配状态。这与前代（FAT12/FAT16/FAT32）有显著不同：前代由 FAT 本身记录 Cluster Heap 中各簇的分配状态。

**表 12 Cluster Heap 结构**

| **字段名** | **偏移** **(扇区)** | **大小** **(扇区)** | **说明** |
| --- | --- | --- | --- |
| Cluster[2] | ClusterHeapOffset | 2^SectorsPerClusterShift^ | 强制；内容见第 5.1.1 节。 |
| ... | ... | ... | ... |
| Cluster[ClusterCount+1] | ClusterHeapOffset + (ClusterCount – 1) \* 2^SectorsPerClusterShift^ | 2^SectorsPerClusterShift^ | 强制；内容见第 5.1.1 节。 |

#### 5.1.1 Cluster[2] ... Cluster[ClusterCount+1] 字段

该数组中每个 Cluster 字段是一段连续扇区，大小由 SectorsPerClusterShift 定义。
