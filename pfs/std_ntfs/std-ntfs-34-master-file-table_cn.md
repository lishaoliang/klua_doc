> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

## NTFS 物理结构

### 主文件表 (Master File Table, MFT)

使用 NTFS 格式化卷时, Windows Server 2003 在分区上创建 MFT 与元数据文件. MFT 是一种关系数据库, 由 file record 行与 file attribute 列组成. 它至少含 NTFS 卷上每个文件的一条项, 包括 MFT 自身.

MFT 存储从 NTFS 分区检索文件所需的信息.

#### MFT 与元数据文件 (MFT and Metadata Files)

由于 MFT 存储关于自身的信息, NTFS 为 MFT 的前 16 条记录 (约 16 KB) 保留给元数据文件, 用于描述 MFT. 以美元符号 ($) 开头的元数据文件见表 Metadata Files Stored in the MFT. MFT 的其余记录含卷上每个文件与文件夹的 file record 与 folder record.

**Metadata Files Stored in the MFT (存储于 MFT 的元数据文件)**

| 系统文件 | 文件名 | MFT Record | 文件用途 |
| --- | --- | --- | --- |
| Master file table (主文件表) | $Mft | 0 | 含 NTFS 卷上每个文件与文件夹的一条 base file record. 若文件或文件夹的分配信息过大无法放入单条 record, 还会分配其他 file record. |
| Master file table mirror (MFT 镜像) | $MftMirr | 1 | 保证在单扇区故障时仍可访问 MFT. 它是 MFT 前四条记录的重复镜像. |
| Log file (日志文件) | $LogFile | 2 | 含 NTFS 用于更快可恢复性的信息. Windows Server 2003 用 log file 在系统故障后将 NTFS 元数据一致性恢复. log file 大小取决于卷大小, 但可用 Chkdsk 命令增大. |
| Volume (卷) | $Volume | 3 | 含卷信息, 如卷标与卷版本. |
| Attribute definitions (属性定义) | $AttrDef | 4 | 列出属性名、编号与描述. |
| Root file name index (根文件名索引) | . | 5 | 根目录. |
| Cluster bitmap (簇位图) | $Bitmap | 6 | 以显示空闲与已用簇的方式表示卷. |
| Boot sector (引导扇区) | $Boot | 7 | 含用于挂载卷的 BPB 以及卷可引导时使用的额外 bootstrap loader code. |
| Bad cluster file (坏簇文件) | $BadClus | 8 | 含卷的坏簇. |
| Security file (安全文件) | $Secure | 9 | 含卷内所有文件的唯一 security descriptor. |
| Upcase table (大写表) | $Upcase | 10 | 将小写字符转换为匹配的 Unicode 大写字符. |
| NTFS extension file (NTFS 扩展文件) | $Extend | 11 | 用于各种可选扩展, 如配额、reparse point 数据与 object identifier. |
| | | 12–15 | 保留供将来使用. |

$Mft 与备份 MFT $MftMirr 的数据段位置均记录在 boot sector 中. $MftMirr 是 $Mft 前四条 record 或 $Mft 第一个簇 (取较大者) 的重复镜像. 若镜像范围内任一 MFT record 损坏或不可读, NTFS 读取 boot sector 以找到 $MftMirr 位置, 然后读取 $MftMirr 并使用其中信息替代 MFT 中对应信息. 若可能, $MftMirr 中的正确数据会写回 MFT 的对应位置.

#### MFT Zone (MFT 区)

为防止 MFT 碎片化, NTFS 默认保留卷容量的 12.5% 专供 MFT 使用. 该空间称 MFT zone, 除非卷其余部分已满, 否则不用于存储数据.

依平均文件大小与其他变量, 当卷接近满载时, MFT zone 或卷上未保留空间会先满.

- 含少量大文件的卷会先耗尽未保留空间.
- 含大量小文件的卷会先耗尽 MFT zone 空间.

无论哪种情况, 当某一区域满时 MFT 会发生碎片化. 可通过注册表设置更改新建卷的 MFT zone 大小, 使其为卷容量的某一百分比. MFT zone 大小设置如下:

- 设置 1 (默认) 保留约 12.5% 卷容量.
- 设置 2 保留约 25%.
- 设置 3 保留约 37.5%.
- 设置 4 保留约 50%.

在大多数计算机上, 默认设置 1 已足够. 默认设置适合平均文件大小 8 KB 的卷. 存储大量较小文件可能需要为新卷增大 MFT zone.

增大 MFT zone 后, NTFS 不会立即分配空间以容纳新 MFT zone 大小. NTFS 会先耗尽原保留空间, 再增大 MFT zone. 原空间耗尽后, NTFS 查找下一个足够大的连续空间以容纳额外 MFT zone, 这可能导致 MFT 碎片化. 若默认值不符合需求, 可调整 MFT zone 大小.
