> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## FAT 类型判定

对此存在大量混淆, 导致许多 **off by 1**、**off by 2**、**off by 10**、**massively off** (差 1、差 2、差 10、严重偏差) 错误. 判定方法其实很简单: FAT 类型 (FAT12、FAT16 或 FAT32 之一) **仅** 由卷上的 **簇计数 (count of clusters)** 决定, 与任何其他因素无关.

请仔细阅读本节全部内容, 每个词都很重要. 例如注意表述是 **count of clusters** (簇计数), 而非 **maximum valid cluster number** (最大有效簇号), 因为第一个数据簇是 2, 而非 0 或 1.

首先说明 **count of clusters** 如何计算. 全部使用卷的 BPB 字段. 首先按前文计算根目录所占扇区数:

```
RootDirSectors = ((BPB_RootEntCnt * 32) + (BPB_BytsPerSec - 1)) / BPB_BytsPerSec;
```

FAT32 卷上 `BPB_RootEntCnt` 始终为 0, 故 `RootDirSectors` 始终为 0.

接下来计算卷数据区扇区数:

```
If (BPB_FATSz16 != 0)
    FATSz = BPB_FATSz16;
Else
    FATSz = BPB_FATSz32;

If (BPB_TotSec16 != 0)
    TotSec = BPB_TotSec16;
Else
    TotSec = BPB_TotSec32;

DataSec = TotSec - (BPB_RsvdSecCnt + (BPB_NumFATs * FATSz) + RootDirSectors);
```

再计算簇数:

```
CountofClusters = DataSec / BPB_SecPerClus;
```

注意: 此计算 **向下取整**.

现在可判定 FAT 类型. 请格外小心, 否则会 off-by-one!

下列示例中, `<` 表示 **小于**, **不是** `<=`. 数字正确: FAT12 阈值为 4085; FAT16 阈值为 65525. 这些数字与 `<` 符号无误.

```
If (CountofClusters < 4085) {
    /* Volume is FAT12 */
} else if (CountofClusters < 65525) {
    /* Volume is FAT16 */
} else {
    /* Volume is FAT32 */
}
```

**这是判定 FAT 类型的唯一方法.** 不存在簇数超过 4084 的 FAT12 卷. 不存在簇数少于 4085 或多于 65,524 的 FAT16 卷. 不存在簇数少于 65,525 的 FAT32 卷. 若试图创建违反此规则的 FAT 卷, Microsoft 操作系统将无法正确处理, 因为它们会认为卷的 FAT 类型与您设想的不同.

**注**: 如前文多次所述, 世界上大量 FAT 代码是错误的. 许多 FAT 类型判定代码差 1、2、8、10 或 16. 因此, 若格式化需与所有现有 FAT 代码最大兼容的 FAT 卷, 应避免创建簇数接近 4,085 或 65,525 的任何类型卷. 应距这些切换簇数至少 16 个簇以上.

另请注意, `CountofClusters` 正是从簇 2 开始的数据簇计数. 卷的最大有效簇号为 `CountofClusters + 1`; **含两个保留簇在内的簇总数** 为 `CountofClusters + 2`.

与 FAT 相关的另一重要计算: 给定任意有效簇号 N, 该簇在 FAT 中的表项位于何处? 仅 FAT12 较复杂; FAT16 与 FAT32 较简单:

```
If (BPB_FATSz16 != 0)
    FATSz = BPB_FATSz16;
Else
    FATSz = BPB_FATSz32;

If (FATType == FAT16)
    FATOffset = N * 2;
Else if (FATType == FAT32)
    FATOffset = N * 4;

ThisFATSecNum = BPB_ResvdSecCnt + (FATOffset / BPB_BytsPerSec);
ThisFATEntOffset = REM(FATOffset / BPB_BytsPerSec);
```

`REM()` 为取余运算符, 即 `FATOffset` 除以 `BPB_BytsPerSec` 的余数. `ThisFATSecNum` 是第一个 FAT 中包含簇 N 表项的 FAT 扇区号 (相对 FAT 卷第 0 扇区). 第二个 FAT 加 `FATSz`; 第三个加 `2*FATSz`, 依此类推.

读取扇区号 `ThisFATSecNum` (相对 FAT 卷扇区 0). 假设读入名为 `SecBuff` 的 8 位字节数组. 假设 `WORD` 为 16 位无符号, `DWORD` 为 32 位无符号.

读取簇表项内容:

```
If (FATType == FAT16)
    FAT16ClusEntryVal = *((WORD *) &SecBuff[ThisFATEntOffset]);
Else
    FAT32ClusEntryVal = (*((DWORD *) &SecBuff[ThisFATEntOffset])) & 0x0FFFFFFF;
```

