> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## FAT32 FSInfo 扇区与备份引导扇区 (FAT32 FSInfo Sector Structure and Backup Boot Sector)

FAT32 卷上 FAT 表可能很大; 相比之下 FAT16 最多约 128 KB 扇区, FAT12 最多约 6 KB 扇区. 因此 FAT32 提供字段存储 **上次已知** 的空闲簇计数, 避免在 API 查询剩余空间时 (例如目录列表末尾) 立即全表扫描. FSInfo 扇区号由 `BPB_FSInfo` 给出; Microsoft 实现 **恒为 1**. FSInfo 扇区结构如下 (对齐内核 `struct fat_boot_fsinfo` / `FAT_FSINFO_SIG1` / `FAT_FSINFO_SIG2`):

**表: FSInfo 扇区结构**

| **字段名** | **偏移 (字节)** | **大小 (字节)** | **说明** |
| --- | --- | --- | --- |
| FSI_LeadSig | 0 | 4 | 值 `0x41615252`. 首部签名, 用于校验本扇区为 FSInfo. |
| FSI_Reserved1 | 4 | 480 | 保留供将来扩展. FAT32 格式化代码 **shall** 将全部字节置 0; 当前 **shall not** 使用. |
| FSI_StrucSig | 484 | 4 | 值 `0x61417272`. 局部签名, 标记后续有效字段位置. |
| FSI_Free_Count | 488 | 4 | 卷上 **上次已知** 空闲簇数. 若为 `0xFFFFFFFF` 表示未知, 须重新计算. 其他值可用但未必准确; 至少应校验 `<=` 卷簇总数. |
| FSI_Nxt_Free | 492 | 4 | 给 FAT 驱动的 **hint**: 从哪个簇号开始查找空闲簇. FAT32 FAT 很大时, 若表前段已分配簇很多而从 cluster 2 开始扫描会很慢. 通常设为上次分配的簇号. 若为 `0xFFFFFFFF` 表示无 hint, 应从 cluster 2 开始. 其他值可用但应先校验为卷上有效簇号. |
| FSI_Reserved2 | 496 | 12 | 保留. 格式化时 **shall** 置 0; 当前 **shall not** 使用. |
| FSI_TrailSig | 508 | 4 | 值 `0xAA550000`. 尾部签名. 高 2 字节 (偏移 510、511) 与扇区 0 同偏移处的引导签名一致. |

FAT16/FAT12 不具备、FAT32 具备的另一特性是 **`BPB_BkBootSec` 字段**. FAT16/FAT12 若卷扇区 0 被覆盖或损坏, 整卷可能无法识别 — 对 FAT12/16 这是 **单点故障**. `BPB_BkBootSec` 降低 FAT32 上该风险: 从该扇区号起存有引导信息备份, 含完整 BPB.

若扇区 0 被意外覆盖, 修复工具只需从备份恢复引导扇区. 若扇区 0 物理不可读, 仍可挂载卷以便用户抢救数据.

**扇区 0 损坏** 正是 **`BPB_BkBootSec` 应恒为 6** 的原因. 扇区 0 不可读时, 多种操作系统 **硬编码** 从 FAT32 卷 **扇区 6** 查找备份引导. 从 `BPB_BkBootSec` 起为 **完整 boot record**. Microsoft FAT32 **boot sector** 实际为 **三个 512 字节扇区**; 自 `BPB_BkBootSec` 起复制全部三扇区. FSInfo 扇区亦有副本, 尽管备份 BPB 中 `BPB_FSInfo` 与扇区 0 BPB 中取值相同.

**注**: 这三个扇区在偏移 510、511 处均有 `0xAA55` 签名, 与首引导扇区相同 (见 BPB 结构说明末尾).
