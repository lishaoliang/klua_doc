# exFAT 标准理解（个人笔记）

> **性质**: 个人对 exFAT 标准的理解与归纳, 非官方规范全文, 非 pfs 实现设计稿.
> **官方镜像**: [std_exfat/](std_exfat/) · 索引 [std-exfat-specification_cn.md](std_exfat/std-exfat-specification_cn.md)
> **冲突**: 官方源 / `std_exfat/` > 本文; 实现见 `portfs/src/exfat/`

## 术语

常用词简略理解; 英文名保留. 展开见 [附录 A](#附录a).

| 中文      | 英文                    | 简略                      |
| ------- | --------------------- | ----------------------- |
| 扇区      | **Sector**            | 最小读写块                   |
| 簇       | **Cluster**           | 分配单位; 数据簇自 **2** 起      |
| 簇堆      | **Cluster Heap**      | 数据区主体                   |
| FAT表    | **FAT**               | 簇链表; 表项 4 字节            |
| Bitmap图 | **Allocation Bitmap** | 分配位图，是否空闲看它 (非只看 FAT=0) |
| 大写表     | **Up-case Table**     | 文件名比较前做大小写折叠            |
| 根目录     | **Root Directory**    | 在簇堆内, 可增长               |
| 目录项(文件) | **Directory Entry**   | 固定 32 字节, EntryType 分种类 |

# 1 简要存储布局(on-disk)

## 1.1 引导扇区(Boot Sector)
* 旧 BPB 区 (`must_be_zero[53]`) 须全 0; 几何在后续字段 (非 FAT 式 BPB)
* 位置: Main Boot 扇区 0; Backup Boot 通常扇区 12 起; 大小: 512B; `fs_name` `"EXFAT   "` (`STR_EXFAT`); 尾标 `0xAA55` (`EXFAT_BOOT_SIGNATURE`)
* 对应: `portfs/src/exfat/pfs_exfat_raw.h` → `exfat_boot_sector_t`

* 核心项：
```
/* EXFAT: Main and Backup Boot Sector (512 bytes) little endian; 仅列核心字段 */
typedef struct _exfat_boot_sector_t {
    uint8_t  fs_name[8];                                // 须为 "EXFAT   "
    /* must_be_zero[53]: 旧 BPB 兼容区, 须全 0 */

    uint64_t vol_length;                                // 卷总扇区数
    uint32_t fat_offset;                                // FAT 表首扇区偏移
    uint32_t fat_length;                                // 单个 FAT 表扇区数

    uint32_t clu_offset;                                // 簇堆首扇区偏移
    uint32_t clu_count;                                 // 簇堆可用簇数
    uint32_t root_cluster;                              // 根目录起始簇号

    uint8_t  sect_size_bits;                            // 扇区大小指数 (9~12 → 512..4096)
    uint8_t  sect_per_clus_bits;                        // 每簇扇区数指数
    uint8_t  num_fats;                                  // FAT 表份数 (1 或 2)

    ....
} exfat_boot_sector_t;
```

可解析得到关键点：
```
1. 逻辑扇区大小、簇大小 (2^sect_size_bits, 再 × 2^sect_per_clus_bits)
2. 卷扇区总数 (vol_length)
3. FAT 表位置与长度 (fat_offset / fat_length)
4. 簇堆位置与簇数 (clu_offset / clu_count)
5. 根目录起始簇 (root_cluster; Bitmap/Up-case 仍须扫根目录项)
```

## 1.2 簇堆
* 相当于数据区
* 包含：目录项、Bitmap图、大写表、文件数据等

## 1.3 目录项
* 单个目录项大小 32B
* exFAT 的 元数据

* 核心数据结构项
```
typedef struct _exfat_dentry_t {
    uint8_t type;                                       // 目录项类型
    union {
        struct {                                        // 主文件目录项 
	        ...
        } file; /* file directory entry */
        struct {                                        // 流扩展目录项 
	        ...
        } stream; /* stream extension directory entry */
        struct {                                        // Bitmap图 
	        ...
        } bitmap; /* allocation bitmap directory entry */
        struct {                                        // 大写表目录项 
	        ...
        } upcase; /* up-case table directory entry */
		...
    } dentry;
} exfat_dentry_t;
```

如果一个簇是存储目录项，那么整个簇 存放的都是目录项；即类似一个32B的数组；

## 1.4 根目录簇

## 1.5 Bitmap图
## 1.6 FAT表
## 1.7 大写表

## 1.8 解析次序
```
1. 读引导扇区(LBA0:512B)，解析得到 
   a. 逻辑扇区大小、簇大小
   b. FAT表的位置
   c. 簇堆位置
   d. 根目录位置 
   
2. 读取根目录簇，解析得到
   a. Bitmap图
   b. 大写表
```

## 1.9 核心布局图

```
  +------------------+  扇区 0
  | 1. Boot Sector   |  Main Boot (512B) + Extended/OEM/Checksum ...
  |    (+ Backup)    |  Backup Boot 紧随其后 (扇区 12 起)
  +------------------+  FatOffset
  | 2. FAT 表        |  FatLength 扇区; 簇链表 (表项 4B)
  +------------------+  ClusterHeapOffset
  | 6. 簇堆          |  ClusterCount 个簇; 数据簇号自 2 起
  |   Cluster Heap   |
  |                  |
  |   +------------+ |  (以下均在簇堆内, 位置由目录项/簇号给出, 非固定扇区)
  |   | 5. 根目录  | |  root_cluster 起; 内含 Bitmap/Upcase 等目录项
  |   +------------+ |
  |   | 3. Bitmap  | |  Allocation Bitmap; 根目录 Bitmap 项指向其首簇
  |   +------------+ |
  |   | 4. Up-case | |  大写表; 根目录 Up-case 项指向其首簇
  |   +------------+ |
  |   | 文件/子目录 | |  其它簇: 用户数据与子目录
  |   +------------+ |
  +------------------+
  | (Excess Space)   |  可选尾部未纳入簇堆的空间
  +------------------+
```

# 2 目录/文件核心思路

## 2.1 空闲与已分配簇

## 2.2 连续簇与FAT链簇

## 2.3 根目录
## 2.4 子目录
## 2.5 文件




# 附录A


# 附录B

## Windows查看分区错误日志
* 运行打开 eventvwr.msc ，事件查看器
* Windows 日志 - 系统
* 级别 - 来源
```
exFAT
```

