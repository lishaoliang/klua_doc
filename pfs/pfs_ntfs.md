# NTFS 标准理解（个人笔记）

> **性质**: 个人对 NTFS 公开说明与 on-disk 的理解与归纳, 非官方规范全文, 非 pfs 实现设计稿.
> **公开说明镜像**: [std_ntfs/](std_ntfs/) · 索引 [std-ntfs-specification_cn.md](std_ntfs/std-ntfs-specification_cn.md)
> **冲突**: 官方源 / `std_ntfs/` / `pfs_ntfs_raw.h` > 本文; 实现见 `portfs/src/ntfs/`

## 术语

常用词简略理解; 英文名保留. 展开见 [附录 A](#附录a).

| 中文    | 英文                        | 简略                                        |
| ----- | ------------------------- | ----------------------------------------- |
| 扇区    | **Sector**                | 最小读写块                                     |
| 簇     | **Cluster**               | 分配单位                                      |
| 逻辑簇号  | **LCN**                   | 卷上物理簇编号 (Logical Cluster Number)          |
| 虚拟簇号  | **VCN**                   | 属性/流内簇编号 (Virtual Cluster Number)         |
| 主文件表  | **MFT**                   | Master File Table; 元数据中心, 文件=MFT 记录集合     |
| MFT记录 | **MFT Record**            | 一条 `FILE` 记录; 常 1024B; 内含属性列表             |
| MFT镜像 | **$MFTMirr**              | 前若干条 MFT 记录副本; Boot 给出其 LCN               |
| MFT引用 | **MFT Reference**         | 48-bit 记录号 + 16-bit 序列号                   |
| 属性    | **Attribute**             | 类型码区分 (`$DATA`/`$FILE_NAME` 等); 常驻或非常驻    |
| 运行列表  | **Runlist** / Data Runs   | 非常驻属性: VCN→LCN 映射链                        |
| 更新序列  | **USA** / Update Sequence | 多扇区记录防撕裂; 读后须做 MST 还原                     |
| 簇位图   | **$Bitmap**               | 簇空闲/占用位图 (按 LCN)                          |
| 大写表   | **$UpCase**               | 文件名比较前做大小写折叠 (UTF-16)                     |
| 目录索引  | **Index** / `$I30`        | 目录名→MFT 引用; 多为 B+ 树索引                     |
| 引导扇区  | **Boot Sector**           | LBA0; OEM `"NTFS    "`; 含 MFT/MFTMirr LCN |

# 1 简要存储布局(on-disk)

## 1.1 引导扇区(Boot Sector)
* BPB (兼容区; 多数 FAT 字段在 NTFS 上为 0)
* 位置: 0x0; 大小: 512B; OEM `"NTFS    "` (`NTFS_OEM_ID`); 尾标 `0xAA55`
* 对应: `portfs/src/ntfs/pfs_ntfs_raw.h` → `ntfs_boot_sector_t` / `ntfs_bios_parameter_block_t`

* 核心项：
```
/* NTFS: Boot Sector (512 bytes) little endian; 仅列核心字段 */
typedef struct _ntfs_boot_sector_t {
    uint64_t oem_id;                                    // 须为 "NTFS    "
    /* bpb */
    uint16_t bytes_per_sector;                          // 逻辑扇区字节数 (常见 512)
    uint8_t  sectors_per_cluster;                       // 每簇扇区数

    uint64_t number_of_sectors;                         // 卷扇区总数
    uint64_t mft_lcn;                                   // $MFT 起始 LCN
    uint64_t mftmirr_lcn;                               // $MFTMirr 起始 LCN

    int8_t   clusters_per_mft_record;                   // MFT 记录大小编码 (>0=簇数; <0=2^|n| 字节)
    int8_t   clusters_per_index_record;                 // 索引块大小编码 (同上)

    ....
} ntfs_boot_sector_t;
```

可解析得到关键点：
```
1. 逻辑扇区大小、簇大小
2. 卷扇区总数
3. MFT 位置 (mft_lcn)
4. MFTMirr 位置 (mftmirr_lcn)
5. MFT 记录大小、索引块大小
```

## 1.2 MFT记录


## 1.3 MFT镜像


## 1.4 目录索引


# 2 核心MFT记录



# 3 目录/文件核心思路



# 附录A

# 附录B

## Windows查看分区错误日志
* 运行打开 eventvwr.msc ，事件查看器
* Windows 日志 - 系统
* 级别 - 来源
```
Ntfs
```

* 检查磁盘元数据错误
```
chkdsk e:
```
