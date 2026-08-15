# 文件系统基础概念（个人笔记）

> **性质**: 个人对块设备 / 分区 / 扇区 / 簇等通识的理解; 非官方规范; 供 `pfs_fat.md` / `pfs_exfat.md` / `pfs_ntfs.md` 共用.
> **冲突**: 各 FS 细节以对应 `std_*` 与官方源为准.


# 1 存储设备(块设备)

```
软盘、硬盘、U盘、SD卡、TF卡等
```

* 本质: 按**块**存取的介质; 对外常看成一块连续的大「字节空间」, 但底层读写以扇区/页为单位对齐.
* 访问地址: 可类比内存, 从 `0x0` 起编址到容量末尾; 实际接口多用 **LBA**(Logical Block Address, 逻辑块号) 或**字节偏移**, 再换算到扇区.
* 与字符设备区别: 块设备可随机定位读写; 字符设备多是流式顺序访问.
* 常见形态:
  * 机械盘 / SSD: 整盘或分区节点
  * 可移动: U盘、SD/TF 卡 (多经读卡器/USB 呈现为块设备)
  * 软盘: 历史形态; 扇区几何与现代盘不同, 通识仍适用「扇区→簇」分层
* 对文件系统: FS **不直接**关心盘片物理几何, 只认「可按扇区/字节偏移读写的块设备」(或其上的一个分区).
* 与 pfs: `pfs_blkio` 即对块设备(或镜像文件)做对齐读写的封装; 上层 fat/exfat/ntfs 挂在 blkio 之上.

# 2 扇区

```
将 512B、1024B、2048B、4096B 组成一组, 称为一个扇区
```

* 扇区 (Sector): 块设备对外的**最小读写单元** (逻辑扇区); 常见 512 / 1024 / 2048 / 4096 字节.
* **逻辑扇区** vs **物理扇区**: 逻辑是 OS/API 看到的单元; 物理是介质真实页/扇区大小 (如 Advanced Format 物理 4K、逻辑仍报 512). 写对齐宜参考物理扇区; 编址换算多用逻辑扇区.
* 地址换算: `字节偏移 = LBA × 扇区大小`; `LBA = 字节偏移 / 扇区大小` (须整除).
* 对 FS: Boot/BPB 等结构按扇区布局; 读写未对齐扇区时, 驱动或库常做 RMW (读-改-写).
* 与 pfs: `pfs_blkio_sector_size` / `sector_size_physical` 探测; `open` / `open_direct` 的对齐粒度取 512~4096.

# 3 簇(块)

* Windows / 多数 FAT 族文档称 **簇 (Cluster)**
* Linux VFS 语境常称 **块 (block)** (与「磁盘扇区」不是同一层)

```
将多个扇区组成一组, 称为一个簇(块)
文件系统一般按簇(块)分配
```

* 簇 = FS 的**空间分配单位**; 大小 = `每簇扇区数 × 扇区大小` (常为 2 的幂).
* 文件/目录内容按簇占坑; 小文件也至少占一簇 (可能浪费尾部空间).
* 簇号: 逻辑编号, 再映射到分区内字节/扇区偏移; **具体编号起点因 FS 而异** (如 exFAT/FAT 数据簇多自 **2** 起, 0/1 保留).
* 与扇区关系: 扇区是设备层; 簇是 FS 层. FS 读写最终仍落到扇区对齐的块 I/O.
* 与 pfs: fat/exfat/ntfs 各自在 raw/超级块里解释簇大小与簇号映射.

# 4 簇堆（块堆)

* **Cluster Heap** (簇堆): 卷上「按簇编号的那一大片数据区」的通称; exFAT 规范正式用此名.
* 典型内容: 分配位图 / 大写表 / 根目录 / 子目录 / 文件数据等 (因 FS 而异; 经典 FAT 的根目录区布局不同).
* 理解要点:
  * Boot / FAT 等元数据区通常**在簇堆之前**(或另有固定区)
  * 簇堆内用**簇号**寻址; 簇链或位图描述哪些簇已用
  * 「堆」不是内存堆, 只是一片连续的簇编号空间
