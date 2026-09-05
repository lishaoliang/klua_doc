> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

# Boot Sector | Microsoft Learn

位于每个卷扇区 1 的 boot sector 是启动计算机的关键磁盘结构. 它含可执行代码及代码所需数据, 包括文件系统访问卷所需信息. 格式化卷时创建 boot sector. boot sector 末尾为 2 字节结构, 称 signature word 或 end of sector marker, 恒为 0x55AA. 在运行 Windows 2000 的计算机上, 活动分区的 boot sector 载入内存并启动 Ntldr, 后者加载操作系统.

Windows 2000 boot sector 由下列元素组成:

- 基于 x86 的 CPU jump instruction.
- Original equipment manufacturer identification (OEM ID).
- BIOS parameter block (BPB), 一种数据结构.
- Extended BPB.
- 启动操作系统的可执行 boot code (或 bootstrap code).

![note-icon](images/cc938288.reskit_note(en-us,technet.10).gif)

**Note**

所有 Windows 2000 boot sector 均含这些元素. 然而 NTFS, FAT16 与 FAT32 的 boot sector 格式各不相同.

BPB 描述卷的物理参数; extended BPB 紧接 BPB 之后. 因字段类型与数据量不同, FAT16, FAT32 与 NTFS boot sector 的 BPB 长度不同.

BPB 与 extended BPB 中的信息供磁盘设备驱动读写与配置卷. extended BPB 之后区域通常含可执行 boot code, 执行继续启动过程所需操作.

### Boot Sector 启动过程

计算机在启动时使用 boot sector 运行指令. 初始启动过程概括如下:

1. 系统 BIOS 与 CPU 发起 power-on self test (POST).
2. BIOS 搜索 boot device (通常为磁盘).
3. BIOS 将 boot device 的第一个物理扇区载入内存并将 CPU 执行转到该内存地址.

若 boot device 在硬盘上, BIOS 载入 MBR. MBR 中的 master boot code 载入 active partition 的 boot sector 并将 CPU 执行转到该地址. 在运行 Windows 2000 的计算机上, boot sector 中的可执行 boot code 找到 Ntldr, 将其载入内存并将执行转交给该文件.

![note-icon](images/cc938288.reskit_note(en-us,technet.10).gif)

**Note**

Windows 2000 无法从运行 dynamic disk 的 spanned, striped 或 RAID-5 卷启动. 这些磁盘结构无法登记到 MBR 的 partition table, 因此使用这些结构的系统分区不可启动. Windows 2000 须完全载入内存后才能使用这些结构.

若驱动器 A 中有软盘, 系统 BIOS 将磁盘第一个扇区 (boot sector) 载入内存. 若磁盘可启动 (由 MS-DOS 格式化并应用核心操作系统文件), boot sector 载入内存并用可执行 boot code 将 CPU 执行转交给 Io.sys (MS-DOS 核心文件). 若软盘不可引导, 可执行 boot code 显示如下错误消息:

`Non-System disk or disk error`

`Replace and press any key when ready`

![note-icon](images/cc938288.reskit_note(en-us,technet.10).gif)

**Note**

配置为先在驱动器 C 查找启动文件的正常系统不会出现此错误. 许多计算机可在 CMOS setup 中设置系统搜索启动文件的磁盘顺序.

若从硬盘启动时出现类似错误, boot sector 可能已损坏. 有关 boot sector 问题故障排除的更多信息, 见本章后文 "Damaged MBRs and Boot Sectors".

初始启动过程与磁盘格式及操作系统无关. 当 boot sector 的可执行 boot code 开始运行时, 操作系统与文件系统的特性才变得重要.

### Boot Sector 的组成部分

MBR 将 CPU 执行转交给 boot sector, 因此 boot sector 前三个字节必须是有效的、可执行的基于 x86 的 CPU 指令. 其中包括跳过其后若干不可执行字节的 jump instruction.

jump instruction 之后为 8 字节 OEM ID, 即标识格式化该卷的操作系统名称与版本号的字符串. 为保持与 MS-DOS 兼容, Windows 2000 在 FAT16 与 FAT32 磁盘此字段写入 `"MSDOS5.0"`. 在 NTFS 磁盘上写入 `"NTFS    "`.

