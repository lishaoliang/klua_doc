> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

## NTFS 物理结构

下列内容说明 NTFS 卷上簇与扇区的组织方式, 卷上引导扇区如何标识文件系统, 以及主文件表 (MFT) 如何在卷上组织各类结构.

### NTFS 卷上的簇与扇区

簇 (cluster, 亦称 allocation unit) 是可为文件分配的最小磁盘空间单位. Windows Server 2003 所用的全部文件系统均按簇大小组织硬盘, 簇大小由每个簇包含的扇区 (硬盘上的存储单位) 数量决定. 例如, 在采用 512 字节扇区的磁盘上, 512 字节簇含 1 个扇区, 而 4 KB 簇含 8 个扇区.

计算机在启动时会访问硬盘上的特定扇区, 以确定启动哪个操作系统以及分区位于何处. 这些扇区上存储的数据因计算机平台而异.

#### NTFS 卷上的簇序号

NTFS 卷上的簇从分区起始处起按顺序编号, 形成逻辑簇号 (LCN). NTFS 使用名为 Master File Table (MFT) 的记录存储文件系统中的全部对象, 其结构类似数据库.

在 NTFS 卷上, 簇从扇区 0 开始; 因此每个簇均对齐于簇边界. 文件存储所用连续簇可加快文件处理速度.

**Note**

- 软盘不使用 NTFS, 始终格式化为 FAT.

#### NTFS 卷上簇大小的限制

由于 NTFS 依卷大小采用不同簇大小, 每种文件系统可支持的簇数有上限. 簇越小, 磁盘在理论上越能高效存储信息, 因为簇内未用空间无法被其他文件复用. 文件系统支持的簇越多, 用该文件系统可创建并格式化的卷就越大. NTFS 使用较小簇, 因而文件组织更高效.

下表 **Default NTFS Cluster Sizes** 列出 NTFS 卷大小与默认簇大小.

**Default NTFS Cluster Sizes**

| Volume Size | NTFS Cluster Size |
| --- | --- |
| 7 megabytes (MB)–512 MB | 512 bytes |
| 513 MB–1,024 MB | 1 KB |
| 1,025 MB–2 GB | 2 KB |
| 2 GB–2 terabytes | 4 KB |

#### NTFS 卷上的最大尺寸

在格式化 NTFS 卷之前, 应评估将要存储的文件类型, 以决定是否使用默认簇大小.

格式化 NTFS 卷时, 可通过 Disk Management 管理单元指定最大 64 KB 的簇大小. 若格式化时未指定簇大小, 则使用默认值. 若要在格式化后更改簇大小, 必须重新格式化该卷.

在选择非默认簇大小之前, 请注意下列重要限制:

- 对 Microsoft Windows NT、Windows 2000、Windows XP 与 Windows Server 2003, 2 GB 至 4 GB 的 FAT16 卷簇大小为 64 KB, 可能与部分应用程序存在兼容性问题. 例如, 安装程序在 64 KB 簇卷上无法正确计算可用空间, 并可能因误以为可用空间不足而无法运行. 因此, 大于 2 GB 的卷可使用 NTFS 或 FAT32 格式化.
- 由于簇大小大于 4 KB 时不支持文件压缩, Windows Server 2003 的默认 NTFS 簇大小不超过 4 KB.

理论上, 最大 NTFS 卷大小为 2^64 个簇减 1 个簇. 然而, Windows Server 2003 实现中的最大 NTFS 卷大小为 2^32 个簇减 1 个簇. 例如, 使用 64 KB 簇时, 最大 NTFS 卷为 256 TB 减 64 KB. 使用默认 4 KB 簇时, 最大 NTFS 卷为 16 TB 减 4 KB.

**Note**

- 若 NTFS 文件夹内文件数量很大 (30 万或更多), 应禁用短文件名生成以提升性能, 尤其当长文件名前六个字符相似时.

下表 **NTFS Size Limits** 列出 NTFS 尺寸上限.

**NTFS Size Limits**

| Description | Limit |
| --- | --- |
| Maximum file size | Architecturally: 16 exabytes minus 1 KB (2^64 bytes minus 1 KB) Implementation: 16 terabytes minus 64 KB (2^44 bytes minus 64 KB) |
| Maximum volume size | Architecturally: 2^64 clusters minus 1 cluster Implementation: 256 terabytes minus 64 KB (2^32 clusters minus 1 cluster) |
| Files per volume | 4,294,967,295 (2^32 minus 1 file) |

