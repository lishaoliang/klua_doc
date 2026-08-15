> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: `portfs/doc/std_exfat/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/exfat/`（Boot Sector / VolumeFlags / BootChecksum）
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 3 主引导区与备份引导区 (Main and Backup Boot Regions)

Main Boot Region（主引导区）提供实现方执行下列操作所需的全部引导指令、标识信息与文件系统参数:

1. 从 exFAT 卷引导启动计算机系统。
2. 识别卷上的文件系统为 exFAT。
3. 发现 exFAT 文件系统结构的位置。

Backup Boot Region（备份引导区）是 Main Boot Region 的备份。当 Main Boot Region 处于不一致状态时, 它有助于恢复 exFAT 卷。除更新引导指令等不常见情形外, 实现方 **should** 不修改 Backup Boot Region 的内容。

### 3.1 Main 与 Backup Boot Sector 子区

Main Boot Sector 含从 exFAT 卷引导启动的代码以及描述卷结构的基本 exFAT 参数 (见表 4)。BIOS、MBR 或其他引导代理 **may** 检查该扇区, 并 **may** 加载并执行其中所含的任何引导指令。

Backup Boot Sector 是 Main Boot Sector 的备份, 结构相同 (见表 4)。Backup Boot Sector **may** 辅助恢复操作; 然而, 实现方 **shall** 将 VolumeFlags 与 PercentInUse 字段的内容视为过期 (stale)。

在使用 Main 或 Backup Boot Sector 的内容之前, 实现方 **shall** 通过校验各自的 Boot Checksum 并确保所有字段处于有效取值范围内来验证其内容。

初始格式化操作会初始化 Main 与 Backup Boot Sector 的内容; 实现方 **may** 按需更新这些扇区 (并 **shall** 同时更新各自的 Boot Checksum)。然而, 实现方 **may** 更新 VolumeFlags 或 PercentInUse 字段而不更新各自的 Boot Checksum (校验和明确排除这两个字段)。

**表 4 Main 与 Backup Boot Sector 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| JumpBoot | 0 | 3 | 强制；内容见第 3.1.1 节。 |
| FileSystemName | 3 | 8 | 强制；内容见第 3.1.2 节。 |
| MustBeZero | 11 | 53 | 强制；内容见第 3.1.3 节。 |
| PartitionOffset | 64 | 8 | 强制；内容见第 3.1.4 节。 |
| VolumeLength | 72 | 8 | 强制；内容见第 3.1.5 节。 |
| FatOffset | 80 | 4 | 强制；内容见第 3.1.6 节。 |
| FatLength | 84 | 4 | 强制；内容见第 3.1.7 节。 |
| ClusterHeapOffset | 88 | 4 | 强制；内容见第 3.1.8 节。 |
| ClusterCount | 92 | 4 | 强制；内容见第 3.1.9 节。 |
| FirstClusterOfRootDirectory | 96 | 4 | 强制；内容见第 3.1.10 节。 |
| VolumeSerialNumber | 100 | 4 | 强制；内容见第 3.1.11 节。 |
| FileSystemRevision | 104 | 2 | 强制；内容见第 3.1.12 节。 |
| VolumeFlags | 106 | 2 | 强制；内容见第 3.1.13 节。 |
| BytesPerSectorShift | 108 | 1 | 强制；内容见第 3.1.14 节。 |
| SectorsPerClusterShift | 109 | 1 | 强制；内容见第 3.1.15 节。 |
| NumberOfFats | 110 | 1 | 强制；内容见第 3.1.16 节。 |
| DriveSelect | 111 | 1 | 强制；内容见第 3.1.17 节。 |
| PercentInUse | 112 | 1 | 强制；内容见第 3.1.18 节。 |
| Reserved | 113 | 7 | 强制；内容为保留 (Reserved)。 |
| BootCode | 120 | 390 | 强制；内容见第 3.1.19 节。 |
| BootSignature | 510 | 2 | 强制；内容见第 3.1.20 节。 |
| ExcessSpace | 512 | 2^BytesPerSectorShift^ – 512 | 强制；若有内容则为 Undefined。注: Main 与 Backup Boot Sector 均含 BytesPerSectorShift 字段。 |

#### 3.1.1 JumpBoot 字段

JumpBoot 字段 **shall** 含个人计算机常见 CPU 的跳转指令; 执行时 CPU "跳转" 至 BootCode 字段中的引导指令。

该字段有效值 (从低字节到高字节) 为 EBh 76h 90h。

