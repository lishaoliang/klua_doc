> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## 通用说明 (General Comments, 适用于各 FAT 类型)

FAT 文件系统最初为 IBM PC 体系结构开发. 因此盘上数据结构均为 **小端 (little endian)**.

若将盘上一条 32 位 FAT 表项视为四个 8 位字节, 记 `byte[0]` 为最低地址字节, `byte[3]` 为最高地址字节, 则 32 个位 (00 为最低有效位, 31 为最高有效位) 布局如下:

```
byte[3]           3 3 2 2 2 2 2 2
                  1 0 9 8 7 6 5 4

byte[2]           2 2 2 2 1 1 1 1
                  3 2 1 0 9 8 7 6

byte[1]           1 1 1 1 1 1 0 0
                  5 4 3 2 1 0 9 8

byte[0]           0 0 0 0 0 0 0 0
                  7 6 5 4 3 2 1 0
```

若主机为 **大端 (big endian)**, 在磁盘与内存间搬运数据时必须做字节序转换.

FAT 卷由四个基本区域组成, 在卷上按下列顺序排列:

| 序号 | 区域 |
| --- | --- |
| 0 | **Reserved Region（保留区）** |
| 1 | **FAT Region（FAT 区）** |
| 2 | **Root Directory Region（根目录区）** — FAT32 卷上不存在 |
| 3 | **File and Directory Data Region（文件与目录数据区）** |
