> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## FAT 卷初始化 (FAT Volume Initialization)

至此, 细心读者应有一个关键问题: FAT 类型 (FAT12 / FAT16 / FAT32) 取决于簇数, 而 FAT 卷数据区可用扇区数又取决于 FAT 大小 — 面对尚无 BPB 的未格式化卷, 如何确定这一切并计算应写入 `BPB_SecPerClus` 以及 `BPB_FATSz16` 或 `BPB_FATSz32` 的值? Microsoft 操作系统采用固定阈值、若干查表项与一段算术完成.

Microsoft 操作系统仅在软盘上使用 FAT12. 软盘格式种类有限且容量固定, 故用简单查表:

> 若为下表所列软盘类型, 则 BPB 按下表填写.

FAT12 无动态计算. 各 FAT12 格式的 `BPB_SecPerClus` 与 `BPB_FATSz16` 均经手工演算后写入表中 (并确保簇数始终小于 4085). 介质大于 4 MB 时不应使用 FAT12; 应减小 `BPB_SecPerClus` 使卷为 FAT16.

本节其余内容 **仅适用于每扇区 512 字节** 的驱动器. 扇区大小非 512 时不可使用这些表或下述算法.

**固定阈值** 即 **FAT16 与 FAT32 分界容量**. 小于该值的卷为 FAT16, 大于等于该值为 FAT32. 对 Windows, 该值为 **512 MB**. 小于 512 MB 的 FAT 卷为 FAT16, 512 MB 及以上默认为 FAT32.

**请勿误解**: 存在大量大于 512 MB 的 FAT16 卷. 可通过多种方式强制格式化为 FAT16, 且各实现的分界策略不同. 此处仅讨论 **MS-DOS / Windows 在未格式化卷上的默认分界**.

有两张表 — 一张 FAT16, 一张 FAT32. 按卷大小 (512 字节扇区数, 即写入 `BPB_TotSec16` 或 `BPB_TotSec32` 的值) 选取表项, 得到 `BPB_SecPerClus`.

```c
struct DSKSZTOSECPERCLUS {
    DWORD   DiskSize;
    BYTE    SecPerClusVal;
};

/*
 * FAT16 驱动器查表. 注意表中包含大于 512 MB 的项,
 * 但通常仅使用 < 512 MB 的项.
 * 用法: 找第一个 DiskSize >= 卷扇区数的表项.
 * 要正常工作, 须 BPB_RsvdSecCnt=1, BPB_NumFATs=2, BPB_RootEntCnt=512.
 * 若这些值不同, 可能须调整首项 DiskSize, 否则 FAT16 簇数可能过低.
 */
DSKSZTOSECPERCLUS DskTableFAT16[] = {
    {        8400,   0}, /* 最大约 4.1 MB; SecPerClusVal=0 触发错误 */
    {      32680,   2},  /* 最大约 16 MB,  1k cluster */
    {    262144,   4},   /* 最大约 128 MB, 2k cluster */
    {   524288,    8},   /* 最大约 256 MB, 4k cluster */
    { 1048576,  16},     /* 最大约 512 MB, 8k cluster */
    /* 以下项仅在强制 FAT16 时使用 */
    { 2097152,  32},     /* 最大约 1 GB,  16k cluster */
    { 4194304,  64},     /* 最大约 2 GB,  32k cluster */
    { 0xFFFFFFFF, 0}     /* 大于 2 GB; SecPerClusVal=0 触发错误 */
};

/*
 * FAT32 驱动器查表. 注意表中包含小于 512 MB 的项,
 * 但通常仅使用 >= 512 MB 的项.
 * 用法: 找第一个 DiskSize >= 卷扇区数的表项.
 * 要正常工作, 须 BPB_RsvdSecCnt=32, BPB_NumFATs=2.
 * 若这些值不同, 可能须调整首项 DiskSize, 否则 FAT32 簇数可能过低.
 */
DSKSZTOSECPERCLUS DskTableFAT32[] = {
    {       66600,   0},  /* 最大约 32.5 MB; SecPerClusVal=0 触发错误 */
    {     532480,   1},   /* 最大约 260 MB,  .5k cluster */
    { 16777216,   8},     /* 最大约 8 GB,    4k cluster */
    { 33554432, 16},      /* 最大约 16 GB,   8k cluster */
    { 67108864, 32},      /* 最大约 32 GB,  16k cluster */
    { 0xFFFFFFFF, 64}     /* 大于 32 GB, 32k cluster */
};
```

给定磁盘大小与 FAT 类型 (FAT16 或 FAT32), 即得 `BPB_SecPerClus`. 余下工作是计算 FAT 占用扇区数, 以设置 `BPB_FATSz16` 或 `BPB_FATSz32`. 此时假定 `BPB_RootEntCnt`、`BPB_RsvdSecCnt`、`BPB_NumFATs` 已正确设置. 并设 `DskSize` 为将写入 `BPB_TotSec32` 或 `BPB_TotSec16` 的卷大小.

```
RootDirSectors = ((BPB_RootEntCnt * 32) + (BPB_BytsPerSec - 1)) / BPB_BytsPerSec;
TmpVal1 = DskSize - (BPB_ResvdSecCnt + RootDirSectors);
TmpVal2 = (256 * BPB_SecPerClus) + BPB_NumFATs;
If (FATType == FAT32)
    TmpVal2 = TmpVal2 / 2;
FATSz = (TmpVal1 + (TmpVal2 - 1)) / TmpVal2;
If (FATType == FAT32) {
    BPB_FATSz16 = 0;
    BPB_FATSz32 = FATSz;
} else {
    BPB_FATSz16 = LOWORD(FATSz);
    /* FAT16 BPB 无 BPB_FATSz32 */
}
```

不必深究该算式的推导. 要点是: **Microsoft 操作系统就是这样算的, 且能工作**. 注意该算法 **并不完美**: FAT16 时 `FATSz` 偶尔偏大最多 2 扇区, FAT32 时最多偏大 8 扇区; **绝不会算得过小**. 因 FAT 略大仅浪费少量扇区, 可接受, 相对其简洁性而言利大于弊.
