> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名保留英文；目录项/位图/Up-case 对齐 linux-7.1.2 `fs/exfat/`（`dir.c`/`nls.c`/`balloc.c`）与 `pfs_exfat_*`；§7.2 推荐 Up-case 表数据保持原文
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 7 目录项定义 (Directory Entry Definitions)

exFAT 文件系统修订版 1.00 定义下列目录项:

- 关键主项 (Critical primary)
 - Allocation Bitmap（第 7.1 节）
 - Up-case Table（第 7.2 节）
 - Volume Label（第 7.3 节）
 - File（第 7.4 节）
- 良性主项 (Benign primary)
 - Volume GUID（第 7.5 节）
 - TexFAT Padding（第 7.10 节）

- 关键次项 (Critical secondary)
 - Stream Extension（第 7.6 节）
 - File Name（第 7.7 节）
- 良性次项 (Benign secondary)
 - Vendor Extension（第 7.8 节）
 - Vendor Allocation（第 7.9 节）

### 7.1 Allocation Bitmap 目录项

在 exFAT 文件系统中, FAT 并不描述簇的分配状态; 而是由 Allocation Bitmap 承担该职责. Allocation Bitmap 位于 Cluster Heap (见第 7.1.5 节), 并在根目录中有对应的 critical primary directory entry (见表 20).

NumberOfFats 字段决定根目录中有效 Allocation Bitmap directory entry 的数量. 若 NumberOfFats 为 1, 则唯一有效的 Allocation Bitmap directory entry 数量为 1; 且该 entry 仅在其描述 First Allocation Bitmap (见第 7.1.2.1 节) 时有效. 若 NumberOfFats 为 2, 则唯一有效的 Allocation Bitmap directory entry 数量为 2; 且二者分别描述 First Allocation Bitmap 与 Second Allocation Bitmap 时方有效.

**表 20 Allocation Bitmap DirectoryEntry 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.1.1 节。 |
| BitmapFlags | 1 | 1 | 强制字段; 内容见第 7.1.2 节。 |
| Reserved | 2 | 18 | 强制字段; 内容为保留。 |
| FirstCluster | 20 | 4 | 强制字段; 内容见第 7.1.3 节。 |
| DataLength | 24 | 8 | 强制字段; 内容见第 7.1.4 节。 |

#### 7.1.1 EntryType 字段

EntryType 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1 节).

##### 7.1.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.1 节).

对于 Allocation Bitmap directory entry, 本字段唯一有效取值为 1.

##### 7.1.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.2 节).

对于 Allocation Bitmap directory entry, 本字段唯一有效取值为 0.

##### 7.1.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.3 节).

##### 7.1.1.4 InUse 字段

InUse 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.4 节).

#### 7.1.2 BitmapFlags 字段

BitmapFlags 字段包含标志位 (见表 21).

**表 21 BitmapFlags 字段结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| BitmapIdentifier | 0 | 1 | 强制字段; 内容见第 7.1.2.1 节。 |
| Reserved | 1 | 7 | 强制字段; 内容为保留。 |

##### 7.1.2.1 BitmapIdentifier 字段

BitmapIdentifier 字段应表明给定 directory entry 所描述的是哪一个 Allocation Bitmap. 实现须将 First Allocation Bitmap 与 First FAT 配合使用, 将 Second Allocation Bitmap 与 Second FAT 配合使用. ActiveFat 字段描述当前活动的 FAT 与 Allocation Bitmap.

本字段有效取值为:

- 0, 表示给定 directory entry 描述 First Allocation Bitmap
- 1, 表示给定 directory entry 描述 Second Allocation Bitmap; 仅当 NumberOfFats 为 2 时方可出现

#### 7.1.3 FirstCluster 字段

FirstCluster 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.5 节).

本字段包含 FAT 所描述的簇链中首簇的索引, 该簇链承载 Allocation Bitmap.

#### 7.1.4 DataLength 字段

DataCluster 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.6 节).

#### 7.1.5 Allocation Bitmap

Allocation Bitmap 记录 Cluster Heap 中各簇的分配状态. Allocation Bitmap 中每一位表明对应簇是否可供分配.

Allocation Bitmap 自最低簇索引向最高簇索引表示各簇 (见表 22). 出于历史原因, 首个簇的索引为 2.

注

位图首 bit 为第一个 byte 的最低有效位.

**表 22 Allocation Bitmap 结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| BitmapEntry[2] | 0 | 1 | 强制字段; 内容见第 7.1.5.1 节. |
| ... | ... | ... | ... |
| BitmapEntry[ClusterCount+1] | ClusterCount - 1 | 1 | 强制字段; 内容见第 7.1.5.1 节. 注: Main 与 Backup Boot Sector 均含 ClusterCount 字段. |
| Reserved | ClusterCount | (DataLength \* 8) – ClusterCount | 强制字段; 内容为保留 (若有). 注: Main 与 Backup Boot Sector 均含 ClusterCount 字段. |

##### 7.1.5.1 BitmapEntry[2] ... BitmapEntry[ClusterCount+1] 字段

该数组中每个 BitmapEntry 字段对应 Cluster Heap 中的一个簇. BitmapEntry[2] 表示 Cluster Heap 中第一个簇, BitmapEntry[ClusterCount+1] 表示最后一个簇.

这些字段的有效取值为:

- 0, 表示对应簇可供分配
- 1, 表示对应簇不可供分配 (簇分配可能已占用该簇, 或活动 FAT 可能将该簇标记为 bad)
### 7.2 Up-case Table 目录项

Up-case Table 定义小写字符到大写字符的转换. 由于 File Name directory entry (见第 7.7 节) 使用 Unicode 字符, 且 exFAT 文件系统大小写不敏感并保留大小写, 该表十分重要. Up-case Table 位于 Cluster Heap (见第 7.2.5 节), 并在根目录中有对应的 critical primary directory entry (见表 23). 有效 Up-case Table directory entry 数量为 1.

鉴于 Up-case Table 与文件名的关系, 除格式化操作外, 实现宜不修改 Up-case Table.

**表 23 Up-case Table DirectoryEntry 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.2.1 节。 |
| Reserved1 | 1 | 3 | 强制字段; 内容为保留。 |
| TableChecksum | 4 | 4 | 强制字段; 内容见第 7.2.2 节。 |
| Reserved2 | 8 | 12 | 强制字段; 内容为保留。 |
| FirstCluster | 20 | 4 | 强制字段; 内容见第 7.2.3 节。 |
| DataLength | 24 | 8 | 强制字段; 内容见第 7.2.4 节。 |

#### 7.2.1 EntryType 字段

EntryType 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1 节).

##### 7.2.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.1 节).

对于 Up-case Table directory entry, 本字段唯一有效取值为 2.

##### 7.2.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.2 节).

对于 Up-case Table directory entry, 本字段唯一有效取值为 0.

##### 7.2.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.3 节).

##### 7.2.1.4 InUse 字段

InUse 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.4 节).

#### 7.2.2 TableChecksum 字段

TableChecksum 字段包含 Up-case Table 的校验和 (由 FirstCluster 与 DataLength 字段描述). 实现于使用 Up-case Table 之前, 须验证本字段内容有效.

**图 3 TableChecksum 计算**

```C
UInt32 TableChecksum
(
    UCHAR  * Table,    // points to an in-memory copy of the up-case table
    UInt64   DataLength
)
{
    UInt32 Checksum = 0;
    UInt64 Index;

    for (Index = 0; Index < DataLength; Index++)
    {
        Checksum = ((Checksum&1) ? 0x80000000 : 0) + (Checksum>>1) + (UInt32)Table[Index];
    }

    return Checksum;
}
```

#### 7.2.3 FirstCluster 字段

FirstCluster 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.5 节).

本字段包含 FAT 所描述的簇链中首簇的索引, 该簇链承载 Up-case Table.

#### 7.2.4 DataLength 字段

DataCluster 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.6 节).

#### 7.2.5 Up-case Table

up-case table 是一系列 Unicode 字符映射. 每个字符映射为 2 字节字段: 字段在 up-case table 中的索引表示待大写化的 Unicode 字符, 2 字节字段值表示大写化后的 Unicode 字符.

前 128 个 Unicode 字符具有强制映射 (见表 24). 若 up-case table 对前 128 个 Unicode 字符中任一字符给出与强制映射不同的映射, 则该表无效.

仅支持强制映射范围内字符的实现, 可忽略 up-case table 其余部分的映射. 此类实现在创建或重命名文件时 (通过 File Name directory entry, 见第 7.7 节), 须仅使用强制映射范围内的字符. 对大写化既有文件名时, 此类实现不得对大写化非强制映射范围内的字符, 而应在结果大写文件名中保持原样 (部分大写化, partial up-casing). 比较文件名时, 此类实现须将与比较名仅在非强制映射范围内 Unicode 字符上不同的文件名视为等价. 此类文件名仅为潜在等价, 实现无法保证完全大写化后的文件名不与比较名冲突.

**表 24 前 128 项强制 up-case table 条目**

| Table index | + 0 | + 1 | + 2 | + 3 | + 4 | + 5 | + 6 | + 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **0000h** | 0000h | 0001h | 0002h | 0003h | 0004h | 0005h | 0006h | 0007h |
| **0008h** | 0008h | 0009h | 000Ah | 000Bh | 000Ch | 000Dh | 000Eh | 000Fh |
| **0010h** | 0010h | 0011h | 0012h | 0013h | 0014h | 0015h | 0016h | 0017h |
| **0018h** | 0018h | 0019h | 001Ah | 001Bh | 001Ch | 001Dh | 001Eh | 001Fh |
| **0020h** | 0020h | 0021h | 0022h | 0023h | 0024h | 0025h | 0026h | 0027h |
| **0028h** | 0028h | 0029h | 002Ah | 002Bh | 002Ch | 002Dh | 002Eh | 002Fh |
| **0030h** | 0030h | 0031h | 0032h | 0033h | 0034h | 0035h | 0036h | 0037h |
| **0038h** | 0038h | 0039h | 003Ah | 003Bh | 003Ch | 003Dh | 003Eh | 003Fh |
| **0040h** | 0040h | 0041h | 0042h | 0043h | 0044h | 0045h | 0046h | 0047h |
| **0048h** | 0048h | 0049h | 004Ah | 004Bh | 004Ch | 004Dh | 004Eh | 004Fh |
| **0050h** | 0050h | 0051h | 0052h | 0053h | 0054h | 0055h | 0056h | 0057h |
| **0058h** | 0058h | 0059h | 005Ah | 005Bh | 005Ch | 005Dh | 005Eh | 005Fh |
| **0060h** | 0060h | **0041h** | **0042h** | **0043h** | **0044h** | **0045h** | **0046h** | **0047h** |
| **0068h** | **0048h** | **0049h** | **004Ah** | **004Bh** | **004Ch** | **004Dh** | **004Eh** | **004Fh** |
| **0070h** | **0050h** | **0051h** | **0052h** | **0053h** | **0054h** | **0055h** | **0056h** | **0057h** |
| **0078h** | **0058h** | **0059h** | **005Ah** | 007Bh | 007Ch | 007Dh | 007Eh | 007Fh |

**(注: 非恒等 up-case 映射的条目以粗体显示)**

格式化卷时, 实现可借助 identity-mapping compression 生成压缩格式的 up-case table, 因 Unicode 字符空间中很大部分无大小写概念 (即 "小写" 与 "大写" 字符等价). 实现以 FFFFh 后接 identity mapping 数量来表示一系列 identity mapping, 从而压缩 up-case table.