#### 3.1.2 FileSystemName 字段

FileSystemName 字段 **shall** 含卷上文件系统的名称。

该字段有效值为 ASCII 字符 "EXFAT ", 含三个尾随空格。

#### 3.1.3 MustBeZero 字段

MustBeZero 字段 **shall** 直接对应 FAT12/16/32 卷上打包 BIOS 参数块所占字节范围。

该字段有效值为 0, 有助于防止 FAT12/16/32 实现误挂载 exFAT 卷。

#### 3.1.4 PartitionOffset 字段

PartitionOffset 字段 **shall** 描述承载给定 exFAT 卷的分区相对于介质的扇区偏移。该字段有助于在个人计算机上通过扩展 INT 13h 从卷引导启动。

该字段所有可能取值均有效; 然而, 值为 0 表示实现方 **shall** 忽略该字段。

#### 3.1.5 VolumeLength 字段

VolumeLength 字段 **shall** 以扇区为单位描述给定 exFAT 卷的大小。

该字段有效取值范围为:

- 至少 2^20^/ 2^BytesPerSectorShift^, 确保最小卷不小于 1MB
- 至多 2^64^- 1, 即该字段能描述的最大值。

 然而, 若 Excess Space 子区大小为 0, 则该字段最大值为 ClusterHeapOffset + (2^32^- 11) \*2^SectorsPerClusterShift^。

#### 3.1.6 FatOffset 字段

FatOffset 字段 **shall** 描述 First FAT 相对于卷的扇区偏移。该字段使实现方能够将 First FAT 对齐到底层存储介质的特性。

该字段有效取值范围为:

- 至少 24, 对应 Main Boot 与 Backup Boot 区所占扇区
- 至多 ClusterHeapOffset - (FatLength \* NumberOfFats), 对应 Cluster Heap 所占扇区

#### 3.1.7 FatLength 字段

FatLength 字段 **shall** 以扇区为单位描述每份 FAT 表的长度 (卷最多可含两份 FAT)。

该字段有效取值范围为:

- 至少 (ClusterCount + 2) \* 2^2^/ 2^BytesPerSectorShift^ 向上取整至最近整数, 确保每份 FAT 有足够空间描述 Cluster Heap 中全部簇
- 至多 (ClusterHeapOffset - FatOffset) / NumberOfFats 向下取整至最近整数, 确保 FAT 位于 Cluster Heap 之前

该字段 **may** 含超出上述下界的值, 以便 Second FAT (若存在) 也能对齐到底层存储介质的特性。超出 FAT 本身所需空间的部分 (若有) 内容为 Undefined。

#### 3.1.8 ClusterHeapOffset 字段

ClusterHeapOffset 字段 **shall** 描述 Cluster Heap 相对于卷的扇区偏移。该字段使实现方能够将 Cluster Heap 对齐到底层存储介质的特性。

该字段有效取值范围为:

- 至少 FatOffset + FatLength \* NumberOfFats, 对应前述全部区域所占扇区
- 至多 2^32^- 1 或 VolumeLength - (ClusterCount \* 2^SectorsPerClusterShift^), 取两者中较小者

#### 3.1.9 ClusterCount 字段

ClusterCount 字段 **shall** 描述 Cluster Heap 所含的簇数。

该字段有效值 **shall** 为下列两者中的较小者:

- (VolumeLength - ClusterHeapOffset) / 2^SectorsPerClusterShift^ 向下取整至最近整数, 即 Cluster Heap 起始至卷末恰好能容纳的簇数
- 2^32^- 11, 即一份 FAT 能描述的最大簇数

ClusterCount 字段的值决定 FAT 的最小大小。为避免 FAT 过大, 实现方 **may** 通过增大簇大小 (经 SectorsPerClusterShift 字段) 控制 Cluster Heap 中的簇数。本规范建议 Cluster Heap 中簇数不超过 2^24^- 2。然而, 实现方 **shall** 能处理 Cluster Heap 中最多 2^32^- 11 个簇的卷。

#### 3.1.10 FirstClusterOfRootDirectory 字段

FirstClusterOfRootDirectory 字段 **shall** 含根目录第一个簇的簇索引。根目录 **shall** 始终在活动 FAT 中以簇链描述, 如同根目录由 GeneralPrimaryFlags 字段的 NoFatChain 标志 (见 section 6.3.4.2) 等于零的目录项描述。根目录的数据长度 **shall** 始终通过加载簇链确定。实现方 **should** 尽力将根目录第一个簇放在 Allocation Bitmap 与 Up-case Table 所占簇之后的第一个非坏簇。