![note-icon](images/cc938288.reskit_note(en-us,technet.10).gif)

**Note**

在 Windows 95 格式化的磁盘上可能看到 OEM ID `"MSWIN4.0"`, 在 Windows 95 OSR2 与 Windows 98 格式化的磁盘上可能看到 `"MSWIN4.1"`. Windows 2000 除验证 NTFS 卷外不使用 boot sector 中的 OEM ID 字段.

OEM ID 之后为 BPB, 提供使可执行 boot code 定位 Ntldr 所需的信息. BPB 始终起始于相同偏移, 因此标准参数位于已知位置. 磁盘大小与几何变量封装在 BPB 中. 由于 boot sector 前半为 x86 jump instruction, 未来可在 BPB 末尾追加新信息以扩展 BPB, jump instruction 仅需小幅调整. BPB 以 packed (未对齐) 格式存储.

### FAT16 Boot Sector

表 1.6 说明 FAT16 文件系统格式化卷的 boot sector.

**Table 1.6 Boot Sector Sections on a FAT16 Volume**

| Byte Offset | Field Length | Field Name |
| --- | --- | --- |
| 0x00 | 3 bytes | Jump Instruction |
| 0x03 | LONGLONG | OEM ID |
| 0x0B | 25 bytes | BPB |
| 0x24 | 26 bytes | Extended BPB |
| 0x3E | 448 bytes | Bootstrap Code |
| 0x01FE | WORD | End of Sector Marker |

下列示例展示 FAT16 卷 boot sector 的十六进制打印. 打印输出分三段:

- 字节 0x00–0x0A 为 jump instruction 与 OEM ID (粗体).
- 字节 0x0B–0x3D 为 BPB 与 extended BPB.
- 其余为 bootstrap code 与 end of sector marker (粗体).

`Physical Sector: Cyl 0, Side 1, Sector 1`

`00000000: EB 3C 90 4D 53 44 4F 53 - 35 2E 30 00 02 40 01 00 .<.MSDOS5.0 ..@..`

`00000010: 02 00 02 00 00 F8 FC 00 - 3F 00 40 00 3F 00 00 00 ........?.@.?...`

`00000020: 01 F0 3E 00 80 00 29 A8 - 8B 36 52 4E 4F 20 4E 41 ..>...)..6RNO NA`

`00000030: 4D 45 20 20 20 20 46 41 - 54 31 36 20 20 20 33 C0 ME FAT16 3.`

**`00000040: 8E D0 BC 00 7C 68 C0 07 - 1F A0 10 00 F7 26 16 00 ....|h......&..`**

(其余 bootstrap code 与 end of sector marker 见英文镜像原文 hex dump.)

表 1.7 与 1.8 说明 FAT16 卷 BPB 与 extended BPB 布局. 示例值对应前述示例数据.

**Table 1.7 BPB Fields for FAT16 Volumes**