例如, 实现可用压缩 up-case table 的下列八个条目表示前 100 (64h) 个字符映射:

>
> FFFFh, 0061h, 0041h, 0042h, 0043h

前两个条目表示前 97 (61h) 个字符 (0000h 至 0060h) 为 identity mapping. 随后字符 0061h 至 0063h 分别映射至 0041h 至 0043h.

格式化卷时提供压缩 up-case table 的能力为可选; 但解释压缩与未压缩 up-case table 的能力为强制. TableChecksum 字段的值始终与卷上 up-case table 的实际形态一致, 该形态可为压缩或未压缩.

##### 7.2.5.1 推荐 Up-case Table

格式化卷时, 实现宜以压缩格式记录推荐 up-case table (见表 25), 其 TableChecksum 字段值为 E619D30Dh.

若实现定义自有 up-case table (压缩或未压缩), 则该表须覆盖完整 Unicode 字符范围 (字符码 0000h 至 FFFFh, 含端点).

**表 25 推荐压缩格式 up-case table**

| Raw offset | + 0 | + 1 | + 2 | + 3 | + 4 | + 5 | + 6 | + 7 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **0000h** | 0000h | 0001h | 0002h | 0003h | 0004h | 0005h | 0006h | 0007h |
| **0008h** | 0008h | 0009h | 000Ah | 000Bh | 000Ch | 000Dh | 000Eh | 000Fh |
| **0010h** | 0010h | 0011h | 0012h | 0013h | 0014h | 0015h | 0016h | 0017h |
| **0018h** | 0018h | 0019h | 001Ah | 001Bh | 001Ch | 001Dh | 001Eh | 001Fh |
| **0020h** | 0020h | 0021h | 0022h | 0023h | 0024h | 0025h | 0026h | 0027h |
| **0028h** | 0028h | 0029h | 002Ah | 002Bh | 002Ch | 002Dh | 002Eh | 002Fh |
| **0030h** | 0030h | 0031h | 0032h | 0033h | 0034h | 0035h | 0036h | 0037h |
| **0038h** | 0038h | 0039h | 003Ah | 003Bh | 003Ch | 003Dh | 003Eh | 003Fh |
| **0040h** | 0040h | 0041h | 0042h | 0043h | 0044h | 0045h | 0046h | 0047h |
| **0048h** | 0048h | 0049h | 004Ah | 004Bh | 004Ch | 004Dh | 004Eh | 004Fh |
| **0050h** | 0050h | 0051h | 0052h | 0053h | 0054h | 0055h | 0056h | 0057h |
| **0058h** | 0058h | 0059h | 005Ah | 005Bh | 005Ch | 005Dh | 005Eh | 005Fh |
| **0060h** | 0060h | 0041h | 0042h | 0043h | 0044h | 0045h | 0046h | 0047h |
| **0068h** | 0048h | 0049h | 004Ah | 004Bh | 004Ch | 004Dh | 004Eh | 004Fh |
| **0070h** | 0050h | 0051h | 0052h | 0053h | 0054h | 0055h | 0056h | 0057h |
| **0078h** | 0058h | 0059h | 005Ah | 007Bh | 007Ch | 007Dh | 007Eh | 007Fh |
| **0080h** | 0080h | 0081h | 0082h | 0083h | 0084h | 0085h | 0086h | 0087h |
| **0088h** | 0088h | 0089h | 008Ah | 008Bh | 008Ch | 008Dh | 008Eh | 008Fh |
| **0090h** | 0090h | 0091h | 0092h | 0093h | 0094h | 0095h | 0096h | 0097h |
| **0098h** | 0098h | 0099h | 009Ah | 009Bh | 009Ch | 009Dh | 009Eh | 009Fh |
| **00A0h** | 00A0h | 00A1h | 00A2h | 00A3h | 00A4h | 00A5h | 00A6h | 00A7h |
| **00A8h** | 00A8h | 00A9h | 00AAh | 00ABh | 00ACh | 00ADh | 00AEh | 00AFh |
| **00B0h** | 00B0h | 00B1h | 00B2h | 00B3h | 00B4h | 00B5h | 00B6h | 00B7h |
| **00B8h** | 00B8h | 00B9h | 00BAh | 00BBh | 00BCh | 00BDh | 00BEh | 00BFh |
| **00C0h** | 00C0h | 00C1h | 00C2h | 00C3h | 00C4h | 00C5h | 00C6h | 00C7h |
| **00C8h** | 00C8h | 00C9h | 00CAh | 00CBh | 00CCh | 00CDh | 00CEh | 00CFh |
| **00D0h** | 00D0h | 00D1h | 00D2h | 00D3h | 00D4h | 00D5h | 00D6h | 00D7h |
| **00D8h** | 00D8h | 00D9h | 00DAh | 00DBh | 00DCh | 00DDh | 00DEh | 00DFh |
| **00E0h** | 00C0h | 00C1h | 00C2h | 00C3h | 00C4h | 00C5h | 00C6h | 00C7h |
| **00E8h** | 00C8h | 00C9h | 00CAh | 00CBh | 00CCh | 00CDh | 00CEh | 00CFh |
| **00F0h** | 00D0h | 00D1h | 00D2h | 00D3h | 00D4h | 00D5h | 00D6h | 00F7h |
| **00F8h** | 00D8h | 00D9h | 00DAh | 00DBh | 00DCh | 00DDh | 00DEh | 0178h |
| **0100h** | 0100h | 0100h | 0102h | 0102h | 0104h | 0104h | 0106h | 0106h |
| **0108h** | 0108h | 0108h | 010Ah | 010Ah | 010Ch | 010Ch | 010Eh | 010Eh |
| **0110h** | 0110h | 0110h | 0112h | 0112h | 0114h | 0114h | 0116h | 0116h |
| **0118h** | 0118h | 0118h | 011Ah | 011Ah | 011Ch | 011Ch | 011Eh | 011Eh |
| **0120h** | 0120h | 0120h | 0122h | 0122h | 0124h | 0124h | 0126h | 0126h |
| **0128h** | 0128h | 0128h | 012Ah | 012Ah | 012Ch | 012Ch | 012Eh | 012Eh |
| **0130h** | 0130h | 0131h | 0132h | 0132h | 0134h | 0134h | 0136h | 0136h |
| **0138h** | 0138h | 0139h | 0139h | 013Bh | 013Bh | 013Dh | 013Dh | 013Fh |
| **0140h** | 013Fh | 0141h | 0141h | 0143h | 0143h | 0145h | 0145h | 0147h |
| **0148h** | 0147h | 0149h | 014Ah | 014Ah | 014Ch | 014Ch | 014Eh | 014Eh |
| **0150h** | 0150h | 0150h | 0152h | 0152h | 0154h | 0154h | 0156h | 0156h |
| **0158h** | 0158h | 0158h | 015Ah | 015Ah | 015Ch | 015Ch | 015Eh | 015Eh |
| **0160h** | 0160h | 0160h | 0162h | 0162h | 0164h | 0164h | 0166h | 0166h |
| **0168h** | 0168h | 0168h | 016Ah | 016Ah | 016Ch | 016Ch | 016Eh | 016Eh |
| **0170h** | 0170h | 0170h | 0172h | 0172h | 0174h | 0174h | 0176h | 0176h |
| **0178h** | 0178h | 0179h | 0179h | 017Bh | 017Bh | 017Dh | 017Dh | 017Fh |
| **0180h** | 0243h | 0181h | 0182h | 0182h | 0184h | 0184h | 0186h | 0187h |
| **0188h** | 0187h | 0189h | 018Ah | 018Bh | 018Bh | 018Dh | 018Eh | 018Fh |
| **0190h** | 0190h | 0191h | 0191h | 0193h | 0194h | 01F6h | 0196h | 0197h |
| **0198h** | 0198h | 0198h | 023Dh | 019Bh | 019Ch | 019Dh | 0220h | 019Fh |
| **01A0h** | 01A0h | 01A0h | 01A2h | 01A2h | 01A4h | 01A4h | 01A6h | 01A7h |
| **01A8h** | 01A7h | 01A9h | 01AAh | 01ABh | 01ACh | 01ACh | 01AEh | 01AFh |
| **01B0h** | 01AFh | 01B1h | 01B2h | 01B3h | 01B3h | 01B5h | 01B5h | 01B7h |
| **01B8h** | 01B8h | 01B8h | 01BAh | 01BBh | 01BCh | 01BCh | 01BEh | 01F7h |
| **01C0h** | 01C0h | 01C1h | 01C2h | 01C3h | 01C4h | 01C5h | 01C4h | 01C7h |
| **01C8h** | 01C8h | 01C7h | 01CAh | 01CBh | 01CAh | 01CDh | 01CDh | 01CFh |
| **01D0h** | 01CFh | 01D1h | 01D1h | 01D3h | 01D3h | 01D5h | 01D5h | 01D7h |
| **01D8h** | 01D7h | 01D9h | 01D9h | 01DBh | 01DBh | 018Eh | 01DEh | 01DEh |
| **01E0h** | 01E0h | 01E0h | 01E2h | 01E2h | 01E4h | 01E4h | 01E6h | 01E6h |
| **01E8h** | 01E8h | 01E8h | 01EAh | 01EAh | 01ECh | 01ECh | 01EEh | 01EEh |
| **01F0h** | 01F0h | 01F1h | 01F2h | 01F1h | 01F4h | 01F4h | 01F6h | 01F7h |
| **01F8h** | 01F8h | 01F8h | 01FAh | 01FAh | 01FCh | 01FCh | 01FEh | 01FEh |
| **0200h** | 0200h | 0200h | 0202h | 0202h | 0204h | 0204h | 0206h | 0206h |
| **0208h** | 0208h | 0208h | 020Ah | 020Ah | 020Ch | 020Ch | 020Eh | 020Eh |
| **0210h** | 0210h | 0210h | 0212h | 0212h | 0214h | 0214h | 0216h | 0216h |
| **0218h** | 0218h | 0218h | 021Ah | 021Ah | 021Ch | 021Ch | 021Eh | 021Eh |
| **0220h** | 0220h | 0221h | 0222h | 0222h | 0224h | 0224h | 0226h | 0226h |
| **0228h** | 0228h | 0228h | 022Ah | 022Ah | 022Ch | 022Ch | 022Eh | 022Eh |
| **0230h** | 0230h | 0230h | 0232h | 0232h | 0234h | 0235h | 0236h | 0237h |
| **0238h** | 0238h | 0239h | 2C65h | 023Bh | 023Bh | 023Dh | 2C66h | 023Fh |
| **0240h** | 0240h | 0241h | 0241h | 0243h | 0244h | 0245h | 0246h | 0246h |
| **0248h** | 0248h | 0248h | 024Ah | 024Ah | 024Ch | 024Ch | 024Eh | 024Eh |
| **0250h** | 0250h | 0251h | 0252h | 0181h | 0186h | 0255h | 0189h | 018Ah |
| **0258h** | 0258h | 018Fh | 025Ah | 0190h | 025Ch | 025Dh | 025Eh | 025Fh |
| **0260h** | 0193h | 0261h | 0262h | 0194h | 0264h | 0265h | 0266h | 0267h |
| **0268h** | 0197h | 0196h | 026Ah | 2C62h | 026Ch | 026Dh | 026Eh | 019Ch |
| **0270h** | 0270h | 0271h | 019Dh | 0273h | 0274h | 019Fh | 0276h | 0277h |
| **0278h** | 0278h | 0279h | 027Ah | 027Bh | 027Ch | 2C64h | 027Eh | 027Fh |
| **0280h** | 01A6h | 0281h | 0282h | 01A9h | 0284h | 0285h | 0286h | 0287h |
| **0288h** | 01AEh | 0244h | 01B1h | 01B2h | 0245h | 028Dh | 028Eh | 028Fh |
| **0290h** | 0290h | 0291h | 01B7h | 0293h | 0294h | 0295h | 0296h | 0297h |
| **0298h** | 0298h | 0299h | 029Ah | 029Bh | 029Ch | 029Dh | 029Eh | 029Fh |
| **02A0h** | 02A0h | 02A1h | 02A2h | 02A3h | 02A4h | 02A5h | 02A6h | 02A7h |
| **02A8h** | 02A8h | 02A9h | 02AAh | 02ABh | 02ACh | 02ADh | 02AEh | 02AFh |
| **02B0h** | 02B0h | 02B1h | 02B2h | 02B3h | 02B4h | 02B5h | 02B6h | 02B7h |
| **02B8h** | 02B8h | 02B9h | 02BAh | 02BBh | 02BCh | 02BDh | 02BEh | 02BFh |
| **02C0h** | 02C0h | 02C1h | 02C2h | 02C3h | 02C4h | 02C5h | 02C6h | 02C7h |
| **02C8h** | 02C8h | 02C9h | 02CAh | 02CBh | 02CCh | 02CDh | 02CEh | 02CFh |
| **02D0h** | 02D0h | 02D1h | 02D2h | 02D3h | 02D4h | 02D5h | 02D6h | 02D7h |
| **02D8h** | 02D8h | 02D9h | 02DAh | 02DBh | 02DCh | 02DDh | 02DEh | 02DFh |
| **02E0h** | 02E0h | 02E1h | 02E2h | 02E3h | 02E4h | 02E5h | 02E6h | 02E7h |
| **02E8h** | 02E8h | 02E9h | 02EAh | 02EBh | 02ECh | 02EDh | 02EEh | 02EFh |
| **02F0h** | 02F0h | 02F1h | 02F2h | 02F3h | 02F4h | 02F5h | 02F6h | 02F7h |
| **02F8h** | 02F8h | 02F9h | 02FAh | 02FBh | 02FCh | 02FDh | 02FEh | 02FFh |
| **0300h** | 0300h | 0301h | 0302h | 0303h | 0304h | 0305h | 0306h | 0307h |
| **0308h** | 0308h | 0309h | 030Ah | 030Bh | 030Ch | 030Dh | 030Eh | 030Fh |
| **0310h** | 0310h | 0311h | 0312h | 0313h | 0314h | 0315h | 0316h | 0317h |
| **0318h** | 0318h | 0319h | 031Ah | 031Bh | 031Ch | 031Dh | 031Eh | 031Fh |
| **0320h** | 0320h | 0321h | 0322h | 0323h | 0324h | 0325h | 0326h | 0327h |
| **0328h** | 0328h | 0329h | 032Ah | 032Bh | 032Ch | 032Dh | 032Eh | 032Fh |
| **0330h** | 0330h | 0331h | 0332h | 0333h | 0334h | 0335h | 0336h | 0337h |
| **0338h** | 0338h | 0339h | 033Ah | 033Bh | 033Ch | 033Dh | 033Eh | 033Fh |
| **0340h** | 0340h | 0341h | 0342h | 0343h | 0344h | 0345h | 0346h | 0347h |
| **0348h** | 0348h | 0349h | 034Ah | 034Bh | 034Ch | 034Dh | 034Eh | 034Fh |
| **0350h** | 0350h | 0351h | 0352h | 0353h | 0354h | 0355h | 0356h | 0357h |
| **0358h** | 0358h | 0359h | 035Ah | 035Bh | 035Ch | 035Dh | 035Eh | 035Fh |
| **0360h** | 0360h | 0361h | 0362h | 0363h | 0364h | 0365h | 0366h | 0367h |
| **0368h** | 0368h | 0369h | 036Ah | 036Bh | 036Ch | 036Dh | 036Eh | 036Fh |
| **0370h** | 0370h | 0371h | 0372h | 0373h | 0374h | 0375h | 0376h | 0377h |
| **0378h** | 0378h | 0379h | 037Ah | 03FDh | 03FEh | 03FFh | 037Eh | 037Fh |
| **0380h** | 0380h | 0381h | 0382h | 0383h | 0384h | 0385h | 0386h | 0387h |
| **0388h** | 0388h | 0389h | 038Ah | 038Bh | 038Ch | 038Dh | 038Eh | 038Fh |
| **0390h** | 0390h | 0391h | 0392h | 0393h | 0394h | 0395h | 0396h | 0397h |
| **0398h** | 0398h | 0399h | 039Ah | 039Bh | 039Ch | 039Dh | 039Eh | 039Fh |
| **03A0h** | 03A0h | 03A1h | 03A2h | 03A3h | 03A4h | 03A5h | 03A6h | 03A7h |
| **03A8h** | 03A8h | 03A9h | 03AAh | 03ABh | 0386h | 0388h | 0389h | 038Ah |
| **03B0h** | 03B0h | 0391h | 0392h | 0393h | 0394h | 0395h | 0396h | 0397h |
| **03B8h** | 0398h | 0399h | 039Ah | 039Bh | 039Ch | 039Dh | 039Eh | 039Fh |
| **03C0h** | 03A0h | 03A1h | 03A3h | 03A3h | 03A4h | 03A5h | 03A6h | 03A7h |
| **03C8h** | 03A8h | 03A9h | 03AAh | 03ABh | 038Ch | 038Eh | 038Fh | 03CFh |
| **03D0h** | 03D0h | 03D1h | 03D2h | 03D3h | 03D4h | 03D5h | 03D6h | 03D7h |
| **03D8h** | 03D8h | 03D8h | 03DAh | 03DAh | 03DCh | 03DCh | 03DEh | 03DEh |
| **03E0h** | 03E0h | 03E0h | 03E2h | 03E2h | 03E4h | 03E4h | 03E6h | 03E6h |
| **03E8h** | 03E8h | 03E8h | 03EAh | 03EAh | 03ECh | 03ECh | 03EEh | 03EEh |
| **03F0h** | 03F0h | 03F1h | 03F9h | 03F3h | 03F4h | 03F5h | 03F6h | 03F7h |
| **03F8h** | 03F7h | 03F9h | 03FAh | 03FAh | 03FCh | 03FDh | 03FEh | 03FFh |
| **0400h** | 0400h | 0401h | 0402h | 0403h | 0404h | 0405h | 0406h | 0407h |
| **0408h** | 0408h | 0409h | 040Ah | 040Bh | 040Ch | 040Dh | 040Eh | 040Fh |
| **0410h** | 0410h | 0411h | 0412h | 0413h | 0414h | 0415h | 0416h | 0417h |
| **0418h** | 0418h | 0419h | 041Ah | 041Bh | 041Ch | 041Dh | 041Eh | 041Fh |
| **0420h** | 0420h | 0421h | 0422h | 0423h | 0424h | 0425h | 0426h | 0427h |
| **0428h** | 0428h | 0429h | 042Ah | 042Bh | 042Ch | 042Dh | 042Eh | 042Fh |
| **0430h** | 0410h | 0411h | 0412h | 0413h | 0414h | 0415h | 0416h | 0417h |
| **0438h** | 0418h | 0419h | 041Ah | 041Bh | 041Ch | 041Dh | 041Eh | 041Fh |
| **0440h** | 0420h | 0421h | 0422h | 0423h | 0424h | 0425h | 0426h | 0427h |
| **0448h** | 0428h | 0429h | 042Ah | 042Bh | 042Ch | 042Dh | 042Eh | 042Fh |
| **0450h** | 0400h | 0401h | 0402h | 0403h | 0404h | 0405h | 0406h | 0407h |
| **0458h** | 0408h | 0409h | 040Ah | 040Bh | 040Ch | 040Dh | 040Eh | 040Fh |
| **0460h** | 0460h | 0460h | 0462h | 0462h | 0464h | 0464h | 0466h | 0466h |
| **0468h** | 0468h | 0468h | 046Ah | 046Ah | 046Ch | 046Ch | 046Eh | 046Eh |
| **0470h** | 0470h | 0470h | 0472h | 0472h | 0474h | 0474h | 0476h | 0476h |
| **0478h** | 0478h | 0478h | 047Ah | 047Ah | 047Ch | 047Ch | 047Eh | 047Eh |
| **0480h** | 0480h | 0480h | 0482h | 0483h | 0484h | 0485h | 0486h | 0487h |
| **0488h** | 0488h | 0489h | 048Ah | 048Ah | 048Ch | 048Ch | 048Eh | 048Eh |
| **0490h** | 0490h | 0490h | 0492h | 0492h | 0494h | 0494h | 0496h | 0496h |
| **0498h** | 0498h | 0498h | 049Ah | 049Ah | 049Ch | 049Ch | 049Eh | 049Eh |
| **04A0h** | 04A0h | 04A0h | 04A2h | 04A2h | 04A4h | 04A4h | 04A6h | 04A6h |
| **04A8h** | 04A8h | 04A8h | 04AAh | 04AAh | 04ACh | 04ACh | 04AEh | 04AEh |
| **04B0h** | 04B0h | 04B0h | 04B2h | 04B2h | 04B4h | 04B4h | 04B6h | 04B6h |
| **04B8h** | 04B8h | 04B8h | 04BAh | 04BAh | 04BCh | 04BCh | 04BEh | 04BEh |
| **04C0h** | 04C0h | 04C1h | 04C1h | 04C3h | 04C3h | 04C5h | 04C5h | 04C7h |
| **04C8h** | 04C7h | 04C9h | 04C9h | 04CBh | 04CBh | 04CDh | 04CDh | 04C0h |
| **04D0h** | 04D0h | 04D0h | 04D2h | 04D2h | 04D4h | 04D4h | 04D6h | 04D6h |
| **04D8h** | 04D8h | 04D8h | 04DAh | 04DAh | 04DCh | 04DCh | 04DEh | 04DEh |
| **04E0h** | 04E0h | 04E0h | 04E2h | 04E2h | 04E4h | 04E4h | 04E6h | 04E6h |
| **04E8h** | 04E8h | 04E8h | 04EAh | 04EAh | 04ECh | 04ECh | 04EEh | 04EEh |
| **04F0h** | 04F0h | 04F0h | 04F2h | 04F2h | 04F4h | 04F4h | 04F6h | 04F6h |
| **04F8h** | 04F8h | 04F8h | 04FAh | 04FAh | 04FCh | 04FCh | 04FEh | 04FEh |
| **0500h** | 0500h | 0500h | 0502h | 0502h | 0504h | 0504h | 0506h | 0506h |
| **0508h** | 0508h | 0508h | 050Ah | 050Ah | 050Ch | 050Ch | 050Eh | 050Eh |
| **0510h** | 0510h | 0510h | 0512h | 0512h | 0514h | 0515h | 0516h | 0517h |
| **0518h** | 0518h | 0519h | 051Ah | 051Bh | 051Ch | 051Dh | 051Eh | 051Fh |
| **0520h** | 0520h | 0521h | 0522h | 0523h | 0524h | 0525h | 0526h | 0527h |
| **0528h** | 0528h | 0529h | 052Ah | 052Bh | 052Ch | 052Dh | 052Eh | 052Fh |
| **0530h** | 0530h | 0531h | 0532h | 0533h | 0534h | 0535h | 0536h | 0537h |
| **0538h** | 0538h | 0539h | 053Ah | 053Bh | 053Ch | 053Dh | 053Eh | 053Fh |
| **0540h** | 0540h | 0541h | 0542h | 0543h | 0544h | 0545h | 0546h | 0547h |
| **0548h** | 0548h | 0549h | 054Ah | 054Bh | 054Ch | 054Dh | 054Eh | 054Fh |
| **0550h** | 0550h | 0551h | 0552h | 0553h | 0554h | 0555h | 0556h | 0557h |
| **0558h** | 0558h | 0559h | 055Ah | 055Bh | 055Ch | 055Dh | 055Eh | 055Fh |
| **0560h** | 0560h | 0531h | 0532h | 0533h | 0534h | 0535h | 0536h | 0537h |
| **0568h** | 0538h | 0539h | 053Ah | 053Bh | 053Ch | 053Dh | 053Eh | 053Fh |
| **0570h** | 0540h | 0541h | 0542h | 0543h | 0544h | 0545h | 0546h | 0547h |
| **0578h** | 0548h | 0549h | 054Ah | 054Bh | 054Ch | 054Dh | 054Eh | 054Fh |
| **0580h** | 0550h | 0551h | 0552h | 0553h | 0554h | 0555h | 0556h | FFFFh |
| **0588h** | 17F6h | 2C63h | 1D7Eh | 1D7Fh | 1D80h | 1D81h | 1D82h | 1D83h |
| **0590h** | 1D84h | 1D85h | 1D86h | 1D87h | 1D88h | 1D89h | 1D8Ah | 1D8Bh |
| **0598h** | 1D8Ch | 1D8Dh | 1D8Eh | 1D8Fh | 1D90h | 1D91h | 1D92h | 1D93h |
| **05A0h** | 1D94h | 1D95h | 1D96h | 1D97h | 1D98h | 1D99h | 1D9Ah | 1D9Bh |
| **05A8h** | 1D9Ch | 1D9Dh | 1D9Eh | 1D9Fh | 1DA0h | 1DA1h | 1DA2h | 1DA3h |
| **05B0h** | 1DA4h | 1DA5h | 1DA6h | 1DA7h | 1DA8h | 1DA9h | 1DAAh | 1DABh |
| **05B8h** | 1DACh | 1DADh | 1DAEh | 1DAFh | 1DB0h | 1DB1h | 1DB2h | 1DB3h |
| **05C0h** | 1DB4h | 1DB5h | 1DB6h | 1DB7h | 1DB8h | 1DB9h | 1DBAh | 1DBBh |
| **05C8h** | 1DBCh | 1DBDh | 1DBEh | 1DBFh | 1DC0h | 1DC1h | 1DC2h | 1DC3h |
| **05D0h** | 1DC4h | 1DC5h | 1DC6h | 1DC7h | 1DC8h | 1DC9h | 1DCAh | 1DCBh |
| **05D8h** | 1DCCh | 1DCDh | 1DCEh | 1DCFh | 1DD0h | 1DD1h | 1DD2h | 1DD3h |
| **05E0h** | 1DD4h | 1DD5h | 1DD6h | 1DD7h | 1DD8h | 1DD9h | 1DDAh | 1DDBh |
| **05E8h** | 1DDCh | 1DDDh | 1DDEh | 1DDFh | 1DE0h | 1DE1h | 1DE2h | 1DE3h |
| **05F0h** | 1DE4h | 1DE5h | 1DE6h | 1DE7h | 1DE8h | 1DE9h | 1DEAh | 1DEBh |
| **05F8h** | 1DECh | 1DEDh | 1DEEh | 1DEFh | 1DF0h | 1DF1h | 1DF2h | 1DF3h |
| **0600h** | 1DF4h | 1DF5h | 1DF6h | 1DF7h | 1DF8h | 1DF9h | 1DFAh | 1DFBh |
| **0608h** | 1DFCh | 1DFDh | 1DFEh | 1DFFh | 1E00h | 1E00h | 1E02h | 1E02h |
| **0610h** | 1E04h | 1E04h | 1E06h | 1E06h | 1E08h | 1E08h | 1E0Ah | 1E0Ah |
| **0618h** | 1E0Ch | 1E0Ch | 1E0Eh | 1E0Eh | 1E10h | 1E10h | 1E12h | 1E12h |
| **0620h** | 1E14h | 1E14h | 1E16h | 1E16h | 1E18h | 1E18h | 1E1Ah | 1E1Ah |
| **0628h** | 1E1Ch | 1E1Ch | 1E1Eh | 1E1Eh | 1E20h | 1E20h | 1E22h | 1E22h |
| **0630h** | 1E24h | 1E24h | 1E26h | 1E26h | 1E28h | 1E28h | 1E2Ah | 1E2Ah |
| **0638h** | 1E2Ch | 1E2Ch | 1E2Eh | 1E2Eh | 1E30h | 1E30h | 1E32h | 1E32h |
| **0640h** | 1E34h | 1E34h | 1E36h | 1E36h | 1E38h | 1E38h | 1E3Ah | 1E3Ah |
| **0648h** | 1E3Ch | 1E3Ch | 1E3Eh | 1E3Eh | 1E40h | 1E40h | 1E42h | 1E42h |
| **0650h** | 1E44h | 1E44h | 1E46h | 1E46h | 1E48h | 1E48h | 1E4Ah | 1E4Ah |
| **0658h** | 1E4Ch | 1E4Ch | 1E4Eh | 1E4Eh | 1E50h | 1E50h | 1E52h | 1E52h |
| **0660h** | 1E54h | 1E54h | 1E56h | 1E56h | 1E58h | 1E58h | 1E5Ah | 1E5Ah |
| **0668h** | 1E5Ch | 1E5Ch | 1E5Eh | 1E5Eh | 1E60h | 1E60h | 1E62h | 1E62h |
| **0670h** | 1E64h | 1E64h | 1E66h | 1E66h | 1E68h | 1E68h | 1E6Ah | 1E6Ah |
| **0678h** | 1E6Ch | 1E6Ch | 1E6Eh | 1E6Eh | 1E70h | 1E70h | 1E72h | 1E72h |
| **0680h** | 1E74h | 1E74h | 1E76h | 1E76h | 1E78h | 1E78h | 1E7Ah | 1E7Ah |
| **0688h** | 1E7Ch | 1E7Ch | 1E7Eh | 1E7Eh | 1E80h | 1E80h | 1E82h | 1E82h |
| **0690h** | 1E84h | 1E84h | 1E86h | 1E86h | 1E88h | 1E88h | 1E8Ah | 1E8Ah |
| **0698h** | 1E8Ch | 1E8Ch | 1E8Eh | 1E8Eh | 1E90h | 1E90h | 1E92h | 1E92h |
| **06A0h** | 1E94h | 1E94h | 1E96h | 1E97h | 1E98h | 1E99h | 1E9Ah | 1E9Bh |
| **06A8h** | 1E9Ch | 1E9Dh | 1E9Eh | 1E9Fh | 1EA0h | 1EA0h | 1EA2h | 1EA2h |
| **06B0h** | 1EA4h | 1EA4h | 1EA6h | 1EA6h | 1EA8h | 1EA8h | 1EAAh | 1EAAh |
| **06B8h** | 1EACh | 1EACh | 1EAEh | 1EAEh | 1EB0h | 1EB0h | 1EB2h | 1EB2h |
| **06C0h** | 1EB4h | 1EB4h | 1EB6h | 1EB6h | 1EB8h | 1EB8h | 1EBAh | 1EBAh |
| **06C8h** | 1EBCh | 1EBCh | 1EBEh | 1EBEh | 1EC0h | 1EC0h | 1EC2h | 1EC2h |
| **06D0h** | 1EC4h | 1EC4h | 1EC6h | 1EC6h | 1EC8h | 1EC8h | 1ECAh | 1ECAh |
| **06D8h** | 1ECCh | 1ECCh | 1ECEh | 1ECEh | 1ED0h | 1ED0h | 1ED2h | 1ED2h |
| **06E0h** | 1ED4h | 1ED4h | 1ED6h | 1ED6h | 1ED8h | 1ED8h | 1EDAh | 1EDAh |
| **06E8h** | 1EDCh | 1EDCh | 1EDEh | 1EDEh | 1EE0h | 1EE0h | 1EE2h | 1EE2h |
| **06F0h** | 1EE4h | 1EE4h | 1EE6h | 1EE6h | 1EE8h | 1EE8h | 1EEAh | 1EEAh |
| **06F8h** | 1EECh | 1EECh | 1EEEh | 1EEEh | 1EF0h | 1EF0h | 1EF2h | 1EF2h |
| **0700h** | 1EF4h | 1EF4h | 1EF6h | 1EF6h | 1EF8h | 1EF8h | 1EFAh | 1EFBh |
| **0708h** | 1EFCh | 1EFDh | 1EFEh | 1EFFh | 1F08h | 1F09h | 1F0Ah | 1F0Bh |
| **0710h** | 1F0Ch | 1F0Dh | 1F0Eh | 1F0Fh | 1F08h | 1F09h | 1F0Ah | 1F0Bh |
| **0718h** | 1F0Ch | 1F0Dh | 1F0Eh | 1F0Fh | 1F18h | 1F19h | 1F1Ah | 1F1Bh |
| **0720h** | 1F1Ch | 1F1Dh | 1F16h | 1F17h | 1F18h | 1F19h | 1F1Ah | 1F1Bh |
| **0728h** | 1F1Ch | 1F1Dh | 1F1Eh | 1F1Fh | 1F28h | 1F29h | 1F2Ah | 1F2Bh |
| **0730h** | 1F2Ch | 1F2Dh | 1F2Eh | 1F2Fh | 1F28h | 1F29h | 1F2Ah | 1F2Bh |
| **0738h** | 1F2Ch | 1F2Dh | 1F2Eh | 1F2Fh | 1F38h | 1F39h | 1F3Ah | 1F3Bh |
| **0740h** | 1F3Ch | 1F3Dh | 1F3Eh | 1F3Fh | 1F38h | 1F39h | 1F3Ah | 1F3Bh |
| **0748h** | 1F3Ch | 1F3Dh | 1F3Eh | 1F3Fh | 1F48h | 1F49h | 1F4Ah | 1F4Bh |
| **0750h** | 1F4Ch | 1F4Dh | 1F46h | 1F47h | 1F48h | 1F49h | 1F4Ah | 1F4Bh |
| **0758h** | 1F4Ch | 1F4Dh | 1F4Eh | 1F4Fh | 1F50h | 1F59h | 1F52h | 1F5Bh |
| **0760h** | 1F54h | 1F5Dh | 1F56h | 1F5Fh | 1F58h | 1F59h | 1F5Ah | 1F5Bh |
| **0768h** | 1F5Ch | 1F5Dh | 1F5Eh | 1F5Fh | 1F68h | 1F69h | 1F6Ah | 1F6Bh |
| **0770h** | 1F6Ch | 1F6Dh | 1F6Eh | 1F6Fh | 1F68h | 1F69h | 1F6Ah | 1F6Bh |
| **0778h** | 1F6Ch | 1F6Dh | 1F6Eh | 1F6Fh | 1FBAh | 1FBBh | 1FC8h | 1FC9h |
| **0780h** | 1FCAh | 1FCBh | 1FDAh | 1FDBh | 1FF8h | 1FF9h | 1FEAh | 1FEBh |
| **0788h** | 1FFAh | 1FFBh | 1F7Eh | 1F7Fh | 1F88h | 1F89h | 1F8Ah | 1F8Bh |
| **0790h** | 1F8Ch | 1F8Dh | 1F8Eh | 1F8Fh | 1F88h | 1F89h | 1F8Ah | 1F8Bh |
| **0798h** | 1F8Ch | 1F8Dh | 1F8Eh | 1F8Fh | 1F98h | 1F99h | 1F9Ah | 1F9Bh |
| **07A0h** | 1F9Ch | 1F9Dh | 1F9Eh | 1F9Fh | 1F98h | 1F99h | 1F9Ah | 1F9Bh |
| **07A8h** | 1F9Ch | 1F9Dh | 1F9Eh | 1F9Fh | 1FA8h | 1FA9h | 1FAAh | 1FABh |
| **07B0h** | 1FACh | 1FADh | 1FAEh | 1FAFh | 1FA8h | 1FA9h | 1FAAh | 1FABh |
| **07B8h** | 1FACh | 1FADh | 1FAEh | 1FAFh | 1FB8h | 1FB9h | 1FB2h | 1FBCh |
| **07C0h** | 1FB4h | 1FB5h | 1FB6h | 1FB7h | 1FB8h | 1FB9h | 1FBAh | 1FBBh |
| **07C8h** | 1FBCh | 1FBDh | 1FBEh | 1FBFh | 1FC0h | 1FC1h | 1FC2h | 1FC3h |
| **07D0h** | 1FC4h | 1FC5h | 1FC6h | 1FC7h | 1FC8h | 1FC9h | 1FCAh | 1FCBh |
| **07D8h** | 1FC3h | 1FCDh | 1FCEh | 1FCFh | 1FD8h | 1FD9h | 1FD2h | 1FD3h |
| **07E0h** | 1FD4h | 1FD5h | 1FD6h | 1FD7h | 1FD8h | 1FD9h | 1FDAh | 1FDBh |
| **07E8h** | 1FDCh | 1FDDh | 1FDEh | 1FDFh | 1FE8h | 1FE9h | 1FE2h | 1FE3h |
| **07F0h** | 1FE4h | 1FECh | 1FE6h | 1FE7h | 1FE8h | 1FE9h | 1FEAh | 1FEBh |
| **07F8h** | 1FECh | 1FEDh | 1FEEh | 1FEFh | 1FF0h | 1FF1h | 1FF2h | 1FF3h |
| **0800h** | 1FF4h | 1FF5h | 1FF6h | 1FF7h | 1FF8h | 1FF9h | 1FFAh | 1FFBh |
| **0808h** | 1FF3h | 1FFDh | 1FFEh | 1FFFh | 2000h | 2001h | 2002h | 2003h |
| **0810h** | 2004h | 2005h | 2006h | 2007h | 2008h | 2009h | 200Ah | 200Bh |
| **0818h** | 200Ch | 200Dh | 200Eh | 200Fh | 2010h | 2011h | 2012h | 2013h |
| **0820h** | 2014h | 2015h | 2016h | 2017h | 2018h | 2019h | 201Ah | 201Bh |
| **0828h** | 201Ch | 201Dh | 201Eh | 201Fh | 2020h | 2021h | 2022h | 2023h |
| **0830h** | 2024h | 2025h | 2026h | 2027h | 2028h | 2029h | 202Ah | 202Bh |
| **0838h** | 202Ch | 202Dh | 202Eh | 202Fh | 2030h | 2031h | 2032h | 2033h |
| **0840h** | 2034h | 2035h | 2036h | 2037h | 2038h | 2039h | 203Ah | 203Bh |
| **0848h** | 203Ch | 203Dh | 203Eh | 203Fh | 2040h | 2041h | 2042h | 2043h |
| **0850h** | 2044h | 2045h | 2046h | 2047h | 2048h | 2049h | 204Ah | 204Bh |
| **0858h** | 204Ch | 204Dh | 204Eh | 204Fh | 2050h | 2051h | 2052h | 2053h |
| **0860h** | 2054h | 2055h | 2056h | 2057h | 2058h | 2059h | 205Ah | 205Bh |
| **0868h** | 205Ch | 205Dh | 205Eh | 205Fh | 2060h | 2061h | 2062h | 2063h |
| **0870h** | 2064h | 2065h | 2066h | 2067h | 2068h | 2069h | 206Ah | 206Bh |
| **0878h** | 206Ch | 206Dh | 206Eh | 206Fh | 2070h | 2071h | 2072h | 2073h |
| **0880h** | 2074h | 2075h | 2076h | 2077h | 2078h | 2079h | 207Ah | 207Bh |
| **0888h** | 207Ch | 207Dh | 207Eh | 207Fh | 2080h | 2081h | 2082h | 2083h |
| **0890h** | 2084h | 2085h | 2086h | 2087h | 2088h | 2089h | 208Ah | 208Bh |
| **0898h** | 208Ch | 208Dh | 208Eh | 208Fh | 2090h | 2091h | 2092h | 2093h |
| **08A0h** | 2094h | 2095h | 2096h | 2097h | 2098h | 2099h | 209Ah | 209Bh |
| **08A8h** | 209Ch | 209Dh | 209Eh | 209Fh | 20A0h | 20A1h | 20A2h | 20A3h |
| **08B0h** | 20A4h | 20A5h | 20A6h | 20A7h | 20A8h | 20A9h | 20AAh | 20ABh |
| **08B8h** | 20ACh | 20ADh | 20AEh | 20AFh | 20B0h | 20B1h | 20B2h | 20B3h |
| **08C0h** | 20B4h | 20B5h | 20B6h | 20B7h | 20B8h | 20B9h | 20BAh | 20BBh |
| **08C8h** | 20BCh | 20BDh | 20BEh | 20BFh | 20C0h | 20C1h | 20C2h | 20C3h |
| **08D0h** | 20C4h | 20C5h | 20C6h | 20C7h | 20C8h | 20C9h | 20CAh | 20CBh |
| **08D8h** | 20CCh | 20CDh | 20CEh | 20CFh | 20D0h | 20D1h | 20D2h | 20D3h |
| **08E0h** | 20D4h | 20D5h | 20D6h | 20D7h | 20D8h | 20D9h | 20DAh | 20DBh |
| **08E8h** | 20DCh | 20DDh | 20DEh | 20DFh | 20E0h | 20E1h | 20E2h | 20E3h |
| **08F0h** | 20E4h | 20E5h | 20E6h | 20E7h | 20E8h | 20E9h | 20EAh | 20EBh |
| **08F8h** | 20ECh | 20EDh | 20EEh | 20EFh | 20F0h | 20F1h | 20F2h | 20F3h |
| **0900h** | 20F4h | 20F5h | 20F6h | 20F7h | 20F8h | 20F9h | 20FAh | 20FBh |
| **0908h** | 20FCh | 20FDh | 20FEh | 20FFh | 2100h | 2101h | 2102h | 2103h |
| **0910h** | 2104h | 2105h | 2106h | 2107h | 2108h | 2109h | 210Ah | 210Bh |
| **0918h** | 210Ch | 210Dh | 210Eh | 210Fh | 2110h | 2111h | 2112h | 2113h |
| **0920h** | 2114h | 2115h | 2116h | 2117h | 2118h | 2119h | 211Ah | 211Bh |
| **0928h** | 211Ch | 211Dh | 211Eh | 211Fh | 2120h | 2121h | 2122h | 2123h |
| **0930h** | 2124h | 2125h | 2126h | 2127h | 2128h | 2129h | 212Ah | 212Bh |
| **0938h** | 212Ch | 212Dh | 212Eh | 212Fh | 2130h | 2131h | 2132h | 2133h |
| **0940h** | 2134h | 2135h | 2136h | 2137h | 2138h | 2139h | 213Ah | 213Bh |
| **0948h** | 213Ch | 213Dh | 213Eh | 213Fh | 2140h | 2141h | 2142h | 2143h |
| **0950h** | 2144h | 2145h | 2146h | 2147h | 2148h | 2149h | 214Ah | 214Bh |
| **0958h** | 214Ch | 214Dh | 2132h | 214Fh | 2150h | 2151h | 2152h | 2153h |
| **0960h** | 2154h | 2155h | 2156h | 2157h | 2158h | 2159h | 215Ah | 215Bh |
| **0968h** | 215Ch | 215Dh | 215Eh | 215Fh | 2160h | 2161h | 2162h | 2163h |
| **0970h** | 2164h | 2165h | 2166h | 2167h | 2168h | 2169h | 216Ah | 216Bh |
| **0978h** | 216Ch | 216Dh | 216Eh | 216Fh | 2160h | 2161h | 2162h | 2163h |
| **0980h** | 2164h | 2165h | 2166h | 2167h | 2168h | 2169h | 216Ah | 216Bh |
| **0988h** | 216Ch | 216Dh | 216Eh | 216Fh | 2180h | 2181h | 2182h | 2183h |
| **0990h** | 2183h | FFFFh | 034Bh | 24B6h | 24B7h | 24B8h | 24B9h | 24BAh |
| **0998h** | 24BBh | 24BCh | 24BDh | 24BEh | 24BFh | 24C0h | 24C1h | 24C2h |
| **09A0h** | 24C3h | 24C4h | 24C5h | 24C6h | 24C7h | 24C8h | 24C9h | 24CAh |
| **09A8h** | 24CBh | 24CCh | 24CDh | 24CEh | 24CFh | FFFFh | 0746h | 2C00h |
| **09B0h** | 2C01h | 2C02h | 2C03h | 2C04h | 2C05h | 2C06h | 2C07h | 2C08h |
| **09B8h** | 2C09h | 2C0Ah | 2C0Bh | 2C0Ch | 2C0Dh | 2C0Eh | 2C0Fh | 2C10h |
| **09C0h** | 2C11h | 2C12h | 2C13h | 2C14h | 2C15h | 2C16h | 2C17h | 2C18h |
| **09C8h** | 2C19h | 2C1Ah | 2C1Bh | 2C1Ch | 2C1Dh | 2C1Eh | 2C1Fh | 2C20h |
| **09D0h** | 2C21h | 2C22h | 2C23h | 2C24h | 2C25h | 2C26h | 2C27h | 2C28h |
| **09D8h** | 2C29h | 2C2Ah | 2C2Bh | 2C2Ch | 2C2Dh | 2C2Eh | 2C5Fh | 2C60h |
| **09E0h** | 2C60h | 2C62h | 2C63h | 2C64h | 2C65h | 2C66h | 2C67h | 2C67h |
| **09E8h** | 2C69h | 2C69h | 2C6Bh | 2C6Bh | 2C6Dh | 2C6Eh | 2C6Fh | 2C70h |
| **09F0h** | 2C71h | 2C72h | 2C73h | 2C74h | 2C75h | 2C75h | 2C77h | 2C78h |
| **09F8h** | 2C79h | 2C7Ah | 2C7Bh | 2C7Ch | 2C7Dh | 2C7Eh | 2C7Fh | 2C80h |
| **0A00h** | 2C80h | 2C82h | 2C82h | 2C84h | 2C84h | 2C86h | 2C86h | 2C88h |
| **0A08h** | 2C88h | 2C8Ah | 2C8Ah | 2C8Ch | 2C8Ch | 2C8Eh | 2C8Eh | 2C90h |
| **0A10h** | 2C90h | 2C92h | 2C92h | 2C94h | 2C94h | 2C96h | 2C96h | 2C98h |
| **0A18h** | 2C98h | 2C9Ah | 2C9Ah | 2C9Ch | 2C9Ch | 2C9Eh | 2C9Eh | 2CA0h |
| **0A20h** | 2CA0h | 2CA2h | 2CA2h | 2CA4h | 2CA4h | 2CA6h | 2CA6h | 2CA8h |
| **0A28h** | 2CA8h | 2CAAh | 2CAAh | 2CACh | 2CACh | 2CAEh | 2CAEh | 2CB0h |
| **0A30h** | 2CB0h | 2CB2h | 2CB2h | 2CB4h | 2CB4h | 2CB6h | 2CB6h | 2CB8h |
| **0A38h** | 2CB8h | 2CBAh | 2CBAh | 2CBCh | 2CBCh | 2CBEh | 2CBEh | 2CC0h |
| **0A40h** | 2CC0h | 2CC2h | 2CC2h | 2CC4h | 2CC4h | 2CC6h | 2CC6h | 2CC8h |
| **0A48h** | 2CC8h | 2CCAh | 2CCAh | 2CCCh | 2CCCh | 2CCEh | 2CCEh | 2CD0h |
| **0A50h** | 2CD0h | 2CD2h | 2CD2h | 2CD4h | 2CD4h | 2CD6h | 2CD6h | 2CD8h |
| **0A58h** | 2CD8h | 2CDAh | 2CDAh | 2CDCh | 2CDCh | 2CDEh | 2CDEh | 2CE0h |
| **0A60h** | 2CE0h | 2CE2h | 2CE2h | 2CE4h | 2CE5h | 2CE6h | 2CE7h | 2CE8h |
| **0A68h** | 2CE9h | 2CEAh | 2CEBh | 2CECh | 2CEDh | 2CEEh | 2CEFh | 2CF0h |
| **0A70h** | 2CF1h | 2CF2h | 2CF3h | 2CF4h | 2CF5h | 2CF6h | 2CF7h | 2CF8h |
| **0A78h** | 2CF9h | 2CFAh | 2CFBh | 2CFCh | 2CFDh | 2CFEh | 2CFFh | 10A0h |
| **0A80h** | 10A1h | 10A2h | 10A3h | 10A4h | 10A5h | 10A6h | 10A7h | 10A8h |
| **0A88h** | 10A9h | 10AAh | 10ABh | 10ACh | 10ADh | 10AEh | 10AFh | 10B0h |
| **0A90h** | 10B1h | 10B2h | 10B3h | 10B4h | 10B5h | 10B6h | 10B7h | 10B8h |
| **0A98h** | 10B9h | 10BAh | 10BBh | 10BCh | 10BDh | 10BEh | 10BFh | 10C0h |
| **0AA0h** | 10C1h | 10C2h | 10C3h | 10C4h | 10C5h | FFFFh | D21Bh | FF21h |
| **0AA8h** | FF22h | FF23h | FF24h | FF25h | FF26h | FF27h | FF28h | FF29h |
| **0AB0h** | FF2Ah | FF2Bh | FF2Ch | FF2Dh | FF2Eh | FF2Fh | FF30h | FF31h |
| **0AB8h** | FF32h | FF33h | FF34h | FF35h | FF36h | FF37h | FF38h | FF39h |
| **0AC0h** | FF3Ah | FF5Bh | FF5Ch | FF5Dh | FF5Eh | FF5Fh | FF60h | FF61h |
| **0AC8h** | FF62h | FF63h | FF64h | FF65h | FF66h | FF67h | FF68h | FF69h |
| **0AD0h** | FF6Ah | FF6Bh | FF6Ch | FF6Dh | FF6Eh | FF6Fh | FF70h | FF71h |
| **0AD8h** | FF72h | FF73h | FF74h | FF75h | FF76h | FF77h | FF78h | FF79h |
| **0AE0h** | FF7Ah | FF7Bh | FF7Ch | FF7Dh | FF7Eh | FF7Fh | FF80h | FF81h |
| **0AE8h** | FF82h | FF83h | FF84h | FF85h | FF86h | FF87h | FF88h | FF89h |
| **0AF0h** | FF8Ah | FF8Bh | FF8Ch | FF8Dh | FF8Eh | FF8Fh | FF90h | FF91h |
| **0AF8h** | FF92h | FF93h | FF94h | FF95h | FF96h | FF97h | FF98h | FF99h |
| **0B00h** | FF9Ah | FF9Bh | FF9Ch | FF9Dh | FF9Eh | FF9Fh | FFA0h | FFA1h |
| **0B08h** | FFA2h | FFA3h | FFA4h | FFA5h | FFA6h | FFA7h | FFA8h | FFA9h |
| **0B10h** | FFAAh | FFABh | FFACh | FFADh | FFAEh | FFAFh | FFB0h | FFB1h |
| **0B18h** | FFB2h | FFB3h | FFB4h | FFB5h | FFB6h | FFB7h | FFB8h | FFB9h |
| **0B20h** | FFBAh | FFBBh | FFBCh | FFBDh | FFBEh | FFBFh | FFC0h | FFC1h |
| **0B28h** | FFC2h | FFC3h | FFC4h | FFC5h | FFC6h | FFC7h | FFC8h | FFC9h |
| **0B30h** | FFCAh | FFCBh | FFCCh | FFCDh | FFCEh | FFCFh | FFD0h | FFD1h |
| **0B38h** | FFD2h | FFD3h | FFD4h | FFD5h | FFD6h | FFD7h | FFD8h | FFD9h |
| **0B40h** | FFDAh | FFDBh | FFDCh | FFDDh | FFDEh | FFDFh | FFE0h | FFE1h |
| **0B48h** | FFE2h | FFE3h | FFE4h | FFE5h | FFE6h | FFE7h | FFE8h | FFE9h |
| **0B50h** | FFEAh | FFEBh | FFECh | FFEDh | FFEEh | FFEFh | FFF0h | FFF1h |
| **0B58h** | FFF2h | FFF3h | FFF4h | FFF5h | FFF6h | FFF7h | FFF8h | FFF9h |
| **0B60h** | FFFAh | FFFBh | FFFCh | FFFDh | FFFEh | FFFFh | | |
### 7.3 Volume Label 目录项