* FAT 族口语也可说「数据区」; 读 exFAT 时与规范用语 **Cluster Heap** 对齐即可.
* 细节: 见各 `pfs_{fs}.md` 与 `std_*` (exFAT 章「Data Region / Cluster Heap」).

# 5 分区

```
将一个物理磁盘分为多个逻辑磁盘
```

* 分区: 在整盘上划出的连续区间; 每个分区可独立格式化一种 FS, 对上层常表现为单独块设备.
* **MBR** (Master Boot Record):
  * 盘首扇区含引导码 + 分区表 (经典最多 4 主分区; 扩展分区可挂逻辑分区)
  * 寻址历史包袱多 (如 2TiB 量级限制等); 老盘/兼容场景仍常见
* **GPT** (GUID Partition Table):
  * UEFI 常用; 分区用 GUID; 表有备份; 支持大容量与更多分区
  * 常带 Protective MBR 以免老工具误判
* FS 视角: mkfs/mount 多针对**某一个分区**(或整盘当单分区镜像); Boot 扇区相对**分区起点**为 0, 不是整盘 0 (除非无分区表、整盘即一体积).
* 注意: 改分区表 ≠ 改 FS; 动分区边界会毁掉其上文件系统.

# 6 系统接口

用户态打开块设备后, 按**字节偏移**读写 (内部再对齐到扇区). pfs 统一走 `pfs_blkio_open` / `pread` / `pwrite`.

## 6.1 Windows

* API 形态: `CreateFile` 打开设备名, 再 `ReadFile` / `WriteFile` (或 overlapped); 常需管理员权限.
* 缓冲 vs 直接: 默认经系统缓存; `FILE_FLAG_NO_BUFFERING` 时 offset/size/缓冲地址须扇区对齐 (pfs: `open_direct`).

```
整盘读写方式:
  \\.\PhysicalDriveN
  例: \\.\PhysicalDrive0  (第一块物理盘; N 从 0 起)
```

```
分区盘读写方式:
  \\.\X:                 (已挂载盘符, 如 \\.\D:)
  \\.\HarddiskVolumeN    (卷设备名; N 视系统枚举)
  也可用卷 GUID 路径: \\.\Volume{GUID}\
```

* 注意: 对已挂载卷做裸写易与系统 FS 驱动冲突; 测试/修复场景常先卸载或只读打开.

## 6.2 Linux

* API 形态: `open` + `pread` / `pwrite` / `lseek`; 设备节点在 `/dev`.
* 直接 I/O: `O_DIRECT`, 同样要求对齐 (pfs: `open_direct`).

```
整盘读写方式:
  /dev/sdX     例: /dev/sda
  /dev/nvme0n1
  /dev/mmcblk0  (SD/eMMC 等)
```

```
分区盘读写方式:
  /dev/sdXN      例: /dev/sda1
  /dev/nvme0n1p1
  /dev/mmcblk0p1
  也可对 loop/镜像: /dev/loop0 或直接 open 镜像文件当「块设备」用
```

* 权限: 通常需 root 或 `disk` 组; 对已挂载分区裸写同样危险.

## 6.3 与 pfs 对应

| 概念 | pfs |
|------|-----|
| 打开整盘/分区/镜像路径 | `pfs_blkio_open` / `open_direct` (`p_devpath`) |
| 按字节偏移读/写 | `pfs_blkio_pread` / `pwrite` |
| 逻辑/物理扇区探测 | `pfs_blkio_sector_size` / `sector_size_physical` |
| Win 整盘示例路径 | `\\\\.\\PhysicalDrive1` (见头文件注释) |
| Linux 整盘示例路径 | `/dev/sda` |