| Byte Offset | Field Length | Value | Field Name and Definition |
| --- | --- | --- | --- |
| 0x0B | WORD | 0x0002 | **Bytes Per Sector**. 硬件扇区大小. 有效十进制值为 512, 1024, 2048 与 4096. 美国常用磁盘多为 512. |
| 0x0D | BYTE | 0x40 | **Sectors Per Cluster**. 每簇扇区数. FAT16 可跟踪的簇数有限 (最多 65,536), 大卷通过增大每簇扇区数支持. 默认簇大小取决于卷大小. 有效十进制值为 1, 2, 4, 8, 16, 32, 64 与 128. 导致簇大于 32 KB (**Bytes Per Sector** * **Sectors Per Cluster**) 的值可能引起磁盘与软件错误. |
| 0x0E | WORD | 0x0100 | **Reserved Sectors**. 第一个 FAT 之前的扇区数, 含 boot sector. 此字段值恒为 1. |
| 0x10 | BYTE | 0x02 | **Number of FATs**. 卷上 FAT 副本数. 此字段值恒为 2. |
| 0x11 | WORD | 0x0002 | **Root Entries**. 根目录可存储的 32 字节文件/文件夹名 entry 总数. 典型硬盘此值为 512. 一条 entry 用作 Volume Label, 长文件名文件/文件夹占用多条 entry. 最大 entry 数通常 511, 但使用长文件名时会在达到该数前耗尽. |
| 0x13 | WORD | 0x0000 | **Small Sectors**. 以 16 位表示的卷扇区数 (< 65,536). 大于 65,536 扇区的卷此字段为零并使用 **Large Sectors**. |
| 0x15 | BYTE | 0xF8 | **Media Descriptor**. 提供介质信息. 0xF8 表示硬盘, 0xF0 表示高密度 3.5 英寸软盘. 介质描述符条目源自 MS-DOS FAT16, Windows 2000 不再使用. |
| 0x16 | WORD | 0xFC00 | **Sectors Per FAT**. 卷上每个 FAT 占用的扇区数. 计算机用此数与 FAT 数量及 hidden sectors 确定根目录起始位置, 亦可根据根目录 entry 数 (512) 确定用户数据区起始. |
| 0x18 | WORD | 0x3F00 | **Sectors Per Track**. 低级格式化磁盘所用 apparent disk geometry 的一部分. |
| 0x1A | WORD | 0x4000 | **Number of Heads**. 低级格式化磁盘所用 apparent disk geometry 的一部分. |
| 0x1C | DWORD | 0x3F000000 | **Hidden Sectors**. boot sector 之前卷上的扇区数. 启动序列用此值计算根目录与数据区的绝对偏移. |
| 0x20 | DWORD | 0x01F03E00 | **Large Sectors**. 若 **Small Sectors** 为零, 此字段含 FAT16 卷扇区总数. 若 **Small Sectors** 非零, 此字段为零. |

**Table 1.8 Extended BPB Fields for FAT16 Volumes**

| Byte Offset | Field Length | Value | Field Name and Definition |
| --- | --- | --- | --- |
| 0x24 | BYTE | 0x80 | **Physical Drive Number**. 与 BIOS 物理驱动器号相关. 软盘驱动器为 0x00, 物理硬盘为 0x80, 与物理磁盘数量无关. 通常在发出 INT 13h BIOS 调用指定访问设备前设置. 仅当设备为 boot device 时相关. |
| 0x25 | BYTE | 0x00 | **Reserved**. FAT16 卷恒为零. |
| 0x26 | BYTE | 0x29 | **Extended Boot Signature**. 须为 0x28 或 0x29 方被 Windows 2000 识别. |
| 0x27 | DWORD | 0xA88B3652 | **Volume Serial Number**. 格式化时生成的随机序列号, 用于区分磁盘. |
| 0x2B | 11 bytes | NO NAME | **Volume Label**. 曾用于存储卷标. 卷标现作为根目录中的特殊文件存储. |
| 0x36 | LONGLONG | FAT16 | **File System Type**. 依磁盘格式为 FAT, FAT12 或 FAT16 的字段. |

### FAT32 Boot Sector

表 1.9 说明 FAT32 文件系统格式化卷的 boot sector.

![note-icon](images/cc938288.reskit_note(en-us,technet.10).gif)

**Note**

FAT32 boot sector 结构与 FAT16 非常相似, 但 FAT32 BPB 含额外字段. FAT32 extended BPB 使用与 FAT16 相同字段, 但在 boot sector 内偏移不同. FAT32 格式化的驱动器对不支持 FAT32 的操作系统不可读.

**Table 1.9 Boot Sector Sections on a FAT32 Volume**

| Byte Offset | Field Length | Field Name |
| --- | --- | --- |
| 0x00 | 3 bytes | Jump Instruction |
| 0x03 | LONGLONG | OEM ID |
| 0x0B | 53 bytes | BPB |
| 0x40 | 26 bytes | Extended BPB |
| 0x5A | 420 bytes | Bootstrap Code |
| 0x01FE | WORD | End of Sector Marker |

下列示例展示 FAT32 卷 boot sector 的十六进制打印. 打印输出分三段:

- 字节 0x00–0x0A 为 jump instruction 与 OEM ID (粗体).
- 字节 0x0B–0x59 为 BPB 与 extended BPB.
- 其余为 bootstrap code 与 end of sector marker (粗体).

`Physical Sector: Cyl 878, Side 0, Sector 1 `

`00000000: EB 58 90 4D 53 44 4F 53 - 35 2E 30 00 02 08 20 00 .X.MSDOS5.0 ... .`