Volume Label 是便于终端用户区分存储卷的 Unicode 字符串. 在 exFAT 文件系统中, Volume Label 以 critical primary directory entry 形式存在于根目录 (见表 26). 有效 Volume Label directory entry 数量为 0 至 1.

**表 26 Volume Label DirectoryEntry 结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.3.1 节。 |
| CharacterCount | 1 | 1 | 强制字段; 内容见第 7.3.2 节。 |
| VolumeLabel | 2 | 22 | 强制字段; 内容见第 7.3.3 节。 |
| Reserved | 24 | 8 | 强制字段; 内容为保留。 |

#### 7.3.1 EntryType 字段

EntryType 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1 节).

##### 7.3.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.1 节).

对于 Volume Label directory entry, 本字段唯一有效取值为 3.

##### 7.3.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.2 节).

对于 Volume Label directory entry, 本字段唯一有效取值为 0.

##### 7.3.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.3 节).

##### 7.3.1.4 InUse 字段

InUse 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.4 节).

#### 7.3.2 CharacterCount 字段

CharacterCount 字段应包含 VolumeLabel 字段中 Unicode 字符串的长度.

本字段有效取值范围为:

- 至少为 0, 表示 Unicode 字符串长度为 0 (即无卷标)
- 至多为 11, 表示 Unicode 字符串长度为 11