该字段有效取值范围为:

- 至少 2, 即 Cluster Heap 中第一个簇的索引
- 至多 ClusterCount + 1, 即 Cluster Heap 中最后一个簇的索引

#### 3.1.11 VolumeSerialNumber 字段

VolumeSerialNumber 字段 **shall** 含唯一序列号。这有助于实现方区分不同 exFAT 卷。实现方 **should** 通过组合格式化 exFAT 卷的日期与时间生成序列号。日期与时间组合成序列号的机制由实现自行决定。

该字段所有可能取值均有效。

#### 3.1.12 FileSystemRevision 字段

FileSystemRevision 字段 **shall** 描述给定卷上 exFAT 结构的主版本号与次版本号。

高字节为主版本号, 低字节为次版本号。例如, 若高字节为 01h 且低字节为 05h, 则 FileSystemRevision 描述版本号 1.05。又如, 若高字节为 0Ah 且低字节为 0Fh, 则描述版本号 10.15。

该字段有效取值范围为:

- 低字节至少 0, 高字节至少 1
- 低字节至多 99, 高字节至多 99

本规范所描述的 exFAT 版本号为 1.00。符合本规范的实现 **should** 挂载主版本号为 1 的任何 exFAT 卷, 且 **shall not** 挂载主版本号为其他任何值的 exFAT 卷。实现方 **shall** 遵守次版本号, 且 **shall not** 执行或创建给定次版本号对应规范未描述的任何文件系统结构。

#### 3.1.13 VolumeFlags 字段

VolumeFlags 字段 **shall** 含指示 exFAT 卷上各文件系统结构状态的标志 (见表 5)。

实现方 **shall not** 在计算各自 Main Boot 或 Backup Boot 区校验和时包含该字段。引用 Backup Boot Sector 时, 实现方 **shall** 将该字段视为过期。

**表 5 VolumeFlags 字段结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| ActiveFat | 0 | 1 | 强制；内容见第 3.1.13.1 节。 |
| VolumeDirty | 1 | 1 | 强制；内容见第 3.1.13.2 节。 |
| MediaFailure | 2 | 1 | 强制；内容见第 3.1.13.3 节。 |
| ClearToZero | 3 | 1 | 强制；内容见第 3.1.13.4 节。 |
| Reserved | 4 | 12 | 强制；内容为保留 (Reserved)。 |

##### 3.1.13.1 ActiveFat 字段

ActiveFat 字段 **shall** 描述哪一份 FAT 与 Allocation Bitmap 为活动 (且实现方 **shall** 使用), 如下:

- 0, 表示 First FAT 与 First Allocation Bitmap 为活动
- 1, 表示 Second FAT 与 Second Allocation Bitmap 为活动; 仅当 NumberOfFats 字段为 2 时可能

实现方 **shall** 将非活动 FAT 与 Allocation Bitmap 视为过期。仅 TexFAT 感知实现 **shall** 切换活动 FAT 与 Allocation Bitmap (见 Section 7.1)。

##### 3.1.13.2 VolumeDirty 字段

VolumeDirty 字段 **shall** 描述卷是否处于脏状态, 如下:

- 0, 表示卷可能处于一致状态
- 1, 表示卷可能处于不一致状态

实现方 **should** 在遇到未自行解决的文件系统元数据不一致时将本字段设为 1。若挂载卷时本字段为 1, 仅解决文件系统元数据不一致的实现 **may** 将本字段清为 0。此类实现 **shall** 仅在确保文件系统处于一致状态后才将本字段清为 0。

若挂载卷时本字段为 0, 实现方 **should** 在更新文件系统元数据前将本字段设为 1, 更新后再清为 0, 类似第 8.1 节建议的写入顺序。

##### 3.1.13.3 MediaFailure 字段

MediaFailure 字段 **shall** 描述实现方是否发现介质故障, 如下:

- 0, 表示承载介质未报告故障, 或已知故障已在 FAT 中记录为 "bad" 簇
- 1, 表示承载介质已报告故障 (即读写操作失败)

实现方 **should** 在下列情况下将本字段设为 1:

1. 承载介质访问卷内任何区域失败
2. 实现方已用尽访问重试算法 (若有)

若挂载卷时本字段为 1, 扫描整个卷以发现介质故障并将全部故障在 FAT 中记录为 "bad" 簇 (或以其他方式解决介质故障) 的实现 **may** 将本字段清为 0。