`00000010: 02 00 00 00 00 F8 00 00 - 3F 00 FF 00 EE 39 D7 00 ........?....9..`

`00000020: 7F 32 4E 00 83 13 00 00 - 00 00 00 00 02 00 00 00 2N.............`

`00000030: 01 00 06 00 00 00 00 00 - 00 00 00 00 00 00 00 00 ................`

`00000040: 80 00 29 8B 93 6D 54 4E - 4F 20 4E 41 4D 45 20 20 ..)..mTNO NAME `

`00000050: 20 20 46 41 54 33 32 20 - 20 20 33 C9 8E D1 BC F4 FAT32 3.....`

(其余 bootstrap code 与 end of sector marker 见英文镜像原文 hex dump.)

表 1.10 与 1.11 说明 FAT32 卷 BPB 与 extended BPB 布局. 示例值对应前述示例数据.

**Table 1.10 BPB Fields for FAT32 Volumes**

| Byte Offset | Field Length | Value | Field Name and Definition |
| --- | --- | --- | --- |
| 0x0B | WORD | 0x0002 | **Bytes Per Sector**. 硬件扇区大小. 有效十进制值为 512, 1024, 2048 与 4096. 美国常用磁盘多为 512. |
| 0x0D | BYTE | 0x08 | **Sectors Per Cluster**. 每簇扇区数. FAT32 可跟踪的簇数有限 (最多 4,294,967,296), 极大卷通过增大每簇扇区数支持. 默认簇大小取决于卷大小. 有效十进制值为 1, 2, 4, 8, 16, 32, 64 与 128. Windows 2000 的 FAT32 实现仅允许创建最大 32 GB 的卷. 其他操作系统 (Windows 95 OSR2 及更高版本) 创建的更大卷在 Windows 2000 中可访问. |
| 0x0E | WORD | 0x0200 | **Reserved Sectors**. 第一个 FAT 之前的扇区数, 含 boot sector. 此字段十进制值通常为 32. |
| 0x10 | BYTE | 0x02 | **Number of FATs**. 卷上 FAT 副本数. 此字段值恒为 2. |
| 0x11 | WORD | 0x0000 | **Root Entries (FAT12/FAT16 only)**. FAT32 卷此字段必须为零. |
| 0x13 | WORD | 0x0000 | **Small Sectors (FAT12/FAT16 only)**. FAT32 卷此字段必须为零. |
| 0x15 | BYTE | 0xF8 | **Media Descriptor**. 提供介质信息. 0xF8 表示硬盘, 0xF0 表示高密度 3.5 英寸软盘. 介质描述符条目源自 MS-DOS FAT16, Windows 2000 不再使用. |
| 0x16 | WORD | 0x0000 | **Sectors Per FAT (FAT12/FAT16 only)**. FAT32 卷此字段必须为零. |
| 0x18 | WORD | 0x3F00 | **Sectors Per Track**. 含使用 INT 13h 的磁盘的 "sectors per track" 几何值. 卷由多磁头与柱面划分为 track. |
| 0x1A | WORD | 0xFF00 | **Number of Heads**. 含使用 INT 13h 的磁盘的 "count of heads" 几何值. 例如 1.44 MB 3.5 英寸软盘此值为 2. |
| 0x1C | DWORD | 0xEE39D700 | **Hidden Sectors**. boot sector 之前卷上的扇区数. 启动序列用此值计算根目录与数据区绝对偏移. 此字段通常仅对 interrupt 13h 可见介质相关. 未分区介质必须恒为零. |
| 0x20 | DWORD | 0x7F324E00 | **Large Sectors**. 含 FAT32 卷扇区总数. |
| 0x24 | DWORD | 0x83130000 | **Sectors Per FAT (FAT32 only)**. 卷上每个 FAT 占用的扇区数. 计算机用此数与 FAT 数量及 hidden sectors 确定根目录起始, 亦可根据根目录 entry 数确定用户数据区起始. |
| 0x28 | WORD | 0x0000 | **Extended Flags (FAT32 only)**. 此两字节的各位含义: Bits 0–3: active FAT 编号 (从 0 计数). 仅镜像禁用时有效. Bits 4–6: Reserved. Bit 7: 0 表示运行时将 FAT 镜像到全部 FAT; 1 表示仅一个 FAT active (由 bits 0-3 引用). Bits 8–15: Reserved. |
| 0x2A | WORD | 0x0000 | **File System Version (FAT32 only)**. 高字节为主版本号, 低字节为次版本号. 支持未来扩展 FAT32 介质类型同时考虑旧 FAT32 驱动挂载. 若非零, 旧版 Windows 不挂载该卷. |
| 0x2C | DWORD | 0x02000000 | **Root Cluster Number (FAT32 only)**. 根目录第一个簇的簇号. 此值通常为 2 但非总是. |
| 0x30 | WORD | 0x0100 | **File System Information Sector Number (FAT32 only)**. FAT32 卷保留区中 File System Information (FSINFO) 结构的扇区号. 值通常为 1. Backup Boot Sector 中保留 FSINFO 副本但不保持更新. |
| 0x34 | WORD | 0x0600 | **Backup Boot Sector (FAT32 only)**. 非零值表示卷保留区中存储 boot sector 副本的扇区号. 此字段值通常为 6. 不建议其他值. |
| 0x36 | 12 bytes | 0x000000000000000000000000 | **Reserved (FAT32 only)**. 保留供将来扩展. 此字段应恒为零. |