#### 7.3.3 VolumeLabel 字段

VolumeLabel 字段应包含 Unicode 字符串, 即卷的用户友好名称. VolumeLabel 字段的非法字符集合与 File Name directory entry 的 FileName 字段相同 (见第 7.7.3 节).
### 7.4 File 目录项

File directory entry 描述文件与目录. 其为 critical primary directory entry, 任意目录可含零个或多个 File directory entry (见表 27). 要使 File directory entry 有效, 须紧接其后恰好一个 Stream Extension directory entry 与至少一个 File Name directory entry (分别见第 7.6 节与第 7.7 节).

**表 27 File DirectoryEntry**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.4.1 节。 |
| SecondaryCount | 1 | 1 | 强制字段; 内容见第 7.4.2 节。 |
| SetChecksum | 2 | 2 | 强制字段; 内容见第 7.4.3 节。 |
| FileAttributes | 4 | 2 | 强制字段; 内容见第 7.4.4 节。 |
| Reserved1 | 6 | 2 | 强制字段; 内容为保留。 |
| CreateTimestamp | 8 | 4 | 强制字段; 内容见第 7.4.5 节。 |
| LastModifiedTimestamp | 12 | 4 | 强制字段; 内容见第 7.4.6 节。 |
| LastAccessedTimestamp | 16 | 4 | 强制字段; 内容见第 7.4.7 节。 |
| Create10msIncrement | 20 | 1 | 强制字段; 内容见第 7.4.5 节。 |
| LastModified10msIncrement | 21 | 1 | 强制字段; 内容见第 7.4.6 节。 |
| CreateUtcOffset | 22 | 1 | 强制字段; 内容见第 7.4.5 节。 |
| LastModifiedUtcOffset | 23 | 1 | 强制字段; 内容见第 7.4.6 节。 |
| LastAccessedUtcOffset | 24 | 1 | 强制字段; 内容见第 7.4.7 节。 |
| Reserved2 | 25 | 7 | 强制字段; 内容为保留。 |

