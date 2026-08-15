> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: `portfs/doc/std_ntfs/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

# NTFS 工作原理: 本地文件系统 | Microsoft Learn

适用对象: Windows Server 2003、Windows Server 2003 R2、Windows Server 2003 with SP1、Windows Server 2003 with SP2

## NTFS 工作原理

**本节内容**

- NTFS 架构
- NTFS 物理结构
- NTFS 进程与交互
- 相关信息

文件系统是操作系统必需的组成部分, 它决定文件在卷上的命名、存储与组织方式. 文件系统管理文件与文件夹, 以及本地与远程用户定位和访问这些项所需的信息.

Microsoft Windows Server 2003 在基本磁盘与动态磁盘上支持 NTFS 文件系统. 基本磁盘与卷是 Windows 操作系统最常用的存储类型. 动态磁盘在卷管理上更灵活, 因为它使用数据库跟踪磁盘上的动态卷信息以及计算机中其他动态磁盘的信息.

在格式化卷时, 你可以为该卷选择文件系统类型. 选择 NTFS 文件系统后, 无论该卷是基本卷还是动态卷, 格式化过程都会在卷上放置关键的 NTFS 文件数据结构.