#### MBR 与 GUID 磁盘上的分区表

主引导记录 (MBR) 磁盘同时使用基本卷与动态卷. 由于 MBR 磁盘分区表仅支持最大 2 TB 的分区, 要创建超过 2 TB 的 NTFS 卷必须使用动态卷. Windows Server 2003 在专用数据库而非分区表中管理动态卷; 因此动态卷不受分区表 2 TB 物理限制. 动态 NTFS 卷可大到 NTFS 所支持的最大卷大小. 使用 GUID 分区表 (GPT) 磁盘的 Itanium 计算机同样支持大于 2 TB 的 NTFS 卷.

### NTFS 卷的组织结构

图 **Organization of an NTFS Volume** 示意 NTFS 如何在卷上组织各类结构.

**Organization of an NTFS Volume**

![Organization of an NTFS Volume](images/cc781134.737c1f18-1bbc-45c7-9cb7-d61387d78324(ws.10).gif)

下表说明 NTFS 卷上各组织结构的含义.

**NTFS Volume Components**

| Component | Description |
| --- | --- |
| NTFS Boot Sector | 含 BIOS parameter block (BPB), 存储卷布局与文件系统结构信息, 以及加载 Windows Server 2003 的引导代码. |
| Master File Table | 含从 NTFS 分区检索文件所需的信息, 例如文件属性. |
| File System Data | 存储不在 Master File Table 内的数据. |
| Master File Table Copy | 含原始副本出现问题时恢复文件系统所必需记录的副本. |

### Boot Sectors

在 MBR 磁盘上, 位于每个分区第一个逻辑扇区的 boot sector 是启动计算机的关键磁盘结构. 它含可执行代码及代码所需数据, 包括文件系统访问卷所需信息. 格式化卷时创建 boot sector. boot sector 末尾为 2 字节结构, 称 signature word 或 end of sector marker, 恒为 0x55AA. 在运行 Windows Server 2003 的计算机上, 活动分区的 boot sector 载入内存并启动 Ntldr; 若安装多个 Windows 版本则加载启动菜单, 若仅一个操作系统则加载该操作系统.

GUID 分区表 (GPT) 磁盘与 MBR 磁盘类似, 但使用位于磁盘开头与末尾的主/备份分区结构提供冗余. GPT 以逻辑块地址 (LBA) 而非相对扇区标识这些结构.

boot sector 由下列元素组成:

- 基于 x86 的 CPU jump instruction.
- Original equipment manufacturer identification (OEM ID).
- BIOS parameter block (BPB), 一种数据结构.
- Extended BPB.
- 启动操作系统的可执行 boot code (或 bootstrap code).

无论基本磁盘还是动态磁盘, 所有 Windows Server 2003 boot sector 均含上述元素.

#### Boot Sector 的组成部分

MBR 将 CPU 执行转交给 boot sector, 因此 boot sector 前三个字节必须是有效的、可执行的基于 x86 的 CPU 指令. 其中包括跳过其后若干不可执行字节的 jump instruction.

jump instruction 之后为 8 字节 OEM ID, 即标识格式化该卷的操作系统名称与版本号的字符串. 为保持与 MS-DOS 兼容, Windows Server 2003 在此字段写入 `"NTFS    "` (含尾部空格).

**Note**

- 在 Windows 95 格式化的磁盘上可能看到 OEM ID `"MSWIN4.0"`, 在 Windows 95 OEM Service Release 2 (OSR2)、Windows 98 与 Windows Millennium Edition 格式化的磁盘上可能看到 `"MSWIN4.1"`. Windows Server 2003 除验证 NTFS 卷外不使用 boot sector 中的 OEM ID 字段.

OEM ID 之后为 BPB, 提供使可执行 boot code 定位 Ntldr 所需的信息. BPB 始终起始于相同偏移, 因此标准参数位于已知位置. 磁盘大小与几何变量封装在 BPB 中. 由于 boot sector 前半为 x86 jump instruction, 未来可在 BPB 末尾追加新信息以扩展 BPB, jump instruction 仅需小幅调整. BPB 以 packed (未对齐) 格式存储.

#### NTFS Boot Sector

下表 **Boot Sector Sections on an NTFS Volume** 说明 NTFS 格式化卷的 boot sector. 格式化 NTFS 卷时, 格式化程序为 boot sector 与 bootstrap code 分配前 16 个扇区.