#### 7.4.1 EntryType 字段

EntryType 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1 节).

##### 7.4.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.1 节).

对于 File directory entry, 本字段唯一有效取值为 5.

##### 7.4.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.2 节).

对于 File directory entry, 本字段唯一有效取值为 0.

##### 7.4.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.3 节).

##### 7.4.1.4 InUse 字段

InUse 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.4 节).

#### 7.4.2 SecondaryCount 字段

SecondaryCount 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.2 节).

#### 7.4.3 SetChecksum 字段

SetChecksum 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.3 节).

#### 7.4.4 FileAttributes 字段

FileAttributes 字段包含标志位 (见表 28).

**表 28 FileAttributes 字段结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| ReadOnly | 0 | 1 | 强制字段; 符合 MS-DOS 定义。 |
| Hidden | 1 | 1 | 强制字段; 符合 MS-DOS 定义。 |
| System | 2 | 1 | 强制字段; 符合 MS-DOS 定义。 |
| Reserved1 | 3 | 1 | 强制字段; 内容为保留。 |
| Directory | 4 | 1 | 强制字段; 符合 MS-DOS 定义。 |
| Archive | 5 | 1 | 强制字段; 符合 MS-DOS 定义。 |
| Reserved2 | 6 | 10 | 强制字段; 内容为保留。 |

