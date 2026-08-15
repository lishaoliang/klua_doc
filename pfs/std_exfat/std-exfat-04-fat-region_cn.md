> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: `portfs/doc/std_exfat/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；FAT / 簇链语义对齐 linux-7.1.2 `fs/exfat/fatent.c`
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 4 文件分配表区 (File Allocation Table Region)

FAT 区最多可含两份 FAT：第一份在 First FAT 子区，第二份在 Second FAT 子区。NumberOfFats 描述本区含几份 FAT，有效值为 1 与 2。因此 First FAT 子区始终含一份 FAT；若 NumberOfFats 为 2，则 Second FAT 子区也含一份 FAT。

VolumeFlags 中的 ActiveFat 字段描述哪一份 FAT 为活动。仅 Main Boot Sector 中的 VolumeFlags 为当前有效。实现方 **shall** 将非活动 FAT 视为过期。非活动 FAT 的使用及在两份 FAT 间切换由实现自行决定。

### 4.1 First 与 Second FAT 子区

FAT **shall** 描述 Cluster Heap（簇堆）中的簇链（见表 11）。簇链是一系列簇，为文件、目录及其他文件系统结构提供记录内容的空间。FAT 将簇链表示为簇号索引的单向链表。除前两个表项外，FAT 中每一表项精确对应一个簇。

**表 11 文件分配表结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| FatEntry[0] | 0 | 4 | 强制；内容见第 4.1.1 节。 |
| FatEntry[1] | 4 | 4 | 强制；内容见第 4.1.2 节。 |
| FatEntry[2] | 8 | 4 | 强制；内容见第 4.1.3 节。 |
| ... | ... | ... | ... |
| FatEntry[ClusterCount+1] | (ClusterCount + 1) \* 4 | 4 | 强制；内容见第 4.1.3 节。ClusterCount + 1 永不超过 FFFFFFF6h。 |
| ExcessSpace | (ClusterCount + 2) \* 4 | (FatLength \* 2^BytesPerSectorShift^) – ((ClusterCount + 2) \* 4) | 强制；若有内容则为 Undefined。 |

#### 4.1.1 FatEntry[0] 字段

FatEntry[0] **shall** 在第一个字节（最低有效字节）描述介质类型，其余三字节为 FFh。

介质类型（第一字节）宜为 F8h。

#### 4.1.2 FatEntry[1] 字段

FatEntry[1] 仅因历史兼容而存在，不描述有用信息。

该字段有效值为 FFFFFFFFh。实现方 **shall** 将其初始化为规定值，且不宜用于任何目的；不宜解释该字段，并在修改周边字段的操作中 **shall** 保留其内容。

#### 4.1.3 FatEntry[2] ... FatEntry[ClusterCount+1] 字段

该数组中每个 FatEntry 表示 Cluster Heap 中的一个簇。FatEntry[2] 表示堆中第一个簇，FatEntry[ClusterCount+1] 表示最后一个簇。（数据簇自 **2** 起，与内核/pfs 一致。）

这些字段的有效取值范围为：

- 闭区间 [2, ClusterCount + 1]，指向给定簇链中的下一个 FatEntry；给定 FatEntry **shall not** 指向同一簇链中位于其之前的任何 FatEntry
- 恰好 FFFFFFF7h，将对应簇标记为 “坏簇” (bad)
- 恰好 FFFFFFFFh，将对应簇标记为簇链的最后一个簇；这是任何簇链末项的唯一有效值（EOF）
