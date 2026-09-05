> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 9 文件系统限制 (File System Limits)

### 9.1 扇区大小限制 (Sector Size Limits)

BytesPerSectorShift 定义扇区大小的上下限（**下限：512 字节；上限：4,096 字节**）。

### 9.2 簇大小限制 (Cluster Size Limits)

SectorsPerClusterShift 定义簇大小的上下限（**下限：1 扇区；上限：25 – BytesPerSectorShift 个扇区**，即最大约 32MB）。

### 9.3 Cluster Heap 大小限制 (Cluster Heap Size Limits)

Cluster Heap **shall** 至少能容纳下列基本文件系统结构：根目录、全部 Allocation Bitmap、以及 Up-case Table。

Cluster Heap 大小下限是上述各基本结构下限的函数。即使取最小簇（512 字节），每个基本结构也各不超过一个簇。因此 **下限为：2 + NumberOfFats 个簇**，按 NumberOfFats 取值合成为 3 或 4 个簇。

Cluster Heap 大小上限是 ClusterCount 所定义的最大可能簇数的简单函数（**上限：2^32^ – 11 个簇**）。无论簇大小如何，这样的簇堆至少足以容纳基本文件系统结构。

### 9.4 卷大小限制 (Volume Size Limits)

VolumeLength 定义卷大小的上下限（下限：**2^20^ / 2^BytesPerSectorShift^ 扇区**，即 1MB；**上限：2^64^ – 1 扇区**，在最大扇区下约合 64ZB）。但本规范建议 Cluster Heap 中簇数不超过 2^24^ – 2（见第 3.1.9 节）。因此建议的卷上限为：ClusterHeapOffset + (2^24^ – 2) \* 2^SectorsPerClusterShift^。取最大簇 32MB，并假设 ClusterHeapOffset 为 96MB（足够 Main/Backup Boot 与仅 First FAT），建议上限约合 512TB。

### 9.5 目录大小限制 (Directory Size Limits)

Stream Extension 目录项的 DataLength 定义目录大小上下限（**下限：0 字节；上限：256MB**）。即一个目录最多可容纳 8,388,608 个目录项（每项 32 字节）。按最小 File 目录项集合（三项）计，一个目录最多可容纳约 2,796,202 个文件。
