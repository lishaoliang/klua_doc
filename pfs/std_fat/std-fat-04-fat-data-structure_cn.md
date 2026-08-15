> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: `portfs/doc/std_fat/`；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## FAT 数据结构 (FAT Data Structure)

下一重要结构是 **FAT 表本身**. 它以 singly linked list 形式描述文件所占 **簇 (cluster)** 链. 注意: FAT 目录或文件容器本质上只是带特殊属性 (表示其为目录) 的普通文件; 目录的额外特殊性在于其 **数据/内容** 为一系列 32 字节的 FAT 目录项 (见下文). 其余方面目录与文件相同. FAT 按簇号映射卷的数据区. **第一个数据簇为 cluster 2**.

cluster 2 首扇区 (数据区起点) 由 BPB 计算如下. 先求根目录占用扇区数:

```
RootDirSectors = ((BPB_RootEntCnt * 32) + (BPB_BytsPerSec - 1)) / BPB_BytsPerSec;
```

FAT32 卷上 `BPB_RootEntCnt` 恒为 0, 故 `RootDirSectors` 恒为 0. 上式中的 32 为单条 FAT 目录项字节数. 该计算 **向上取整**.

数据区起点, 即 cluster 2 首扇区, 计算如下:

```
If (BPB_FATSz16 != 0)
    FATSz = BPB_FATSz16;
Else
    FATSz = BPB_FATSz32;

FirstDataSector = BPB_ResvdSecCnt + (BPB_NumFATs * FATSz) + RootDirSectors;
```

**注**: 扇区号相对含 BPB 的卷首扇区 (含 BPB 的扇区为扇区 0). 因分区原因, 卷扇区 0 未必对应物理驱动器扇区 0.

对任意有效数据簇号 N, 该簇首扇区号 (仍相对 FAT 卷扇区 0) 为:

```
FirstSectorofCluster = ((N - 2) * BPB_SecPerClus) + FirstDataSector;
```

**注**: `BPB_SecPerClus` 限制为 2 的幂 (1, 2, 4, 8, 16, 32, …), 故在支持 2 补码的架构上, 对 `BPB_SecPerClus` 的乘除可用移位实现, 通常快于 MULT/DIV. 对当前 Intel x86, 该优化意义已不大, 因对 2 的幂的乘除指令已高度优化.