##### 3.1.13.4 ClearToZero 字段

ClearToZero 字段在本规范中无显著含义。

该字段有效值为:

- 0, 无特定含义
- 1, 表示实现方 **shall** 在修改任何文件系统结构、目录或文件之前将本字段清为 0

#### 3.1.14 BytesPerSectorShift 字段

BytesPerSectorShift 字段 **shall** 以 log2(N) 描述每扇区字节数, 其中 N 为每扇区字节数。例如, 512 字节/扇区时本字段值为 9。

该字段有效取值范围为:

- 至少 9 (扇区大小 512 字节), 即 exFAT 卷可能的最小扇区
- 至多 12 (扇区大小 4096 字节), 即个人计算机常见 CPU 的内存页大小

#### 3.1.15 SectorsPerClusterShift 字段

SectorsPerClusterShift 字段 **shall** 以 log2(N) 描述每簇扇区数, 其中 N 为每簇扇区数。例如, 8 扇区/簇时本字段值为 3。

该字段有效取值范围为:

- 至少 0 (1 扇区/簇), 即可能的最小簇
- 至多 25 - BytesPerSectorShift, 对应簇大小 32MB

#### 3.1.16 NumberOfFats 字段

NumberOfFats 字段 **shall** 描述卷所含的 FAT 与 Allocation Bitmap 数量。

该字段有效取值范围为:

- 1, 表示卷仅含 First FAT 与 First Allocation Bitmap
- 2, 表示卷含 First FAT、Second FAT、First Allocation Bitmap 与 Second Allocation Bitmap; 该值仅对 TexFAT 卷有效

#### 3.1.17 DriveSelect 字段

DriveSelect 字段 **shall** 含扩展 INT 13h 驱动器号, 有助于在个人计算机上通过扩展 INT 13h 从本卷引导启动。

该字段所有可能取值均有效。先前基于 FAT 的文件系统中类似字段常为 80h。

#### 3.1.18 PercentInUse 字段

PercentInUse 字段 **shall** 描述 Cluster Heap 中已分配簇的百分比。

该字段有效取值范围为:

- 0 至 100 (含), 即 Cluster Heap 中已分配簇百分比, 向下取整至最近整数
- 恰好 FFh, 表示 Cluster Heap 中已分配簇百分比不可用

实现方 **shall** 修改本字段以反映 Cluster Heap 中簇分配的变化, 或 **shall** 将其改为 FFh。

实现方 **shall not** 在计算各自 Main Boot 或 Backup Boot 区校验和时包含该字段。引用 Backup Boot Sector 时, 实现方 **shall** 将该字段视为过期。

#### 3.1.19 BootCode 字段

BootCode 字段 **shall** 含引导指令。实现方 **may** 用引导计算机系统所需的 CPU 指令填充该字段。不提供引导指令的实现 **shall** 在格式化操作中将该字段每字节初始化为 F4h (个人计算机常见 CPU 的 halt 指令)。

#### 3.1.20 BootSignature 字段

BootSignature 字段 **shall** 描述给定扇区是否意图作为 Boot Sector。

该字段有效值为 AA55h。本字段任何其他值均使其各自 Boot Sector 无效。实现方 **should** 在依赖各自 Boot Sector 中任何其他字段之前验证本字段内容。

### 3.2 Main 与 Backup Extended Boot Sectors 子区

Main Extended Boot Sectors 的每个扇区结构相同; 然而, 各扇区 **may** 含不同的引导指令 (见表 6)。引导代理 (如 Main Boot Sector 中的引导指令、替代 BIOS 实现或嵌入式系统固件) **may** 加载这些扇区并执行其中指令。

Backup Extended Boot Sectors 是 Main Extended Boot Sectors 的备份, 结构相同 (见表 6)。

在执行 Main 或 Backup Extended Boot Sectors 的指令之前, 实现方 **should** 通过确保各扇区的 ExtendedBootSignature 字段含规定值来验证其内容。

初始格式化操作会初始化 Main 与 Backup Extended Boot Sectors 的内容; 实现方 **may** 按需更新这些扇区 (并 **shall** 同时更新各自的 Boot Checksum)。

**表 6 Extended Boot Sector 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| ExtendedBootCode | 0 | 2^BytesPerSectorShift^ – 4 | 强制；内容见第 3.2.1 节。注: Main 与 Backup Boot Sector 均含 BytesPerSectorShift 字段。 |
| ExtendedBootSignature | 2^BytesPerSectorShift^ – 4 | 4 | 强制；内容见第 3.2.2 节。注: Main 与 Backup Boot Sector 均含 BytesPerSectorShift 字段。 |

