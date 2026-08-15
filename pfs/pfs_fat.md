# FAT 标准理解（个人笔记）

> **性质**: 个人对 FAT 标准的理解与归纳, 非官方规范全文, 非 pfs 实现设计稿.
> **官方镜像**: [std_fat/](std_fat/) · 索引 [std-fat-specification_cn.md](std_fat/std-fat-specification_cn.md)
> **冲突**: 官方源 / `std_fat/` / 锁定 `pfs_fat_raw.h` > 本文; 实现见 `portfs/src/fat/`

## 术语

常用词简略理解; 英文名保留. 展开见 [附录 A](#附录a).

| 中文   | 英文                 | 简略                                      |
| ---- | ------------------ | --------------------------------------- |
| 扇区   | **Sector**         | 最小读写块                                   |
| 簇    | **Cluster**        | 分配单位; 数据簇自 **2** 起                      |
| 保留区  | **Reserved**       | Boot 起的保留扇区; 含 BPB / (FAT32) FSINFO 等   |
| FAT表 | **FAT**            | 簇链表; 表项宽度 12/16/32 位                    |
| 根目录  | **Root Directory** | FAT12/16 固定区; FAT32 在数据区 (root_cluster) |
| 目录项  | **Directory Entry**| 32 字节; 短名 + 可选 LFN 槽                    |
| 引导扇区 | **Boot Sector**    | LBA0; BPB; 尾标 `0xAA55`                   |

# 1 简要存储布局(on-disk)

## 1.1 引导扇区(Boot Sector)
* BPB (BIOS Parameter Block; FAT12/16/32 共用前部, 后部 union 分扩展)
* 位置: 0x0; 大小: 512B (结构体至扩展区; 扇区尾 `0xAA55` 在 offset 510); OEM `system_id[8]`; 尾标 `FAT_BOOT_SIGNATURE`
* 变体判别: 根目录项数 / `fat_length` / `fat32.length` 等几何 → FAT12 / FAT16 / FAT32 (非仅看 `fs_type` 字符串)
* 对应: `portfs/src/fat/pfs_fat_raw.h` → `fat_boot_sector_t`

* 核心项：
```
/* FAT12/16/32: Boot Sector (BPB) little endian; 仅列核心字段 */
typedef struct _fat_boot_sector_t {
    uint8_t  system_id[8];                              // OEM 名
    uint8_t  sector_size[2];                            // 逻辑扇区字节数 (LE, 未对齐)
    uint8_t  sec_per_clus;                              // 每簇扇区数
    uint16_t reserved;                                  // 保留扇区数 (含本 Boot)
    uint8_t  fats;                                      // FAT 表份数
    uint8_t  dir_entries[2];                            // 根目录项数 (FAT32 为 0)
    uint16_t fat_length;                                // 每 FAT 扇区数 (FAT32 为 0)
    uint32_t total_sect;                                // 总扇区 (小字段 sectors==0 时用)

    union {
        struct { /* FAT12 / FAT16 扩展 */
            uint8_t  fs_type[8];                        // 如 "FAT16   "
            ....
        } fat16;
        struct { /* FAT32 */
            uint32_t length;                            // 每 FAT 扇区数
            uint32_t root_cluster;                      // 根目录起始簇
            uint16_t info_sector;                       // FSINFO 扇区号
            uint16_t backup_boot;                       // 备份 Boot 扇区号
            uint8_t  fs_type[8];                        // 如 "FAT32   "
            ....
        } fat32;
    };
} fat_boot_sector_t;
```

可解析得到关键点：
```
1. 逻辑扇区大小、簇大小
2. 保留区长度 → FAT 表起始扇区
3. 每 FAT 长度与份数 (fat_length 或 fat32.length; fats)
4. 根目录: FAT12/16 为 FAT 后固定区; FAT32 为 root_cluster
5. 数据区起始簇号 (=2) 与总簇数 → 判别 12/16/32
```

# 附录A

# 附录B

## Windows查看分区错误日志
* 运行打开 eventvwr.msc ，事件查看器
* Windows 日志 - 系统
* 级别 - 来源
```
Fastfat
```
