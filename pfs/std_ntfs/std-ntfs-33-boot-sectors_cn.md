> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: `portfs/doc/std_ntfs/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

## NTFS 物理结构

### 引导扇区 (Boot Sectors)

在 MBR 磁盘上, 引导扇区位于每个分区的第一个逻辑扇区, 是启动计算机的关键磁盘结构. 它含可执行代码以及代码所需的数据, 包括文件系统访问卷所需的信息. 引导扇区在格式化卷时创建. 引导扇区末尾是 2 字节结构, 称 signature word 或 end of sector marker (扇区结束标记), 始终设为 0x55AA. 在运行 Windows Server 2003 的计算机上, 活动分区的引导扇区加载到内存并启动 Ntldr; 若安装多个 Windows 版本则加载引导菜单, 若仅安装一个操作系统则直接加载操作系统.

GUID 分区表 (GPT) 磁盘与 MBR 磁盘类似, 但使用位于磁盘开头与末尾的主分区结构与备份分区结构以提供冗余. GPT 以逻辑块地址 (LBA) 而非相对扇区标识这些结构.

引导扇区由下列元素组成:

- 基于 x86 的 CPU 跳转指令 (jump instruction).
- 原始设备制造商标识 (OEM ID).
- BIOS parameter block (BPB), 一种数据结构.
- Extended BPB (扩展 BPB).
- 启动操作系统的可执行引导代码 (bootstrap code).

所有 Windows Server 2003 引导扇区均含上述元素, 无论磁盘类型 (基本磁盘或动态磁盘).

#### 引导扇区组件

MBR 将 CPU 执行转交给引导扇区, 因此引导扇区前 3 字节必须是有效的、可执行的 x86 CPU 指令. 其中包括跳过随后若干不可执行字节的跳转指令.

跳转指令之后是 8 字节 OEM ID, 标识格式化该卷的操作系统名称与版本号的字符串. 为保持与 MS-DOS 兼容, Windows Server 2003 在此字段记录 `"NTFS"`.

**注意**

- 在 Windows 95 格式化的磁盘上也可能看到 OEM ID `"MSWIN4.0"`, 在 Windows 95 OEM Service Release 2 (OSR2)、Windows 98 与 Windows Millennium Edition 格式化的磁盘上可能看到 `"MSWIN4.1"`. Windows Server 2003 除验证 NTFS 卷外不使用引导扇区中的 OEM ID 字段.

OEM ID 之后是 BPB, 提供使可执行引导代码定位 Ntldr 所需的信息. BPB 始终起始于相同偏移, 因此标准参数位于已知位置. 磁盘大小与几何变量封装在 BPB 中. 由于引导扇区第一部分是 x86 跳转指令, 未来可在 BPB 末尾追加新信息以扩展 BPB. 跳转指令只需小幅调整即可适应此变更. BPB 以 packed (未对齐) 格式存储.

#### NTFS Boot Sector (NTFS 引导扇区)

表 Boot Sector Sections on an NTFS Volume 描述格式化为 NTFS 的卷的引导扇区. 格式化 NTFS 卷时, 格式化程序为引导扇区与 bootstrap code 分配前 16 个扇区.

on-disk 布局见 `ntfs_boot_sector` (`fs/ntfs/layout.h`, `pfs_ntfs_raw.h`).

**Boot Sector Sections on an NTFS Volume (NTFS 卷引导扇区区段)**

| 字节偏移 | 字段长度 | 字段名 |
| --- | --- | --- |
| 0x00 | 3 字节 | Jump instruction (跳转指令) |
| 0x03 | 8 字节 | OEM ID |
| 0x0B | 25 字节 | BPB |
| 0x24 | 48 字节 | Extended BPB |
| 0x54 | 426 字节 | Bootstrap code |
| 0x01FE | 2 字节 | End of sector marker (扇区结束标记) |

在 NTFS 卷上, BPB 之后的字段构成 Extended BPB. 这些字段中的数据使 Ntldr 能在启动时找到 MFT. 在 NTFS 卷上, MFT 不位于预定义扇区. 因此, 若 MFT 当前位置出现坏扇区, NTFS 可移动 MFT. 然而, 若数据损坏导致无法定位 MFT, Windows Server 2003 会假定该卷尚未格式化.

下列示例展示使用 Windows Server 2003 格式化的 NTFS 卷的引导扇区. 打印输出分三段:

- 字节 0x00–0x0A 为跳转指令与 OEM ID (粗体).
- 字节 0x0B–0x53 为 BPB 与 Extended BPB.
- 其余为 bootstrap code 与扇区结束标记 (粗体).

```
Physical Sector: Cyl 0, Side 1, Sector 1
00000000: EB 52 90 4E 54 46 53 20 - 20 20 20 00 02 08 00 00 .R.NTFS ..... ..
00000010: 00 00 00 00 00 F8 00 00 - 3F 00 FF 00 3F 00 00 00 ........?...?...
00000020: 00 00 00 00 80 00 80 00 - 1C 91 11 01 00 00 00 00 ................
00000030: 00 00 04 00 00 00 00 00 - 11 19 11 00 00 00 00 00 ................
00000040: F6 00 00 00 01 00 00 00 - 3A B2 7B 82 CD 7B 82 14 ........:.{..{..
00000050: 00 00 00 00 FA 33 C0 8E - D0 BC 00 7C FB B8 C0 07 .....3.....|....
```

表 BPB and Extended BPB Fields on NTFS Volumes 描述 NTFS 卷上 BPB 与 Extended BPB 的字段. 起始于 0x0B、0x0D、0x15、0x18、0x1A 与 0x1C 的字段与 FAT16、FAT32 卷相同. 示例值对应本例数据.

**BPB and Extended BPB Fields on NTFS Volumes (NTFS 卷 BPB 与 Extended BPB 字段)**

| 字节偏移 | 字段长度 | 示例值 | 字段名与定义 |
| --- | --- | --- | --- |
| 0x0B | 2 字节 | 00 02 | **Bytes Per Sector** (每扇区字节数). 硬件扇区大小. 美国常用磁盘此字段值为 512. |
| 0x0D | 1 字节 | 08 | **Sectors Per Cluster** (每簇扇区数). 一个簇中的扇区数. |
| 0x0E | 2 字节 | 00 00 | **Reserved Sectors** (保留扇区数). NTFS 始终为 0, 因为引导扇区位于分区起始. 若非 0, NTFS 无法挂载卷. |
| 0x10 | 3 字节 | 00 00 00 | 值必须为 0, 否则 NTFS 无法挂载卷. |
| 0x13 | 2 字节 | 00 00 | 值必须为 0, 否则 NTFS 无法挂载卷. |
| 0x15 | 1 字节 | F8 | **Media Descriptor** (介质描述符). 提供所用介质信息. F8 表示硬盘, F0 表示高密度 3.5 英寸软盘. 介质描述符项是 MS-DOS FAT16 磁盘的遗留, Windows Server 2003 不使用. |
| 0x16 | 2 字节 | 00 00 | 值必须为 0, 否则 NTFS 无法挂载卷. |
| 0x18 | 2 字节 | 3F 00 | NTFS 不使用或检查. |
| 0x1A | 2 字节 | FF 00 | NTFS 不使用或检查. |
| 0x1C | 4 字节 | 3F 00 00 00 | NTFS 不使用或检查. |
| 0x20 | 4 字节 | 00 00 00 00 | 值必须为 0, 否则 NTFS 无法挂载卷. |
| 0x24 | 4 字节 | 80 00 80 00 | NTFS 不使用或检查. |
| 0x28 | 8 字节 | 1C 91 11 01 00 00 00 00 | **Total Sectors** (总扇区数, `number_of_sectors`). 硬盘上的扇区总数. |
| 0x30 | 8 字节 | 00 00 04 00 00 00 00 00 | **Logical Cluster Number for the File $MFT** (`mft_lcn`). 以 LCN 标识 MFT 位置. |
| 0x38 | 8 字节 | 11 19 11 00 00 00 00 00 | **Logical Cluster Number for the File $MftMirr** (`mftmirr_lcn`). 以 LCN 标识 MFT 镜像拷贝位置. |
| 0x40 | 1 字节 | F6 | **Clusters Per MFT Record** (`clusters_per_mft_record`). 每条 MFT 记录的大小. NTFS 为卷上创建的每个文件生成 file record, 为每个文件夹生成 folder record. 小于此大小的文件与文件夹包含在 MFT 内. 若此数为正 (最大 7F), 表示每条 MFT 记录的簇数. 若为负 (80 至 FF), 则 file record 大小为 2 的该数绝对值次方. |
| 0x41 | 3 字节 | 00 00 00 | NTFS 不使用. |
| 0x44 | 1 字节 | 01 | **Clusters Per Index Buffer** (`clusters_per_index_record`). 每个 index buffer 的大小, 用于为目录分配空间. 若此数为正 (最大 7F), 表示每个 index buffer 的簇数. 若为负 (80 至 FF), 则 index record 大小为 2 的该数绝对值次方. |
| 0x45 | 3 字节 | 00 00 00 | NTFS 不使用. |
| 0x48 | 8 字节 | 3A B2 7B 82 CD 7B 82 14 | **Volume Serial Number** (`volume_serial_number`). 卷的序列号. |
| 0x50 | 4 字节 | 00 00 00 00 | NTFS 不使用. |