#### 3.2.1 ExtendedBootCode 字段

ExtendedBootCode 字段 **shall** 含引导指令。实现方 **may** 用引导计算机系统所需的 CPU 指令填充该字段。不提供引导指令的实现 **shall** 在格式化操作中将该字段每字节初始化为 00h。

#### 3.2.2 ExtendedBootSignature 字段

ExtendedBootSignature 字段 **shall** 描述给定扇区是否意图作为 Extended Boot Sector。

该字段有效值为 AA550000h。本字段任何其他值均使其各自 Main 或 Backup Extended Boot Sector 无效。实现方 **should** 在依赖各自 Extended Boot Sector 中任何其他字段之前验证本字段内容。

### 3.3 Main 与 Backup OEM Parameters 子区

Main OEM Parameters 子区含十个参数结构, **may** 含制造商特定信息 (见表 7)。十个参数结构均派生自 Generic Parameters 模板 (见 Section 3.3.2)。制造商 **may** 从 Generic Parameters 模板派生自定义参数结构。本规范自身定义两种参数结构: Null Parameters (见 Section 3.3.3) 与 Flash Parameters (见 Section 3.3.4)。

Backup OEM Parameters 是 Main OEM Parameters 的备份, 结构相同 (见表 7)。

在使用 Main 或 Backup OEM Parameters 的内容之前, 实现方 **shall** 通过校验各自的 Boot Checksum 验证其内容。

制造商 **should** 用自定义参数结构 (若有) 及其他参数结构填充 Main 与 Backup OEM Parameters。后续格式化操作 **shall** 保留 Main 与 Backup OEM Parameters 的内容。

实现方 **may** 按需更新 Main 与 Backup OEM Parameters (并 **shall** 同时更新各自的 Boot Checksum)。

**表 7 OEM Parameters 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| Parameters[0] | 0 | 48 | 强制；内容见第 3.3.1 节。 |
| ... | ... | ... | ... |
| Parameters[9] | 432 | 48 | 强制；内容见第 3.3.1 节。 |
| Reserved | 480 | 2^BytesPerSectorShift^ – 480 | 强制；内容为保留 (Reserved)。注: Main 与 Backup Boot Sector 均含 BytesPerSectorShift 字段。 |

#### 3.3.1 Parameters[0] ... Parameters[9]

本数组中每个 Parameters 字段含一个参数结构, 派生自 Generic Parameters 模板 (见 Section 3.3.2)。任何未使用的 Parameters 字段 **shall** 描述为含 Null Parameters 结构 (见 Section 3.3.3)。

#### 3.3.2 Generic Parameters 模板

Generic Parameters 模板提供参数结构的基本定义 (见表 8)。所有参数结构均派生自该模板。支持 Generic Parameters 模板为强制。

**表 8 Generic Parameters 模板**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| ParametersGuid | 0 | 16 | 强制；内容见第 3.3.2.1 节。 |
| CustomDefined | 16 | 32 | 强制；派生自该模板的结构定义其内容。 |

##### 3.3.2.1 ParametersGuid 字段

ParametersGuid 字段 **shall** 描述 GUID, 决定给定参数结构其余部分的布局。

该字段所有可能取值均有效; 然而, 制造商 **should** 在从该模板派生自定义参数结构时使用 GUID 生成工具 (如 GuidGen.exe) 选择 GUID。

#### 3.3.3 Null Parameters

Null Parameters 结构派生自 Generic Parameters 模板 (见 Section 3.3.2), **shall** 描述未使用的 Parameters 字段 (见表 9)。创建或更新 OEM Parameters 结构时, 实现方 **shall** 用 Null Parameters 结构填充未使用的 Parameters 字段。创建或更新 OEM Parameters 结构时, 实现方 **should** 将 Null Parameters 结构合并至数组末尾, 从而将其余 Parameters 结构留在 OEM Parameters 结构开头。

支持 Null Parameters 结构为强制。

**表 9 Null Parameters 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| ParametersGuid | 0 | 16 | 强制；内容见第 3.3.3.1 节。 |
| Reserved | 16 | 32 | 强制；内容为保留 (Reserved)。 |

##### 3.3.3.1 ParametersGuid 字段

ParametersGuid 字段 **shall** 符合 Generic Parameters 模板提供的定义 (见 Section 3.3.2.1)。

