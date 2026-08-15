> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: `portfs/doc/std_exfat/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名保留英文；目录项术语对齐 linux-7.1.2 `fs/exfat/dir.c`（Directory Entry / EntryType / NoFatChain）
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 6 目录结构 (Directory Structure)

exFAT 文件系统采用目录树方式管理 Cluster Heap 中的文件系统结构与文件。目录树中父目录与子目录为一对多关系。

FirstClusterOfRootDirectory 字段所指向的目录为目录树根。除根目录外, 所有目录均以单向链接方式自根目录向下延伸。

每个目录由一系列 Directory Entry（目录项, 见表 13）组成。

一个或多个 Directory Entry 组合成一个 directory entry set（目录项集）, 用于描述某一对象, 例如文件系统结构、子目录或文件。

**表 13 目录结构**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| DirectoryEntry[0] | 0 | 32 | 强制字段; 内容见第 6.1 节。 |
| ... | ... | ... | ... |
| DirectoryEntry[N–1] | (N – 1) \* 32 | 32 | 强制字段; 内容见第 6.1 节。N 为 DirectoryEntry 字段个数, 等于包含该目录的簇链字节长度除以单个 DirectoryEntry 大小 32 字节。 |

### 6.1 DirectoryEntry[0] ... DirectoryEntry[N--1]

该数组中每个 DirectoryEntry 字段均派生自 Generic DirectoryEntry 模板（见第 6.2 节）。

### 6.2 Generic DirectoryEntry 模板

Generic DirectoryEntry 模板为 Directory Entry 提供基础定义（见表 14）。所有 Directory Entry 结构均派生自此模板, 且仅 Microsoft 定义的 Directory Entry 结构有效（exFAT 除第 7.8 节与第 7.9 节所定义者外, 不提供厂商自定义 Directory Entry 结构）。解析 Generic DirectoryEntry 模板为强制要求。

**表 14 Generic DirectoryEntry 模板**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 6.2.1 节。 |
| CustomDefined | 1 | 19 | 强制字段; 派生结构可定义其内容。 |
| FirstCluster | 20 | 4 | 强制字段; 内容见第 6.2.2 节。 |
| DataLength | 24 | 8 | 强制字段; 内容见第 6.2.3 节。 |

#### 6.2.1 EntryType 字段

EntryType 字段有三种用法模式, 由字段取值决定（见下列列表）。

- 00h, 表示目录结束标记, 适用条件如下:
 - 该 DirectoryEntry 中其余字段实际均为保留
 - 该目录中后续所有 Directory Entry 亦为目录结束标记
 - 目录结束标记仅允许出现在 directory entry set 之外
 - 实现可按需覆盖目录结束标记
- 01h 至 7Fh（含）, 表示未使用 Directory Entry 标记, 适用条件如下:
 - 该 DirectoryEntry 中其余字段实际均为未定义
 - 未使用 Directory Entry 仅允许出现在 directory entry set 之外
 - 实现可按需覆盖未使用 Directory Entry
 - 该取值范围对应 InUse 字段（见第 6.2.1.4 节）为 0
- 81h 至 FFh（含）, 表示常规 Directory Entry, 适用条件如下:
 - EntryType 字段内容（见表 15）决定 DirectoryEntry 结构其余部分的布局
 - 仅该取值范围在 directory entry set 内有效
 - 该取值范围直接对应 InUse 字段（见第 6.2.1.4 节）为 1

为防止误改 InUse 字段（见第 6.2.1.4 节）导致出现目录结束标记, 取值 80h 无效。

**表 15 Generic EntryType 字段结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| TypeCode | 0 | 5 | 强制字段; 内容见第 6.2.1.1 节。 |
| TypeImportance | 5 | 1 | 强制字段; 内容见第 6.2.1.2 节。 |
| TypeCategory | 6 | 1 | 强制字段; 内容见第 6.2.1.3 节。 |
| InUse | 7 | 1 | 强制字段; 内容见第 6.2.1.4 节。 |

##### 6.2.1.1 TypeCode 字段

TypeCode 字段部分描述给定 Directory Entry 的具体类型。该字段与 TypeImportance、TypeCategory 字段（分别见第 6.2.1.2 节与第 6.2.1.3 节）共同唯一标识给定 Directory Entry 的类型。

除 TypeImportance 与 TypeCategory 均为 0 的情况外, 本字段所有取值均有效; 在该情况下, 本字段取值 0 无效。

##### 6.2.1.2 TypeImportance 字段

TypeImportance 字段描述给定 Directory Entry 的重要性。

本字段有效取值为:

- 0, 表示给定 Directory Entry 为 Critical（见第 6.3.1.2.1 节 Critical Primary 与第 6.4.1.2.1 节 Critical Secondary Directory Entry）
- 1, 表示给定 Directory Entry 为 Benign（见第 6.3.1.2.2 节 Benign Primary 与第 6.4.1.2.2 节 Benign Secondary Directory Entry）

##### 6.2.1.3 TypeCategory 字段

TypeCategory 字段描述给定 Directory Entry 的类别。

本字段有效取值为:

- 0, 表示给定 Directory Entry 为 Primary（见第 6.3 节）
- 1, 表示给定 Directory Entry 为 Secondary（见第 6.4 节）

##### 6.2.1.4 InUse 字段

InUse 字段描述给定 Directory Entry 是否在使用中。

本字段有效取值为:

- 0, 表示给定 Directory Entry 未使用; 即该结构实际为未使用 Directory Entry
- 1, 表示给定 Directory Entry 在使用中; 即该结构为常规 Directory Entry

#### 6.2.2 FirstCluster 字段

FirstCluster 字段应包含与给定 Directory Entry 关联的 Cluster Heap 分配中首簇的索引。

本字段有效取值范围为:

- 恰为 0, 表示不存在簇分配
- 2 至 ClusterCount + 1, 为有效簇索引范围

若簇分配与派生结构不兼容, 派生自此模板的结构可重定义 FirstCluster 与 DataLength 字段。

#### 6.2.3 DataLength 字段

DataLength 字段描述关联簇分配所含数据的字节长度。

本字段有效取值范围为:

- 至少为 0; 若 FirstCluster 字段为 0, 则本字段唯一有效取值为 0
- 至多为 ClusterCount \* 2^SectorsPerClusterShift^ \* 2^BytesPerSectorShift^

若派生结构无法使用簇分配, 派生自此模板的结构可重定义 FirstCluster 与 DataLength 字段。

### 6.3 Generic Primary DirectoryEntry 模板

directory entry set 中第一个 Directory Entry 应为 Primary Directory Entry。该 set 中其余 Directory Entry（若有）均应为 Secondary Directory Entry（见第 6.4 节）。

解析 Generic Primary DirectoryEntry 模板为强制要求。

所有 Primary Directory Entry 结构均派生自 Generic Primary DirectoryEntry 模板（见表 16）, 该模板又派生自 Generic DirectoryEntry 模板（见第 6.2 节）。

**表 16 Generic Primary DirectoryEntry 模板**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 6.3.1 节。 |
| SecondaryCount | 1 | 1 | 强制字段; 内容见第 6.3.2 节。 |
| SetChecksum | 2 | 2 | 强制字段; 内容见第 6.3.3 节。 |
| GeneralPrimaryFlags | 4 | 2 | 强制字段; 内容见第 6.3.4 节。 |
| CustomDefined | 6 | 14 | 强制字段; 派生结构定义其内容。 |
| FirstCluster | 20 | 4 | 强制字段; 内容见第 6.3.5 节。 |
| DataLength | 24 | 8 | 强制字段; 内容见第 6.3.6 节。 |

#### 6.3.1 EntryType 字段

EntryType 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1 节）。

##### 6.3.1.1 TypeCode 字段

TypeCode 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1.1 节）。

##### 6.3.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1.2 节）。

###### 6.3.1.2.1 Critical Primary Directory Entry

Critical Primary Directory Entry 包含对正确管理 exFAT 卷至关重要的信息。仅根目录包含 Critical Primary Directory Entry（File Directory Entry 为例外, 见第 7.4 节）。

Critical Primary Directory Entry 的定义与 exFAT 主修订号相关。实现应支持所有 Critical Primary Directory Entry, 且仅应记录本规范定义的 Critical Primary Directory Entry 结构。

###### 6.3.1.2.2 Benign Primary Directory Entry

Benign Primary Directory Entry 包含对管理 exFAT 卷可能有用的附加信息。任意目录均可包含 Benign Primary Directory Entry。

Benign Primary Directory Entry 的定义与 exFAT 次修订号相关。对本规范或后续规范所定义的任何 Benign Primary Directory Entry 的支持均为可选。无法识别的 Benign Primary Directory Entry 将使整个 directory entry set 无法识别（超出适用 Directory Entry 模板定义的范围）。

##### 6.3.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1.3 节）。

对本模板, 本字段唯一有效取值为 0。

##### 6.3.1.4 InUse 字段

InUse 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1.4 节）。

#### 6.3.2 SecondaryCount 字段

SecondaryCount 字段描述紧接给定 Primary Directory Entry 之后的 Secondary Directory Entry 数量。这些 Secondary Directory Entry 与给定 Primary Directory Entry 共同构成 directory entry set。

本字段有效取值范围为:

- 至少为 0, 表示该 Primary Directory Entry 为 directory entry set 中唯一项
- 至多为 255, 表示紧随其后的 255 个 Directory Entry 与该 Primary Directory Entry 共同构成 directory entry set

派生自此模板的 Critical Primary Directory Entry 结构可重定义 SecondaryCount 与 SetChecksum 字段。

#### 6.3.3 SetChecksum 字段

SetChecksum 字段应包含给定 directory entry set 中所有 Directory Entry 的校验和, 但不含本字段自身（见图 2）。实现于使用 directory entry set 中任何其他 Directory Entry 之前, 应验证本字段内容有效。

派生自此模板的 Critical Primary Directory Entry 结构可重定义 SecondaryCount 与 SetChecksum 字段。

**图 2 EntrySetChecksum 计算**

```C
UInt16 EntrySetChecksum
(
    UCHAR * Entries,       // points to an in-memory copy of the directory entry set
    UCHAR   SecondaryCount
)
{
    UInt16 NumberOfBytes = ((UInt16)SecondaryCount + 1) * 32;
    UInt16 Checksum = 0;
    UInt16 Index;

    for (Index = 0; Index < NumberOfBytes; Index++)
    {
        if ((Index == 2) || (Index == 3))
        {
            continue;
        }
        Checksum = ((Checksum&1) ? 0x8000 : 0) + (Checksum>>1) +  (UInt16)Entries[Index];
    }
    return Checksum;
}
```

#### 6.3.4 GeneralPrimaryFlags 字段

GeneralPrimaryFlags 字段包含标志位（见表 17）。

派生自此模板的 Critical Primary Directory Entry 结构可重定义本字段。

**表 17 Generic GeneralPrimaryFlags 字段结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| AllocationPossible | 0 | 1 | 强制字段; 内容见第 6.3.4.1 节。 |
| NoFatChain | 1 | 1 | 强制字段; 内容见第 6.3.4.2 节。 |
| CustomDefined | 2 | 14 | 强制字段; 派生结构可定义本字段。 |

##### 6.3.4.1 AllocationPossible 字段

AllocationPossible 字段描述给定 Directory Entry 是否可能在 Cluster Heap 中进行分配。

本字段有效取值为:

- 0, 表示无法关联簇分配, FirstCluster 与 DataLength 字段实际均为未定义（派生结构可重定义这些字段）
- 1, 表示可关联簇分配, FirstCluster 与 DataLength 字段按定义解释

##### 6.3.4.2 NoFatChain 字段

NoFatChain 字段指示活动 FAT 是否描述给定分配的簇链。

本字段有效取值为:

- 0, 表示该分配簇链对应 FAT 表项有效, 实现应解析它们; 若 AllocationPossible 为 0, 或 AllocationPossible 为 1 且 FirstCluster 为 0, 则本字段唯一有效取值为 0
- 1, 表示关联分配为一段连续簇; 对应簇的 FAT 表项无效, 实现不得解析它们; 实现可用下列公式计算关联分配大小: DataLength / (2^SectorsPerClusterShift^ \* 2^BytesPerSectorShift^) 向上取整至最近整数

若派生自此模板的 Critical Primary Directory Entry 结构重定义了 GeneralPrimaryFlags 字段, 则任何关联分配簇链的对应 FAT 表项均有效。

#### 6.3.5 FirstCluster 字段

FirstCluster 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.2 节）。

若 NoFatChain 位为 1, 则 FirstCluster 必须指向 cluster heap 中的有效簇。

派生自此模板的 Critical Primary Directory Entry 结构可重定义 FirstCluster 与 DataLength 字段。其他派生结构仅当 AllocationPossible 为 0 时可重定义 FirstCluster 与 DataLength 字段。

#### 6.3.6 DataLength 字段

DataLength 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.3 节）。

若 NoFatChain 位为 1, 则 DataLength 不得为零。若 FirstCluster 字段为零, 则 DataLength 亦须为零。

派生自此模板的 Critical Primary Directory Entry 结构可重定义 FirstCluster 与 DataLength 字段。其他派生结构仅当 AllocationPossible 为 0 时可重定义 FirstCluster 与 DataLength 字段。

### 6.4 Generic Secondary DirectoryEntry 模板

Secondary Directory Entry 的核心用途是为 directory entry set 提供附加信息。解析 Generic Secondary DirectoryEntry 模板为强制要求。

Critical 与 Benign Secondary Directory Entry 的定义均与 exFAT 次修订号相关。对本规范或后续规范所定义的任何 Critical 或 Benign Secondary Directory Entry 的支持均为可选。

所有 Secondary Directory Entry 结构均派生自 Generic Secondary DirectoryEntry 模板（见表 18）, 该模板又派生自 Generic DirectoryEntry 模板（见第 6.2 节）。

**表 18 Generic Secondary DirectoryEntry 模板**

| **字段名** | **偏移** **(字节)** | **大小** **(字节)** | **说明** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | 强制字段; 内容见第 6.4.1 节。 |
| GeneralSecondaryFlags | 1 | 1 | 强制字段; 内容见第 6.4.2 节。 |
| CustomDefined | 2 | 18 | 强制字段; 派生结构定义其内容。 |
| FirstCluster | 20 | 4 | 强制字段; 内容见第 6.4.3 节。 |
| DataLength | 24 | 8 | 强制字段; 内容见第 6.4.4 节。 |

#### 6.4.1 EntryType 字段

EntryType 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1 节）

##### 6.4.1.1 TypeCode 字段

TypeCode 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1.1 节）。

##### 6.4.1.2 TypeImportance 字段

TypeImportance 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1.2 节）。

###### 6.4.1.2.1 Critical Secondary Directory Entry

Critical Secondary Directory Entry 包含对其所属 directory entry set 正确管理至关重要的信息。对任何特定 Critical Secondary Directory Entry 的支持均为可选, 但无法识别的 Critical Directory Entry 将使整个 directory entry set 无法识别（超出适用 Directory Entry 模板定义的范围）。

然而, 若 directory entry set 包含实现无法识别的至少一个 Critical Secondary Directory Entry, 则实现至多只能解析该 set 中 Directory Entry 的模板, 而不得解析 set 中任何 Directory Entry 关联分配所含数据（File Directory Entry 为例外, 见第 7.4 节）。

###### 6.4.1.2.2 Benign Secondary Directory Entry

Benign Secondary Directory Entry 包含对其所属 directory entry set 管理可能有用的附加信息。对任何特定 Benign Secondary Directory Entry 的支持均为可选。无法识别的 Benign Secondary Directory Entry 不会使整个 directory entry set 无法识别。

实现可忽略其无法识别的任何 Benign Secondary Entry。

##### 6.4.1.3 TypeCategory 字段

TypeCategory 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1.3 节）。

对本模板, 本字段有效取值为 1。

##### 6.4.1.4 InUse 字段

InUse 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.1.4 节）。

#### 6.4.2 GeneralSecondaryFlags 字段

GeneralSecondaryFlags 字段包含标志位（见表 19）。

**表 19 Generic GeneralSecondaryFlags 字段结构**

| **字段名** | **偏移** **(位)** | **大小** **(位)** | **说明** |
| --- | --- | --- | --- |
| AllocationPossible | 0 | 1 | 强制字段; 内容见第 6.4.2.1 节。 |
| NoFatChain | 1 | 1 | 强制字段; 内容见第 6.4.2.2 节。 |
| CustomDefined | 2 | 6 | 强制字段; 派生结构可定义本字段。 |

##### 6.4.2.1 AllocationPossible 字段

AllocationPossible 字段与 Generic Primary DirectoryEntry 模板中同名字段定义相同（见第 6.3.4.1 节）。

##### 6.4.2.2 NoFatChain 字段

NoFatChain 字段与 Generic Primary DirectoryEntry 模板中同名字段定义相同（见第 6.3.4.2 节）。

#### 6.4.3 FirstCluster 字段

FirstCluster 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.2 节）。

若 NoFatChain 位为 1, 则 FirstCluster 必须指向 cluster heap 中的有效簇。

#### 6.4.4 DataLength 字段

DataLength 字段应符合 Generic DirectoryEntry 模板中的定义（见第 6.2.3 节）。

若 NoFatChain 位为 1, 则 DataLength 不得为零。若 FirstCluster 字段为零, 则 DataLength 亦须为零。
