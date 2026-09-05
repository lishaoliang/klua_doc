> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

# Master File Table (Developer Notes) — 主文件表 (开发者说明)

[本文档仅适用于 NTFS 卷 version 3.]

主文件表 (Master File Table, MFT) 存储从 NTFS 分区检索文件所需的信息.

一个文件可有一条或多条 MFT record, 并可含一个或多个 attribute. 在 NTFS 中, file reference 是 base file record 的 MFT segment reference. 详见 [MFT_SEGMENT_REFERENCE](https://learn.microsoft.com/en-us/windows/win32/devnotes/mft-segment-reference).

MFT 含 file record segment; 其中前 16 个保留给特殊文件, 例如:

- 0: MFT ($Mft)
- 5: root directory (\)
- 6: volume cluster allocation file ($Bitmap)
- 8: bad-cluster file ($BadClus)

每个 file record segment 以 `FILE_RECORD_SEGMENT_HEADER` 开头. 详见 [FILE_RECORD_SEGMENT_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/file-record-segment-header). 每个 file record segment 之后跟一个或多个 attribute. 每个 attribute 以 `ATTRIBUTE_RECORD_HEADER` 开头. 详见 [ATTRIBUTE_RECORD_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-record-header). attribute record 含 attribute type (如 `$DATA` 或 `$BITMAP`)、可选名称与 attribute value. 用户 data stream 是一个 attribute, 所有 stream 亦然. attribute list 以 0xFFFFFFFF (`$END`, `NTFS_AT_END`) 终止.

下列为部分 attribute 示例.

- $Mft 文件含未命名的 `$DATA` attribute, 即 MFT record segment 的有序序列.
- $Mft 文件含未命名的 `$BITMAP` attribute, 指示哪些 MFT record 正在使用.
- $Bitmap 文件含未命名的 `$DATA` attribute, 指示哪些簇正在使用.
- $BadClus 文件含名为 `$BAD` 的 `$DATA` attribute, 其中每个 entry 对应一个坏簇.

当 file record segment 中再无空间存储 attribute 时, 会分配额外的 file record segment, 并在第一条 (base) file record segment 中以称 attribute list 的 attribute 链接. attribute list 指示与该文件关联的每个 attribute 的位置. 这包括 base file record 中的所有 attribute, attribute list 自身除外. 详见 [ATTRIBUTE_LIST_ENTRY](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-list-entry).

与 MFT 相关的结构包括:

- [ATTRIBUTE_LIST_ENTRY](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-list-entry)
- [ATTRIBUTE_RECORD_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-record-header)
- [FILE_NAME](https://learn.microsoft.com/en-us/windows/win32/devnotes/file-name)
- [FILE_RECORD_SEGMENT_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/file-record-segment-header)
- [MFT_SEGMENT_REFERENCE](https://learn.microsoft.com/en-us/windows/win32/devnotes/mft-segment-reference)
- [MULTI_SECTOR_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/multi-sector-header)
- [STANDARD_INFORMATION](https://learn.microsoft.com/en-us/windows/win32/devnotes/standard-information)

on-disk 布局与上述结构的对照见 `fs/ntfs/layout.h` 与 [portfs/src/ntfs/pfs_ntfs_raw.h](https://gitee.com/klua/portfs/blob/trunk/src/ntfs/pfs_ntfs_raw.h).
