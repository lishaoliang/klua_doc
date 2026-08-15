> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: `portfs/doc/std_ntfs/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

## NTFS 架构 (NTFS Architecture)

在硬盘上格式化并设置卷文件系统时, 会创建主引导记录 (Master Boot Record, MBR). MBR 含少量可执行代码 (称为主引导代码) 以及磁盘分区表. 当卷被挂载时, MBR 执行主引导代码并将控制权转交给磁盘上的引导扇区 (Boot Sector), 使服务器能从该卷的文件系统启动操作系统.

**注意**

- 分区表含多个用于描述分区的字段. 其中 System ID 字段定义分区上的文件系统 (如 NTFS). 对 NTFS 卷, System ID 为 0x07.

图 NTFS Architecture 展示了该过程的架构.

**NTFS Architecture**

![NTFS Architecture](images/cc781134.91d1303a-c92d-4e1e-a98e-aca7bfa54bf4(ws.10).gif)

下表描述 NTFS 文件系统的各组件.

**x86 系统上的 NTFS 架构组件**

| 组件 | 组件说明 |
| --- | --- |
| Hard disk (硬盘) | 含一个或多个分区. |
| Boot sector (引导扇区) | 可引导分区, 存储卷布局与文件系统结构信息, 以及加载 Ntdlr 的引导代码. |
| Master Boot Record (MBR) | 含系统 BIOS 加载到内存的可执行代码. 该代码扫描 MBR 以查找分区表, 确定哪个分区为活动 (可引导) 分区. |
| Ntldlr.dll | 将 CPU 切换到保护模式, 启动文件系统, 然后读取 Boot.ini 文件内容. 该信息决定启动选项与初始引导菜单选择. |
| Ntfs.sys | NTFS 的系统文件驱动. |
| Ntoskrnl.exe | 提取应加载哪些系统设备驱动及其加载顺序的信息. |
| Kernel mode (内核模式) | 允许代码直接访问系统中全部硬件与内存的处理模式. |
| User mode (用户模式) | 应用程序运行的处理模式. |