写入同一簇表项:

```
If (FATType == FAT16)
    *((WORD *) &SecBuff[ThisFATEntOffset]) = FAT16ClusEntryVal;
Else {
    FAT32ClusEntryVal = FAT32ClusEntryVal & 0x0FFFFFFF;
    *((DWORD *) &SecBuff[ThisFATEntOffset]) =
        (*((DWORD *) &SecBuff[ThisFATEntOffset])) & 0xF0000000;
    *((DWORD *) &SecBuff[ThisFATEntOffset]) =
        (*((DWORD *) &SecBuff[ThisFATEntOffset])) | FAT32ClusEntryVal;
}
```

注意 FAT32 代码: FAT32 FAT 表项实际仅为 28 位, 高 4 位保留. 仅格式化时应修改 FAT32 表项高 4 位, 此时整个 32 位表项 (含高 4 位) 应清零.

关于 FAT32 表项还有一点常引起混淆: 32 位 FAT 表项并非真正的 32 位值, 仅为 28 位. 例如 0x10000000、0xF0000000、0x00000000 均表示簇 **FREE** (空闲), 因读取时忽略高 4 位. 若 32 位空闲簇值为 0x30000000, 欲以 0x0FFFFFF7 标记为坏簇, 完成后 32 位表项为 0x3FFFFFF7, 因写入 0x0FFFFFF7 时必须保留高 4 位.

另请注意, 因 `BPB_BytsPerSec` 总能被 2 和 4 整除, FAT16/FAT32 表项 never 跨扇区边界 (FAT12 则不然).

FAT12 代码更复杂, 因每个表项 1.5 字节 (12 位):

```
if (FATType == FAT12)
    FATOffset = N + (N / 2);
/* 不用浮点实现乘以 1.5, 除以 2 向下取整 */

ThisFATSecNum = BPB_RsvdSecCnt + (FATOffset / BPB_BytsPerSec);
ThisFATEntOffset = REM(FATOffset / BPB_BytsPerSec);
```

须检查跨扇区边界情况:

```
If (ThisFATEntOffset == (BPB_BytsPerSec - 1)) {
    /* 此次簇访问跨越 FAT 中的扇区边界 */
    /* 有多种处理策略. 最简单的是 FAT12 卷始终成对加载 FAT 扇区 */
    /* (若要加载 FAT 扇区 N, 除非 N 是最后一个 FAT 扇区, 否则同时在内存中加载 N+1) */
    /* 此处假设采用该策略, 使跨扇区边界测试不必要 */
}
```

随后像 FAT16 一样以 WORD 访问表项, 但若簇号为 **偶数**, 只取 16 位中的低 12 位; 若为奇数, 只取高 12 位:

```
FAT12ClusEntryVal = *((WORD *) &SecBuff[ThisFATEntOffset]);
If (N & 0x0001)
    FAT12ClusEntryVal = FAT12ClusEntryVal >> 4;   /* 簇号为奇数 */
Else
    FAT12ClusEntryVal = FAT12ClusEntryVal & 0x0FFF; /* 簇号为偶数 */
```

写入:

```
If (N & 0x0001) {
    FAT12ClusEntryVal = FAT12ClusEntryVal << 4;   /* 簇号为奇数 */
    *((WORD *) &SecBuff[ThisFATEntOffset]) =
        (*((WORD *) &SecBuff[ThisFATEntOffset])) & 0x000F;
} Else {
    FAT12ClusEntryVal = FAT12ClusEntryVal & 0x0FFF; /* 簇号为偶数 */
    *((WORD *) &SecBuff[ThisFATEntOffset]) =
        (*((WORD *) &SecBuff[ThisFATEntOffset])) & 0xF000;
}
*((WORD *) &SecBuff[ThisFATEntOffset]) =
    (*((WORD *) &SecBuff[ThisFATEntOffset])) | FAT12ClusEntryVal;
```

**注**: 假设 `>>` 将 0 移入高 4 位, `<<` 将 0 移入低 4 位.

文件数据与文件的关联方式如下: 目录项中记录文件第一个簇的簇号. 文件第一个簇 (extent) 即与该簇号关联的数据, 其在卷上的位置按前文 (FirstSectorofCluster 计算) 由簇号得出.

**零长度文件** (未分配数据的文件) 在目录项中首簇号置 0. FAT 中该簇位置 (见 `ThisFATSecNum`/`ThisFATEntOffset` 计算) 含 EOC 标记 (End Of Clusterchain, 簇链结束) 或文件下一簇的簇号. EOC 值依 FAT 类型而定 (设 `FATContent` 为被检查是否为 EOC 的 FAT 表项内容):