#### 7.4.5 CreateTimestamp, Create10msIncrement 与 CreateUtcOffset 字段

CreateTimestamp 与 CreateTime10msIncrement 字段共同应描述给定文件/目录创建时的本地日期与时间. CreateUtcOffset 字段描述本地日期与时间相对 UTC 的偏移. 实现于创建给定 directory entry set 时, 应设置这些字段.

这些字段应符合 Timestamp, 10msIncrement 与 UtcOffset 字段的定义 (分别见第 7.4.8, 7.4.9 与 7.4.10 节).

#### 7.4.6 LastModifiedTimestamp, LastModified10msIncrement 与 LastModifiedUtcOffset 字段

LastModifiedTimestamp 与 LastModifiedTime10msIncrement 字段共同应描述与给定 Stream Extension directory entry 关联的任一簇内容上次修改时的本地日期与时间. LastModifiedUtcOffset 字段描述本地日期与时间相对 UTC 的偏移. 实现须于下列情况后更新这些字段:

1. 修改与给定 Stream Extension directory entry 关联的任一簇内容之后 (ValidDataLength 字段所描述范围之外的内容除外)
2. 更改 ValidDataLength 或 DataLength 字段取值之后

这些字段应符合 Timestamp, 10msIncrement 与 UtcOffset 字段的定义 (分别见第 7.4.8, 7.4.9 与 7.4.10 节).

