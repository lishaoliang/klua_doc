> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/)；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

## NTFS 物理结构

### NTFS 卷的组织 (Organization of an NTFS Volume)

图 Organization of an NTFS Volume 说明 NTFS 如何在卷上组织各结构.

**Organization of an NTFS Volume**

![Organization of an NTFS Volume](images/cc781134.737c1f18-1bbc-45c7-9cb7-d61387d78324(ws.10).gif)

下表描述 NTFS 卷上的各组织结构.

**NTFS 卷组件 (NTFS Volume Components)**

| 组件 | 说明 |
| --- | --- |
| NTFS Boot Sector (NTFS 引导扇区) | 含 BIOS parameter block (BPB), 存储卷布局与文件系统结构信息, 以及加载 Windows Server 2003 的引导代码. |
| Master File Table (MFT, 主文件表) | 含从 NTFS 分区检索文件所需的信息, 例如文件的属性. |
| File System Data (文件系统数据) | 存储不在 MFT 内的数据. |
| Master File Table Copy (MFT 副本) | 含原 MFT 副本中关键记录的拷贝, 用于文件系统出问题时恢复. |