```
IsEOF = FALSE;
If (FATType == FAT12) {
    If (FATContent >= 0x0FF8)
        IsEOF = TRUE;
} else if (FATType == FAT16) {
    If (FATContent >= 0xFFF8)
        IsEOF = TRUE;
} else if (FATType == FAT32) {
    If (FATContent >= 0x0FFFFFF8)
        IsEOF = TRUE;
}
```

注意: FAT 表项含 EOC 标记的簇号已分配给文件, 且为文件最后一个已分配簇. Microsoft 操作系统 FAT 驱动设置 EOC 时使用: FAT12 为 0x0FFF, FAT16 为 0xFFFF, FAT32 为 0x0FFFFFFF. 部分 Microsoft 操作系统磁盘工具使用不同值.

另有特殊 **BAD CLUSTER** (坏簇) 标记. FAT 表项含 BAD CLUSTER 值的簇不应放入空闲列表, 因其易出现磁盘错误. BAD CLUSTER 值: FAT12 为 0x0FF7, FAT16 为 0xFFF7, FAT32 为 0x0FFFFFF7. 这些坏簇也是 **lost clusters** (丢失簇): 表项非零但不在任何文件分配链中. 磁盘修复工具须将含此特殊值的丢失簇识别为坏簇, 不得修改表项内容.

**注**: 坏簇标记在 FAT12/FAT16 卷上不可能是可分配簇号, 但在 FAT32 卷上 0x0FFFFFF7 可能是可分配簇号. 为避免磁盘工具混淆, 任何 FAT32 卷都不应配置成使 0x0FFFFFF7 成为可分配簇号.

FAT 中的空闲簇列表就是所有 FAT 表项值为 0 的簇的列表. 须按前文方式读取非空闲表项一样读取此值. 空闲簇列表不存储在卷上; 挂载时必须扫描 FAT 中值为 0 的表项计算. FAT32 卷上 `BPB_FSInfo` 扇区可能含卷上有效空闲簇计数. 见 FAT32 FSInfo 扇区文档.

FAT 起始处两个保留簇有何用途? 第一个保留簇 FAT[0] 低 8 位含 `BPB_Media` 字节值, 其余位全为 1. 例如 `BPB_Media` 为 0xF8: FAT12 时 FAT[0]=0x0FF8, FAT16 时 0xFFF8, FAT32 时 0x0FFFFFF8. 第二个保留簇 FAT[1] 由 FORMAT 设为 EOC 标记. FAT12 卷未使用, 始终含 EOC. FAT16/FAT32 上文件系统驱动可能将 FAT[1] 表项高 2 位用作脏卷标志 (其余位始终为 1). 注意 FAT16 与 FAT32 位位置不同, 因它们分别是表项的高 2 位.

FAT16:

```
ClnShutBitMask  = 0x8000;
HrdErrBitMask   = 0x4000;
```

FAT32:

```
ClnShutBitMask  = 0x08000000;
HrdErrBitMask   = 0x04000000;
```

| 位 | 含义 |
| --- | --- |
| `ClnShutBitMask` | 若为 1, 卷为 **clean** (正常卸载); 若为 0, 卷为 **dirty** (上次挂载时未正确 Dismount). 建议运行 Chkdsk/Scandisk 修复, 卷可能已损坏. |
| `HrdErrBitMask` | 若为 1, 上次挂载未遇磁盘读写错误; 若为 0, 上次挂载遇磁盘 I/O 错误, 表示部分扇区可能已坏. 建议运行含表面分析的 Chkdsk/Scandisk 查找新坏扇区. |

关于 FAT 区域还有两条重要说明:

1. FAT 最后一个扇区不一定全部属于 FAT. FAT 在最后一个 FAT 扇区中止于对应簇号 `CountofClusters + 1` 的表项 (见前文 `CountofClusters` 计算), 该表项不一定位于最后 FAT 扇区末尾. FAT 代码不应对 `CountofClusters + 1` 表项之后最后 FAT 扇区内容作任何假设. FAT 格式化代码应将此表项之后的字节清零.

2. `BPB_FATSz16` (FAT32 为 `BPB_FATSz32`) 可能大于实际需要. 即每个 FAT 末尾可能有完全未使用的 FAT 扇区. 因此最后 FAT 扇区始终用 `CountofClusters + 1` 计算, 从不单独用 `BPB_FATSz16/32`. FAT 代码不应对这些额外 FAT 扇区内容作假设. FAT 格式化代码应将这些额外 FAT 扇区内容清零.