该字段有效值 (GUID 记法) 为 {00000000-0000-0000-0000-000000000000}。

#### 3.3.4 Flash Parameters

Flash Parameters 结构派生自 Generic Parameters 模板 (见 Section 3.3.2), 含 flash 介质参数 (见表 10)。基于 flash 的存储设备制造商 **may** 用该参数结构填充 Parameters 字段 (宜为 Parameters[0] 字段)。实现方 **may** 使用 Flash Parameters 结构中的信息优化读写访问操作, 以及在格式化介质时对齐文件系统结构。

支持 Flash Parameters 结构为可选。

**表 10 Flash Parameters 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| ParametersGuid | 0 | 16 | 强制；内容见第 3.3.4.1 节。 |
| EraseBlockSize | 16 | 4 | 强制；内容见第 3.3.4.2 节。 |
| PageSize | 20 | 4 | 强制；内容见第 3.3.4.3 节。 |
| SpareSectors | 24 | 4 | 强制；内容见第 3.3.4.4 节。 |
| RandomAccessTime | 28 | 4 | 强制；内容见第 3.3.4.5 节。 |
| ProgrammingTime | 32 | 4 | 强制；内容见第 3.3.4.6 节。 |
| ReadCycle | 36 | 4 | 强制；内容见第 3.3.4.7 节。 |
| WriteCycle | 40 | 4 | 强制；内容见第 3.3.4.8 节。 |
| Reserved | 44 | 4 | 强制；内容为保留 (Reserved)。 |

除 ParametersGuid 字段外, Flash Parameters 全部字段的所有可能取值均有效。然而, 值为 0 表示该字段实际无意义 (实现方 **shall** 忽略给定字段)。

##### 3.3.4.1 ParametersGuid 字段

ParametersGuid 字段 **shall** 符合 Generic Parameters 模板中的定义 (见 Section 3.3.2.1)。

该字段有效值 (GUID 记法) 为 {0A0C7E46-3399-4021-90C8-FA6D389C4BA2}。

##### 3.3.4.2 EraseBlockSize 字段

EraseBlockSize 字段 **shall** 以字节描述 flash 介质的擦除块大小。

##### 3.3.4.3 PageSize 字段

PageSize 字段 **shall** 以字节描述 flash 介质的页大小。

##### 3.3.4.4 SpareSectors 字段

SpareSectors 字段 **shall** 描述 flash 介质可用于内部备用操作的扇区数。

##### 3.3.4.5 RandomAccessTime 字段

RandomAccessTime 字段 **shall** 以纳秒描述 flash 介质的平均随机访问时间。

##### 3.3.4.6 ProgrammingTime 字段

ProgrammingTime 字段 **shall** 以纳秒描述 flash 介质的平均编程时间。

##### 3.3.4.7 ReadCycle 字段

ReadCycle 字段 **shall** 以纳秒描述 flash 介质的平均读周期时间。

##### 3.3.4.8 WriteCycle 字段

WriteCycle 字段 **shall** 以纳秒描述平均写周期时间。

### 3.4 Main 与 Backup Boot Checksum 子区

Main 与 Backup Boot Checksum 各含其四字节校验和的重复模式, 该校验和针对各自 Boot 区中所有其他子区的内容计算。校验和计算 **shall not** 包含各自 Boot Sector 中的 VolumeFlags 与 PercentInUse 字段 (见 Figure 1)。四字节校验和的重复模式自子区起始填充至 Boot Checksum 子区末尾。

在使用 Main 或 Backup Boot 区中任何其他子区的内容之前, 实现方 **shall** 通过校验各自的 Boot Checksum 验证其内容。

初始格式化操作会用重复校验和模式填充 Main 与 Backup Boot Checksum; 实现方 **shall** 在各自 Boot 区中其他扇区内容变化时更新这些扇区。

**Figure 1 Boot Checksum 计算**

```c
UInt32 BootChecksum
(
    UCHAR  * Sectors,        // points to an in-memory copy of the 11 sectors
    USHORT   BytesPerSector
)
{
    UInt32 NumberOfBytes = (UInt32)BytesPerSector * 11;
    UInt32 Checksum = 0;
    UInt32 Index;

    for (Index = 0; Index < NumberOfBytes; Index++)
    {
        if ((Index == 106) || (Index == 107) || (Index == 112))
        {
            continue;
        }
        Checksum = ((Checksum&1) ? 0x80000000 : 0) + (Checksum>>1) + (UInt32)Sectors[Index];
    }

    return Checksum;
}
```