**Boot Sector Sections on an NTFS Volume**

| Byte Offset | Field Length | Field Name |
| --- | --- | --- |
| 0x00 | 3 bytes | Jump instruction |
| 0x03 | 8 bytes | OEM ID |
| 0x0B | 25 bytes | BPB |
| 0x24 | 48 bytes | Extended BPB |
| 0x54 | 426 bytes | Bootstrap code |
| 0x01FE | 2 bytes | End of sector marker |

在 NTFS 卷上, BPB 之后的字段构成 extended BPB. 这些字段中的数据使 Ntldr 在启动时能找到 MFT. 在 NTFS 卷上, MFT 不在预定义扇区; 因此若当前 MFT 位置出现坏扇区, NTFS 可移动 MFT. 然而, 若数据损坏则无法定位 MFT, Windows Server 2003 将认为该卷未格式化.

下列示例展示 Windows Server 2003 格式化的 NTFS 卷 boot sector. 打印输出分三段:

- 字节 0x00–0x0A 为 jump instruction 与 OEM ID (粗体).
- 字节 0x0B–0x53 为 BPB 与 extended BPB.
- 其余为 bootstrap code 与 end of sector marker (粗体).

```
Physical Sector: Cyl 0, Side 1, Sector 1
00000000: EB 52 90 4E 54 46 53 20 - 20 20 20 00 02 08 00 00 .R.NTFS ..... ..
00000010: 00 00 00 00 00 F8 00 00 - 3F 00 FF 00 3F 00 00 00 ........?...?...
00000020: 00 00 00 00 80 00 80 00 - 1C 91 11 01 00 00 00 00 ................
00000030: 00 00 04 00 00 00 00 00 - 11 19 11 00 00 00 00 00 ................
00000040: F6 00 00 00 01 00 00 00 - 3A B2 7B 82 CD 7B 82 14 ........:.{..{..
00000050: 00 00 00 00 FA 33 C0 8E - D0 BC 00 7C FB B8 C0 07 .....3.....|....
```

下表 **BPB and Extended BPB Fields on NTFS Volumes** 说明 NTFS 卷 BPB 与 extended BPB 各字段. 起始于 0x0B、0x0D、0x15、0x18、0x1A 与 0x1C 的字段与 FAT16、FAT32 卷相同. 示例值对应该示例中的数据 (布局对齐 `pfs_ntfs_raw.h` / `ntfs_boot_sector_t`).

**BPB and Extended BPB Fields on NTFS Volumes**

| Byte Offset | Field Length | Sample Value | Field Name and Definition |
| --- | --- | --- | --- |
| 0x0B | 2 bytes | 00 02 | **Bytes Per Sector**. 硬件扇区大小. 美国常用磁盘此字段多为 512. |
| 0x0D | 1 byte | 08 | **Sectors Per Cluster**. 每簇扇区数. |
| 0x0E | 2 bytes | 00 00 | **Reserved Sectors**. NTFS 将 boot sector 置于分区开头, 故恒为 0. 若非 0, NTFS 无法挂载该卷. |
| 0x10 | 3 bytes | 00 00 00 | 必须为 0, 否则 NTFS 无法挂载该卷. |
| 0x13 | 2 bytes | 00 00 | 必须为 0, 否则 NTFS 无法挂载该卷. |
| 0x15 | 1 byte | F8 | **Media Descriptor**. 提供介质信息. F8 表示硬盘, F0 表示高密度 3.5 英寸软盘. 介质描述符条目源自 MS-DOS FAT16, Windows Server 2003 不再使用. |
| 0x16 | 2 bytes | 00 00 | 必须为 0, 否则 NTFS 无法挂载该卷. |
| 0x18 | 2 bytes | 3F 00 | NTFS 不使用也不检查. |
| 0x1A | 2 bytes | FF 00 | NTFS 不使用也不检查. |
| 0x1C | 4 bytes | 3F 00 00 00 | NTFS 不使用也不检查. |
| 0x20 | 4 bytes | 00 00 00 00 | 必须为 0, 否则 NTFS 无法挂载该卷. |
| 0x24 | 4 bytes | 80 00 80 00 | NTFS 不使用也不检查. |
| 0x28 | 8 bytes | 1C 91 11 01 00 00 00 00 | **Total Sectors** (`number_of_sectors`). 硬盘扇区总数. |
| 0x30 | 8 bytes | 00 00 04 00 00 00 00 00 | **Logical Cluster Number for the File $MFT** (`mft_lcn`). 以 LCN 标识 MFT 位置. |
| 0x38 | 8 bytes | 11 19 11 00 00 00 00 00 | **Logical Cluster Number for the File $MftMirr** (`mftmirr_lcn`). 以 LCN 标识 MFT 镜像副本位置. |
| 0x40 | 1 byte | F6 | **Clusters Per MFT Record** (`clusters_per_mft_record`). 每条 MFT 记录的大小. NTFS 为卷上创建的每个文件生成 file record, 为每个文件夹生成 folder record. 小于该大小的文件与文件夹内容驻留在 MFT 内. 若该数为正 (最大 7F), 表示每条 MFT 记录占用的簇数. 若为负 (80 至 FF), 则记录大小为 2 的该数绝对值次方字节. |
| 0x41 | 3 bytes | 00 00 00 | NTFS 不使用. |
| 0x44 | 1 byte | 01 | **Clusters Per Index Record** (`clusters_per_index_record`). 每个 index block 的大小, 用于为目录分配空间. 编码规则同 **Clusters Per MFT Record**. |
| 0x45 | 3 bytes | 00 00 00 | NTFS 不使用. |
| 0x48 | 8 bytes | 3A B2 7B 82 CD 7B 82 14 | **Volume Serial Number** (`volume_serial_number`). 卷序列号. |
| 0x50 | 4 bytes | 00 00 00 00 | NTFS 不使用 (部分布局中此处为 **Checksum**). |