**Table 1.11 Extended BPB Fields for FAT32 Volumes**

| Byte Offset | Field Length | Value | Field Name and Definition |
| --- | --- | --- | --- |
| 0x40 | BYTE | 0x80 | **Physical Drive Number**. 与 BIOS 物理驱动器号相关. 软盘为 0x00, 物理硬盘为 0x80. 通常在 INT 13h 调用前设置. 仅 boot device 时相关. |
| 0x41 | BYTE | 0x00 | **Reserved**. FAT32 卷恒为零. |
| 0x42 | BYTE | 0x29 | **Extended Boot Signature**. 须为 0x28 或 0x29 方被 Windows 2000 识别. |
| 0x43 | DWORD | 0xA88B3652 | **Volume Serial Number**. 格式化时生成的随机序列号. |
| 0x47 | 11 bytes | NO NAME | **Volume Label**. 曾用于存储卷标. 卷标现作为根目录特殊文件存储. |
| 0x52 | LONGLONG | FAT32 | **System ID**. 值为 FAT32 的文本字段. |

### NTFS Boot Sector

表 1.12 说明 NTFS 格式化卷的 boot sector. 如表 1.12 所示, NTFS 卷的 bootstrap code 长于 426 字节. 格式化 NTFS 卷时, 格式化程序为 boot sector 与 bootstrap code 分配前 16 个扇区.

**Table 1.12 Boot Sector Sections on an NTFS Volume**

| Byte Offset | Field Length | Field Name |
| --- | --- | --- |
| 0x00 | 3 bytes | Jump Instruction |
| 0x03 | LONGLONG | OEM ID |
| 0x0B | 25 bytes | BPB |
| 0x24 | 48 bytes | Extended BPB |
| 0x54 | 426 bytes | Bootstrap Code |
| 0x01FE | WORD | End of Sector Marker |

在 NTFS 卷上, BPB 之后的字段构成 extended BPB. 这些字段使 Ntldr 在启动时找到 master file table (MFT). 在 NTFS 卷上, MFT 不在预定义扇区 (与 FAT16/FAT32 不同). 因此若正常位置出现坏扇区可移动 MFT. 然而若数据损坏则无法定位 MFT, Windows 2000 认为卷未格式化.

下列示例展示 Windows 2000 下格式化的 NTFS 卷 boot sector. 打印输出分三段:

- 字节 0x00–0x0A 为 jump instruction 与 OEM ID (粗体).
- 字节 0x0B–0x53 为 BPB 与 extended BPB.
- 其余为 bootstrap code 与 end of sector marker (粗体).

`Physical Sector: Cyl 0, Side 1, Sector 1 `

`00000000: EB 52 90 4E 54 46 53 20 - 20 20 20 00 02 08 00 00 .R.NTFS .....`

`00000010: 00 00 00 00 00 F8 00 00 - 3F 00 FF 00 3F 00 00 00 ........?...?...`

`00000020: 00 00 00 00 80 00 80 00 - 4A F5 7F 00 00 00 00 00 ........J......`