#### 7.4.7 LastAccessedTimestamp 与 LastAccessedUtcOffset 字段

LastAccessedTimestamp 字段应描述与给定 Stream Extension directory entry 关联的任一簇内容上次访问时的本地日期与时间. LastAccessedUtcOffset 字段描述本地日期与时间相对 UTC 的偏移. 实现须于下列情况后更新这些字段:

1. 修改与给定 Stream Extension directory entry 关联的任一簇内容之后 (ValidDataLength 之外的内容除外)
2. 更改 ValidDataLength 或 DataLength 字段取值之后

实现宜于读取与给定 Stream Extension directory entry 关联的任一簇内容之后更新这些字段.

这些字段应符合 Timestamp 与 UtcOffset 字段的定义 (分别见第 7.4.8 与 7.4.10 节).

#### 7.4.8 Timestamp 字段

Timestamp 字段描述本地日期与时间, 精度至两秒 (见表 29).

**表 29 Timestamp 字段结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| DoubleSeconds | 0 | 5 | 强制字段; 内容见第 7.4.8.1 节。 |
| Minute | 5 | 6 | 强制字段; 内容见第 7.4.8.2 节。 |
| Hour | 11 | 5 | 强制字段; 内容见第 7.4.8.3 节。 |
| Day | 16 | 5 | 强制字段; 内容见第 7.4.8.4 节。 |
| Month | 21 | 4 | 强制字段; 内容见第 7.4.8.5 节。 |
| Year | 25 | 7 | 强制字段; 内容见第 7.4.8.6 节。 |

##### 7.4.8.1 DoubleSeconds 字段

DoubleSeconds 字段应以两秒为倍数描述 Timestamp 字段的秒部分.

本字段有效取值范围为:

- 0, 表示 0 秒
- 29, 表示 58 秒

##### 7.4.8.2 Minute 字段

Minute 字段应描述 Timestamp 字段的分钟部分.

本字段有效取值范围为:

- 0, 表示 0 分
- 59, 表示 59 分

##### 7.4.8.3 Hour 字段

Hour 字段应描述 Timestamp 字段的小时部分.

本字段有效取值范围为:

- 0, 表示 00:00
- 23, 表示 23:00

##### 7.4.8.4 Day 字段

Day 字段应描述 Timestamp 字段的日部分.

本字段有效取值范围为:

- 1, 表示给定月份第一天
- 给定月份最后一天 (有效日数由月份决定)

##### 7.4.8.5 Month 字段

Month 字段应描述 Timestamp 字段的月部分.

本字段有效取值范围为:

- 至少为 1, 表示一月
- 至多为 12, 表示十二月

##### 7.4.8.6 Year 字段

Year 字段应描述 Timestamp 字段的年部分, 相对 1980 年. 取值 0 表示 1980 年, 127 表示 2107 年.

本字段所有可能取值均有效.

#### 7.4.9 10msIncrement 字段

10msIncrement 字段应以 10 毫秒为倍数为其对应 Timestamp 字段提供附加时间分辨率.

这些字段有效取值范围为:

- 至少为 0, 表示 0 毫秒
- 至多为 199, 表示 1990 毫秒

#### 7.4.10 UtcOffset 字段

UtcOffset 字段 (见表 30) 应描述自 UTC 至其对应 Timestamp 与 10msIncrement 字段所描述本地日期与时间的偏移. 该偏移含时区效应及其他日期时间调整 (如夏令时与地区性夏时变更).

**表 30 UtcOffset 字段结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| OffsetFromUtc | 0 | 7 | 强制字段; 内容见第 7.4.10.1 节. |
| OffsetValid | 7 | 1 | 强制字段; 内容见第 7.4.10.2 节. |

##### 7.4.10.1 OffsetFromUtc 字段

OffsetFromUtc 字段应描述相关 Timestamp 与 10msIncrement 字段所含本地日期与时间相对 UTC 的偏移. 本字段以 15 分钟为间隔描述相对 UTC 的偏移 (见表 31).

**表 31 OffsetFromUtc 字段取值含义**

| **取值** | **有符号十进制等价** | **描述** |
| --- | --- | --- |
| 3Fh | 63 | 本地日期与时间为 UTC + 15:45 |
| 3Eh | 62 | 本地日期与时间为 UTC + 15:30 |
| ... | ... | ... |
| 01h | 1 | 本地日期与时间为 UTC + 00:15 |
| 00h | 0 | 本地日期与时间为 UTC |
| 7Fh | -1 | 本地日期与时间为 UTC – 00:15 |
| ... | ... | ... |
| 41h | -63 | 本地日期与时间为 UTC – 15:45 |
| 40h | -64 | 本地日期与时间为 UTC – 16:00 |

如上表所示, 本字段所有可能取值均有效. 然而, 实现宜仅在下列情况下将本字段记录为 00h:

1. 本地日期与时间实际上与 UTC 相同, 此时 OffsetValid 字段须为 1
2. 本地日期与时间未知, 此时 OffsetValid 字段须为 1, 且实现须视 UTC 为本地日期与时间
3. UTC 未知, 此时 OffsetValid 字段须为 0

若本地日期与时间相对 UTC 的偏移并非 15 分钟间隔的整数倍, 则实现须在 OffsetFromUtc 字段中记录 00h, 并须视 UTC 为本地日期与时间.

##### 7.4.10.2 OffsetValid 字段

OffsetValid 字段应描述 OffsetFromUtc 字段内容是否有效, 如下:

- 0, 表示 OffsetFromUtc 字段内容无效

>
> 且须为 00h
- 1, 表示 OffsetFromUtc 字段内容有效

实现宜仅在无法获得 UTC 以计算 OffsetFromUtc 字段取值时将本字段设为 0. 若本字段为 0, 则实现须将 Timestamp 与 10msIncrement 字段视为与当前本地日期与时间具有相同 UTC 偏移.
### 7.5 Volume GUID 目录项

Volume GUID directory entry 包含 GUID, 使实现能够以唯一且可编程方式区分卷. Volume GUID 以 benign primary directory entry 形式存在于根目录 (见表 32). 有效 Volume GUID directory entry 数量为 0 至 1.

**表 32 Volume GUID DirectoryEntry**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.5.1 节。 |
| SecondaryCount | 1 | 1 | 强制字段; 内容见第 7.5.2 节。 |
| SetChecksum | 2 | 2 | 强制字段; 内容见第 7.5.3 节。 |
| GeneralPrimaryFlags | 4 | 2 | 强制字段; 内容见第 7.5.4 节。 |
| VolumeGuid | 6 | 16 | 强制字段; 内容见第 7.5.5 节。 |
| Reserved | 22 | 10 | 强制字段; 内容为保留。 |

#### 7.5.1 EntryType 字段

EntryType 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1 节).

##### 7.5.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.1 节).

对于 Volume GUID directory entry, 本字段唯一有效取值为 0.

##### 7.5.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.2 节).

对于 Volume GUID directory entry, 本字段唯一有效取值为 1.

##### 7.5.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.3 节).

##### 7.5.1.4 InUse 字段

InUse 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.1.4 节).

#### 7.5.2 SecondaryCount 字段

SecondaryCount 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.2 节).

对于 Volume GUID directory entry, 本字段唯一有效取值为 0.

#### 7.5.3 SetChecksum 字段

SetChecksum 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.3 节).

#### 7.5.4 GeneralPrimaryFlags 字段

GeneralPrimaryFlags 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.4 节), 并将 CustomDefined 字段内容定义为保留.

##### 7.5.4.1 AllocationPossible 字段

AllocationPossible 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.4.1 节).

对于 Volume GUID directory entry, 本字段唯一有效取值为 0.

##### 7.5.4.2 NoFatChain 字段

NoFatChain 字段应符合 Generic Primary DirectoryEntry 模板中的定义 (见第 6.3.4.2 节).

#### 7.5.5 VolumeGuid 字段

VolumeGuid 字段应包含唯一标识给定卷的 GUID.

本字段所有可能取值均有效, 空 GUID {00000000-0000-0000-0000-000000000000} 除外.
### 7.6 Stream Extension 目录项

Stream Extension directory entry 是 File directory entry set 中的 critical secondary directory entry (见表 33). 每个 File directory entry set 中有效 Stream Extension directory entry 数量为 1; 且该 entry 仅在其紧接 File directory entry 之后时有效.

**表 33 Stream Extension DirectoryEntry**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.6.1 节。 |
| GeneralSecondaryFlags | 1 | 1 | 强制字段; 内容见第 7.6.2 节。 |
| Reserved1 | 2 | 1 | 强制字段; 内容为保留。 |
| NameLength | 3 | 1 | 强制字段; 内容见第 7.6.3 节。 |
| NameHash | 4 | 2 | 强制字段; 内容见第 7.6.4 节。 |
| Reserved2 | 6 | 2 | 强制字段; 内容为保留。 |
| ValidDataLength | 8 | 8 | 强制字段; 内容见第 7.6.5 节。 |
| Reserved3 | 16 | 4 | 强制字段; 内容为保留。 |
| FirstCluster | 20 | 4 | 强制字段; 内容见第 7.6.6 节。 |
| DataLength | 24 | 8 | 强制字段; 内容见第 7.6.7 节。 |

#### 7.6.1 EntryType 字段

EntryType 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1 节).

##### 7.6.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.1 节).

对于 Stream Extension directory entry, 本字段唯一有效取值为 0.

##### 7.6.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.2 节).

对于 Stream Extension directory entry, 本字段唯一有效取值为 0.

##### 7.6.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.3 节).

##### 7.6.1.4 InUse 字段

InUse 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.4 节).

#### 7.6.2 GeneralSecondaryFlags 字段

GeneralSecondaryFlags 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2 节), 并将 CustomDefined 字段内容定义为保留.

##### 7.6.2.1 AllocationPossible 字段

AllocationPossible 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2.1 节).

对于本 directory entry, 本字段唯一有效取值为 1.

##### 7.6.2.2 NoFatChain 字段

NoFatChain 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2.2 节).

#### 7.6.3 NameLength 字段

NameLength 字段应包含后续 File Name directory entry (见第 7.7 节) 共同构成的 Unicode 字符串长度.

本字段有效取值范围为:

- 至少为 1, 即最短文件名
- 至多为 255, 即最长文件名

NameLength 字段取值亦影响 File Name Directory Entry 的数量 (见第 7.7 节).

#### 7.6.4 NameHash 字段

NameHash 字段应包含大写化文件名的 2 字节哈希 (见图 4). 这使实现按名搜索文件时可快速比较. 重要的是, NameHash 可确证不匹配. 实现须以比较大写化文件名验证所有 NameHash 匹配.

**图 4 NameHash 计算**