### Master File Table

用 NTFS 格式化卷时, Windows Server 2003 在分区上创建 MFT 与元数据文件. MFT 是关系型数据库, 由 file record 行与 file attribute 列组成. 它至少包含 NTFS 卷上每个文件的一条记录, 包括 MFT 自身.

MFT 存储从 NTFS 分区检索文件所需的信息.

#### MFT 与元数据文件

由于 MFT 存储关于自身的信息, NTFS 保留 MFT 前 16 条记录 (约 16 KB) 给元数据文件, 用于描述 MFT. 以美元符号 ($) 开头的元数据文件见下表 **Metadata Files Stored in the MFT**. MFT 其余记录包含卷上各文件与文件夹的 file/folder record.

**Metadata Files Stored in the MFT**

| System File | File Name | MFT Record | Purpose of the File |
| --- | --- | --- | --- |
| Master file table | $Mft | 0 | 含 NTFS 卷上每个文件与文件夹的一条 base file record. 若文件或文件夹的分配信息过大无法放入单条记录, 则再分配其他 file record. |
| Master file table mirror | $MftMirr | 1 | 保证单扇区故障时仍可访问 MFT. 为 MFT 前四条记录的 duplicate image. |
| Log file | $LogFile | 2 | 含 NTFS 用于更快可恢复性的信息. Windows Server 2003 用 log file 在系统故障后恢复 NTFS 元数据一致性. log file 大小取决于卷大小, 可用 Chkdsk 增大. |
| Volume | $Volume | 3 | 含卷信息, 如卷标与卷版本. |
| Attribute definitions | $AttrDef | 4 | 列出 attribute 名称, 编号与描述. |
| Root file name index | . | 5 | 根目录. |
| Cluster bitmap | $Bitmap | 6 | 以空闲与已用簇表示整个卷. |
| Boot sector | $Boot | 7 | 含挂载卷所用的 BPB 以及卷可引导时的额外 bootstrap loader code. |
| Bad cluster file | $BadClus | 8 | 含卷上的坏簇. |
| Security file | $Secure | 9 | 含卷内所有文件的唯一 security descriptor. |
| Upcase table | $Upcase | 10 | 将小写字符转换为匹配的 Unicode 大写字符. |
| NTFS extension file | $Extend | 11 | 用于配额, reparse point 数据, object identifier 等可选扩展. |
| | | 12–15 | 保留供将来使用. |

$Mft 与备份 MFT $MftMirr 的数据段位置均记录在 boot sector 中. $MftMirr 是 $Mft 前四条记录或 $Mft 第一个簇 (取较大者) 的 duplicate image. 若镜像范围内任一 MFT record 损坏或不可读, NTFS 读取 boot sector 以定位 $MftMirr, 然后用 $MftMirr 中的信息替代 MFT 中对应信息. 若可能, 将 $MftMirr 中的正确数据写回 MFT 相应位置.

#### MFT Zone