`00000030: 04 00 00 00 00 00 00 00 - 54 FF 07 00 00 00 00 00 ........T.......`

`00000040: F6 00 00 00 01 00 00 00 - 14 A5 1B 74 C9 1B 74 1C ...........t..t.`

`00000050: 00 00 00 00 FA 33 C0 8E - D0 BC 00 7C FB B8 C0 07 .....3.....|....`

(其余 bootstrap code 与 end of sector marker 见英文镜像原文 hex dump.)

表 1.13 说明 NTFS 卷 BPB 与 extended BPB 各字段. 起始于 0x0B, 0x0D, 0x15, 0x18, 0x1A 与 0x1C 的字段与 FAT16/FAT32 卷匹配. 示例值对应前述示例 (布局对齐 `pfs_ntfs_raw.h` / `ntfs_boot_sector_t`).

**Table 1.13 BPB and Extended BPB Fields on NTFS Volumes**

| Byte Offset | Field Length | Sample Value | Field Name |
| --- | --- | --- | --- |
| 0x0B | WORD | 0x0002 | Bytes Per Sector |
| 0x0D | BYTE | 0x08 | Sectors Per Cluster |
| 0x0E | WORD | 0x0000 | Reserved Sectors |
| 0x10 | 3 BYTES | 0x000000 | *always 0* |
| 0x13 | WORD | 0x0000 | *not used by NTFS* |
| 0x15 | BYTE | 0xF8 | Media Descriptor |
| 0x16 | WORD | 0x0000 | *always 0* |
| 0x18 | WORD | 0x3F00 | Sectors Per Track |
| 0x1A | WORD | 0xFF00 | Number Of Heads |
| 0x1C | DWORD | 0x3F000000 | Hidden Sectors |
| 0x20 | DWORD | 0x00000000 | *not used by NTFS* |
| 0x24 | DWORD | 0x80008000 | *not used by NTFS* |
| 0x28 | LONGLONG | 0x4AF57F0000000000 | Total Sectors (`number_of_sectors`) |
| 0x30 | LONGLONG | 0x0400000000000000 | Logical Cluster Number for the file $MFT (`mft_lcn`) |
| 0x38 | LONGLONG | 0x54FF070000000000 | Logical Cluster Number for the file $MftMirr (`mftmirr_lcn`) |
| 0x40 | DWORD | 0xF6000000 | Clusters Per File Record Segment (`clusters_per_mft_record`) |
| 0x44 | DWORD | 0x01000000 | Clusters Per Index Block (`clusters_per_index_record`) |
| 0x48 | LONGLONG | 0x14A51B74C91B741C | Volume Serial Number (`volume_serial_number`) |
| 0x50 | DWORD | 0x00000000 | Checksum (`checksum`) |

**Table 1.13 字段说明 (Description)**

| Field Name | Definition |
| --- | --- |
| Bytes Per Sector | 硬件扇区大小. |
| Sectors Per Cluster | 每簇扇区数. |
| Reserved Sectors | NTFS 将 boot sector 置于分区开头, 故恒为 0. |
| Media Descriptor | 0xF8 表示固定磁盘. Windows 2000 不用于 NTFS 操作. |
| Sectors Per Track / Number Of Heads | CHS 几何, NTFS 不依赖. |
| Hidden Sectors | boot sector 之前扇区数; NTFS boot sector 中通常为 0. |
| Total Sectors | 卷扇区总数. |
| Logical Cluster Number for the file $MFT | $Mft 数据 attribute 的 LCN, 即 MFT 位置. |
| Logical Cluster Number for the file $MftMirr | $MftMirr 的 LCN, 即 MFT 前 3–4 条 record 镜像. |
| Clusters Per File Record Segment | 每条 MFT record 大小 (正数为簇数, 负数为 2^|n| 字节). |
| Clusters Per Index Block | 每个 index block 大小, 编码规则同上. |
| Volume Serial Number | 64 位卷序列号. |
| Checksum | boot sector 校验和 (不含本字段). |

### 保护 Boot Sector

正常运行的系统依赖 boot sector 访问卷, 强烈建议定期运行 Chkdsk 等磁盘扫描工具, 并备份全部数据文件, 以防失去卷访问权限时数据丢失.