```C
UInt16 NameHash
(
    WCHAR * FileName,    // points to an in-memory copy of the up-cased file name
    UCHAR   NameLength
)
{
    UCHAR  * Buffer = (UCHAR *)FileName;
    UInt16   NumberOfBytes = (UInt16)NameLength * 2;
    UInt16   Hash = 0;
    UInt16   Index;

    for (Index = 0; Index < NumberOfBytes; Index++)
    {
        Hash = ((Hash&1) ? 0x8000 : 0) + (Hash>>1) + (UInt16)Buffer[Index];
    }
    return Hash;
}
```

#### 7.6.5 ValidDataLength 字段

ValidDataLength 字段应描述用户数据已写入数据流的范围. 实现于向数据流更远位置写入数据时, 须更新本字段. 存储介质上, 有效数据长度与数据流 DataLength 之间的数据为未定义. 实现须对超出有效数据长度的读操作返回零.

若对应 File directory entry 描述目录, 则本字段唯一有效取值等于 DataLength 字段取值. 否则, 本字段有效取值范围为:

- 至少为 0, 表示尚未向数据流写入用户数据
- 至多为 DataLength, 表示用户数据已写入数据流全长

#### 7.6.6 FirstCluster 字段

FirstCluster 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.3 节).

本字段应包含承载用户数据的数据流首簇索引.

#### 7.6.7 DataLength 字段

DataLength 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.4 节).

若对应 File directory entry 描述目录, 则本字段有效取值为关联分配的全部字节大小 (可为 0). 此外, 对目录, 本字段最大值为 256MB.
### 7.7 File Name 目录项

File Name directory entry 是 File directory entry set 中的 critical secondary directory entry (见表 34). 每个 set 中有效 File Name directory entry 数量为 NameLength / 15 向上取整. 且 File Name directory entry 仅在其作为连续序列紧接 Stream Extension directory entry 之后时有效. 多个 File Name directory entry 共同构成 File directory entry set 的文件名.

给定目录的所有子项须具有唯一的 File Name Directory Entry Set. 即在同一目录内, 大写化后不得出现重复的文件或目录名.

**表 34 File Name DirectoryEntry**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.7.1 节。 |
| GeneralSecondaryFlags | 1 | 1 | 强制字段; 内容见第 7.7.2 节。 |
| FileName | 2 | 30 | 强制字段; 内容见第 7.7.3 节。 |

#### 7.7.1 EntryType 字段

EntryType 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1 节).

##### 7.7.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.1 节).

对于 File Name directory entry, 本字段唯一有效取值为 1.

##### 7.7.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.2 节).

对于 File Name directory entry, 本字段唯一有效取值为 0.

##### 7.7.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.3 节).

##### 7.7.1.4 InUse 字段

InUse 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.4 节).

#### 7.7.2 GeneralSecondaryFlags 字段

GeneralSecondaryFlags 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2 节), 并将 CustomDefined 字段内容定义为保留.

##### 7.7.2.1 AllocationPossible 字段

AllocationPossible 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2.1 节).

对于本 directory entry, 本字段唯一有效取值为 0.

##### 7.7.2.2 NoFatChain 字段

NoFatChain 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2.2 节).

#### 7.7.3 FileName 字段

FileName 字段应包含 Unicode 字符串, 为文件名的一部分. 按 File Name directory entry 在 File directory entry set 中的顺序, FileName 字段拼接构成该 set 的文件名. 给定 FileName 字段长度 15 字符与最多 17 个 File Name directory entry, 最终拼接文件名最大长度为 255.

拼接后的文件名具有与其他 FAT 类文件系统相同的非法字符集合 (见表 35). 实现宜将 FileName 字段未使用字符设为 0000h.

**表 35 非法 FileName 字符**

| **字符码** | **描述** | **字符码** | **描述** | **字符码** | **描述** |
| --- | --- | --- | --- | --- | --- |
| 0000h | 控制码 | 0001h | 控制码 | 0002h | 控制码 |
| 0003h | 控制码 | 0004h | 控制码 | 0005h | 控制码 |
| 0006h | 控制码 | 0007h | 控制码 | 0008h | 控制码 |
| 0009h | 控制码 | 000Ah | 控制码 | 000Bh | 控制码 |
| 000Ch | 控制码 | 000Dh | 控制码 | 000Eh | 控制码 |
| 000Fh | 控制码 | 0010h | 控制码 | 0011h | 控制码 |
| 0012h | 控制码 | 0013h | 控制码 | 0014h | 控制码 |
| 0015h | 控制码 | 0016h | 控制码 | 0017h | 控制码 |
| 0018h | 控制码 | 0019h | 控制码 | 001Ah | 控制码 |
| 001Bh | 控制码 | 001Ch | 控制码 | 001Dh | 控制码 |
| 001Eh | 控制码 | 001Fh | 控制码 | 0022h | 双引号 |
| 002Ah | 星号 | 002Fh | 正斜杠 | 003Ah | 冒号 |
| 003Ch | 小于号 | 003Eh | 大于号 | 003Fh | 问号 |
| 005Ch | 反斜杠 | 007Ch | 竖线 | | |

文件名 "." 与 ".." 分别表示 "本目录" 与 "父目录". 实现不得在 FileName 字段中记录这两个保留文件名. 然而, 实现可在目录列表中生成这两个文件名, 以指代被列目录及其父目录.

实现可希望将文件与目录名限制为 ASCII 字符集. 若如此, 宜将字符使用限制于前 128 个 Unicode 条目的有效字符范围. 仍须在卷上以 Unicode 存储文件与目录名, 并在与用户接口交互时在 ASCII/Unicode 间转换.
### 7.8 Vendor Extension 目录项

Vendor Extension directory entry 是 File directory entry set 中的 benign secondary directory entry (见表 36). 每个 set 可含任意数量的 Vendor Extension directory entry, 上限为 secondary directory entry 总数减去其他 secondary directory entry 数量. 且 Vendor Extension directory entry 仅在其不位于必需的 Stream Extension 与 File Name directory entry 之前时有效.

Vendor Extension directory entry 通过 VendorGuid 字段 (见表 36) 使厂商能在单个 File directory entry set 中拥有唯一、厂商特定的 directory entry. 唯一目录项实质上使厂商可扩展 exFAT 文件系统. 厂商可定义 VendorDefined 字段 (见表 36) 的内容. 厂商实现可维护 VendorDefined 字段内容并提供厂商特定功能.

无法识别 Vendor Extension directory entry 之 GUID 的实现, 须将该 directory entry 与任何其他无法识别的 benign secondary directory entry 同等对待 (见第 8.2 节).

**表 36 Vendor Extension DirectoryEntry**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.8.1 节。 |
| GeneralSecondaryFlags | 1 | 1 | 强制字段; 内容见第 7.8.2 节。 |
| VendorGuid | 2 | 16 | 强制字段; 内容见第 7.8.3 节。 |
| VendorDefined | 18 | 14 | 强制字段; 厂商可定义其内容。 |

#### 7.8.1 EntryType 字段

EntryType 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1 节).

##### 7.8.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.1 节).

对于 Vendor Extension directory entry, 本字段唯一有效取值为 0.

##### 7.8.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.2 节).

对于 Vendor Extension directory entry, 本字段唯一有效取值为 1.

##### 7.8.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.3 节).

##### 7.8.1.4 InUse 字段

InUse 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.4 节).

#### 7.8.2 GeneralSecondaryFlags 字段

GeneralSecondaryFlags 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2 节), 并将 CustomDefined 字段内容定义为保留.

##### 7.8.2.1 AllocationPossible 字段

AllocationPossible 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2.1 节).

对于本 directory entry, 本字段唯一有效取值为 0.

##### 7.8.2.2 NoFatChain 字段

NoFatChain 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2.2 节).

#### 7.8.3 VendorGuid 字段

VendorGuid 字段应包含唯一标识给定 Vendor Extension 的 GUID.

本字段所有可能取值均有效, 空 GUID {00000000-0000-0000-0000-000000000000} 除外. 然而, 厂商于定义扩展时宜使用 GUID 生成工具 (如 GuidGen.exe) 选取 GUID.

本字段取值决定 VendorDefined 字段的厂商特定结构.
### 7.9 Vendor Allocation 目录项

Vendor Allocation directory entry 是 File directory entry set 中的 benign secondary directory entry (见表 37). 每个 set 可含任意数量的 Vendor Allocation directory entry, 上限为 secondary directory entry 总数减去其他 secondary directory entry 数量. 且 Vendor Allocation directory entry 仅在其不位于必需的 Stream Extension 与 File Name directory entry 之前时有效.

Vendor Allocation directory entry 通过 VendorGuid 字段 (见表 37) 使厂商能在单个 File directory entry set 中拥有唯一、厂商特定的 directory entry. 唯一目录项实质上使厂商可扩展 exFAT 文件系统. 厂商可定义关联簇 (若有) 的内容. 厂商实现可维护关联簇 (若有) 的内容并提供厂商特定功能.

无法识别 Vendor Allocation directory entry 之 GUID 的实现, 须将该 directory entry 与任何其他无法识别的 benign secondary directory entry 同等对待 (见第 8.2 节).

**表 37 Vendor Allocation DirectoryEntry**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 7.9.1 节。 |
| GeneralSecondaryFlags | 1 | 1 | 强制字段; 内容见第 7.9.2 节。 |
| VendorGuid | 2 | 16 | 强制字段; 内容见第 7.9.3 节。 |
| VendorDefined | 18 | 2 | 强制字段; 厂商可定义其内容。 |
| FirstCluster | 20 | 4 | 强制字段; 内容见第 7.9.4 节。 |
| DataLength | 24 | 8 | 强制字段; 内容见第 7.9.5 节。 |

#### 7.9.1 EntryType 字段

EntryType 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1 节).

##### 7.9.1.1 TypeCode 字段

TypeCode 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.1 节).

对于 Vendor Allocation directory entry, 本字段唯一有效取值为 1.

##### 7.9.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.2 节).

对于 Vendor Allocation directory entry, 本字段唯一有效取值为 1.

##### 7.9.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.3 节).

##### 7.9.1.4 InUse 字段

InUse 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.1.4 节).

#### 7.9.2 GeneralSecondaryFlags 字段

GeneralSecondaryFlags 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2 节), 并将 CustomDefined 字段内容定义为保留.

##### 7.9.2.1 AllocationPossible 字段

AllocationPossible 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2.1 节).

对于本 directory entry, 本字段唯一有效取值为 1.

##### 7.9.2.2 NoFatChain 字段

NoFatChain 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.2.2 节).

#### 7.9.3 VendorGuid 字段

VendorGuid 字段应包含唯一标识给定 Vendor Allocation 的 GUID.

本字段所有可能取值均有效, 空 GUID {00000000-0000-0000-0000-000000000000} 除外. 然而, 厂商于定义扩展时宜使用 GUID 生成工具 (如 GuidGen.exe) 选取 GUID.

本字段取值决定关联簇 (若有) 内容的厂商特定结构.

#### 7.9.4 FirstCluster 字段

FirstCluster 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.3 节).

#### 7.9.5 DataLength 字段

DataLength 字段应符合 Generic Secondary DirectoryEntry 模板中的定义 (见第 6.4.4 节).

### 7.10 TexFAT Padding 目录项

本规范, exFAT Revision 1.00 File System Basic Specification, 未定义 TexFAT Padding directory entry. 然而, 其 type code 为 1, type importance 为 1. 本规范的实现须将 TexFAT Padding directory entry 与任何其他无法识别的 benign primary directory entry 同等对待; 实现不得移动 TexFAT Padding directory entry.