为防止 MFT 碎片化, NTFS 默认保留卷容量的 12.5% 专供 MFT 使用. 该空间称 MFT zone, 除非卷其余部分已满, 否则不用于存储普通数据.

依平均文件大小等因素, 当卷接近满时, MFT zone 或未保留空间会先耗尽.

- 含少量大文件的卷会先耗尽未保留空间.
- 含大量小文件的卷会先耗尽 MFT zone 空间.

无论哪种情况, 当某一区域满时 MFT 会发生碎片化. 可为新创建卷更改 MFT zone 占卷百分比. MFT zone 大小设置如下:

- 设置 1 (默认) 约保留 12.5% 卷容量.
- 设置 2 约保留 25%.
- 设置 3 约保留 37.5%.
- 设置 4 约保留 50%.

多数计算机默认设置 1 已足够, 适用于平均文件大小 8 KB 的卷. 若存储大量更小文件, 可能需要为新卷增大 MFT zone.

增大 MFT zone 后, NTFS 不会立即分配空间以容纳新 zone 大小, 而是先耗尽原保留空间再扩展 MFT zone. 原空间耗尽后, NTFS 寻找足够大的下一连续空间以容纳额外 MFT zone, 可能导致 MFT 碎片化. 若默认值不合适, 可调整 MFT zone 大小.

### NTFS File Record Attributes

NTFS 卷上每个已分配扇区都属于某个文件. 甚至文件系统元数据也是文件的一部分. NTFS 将每个文件 (或文件夹) 视为一组 file attribute. 文件名, 安全信息乃至数据本身都是 file attribute. 每个 attribute 由 attribute type code 与可选 attribute name 标识.

file record 与 folder record 各 1 KB, 存储在 MFT 中, 其 attribute 写入 MFT 中已分配空间. 除 file attribute 外, 每条 file record 还含该记录在 MFT 中位置的信息.

当文件 attribute 能放入该文件的 MFT file record 时, 称为 resident attribute. 文件名与时间戳等 attribute 始终为 resident. 当文件信息量超出其 MFT file record 时, 部分 attribute 变为 nonresident. nonresident attribute 分配一个或多个磁盘簇. nonresident attribute 的一部分仍留在 MFT 中并指向外部簇. NTFS 创建 Attribute List attribute 以描述全部 attribute record 的位置. 下表 **NTFS File Attribute Types** 列出 NTFS 当前定义的 file attribute (类型码对齐 `NTFS_AT_*`).

**NTFS File Attribute Types**

| Attribute Type | Description |
| --- | --- |
| Standard Information | 访问模式 (只读, 读写等), 时间戳与 link count 等信息. |
| Attribute List | 无法放入 MFT record 的全部 attribute record 的位置. |
| File Name | 长文件名与短文件名的可重复 attribute. 长文件名最多 255 个 Unicode 字符. 短文件名为 8.3 不区分大小写名. POSIX 所需的额外 hard link 名可作为额外 File Name attribute. |
| Data | 文件数据. NTFS 支持每文件多个 Data attribute. 典型文件有一个未命名 Data attribute, 也可有一个或多个命名 Data attribute. |
| Object ID | 卷内唯一的 file identifier. 供 distributed link tracking 使用. 并非所有文件都有 object identifier. |
| Logged Utility Stream | 类似 data stream, 但操作像 NTFS 元数据变更一样记入 NTFS log file. EFS 使用此 attribute. |
| Reparse Point | 用于 mounted drive. 亦供 Installable File System (IFS) filter driver 将特定文件标记为该驱动专用. |
| Index Root | 实现文件夹与其他 index. |
| Index Allocation | 实现大文件夹与其他大 index 的 B-tree 结构. |
| Bitmap | 实现大文件夹与其他大 index 的 B-tree 结构. |
| Volume Information | 仅用于 $Volume 系统文件. 含卷版本. |

NTFS 为卷上创建的每个文件生成 file record, 为每个文件夹生成 folder record. MFT 亦含 MFT 自身的单独 file record. 这些 file/folder record 各 1 KB, 存储在 MFT 中, attribute 写入 MFT 已分配空间. 除 file attribute 外, 每条 record 还含其在 MFT 中的位置. 图 **MFT Entry with Resident Record** 展示小文件或文件夹的 MFT record 内容. 小文件与文件夹 (通常 900 字节或更小) 完全包含在其 MFT record 内.

**MFT Entry with Resident Record**

