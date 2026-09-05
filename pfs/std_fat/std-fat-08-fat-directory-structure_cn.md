> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## FAT 目录结构

本节先讨论短目录项 (short directory entries), 暂不考虑长目录项.

FAT 目录无非是由 32 字节结构组成的线性列表构成的 **file** (文件). 唯一必须始终存在的特殊目录是根目录. 对 FAT12/FAT16 介质, 根目录位于磁盘固定位置, 紧接在最后一个 FAT 之后, 大小 (扇区数) 由 `BPB_RootEntCnt` 固定计算 (见本文 RootDirSectors 计算). FAT12/FAT16 根目录第一扇区号 (相对 FAT 卷第一扇区):

```
FirstRootDirSecNum = BPB_ResvdSecCnt + (BPB_NumFATs * BPB_FATSz16);
```

FAT32 上根目录大小可变, 为簇链, 与其他目录相同. FAT32 卷根目录第一个簇存储在 `BPB_RootClus`. 与其他目录不同, 任何 FAT 类型的根目录本身无日期/时间戳, 无文件名 (除隐含的 `\` 外), 且目录前两个目录项 **不是** `.` 与 `..` 文件. 根目录另一特殊之处: 它是 FAT 卷上唯一允许存在仅设置 `ATTR_VOLUME_ID` 属性位的文件的目录 (见下文).

### FAT 32 字节目录项结构

| **名称** | **偏移 (字节)** | **大小 (字节)** | **说明** |
| --- | --- | --- | --- |
| DIR_Name | 0 | 11 | 短文件名. |
| DIR_Attr | 11 | 1 | 文件属性: `ATTR_READ_ONLY` 0x01; `ATTR_HIDDEN` 0x02; `ATTR_SYSTEM` 0x04; `ATTR_VOLUME_ID` 0x08; `ATTR_DIRECTORY` 0x10; `ATTR_ARCHIVE` 0x20; `ATTR_LONG_NAME` = `ATTR_READ_ONLY \| ATTR_HIDDEN \| ATTR_SYSTEM \| ATTR_VOLUME_ID`. 属性字节高 2 位保留, 创建文件时应始终设为 0, 之后不得修改或读取. |
| DIR_NTRes | 12 | 1 | Windows NT 保留. 创建时设为 0, 之后不得修改或读取. |
| DIR_CrtTimeTenth | 13 | 1 | 创建时间的毫秒戳. 实际为十分之一秒计数. `DIR_CrtTime` 秒部分粒度为 2 秒, 故本字段为十分之一秒计数, 有效范围 0–199 (含). |
| DIR_CrtTime | 14 | 2 | 文件创建时间. |
| DIR_CrtDate | 16 | 2 | 文件创建日期. |
| DIR_LstAccDate | 18 | 2 | 最后访问日期. 无最后访问 **时间**, 仅有日期. 为最后读或写日期. 写操作时应与 `DIR_WrtDate` 设为同一日期. |
| DIR_FstClusHI | 20 | 2 | 本目录项首簇号的高字 (FAT12/FAT16 卷始终为 0). |
| DIR_WrtTime | 22 | 2 | 最后写入时间. 文件创建视为一次写入. |
| DIR_WrtDate | 24 | 2 | 最后写入日期. 文件创建视为一次写入. |
| DIR_FstClusLO | 26 | 2 | 本目录项首簇号的低字. |
| DIR_FileSize | 28 | 4 | 32 位 DWORD, 文件大小 (字节). |

### DIR_Name[0] 的特殊说明

FAT 目录项第一字节 (`DIR_Name[0]`) 的特殊规则:

- 若 `DIR_Name[0] == 0xE5`, 目录项为 **free** (空闲), 无文件或目录名.
- 若 `DIR_Name[0] == 0x00`, 目录项为 free (同 0xE5), 且其后无已分配目录项 (其后所有目录项的 `DIR_Name[0]` 也为 0). 特殊值 0 (而非 0xE5) 告知 FAT 驱动无需检查本目录后续条目, 因全部空闲.
- 若 `DIR_Name[0] == 0x05`, 该字节实际文件名字符为 0xE5. 0xE5 在日本所用字符集中是有效 KANJI Lead byte (首字节). 使用特殊值 0x05 以便正确处理日本此特殊文件名, 避免 FAT 代码误判为空闲.

`DIR_Name` 分为两部分: 8 字符主名与 3 字符扩展名. 两部分均以 0x20 字节 **trailing space padded** (尾随空格填充).

`DIR_Name[0]` 不得等于 0x20. 主名与扩展名之间有隐含的 `.` 字符, 不在 `DIR_Name` 中. `DIR_Name` 中不允许小写字符 (具体字符集依国家/地区而定).

下列字符在 `DIR_Name` 任何字节中 **不合法**:

- 小于 0x20 的值 (除上文 `DIR_Name[0]==0x05` 特殊情形).
- 0x22, 0x2A, 0x2B, 0x2C, 0x2E, 0x2F, 0x3A, 0x3B, 0x3C, 0x3D, 0x3E, 0x3F, 0x5B, 0x5C, 0x5D, 0x7C.

用户输入名称映射到 `DIR_Name` 的示例:

| 用户输入 | DIR_Name (8+3, 空格填充) |
| --- | --- |
| `FOO BAR` | `FOO     BAR` |
| `FOO.BAR` | `FOO     BAR` |
| `FOO.BAR ` | `FOO     BAR` |
| `FOO. BAR` | `FOO` (扩展名部分为空) |
| `FOO .BAR` | `FOO` |
| `PICKLE.A` | `PICKLE  A` |
| `prettybg.big` | `PRETTYBGBIG` |
| ` FOO.BAR` | 非法, `DIR_Name[0]` 不能为 0x20 |

FAT 目录中所有名称唯一. 上表前三个示例名称不同, 但均映射为同一 `DIR_Name` `FOO     BAR`, 同一目录中只能有一个该 `DIR_Name` 的文件.

### DIR_Attr 属性说明

| 属性 | 含义 |
| --- | --- |
| `ATTR_READ_ONLY` | 对文件的写入应失败. |
| `ATTR_HIDDEN` | 正常目录列表不应显示此文件. |
| `ATTR_SYSTEM` | 表示操作系统文件. |
| `ATTR_VOLUME_ID` | 卷上应仅有一个 `\` 设置此属性, 且必须在根目录. 该文件名为卷标. 卷标文件的 `DIR_FstClusHI` 与 `DIR_FstClusLO` 必须始终为 0 (不分配数据簇). |
| `ATTR_DIRECTORY` | 表示此文件实际为其他文件的容器 (目录). |
| `ATTR_ARCHIVE` | 支持备份工具. FAT 驱动在创建、重命名或写入文件时设置此位. 备份工具可用此属性标记自上次备份以来已修改的文件. |

`ATTR_LONG_NAME` 属性组合表示该 **entry** (目录项) 实际是另一文件长文件名目录项的一部分. 见下一节.

### 创建目录

创建目录时 (在 `DIR_Attr` 中设置 `ATTR_DIRECTORY` 位):

1. 将 `DIR_FileSize` 设为 0. 对 `ATTR_DIRECTORY` 文件, `DIR_FileSize` 不使用, 始终为 0 (目录大小通过沿簇链跟踪至 EOC 确定).
2. 分配一个簇 (FAT16/FAT12 根目录除外), 将 `DIR_FstClusLO`/`DIR_FstClusHI` 设为该簇号, 在 FAT 中为该簇写入 EOC 标记.
3. 将该簇全部字节初始化为 0.
4. 若为根目录, 完成 (根目录无 `.`/`..` 项).
5. 若非根目录, 在刚分配簇的数据区前两个 32 字节目录项中创建两个特殊项:
   - 第一项 `DIR_Name` 设为 `.`
   - 第二项 `DIR_Name` 设为 `..`
   - 两项 `DIR_FileSize` 均为 0; 所有日期/时间字段与刚创建目录的目录项相同.
   - `.` 项 (第一项) 的 `DIR_FstClusLO`/`DIR_FstClusHI` 与目录自身目录项相同 (即含 `.`/`..` 的簇号).
   - `..` 项 (第二项) 的 `DIR_FstClusLO`/`DIR_FstClusHI` 设为父目录的首簇号 (若父目录为根目录则为 0, FAT32 卷亦同).

**`.`/`..` 摘要**:

- `.` 项: 指向自身的目录.
- `..` 项: 指向本目录父目录的起始簇 (父为根目录则为 0).

### 日期与时间格式

许多 FAT 文件系统除 `DIR_WrtTime`/`DIR_WrtDate` 外不支持其他日期/时间. 因此 `DIR_CrtTimeMil`、`DIR_CrtTime`、`DIR_CrtDate`、`DIR_LstAccDate` 实际为可选字段. 但 `DIR_WrtTime`/`DIR_WrtDate` 必须支持. 若不支持其他日期/时间字段, 创建文件时应设为 0, 其他文件操作中忽略.

**日期格式**: FAT 目录项日期戳为 16 位字段, 基本为相对 MS-DOS 纪元 1980-01-01 的日期 (bit 0 为 16 位字 LSB, bit 15 为 MSB):

| 位 | 含义 |
| --- | --- |
| Bits 0–4 | 月中日, 有效范围 1–31 (含). |
| Bits 5–8 | 年中月, 1=一月, 有效范围 1–12 (含). |
| Bits 9–15 | 自 1980 起的年数, 有效范围 0–127 (含), 即 1980–2107. |

**时间格式**: FAT 目录项时间戳为 16 位字段, 粒度 2 秒 (bit 0 为 LSB, bit 15 为 MSB):

| 位 | 含义 |
| --- | --- |
| Bits 0–4 | 2 秒计数, 有效范围 0–29 (含), 即 0–58 秒. |
| Bits 5–10 | 分钟, 有效范围 0–59 (含). |
| Bits 11–15 | 小时, 有效范围 0–23 (含). |

有效时间范围: 00:00:00 至 23:59:58.