![MFT Entry with Resident Record](images/cc781134.86787c15-cf0a-4cb9-8ba1-ff1afd37aaf5(ws.10).gif)

通常每个文件使用一条 file record. 然而, 若文件 attribute 很多或高度碎片化, 可能需要多条 file record. 此时该文件的第一条 record, 即 base file record, 存储其余所需 file record 的位置.

folder record 含 index 信息. 小 folder record 完全位于 MFT 结构内, 大文件夹则组织为 B-tree, 其 record 含指向外部簇的指针, 存放无法放入 MFT 的 folder entry.

使用 B-tree 的好处在大文件夹枚举时明显: NTFS 可对相似文件名分组或 index, 仅搜索含目标文件的组, 减少定位特定文件所需的磁盘访问, 尤其对大文件夹. 因此 NTFS 在大文件夹上性能优于 FAT, 因 FAT 须在大文件夹中扫描全部文件名才能列出所有文件.

#### Last Access Time

NTFS 卷上每个文件与文件夹均含 Last Access Time attribute, 表示上次访问时间, 例如用户列出文件夹, 向文件夹添加文件, 读取文件或修改文件时. 最新的 Last Access Time 始终保存在内存中, 最终写入磁盘两处:

- 文件的 attribute, 即其 MFT record 的一部分.
- 文件的 directory entry. directory entry 存储于包含该文件的文件夹中. 具有多个 hard link 的文件有多个 directory entry.

磁盘上的 Last Access Time 并非始终最新, 因为 NTFS 在强制将 Last Access Time 更新写入磁盘前会等待约一小时. 当用户或程序对文件或文件夹执行只读操作 (如列出文件夹内容或读取但不修改文件) 时, NTFS 亦延迟写入 Last Access Time. 若每次读操作都更新磁盘上的 Last Access Time, 则所有读操作都变成写操作, 影响 NTFS 性能.

**Note**

- 基于文件的 Last Access Time 查询即使磁盘值未全部最新仍然准确, 因为准确值保存在内存中.

NTFS 最终按下列方式将内存中的 Last Access Time 写入磁盘.

##### 在文件的 attribute 内

若内存中当前 Last Access Time 与磁盘存储值相差超过一小时, 或该文件全部内存引用消失 (以较晚者为准), NTFS 通常更新磁盘上的文件 attribute. 例如, 若文件当前 Last Access Time 为下午 1:00, 下午 1:30 读取该文件, NTFS 不更新 Last Access Time. 若下午 2:00 再次读取, NTFS 将 attribute 中的 Last Access Time 更新为 2:00, 因为 attribute 显示 1:00 而内存中为 2:00.

##### 在文件的 directory entry 内

NTFS 在下列事件时更新文件的 directory entry:

- NTFS 更新文件 Last Access Time 并发现与 directory entry 中存储值相差超过一小时. 这通常发生在程序关闭用于访问该目录内文件的句柄之后. 若程序长时间保持句柄打开, directory entry 中的变更会有延迟.
- NTFS 更新其他文件 attribute (如 Last Modify Time) 且 Last Access Time 更新尚 pending. 此时 NTFS 与其他更新一并写入 Last Access Time, 无额外性能开销.

**Note**

- 当文件全部内存引用消失时, NTFS 不更新其 directory entry.

若 NTFS 卷含大量文件夹或文件, 且有程序依次短暂访问其中每一项, 生成 Last Access Time 更新所用的 I/O 带宽可能占总体 I/O 带宽的显著比例.

#### Multiple Data Streams

data stream 是字节序列. 应用程序在 stream 内特定偏移写入数据, 读取时在相同偏移读取. 无论何种文件系统, 每个文件都有一个主 unnamed stream.

然而, NTFS 还支持额外的 named data stream; 每个 data stream 是 alternate 字节序列, 如图 **Unnamed and Named Streams** 所示. 应用程序可创建额外 named stream 并通过名称访问. 该特性允许将相关数据作为单一单元管理. 例如, 图形程序可在含图像的 NTFS 文件内以 named data stream 存储 bitmap 缩略图.

**Unnamed and Named Streams**

![Unnamed and Named Streams](images/cc781134.71c1e60b-b8a4-4e65-8bfc-a50995dbcfa8(ws.10).gif)

FAT 卷仅支持主 unnamed stream, 因此若尝试将 Streamexample.doc 复制或移动到 FAT 卷或软盘, 会收到错误消息.
