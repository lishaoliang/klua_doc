# pfs 用户工具

> **性质**: 用户 CLI 说明; 实现 `portfs/src_tool/`; 产物 `portfs/bin/pfs`
> **与自测分界**: 回归 harness 见 [pfs_test.md](pfs_test.md); 本工具**无** doc_id / TID / `a`/`all` 语法

---

## 1 总形态

```text
pfs <command> [options] [args]
pfs help | -h | --h
pfs version | -v | --v
```

| 项 | 约定 |
|----|------|
| `<command>` | 一级动词或动词组 (`probe`, `mkfs`, `mkpt`, `ls`, `image`, `mtd`, `info`, `version`) |
| 二级子动词 | 空格分隔: `probe part`, `mkpt gpt`, `image create` |
| `[options]` | 全局选项 + 子命令专属选项; 短选项优先 (`-d` `-o` `-t`); 子命令 `help` / `-h` / `--h` 打印该命令用法 |
| `[args]` | 路径、源目录、扇区号等; UTF-8 |
| `help` | 打印已实现命令总览 (同 `-h` / `--h`; 分组 + 一行说明, 仿 git 首屏) |
| `version` | 打印版本 (同 `-v` / `--v`) |

**`pfs help` 默认总览** (仅列已实现命令):

```text
pfs - port file system tool

usage: pfs <command> [options] [args]
       pfs help | -h | --h
       pfs version | -v | --v

These are common pfs commands:

inspect
   probe              container, partition table, and file systems
   probe part         partition table only (MBR/GPT)
   probe fs           mount probe at one offset
   dump               hex dump sectors or bytes

create / format
   image create       create VHD/VHDX/QCOW2 image shell
   mkpt               write partition table (MBR/GPT)
   mkfs               format empty file system
   mkvol              format and populate from host directory

flash / mtd
   mtd mkfs           format empty flash/MTD file system image
   mtd mkvol          format flash FS and populate from directory

file operations (mounted volume)
   ls                 list directory entries
   cat                print file contents
   mkdir              create directory (-w)
   rm                 remove file (-w)
   rmdir              remove empty directory (-w)
   mv                 rename path (-w)
   cp                 copy file (-w)

See 'pfs help <command>' or 'pfs <command> -h' for syntax.
```

**编译**: WSL/Linux `cd portfs && make tool` → `bin/pfs`; Windows VS2015 → `bin/pfs.exe`.

---

## 2 全局选项

适用于 `probe` / `mkpt` / `mkfs` / `mkvol` / `mtd mkfs` / `mtd mkvol` / `ls` / `cat` / `mkdir` / `rm` / `rmdir` / `mv` / `cp` 等; 解析见 `pfs_tool_opts.c` 与各 `pfs_cmd_*`.

| 选项 | 含义 |
|------|------|
| `-d <path>` | 块设备或镜像路径 |
| `-o <offset>` | 分区内字节偏移; 默认 `0` (裸卷/整镜像) |
| `-r` | 只读 open / RDONLY mount (默认) |
| `-w` | 读写 open / RDWR mount; **破坏性**写 (`mkpt` / `mkfs` / `mkvol` / `mtd *` / 卷内写 / `image create` 覆盖) **须** `-w` |
| `-q` | 少输出 |
| `-v` | 详细 (子命令选项; 顶层 `pfs -v` / `--v` 为 version) |
| `--force` | 与 `-w` 等价 (兼容别名) |

**路径写法**:

| 平台 | 示例 |
|------|------|
| Linux / WSL | `/dev/sdb`, `disk.img`, `test.vhdx` |
| Windows | `\\.\PhysicalDrive1`, `C:\path\disk.vhdx` |

**默认安全**: 无 `-w` 时 blkio 只读 open; mount 先 RDONLY, 失败再试 RDWR (ISOFS 等只读 FS).

### 2.1 退出码

| 码 | 含义 |
|----|------|
| `0` | 成功 (信息性命令如 `probe` 未识别 FS 仍可为 0) |
| `1` | 用法错误 / 未知 command / 未实现命令 |
| `2` | 业务失败 (blkio open、part_probe、mount 等) |

stderr 前缀: `pfs: <command>:` 或 `pfs: session:`.

---

## 3 已实现命令

| 命令 | 状态 | 说明 |
|------|------|------|
| `help` | **已实现** | 命令总览 |
| `version` | **已实现** | 打印 `pfs tool (libpfs)` |
| `probe [<path>]` | **已实现** | 组合探测: 容器 → 分区表 → 各分区 FS; 无分区则裸卷 FS |
| `probe -d <path>` | **已实现** | 同上, 路径用 `-d` |
| `probe part -d <path>` | **已实现** | 仅打印 MBR/GPT |
| `probe fs -d <path>` | **已实现** | 仅 mount 探测 FS (非 raw 容器自动取**首分区**) |
| `image create <path>` | **已实现** | 建 VHD/VHDX/QCOW2 容器壳; 默认 vhdx / 8G / dynamic |
| `mkpt -d <path>` | **已实现** | 写单分区表; 默认 MBR; `-p gpt` 或 `mkpt gpt` 选 GPT |
| `mkfs -d <path> [-t <type>]` | **已实现** | 空卷格式化 (`pfs_mkfs`); 默认 `-t exfat`; 须 `-w` |
| `mkvol -d <path> [-t <type>] -s <srcdir>` | **已实现** | mkfs + 灌宿主目录 (`pfs_mkvol`); 默认 `-t exfat`; 须 `-w` |
| `dump -d <path> [-o <offset>] …` | **已实现** | 扇区/字节 hex dump (`pfs_blkio_pread`) |
| `mtd mkfs -d <path> -t <type>` | **已实现** | Flash/MTD 空卷 (`pfsmtd_mkfs`); 须 `-w` |
| `mtd mkvol -d <path> -t <type> -s <srcdir>` | **已实现** | mkfs + 灌目录 (`pfsmtd_mkvol`); 须 `-w` |
| `ls -d <path> [vfs_path]` | **已实现** | 列目录 (`pfs_opendir` / `pfs_readdir`); 默认 `vfs_path=/` |
| `cat -d <path> <vfs_path>` | **已实现** | 读文件输出 stdout (`pfs_fopen` / `pfs_fread`) |
| `mkdir -d <path> <vfs_path> -w` | **已实现** | 建目录 (`pfs_mkdir`); 须 `-w` |
| `rm -d <path> <vfs_path> -w` | **已实现** | 删文件 (`pfs_unlink`); 须 `-w` |
| `rmdir -d <path> <vfs_path> -w` | **已实现** | 删空目录 (`pfs_rmdir`); 须 `-w` |
| `mv -d <path> <old> <new> -w` | **已实现** | 重命名/移动 (`pfs_rename`); 须 `-w` |
| `cp -d <path> <src> <dst> [-w]` | **已实现** | 单文件拷贝; `/` 卷内, `@` 宿主; 写 VFS 须 `-w` |

### 3.1 `probe` (组合)

**语法**:

```text
pfs probe <path> [-o <offset>] [-q] [-v] [-r|-w]
pfs probe -d <path> [-o <offset>] [-q] [-v] [-r|-w]
pfs probe help | -h | --h
```

子命令 `probe part` / `probe fs` 亦支持末尾或独占的 `help` / `-h` / `--h`.

**步骤**:

1. `pfs_blkio_open` 识别容器 (vhdx → vhd → qcow2 → vmdk → vdi → raw); 打印 `path` / `container` / `sector_size` / `size`
2. `pfs_part_probe` (GPT → MBR); 打印 `scheme` / `count` / 各分区 `part_offset` / `vol_bytes`
3. `count > 0`: 对每个**非空**分区在 `part_offset` 上 mount 探测 FS
4. `count == 0`: 在 `-o` (默认 0) 上探测裸卷 FS

**输出字段** (节选):

```text
path=disk.vhdx
container=vhdx type=... sector_size=512 size=...

scheme=gpt (2) count=1 sector_size=512
[0] start_lba=... size_lba=... bootable=... sys_ind=0x.. part_offset=... vol_bytes=...

[0] fs=exfat type=exfat(4) mode=RDONLY part_offset=1048576
```

未识别 FS 时打印 `fs=(none) rc=...`; 全部失败时末尾 `no mountable file system found`.

**示例**:

```bash
# 位置参数
pfs probe test.vhdx
pfs probe /dev/sdb

# -d 形式
pfs probe -d dynamic_32g.vhd

# 裸卷指定偏移 (无分区表时)
pfs probe bare.img -o 0

# 安静模式
pfs probe -d disk.img -q
```

### 3.2 `probe part`

**语法**:

```text
pfs probe part -d <path> [-q] [-v]
```

仅调用 `pfs_part_probe` 并打印分区表; **不** mount FS.

**示例**:

```bash
pfs probe part -d disk.img
pfs probe part -d \\.\PhysicalDrive1
```

### 3.3 `probe fs`

**语法**:

```text
pfs probe fs -d <path> [-o <offset>] [-q] [-v] [-r|-w]
```

在指定设备上 mount 探测; 对 VHD/VHDX 等非 raw 容器, 若存在分区表则 session **自动取首分区** (`-o` 被覆盖); raw 或裸卷用 `-o` (默认 0).

**与组合 `probe` 差异**: 组合 probe **遍历全部分区**; `probe fs` 只挂**一个**偏移 (首分区或 CLI `-o`).

**示例**:

```bash
pfs probe fs -d u.vhdx
pfs probe fs -d bare_fat32.img -o 0
```

### 3.4 `image create`

**最简**:

```bash
pfs image create disk.vhdx
```

**语法**:

```text
pfs image create <path> [-t <type>] [-s <size>] [--fixed|--dynamic] [-w] [-q]
```

| 项 | 默认 |
|----|------|
| `-t` | `vhdx` (`vhd` / `vhdx` / `qcow2`; `vmdk`/`vdi` 暂无 create API) |
| `-s` | `8G` (须 512 字节对齐) |
| 稀疏/固定 | `--dynamic` (vhd/vhdx); qcow2 仅稀疏 |
| 输出 | 详细 (打印 type/mode/path/size); `-q` 安静 |
| `-w` | 覆盖已存在文件时须 `-w` (或 `--force`) |

**不用 `-d`**: 输出路径为位置参数 `<path>`.

**示例**:

```bash
pfs image create disk.vhdx
pfs image create -t qcow2 -s 32GiB big.qcow2
pfs image create -t vhdx -s 16GiB --fixed fixed.vhdx -w   # 覆盖已有文件
pfs probe disk.vhdx
```

### 3.5 `mkpt`

**破坏性**; 须 `-w`.

**语法**:

```text
pfs mkpt -d <path> [-p mbr|gpt] [-t <fs_hint>] [-w] [-q] [-v]
pfs mkpt mbr -d <path> [-t <fs_hint>] [-w] [-q] [-v]
pfs mkpt gpt -d <path> [-t <fs_hint>] [-w] [-q] [-v]
```

| 项 | 默认 |
|----|------|
| 分区数 | **1** (整盘单分区; 无多分区 CLI; 库现仅 `count==1`) |
| 分区表 | **mbr** (`-p mbr` 或省略; 或位置参数 `mkpt mbr`) |
| GPT | `mkpt gpt` 或 `-p gpt` |
| `-t` | `exfat` (FS 提示, 非分区表类型) |

位置参数 `mbr`/`gpt` 与 `-p` 不可同时指定不同 scheme.

**示例**:

```bash
pfs mkpt -d disk.img -w
pfs mkpt gpt -d disk.vhdx -w
pfs mkpt -d disk.vhdx -p gpt -t fat32 -w
pfs mkpt -d disk.vhdx -w -v          # 写表后 probe 校验
pfs probe part -d disk.vhdx
```

成功时打印 `scheme=... fs_hint=... part_offset=...` (`-v` 时打印完整分区表).

### 3.6 `mkfs`

**破坏性**; 须 `-w`; RDWR blkio open; 在已有容器或 raw 盘上建**空**文件系统; **不**建分区表, **不**灌目录.

**语法**:

```text
pfs mkfs -d <path> [-t <type>] [-o <offset>] [-s <size>] [-w] [-q] [-v]
```

| 项 | 约定 |
|----|------|
| `-d` | 块设备或镜像 (**必填**) |
| `-t` | 文件系统类型; 默认 `exfat`; 见下表 |
| `-o` | 分区字节偏移; 省略时先探测 MBR/GPT, 有分区则**首分区**, 无分区则 `0` (裸卷) |
| `-s` | 卷容量 (`32GiB` / `16G` / 十进制字节); 省略则按解析后的 `part_offset` 命中分区表项的 `vol_bytes`, 否则 `设备容量 - part_offset` |
| `-w` | 破坏性写 (**必填**); `--force` 等价 |
| `--label` | **不支持** (仅 `mkvol` 支持; 指定时报用法错误) |
| `-v` | 格式化后 mount 探测并打印 FS 信息 |

**`-t` 取值**:

| 用户 | 说明 |
|------|------|
| `fat12` / `fat16` / `fat32` | FAT |
| `exfat` | exFAT |
| `ntfs` | NTFS |
| `isofs` / `udf` | 空卷 (无目录内容) |
| `ext2` / `ext3` / `ext4` | ext2/3/4 |

**分区偏移**: 未指定 `-o` 时先 `pfs_part_probe` (MBR/GPT); 有分区则格式化**首分区**; 无分区则在 `-o` 默认 `0` 处格式化裸卷. 显式 `-o` 则不再自动探测.

**示例**:

```bash
pfs mkfs -d disk.img -o 0 -t fat32 -s 32GiB -w
pfs mkfs -d dynamic_32g.vhd -w                      # 默认 exfat; 有分区则首分区
pfs mkfs -d bare_fat32.img -w                       # 无分区表, 裸卷 -o 0
pfs mkfs -d disk.vhdx -o 1048576 -w               # 显式 -o, 不自动探测
pfs mkfs -d disk.vhdx -o 1048576 -t exfat -w -v   # 写盘后 mount 校验
pfs probe disk.vhdx
```

成功时打印 `fs=... part_offset=... vol_bytes=...` (`-q` 安静).

### 3.7 `mkvol`

**破坏性**; 须 `-w`; RDWR blkio open; 等价 `mkfs` + 将宿主目录灌入卷 (只读 FS 如 ISOFS/UDF 为一次打包).

**语法**:

```text
pfs mkvol -d <path> [-t <type>] -s <srcdir> [-o <offset>] [--label <id>] [-w] [-q] [-v]
```

| 项 | 约定 |
|----|------|
| `-d` | 块设备或镜像 (**必填**) |
| `-t` | 文件系统类型; 默认 `exfat`; 取值同 §3.6 |
| `-s` | 宿主源目录 UTF-8 路径 (**必填**; 与 `mkfs` 的 `-s <size>` 语义不同) |
| `-o` | 分区字节偏移; 省略时规则同 §3.6 (探测 MBR/GPT, 有分区则首分区) |
| `--label` | 卷标 UTF-8; 传给 `pfs_mkvol` 的 `volume_id` (可省略) |
| 卷容量 | **无**独立 `-s <size>` 选项; 自动解析 (规则同 `mkfs` 省略 `-s`) |
| `-w` | 破坏性写 (**必填**); `--force` 等价 |
| `-v` | 灌入后 mount 探测并打印 FS 信息 |

**示例**:

```bash
pfs mkvol -d disk.vhdx -s ./rootfs -w                      # 默认 exfat
pfs mkvol -d disk.vhdx -o 1048576 -t fat32 -s ./rootfs -w
pfs mkvol -d disk.vhdx -s ./rootfs --label USB -w -v
pfs probe disk.vhdx
```

成功时打印 `fs=... part_offset=... vol_bytes=...` 与 `src_dir=...` (`-q` 安静).

### 3.8 `dump`

**只读**; RDONLY blkio open; 经 `pfs_blkio_pread` 输出 hex.

**语法**:

```text
pfs dump -d <path> [-o <offset>] [sector] [-c <count>] [-l <bytes>] [-q] [-v]
pfs dump <path> [sector] [-o <offset>] ...
```

| 项 | 约定 |
|----|------|
| `-d` / 位置参数 | 块设备或镜像 (**必填**) |
| `-o` | 分区/卷起始字节偏移; 默认 `0` |
| `sector` | 相对 `-o` 的扇区号; 默认 `0` |
| `-c` | 连续扇区数; 默认 `1`; 与 `-l` 互斥 |
| `-l` | 字节长度; 与 `-c` 互斥 |
| `-v` | 打印 `sector` / `part_offset` / `byte_off` / `len` 摘要 |
| `-w` | **不支持** (只读) |

绝对读偏移: `byte_off = part_offset + sector * sector_size`.

**示例**:

```bash
pfs dump -d disk.img                    # LBA0 单扇区
pfs dump -d disk.vhdx -o 1048576 0    # 首分区起始扇区
pfs dump disk.img 0 -c 4                # 从 LBA0 起 4 扇区
pfs dump -d disk.img -l 64 -v           # 从 LBA0 起 64 字节
pfs probe part -d disk.vhdx             # 取 part_offset 后再 dump
```

### 3.9 `mtd mkfs` / `mtd mkvol`

**Flash/MTD 文件系统**; 调用 **`pfsmtd_mkfs` / `pfsmtd_mkvol`** (非顶层 `pfs_mkfs`); RDWR blkio open; 目标介质多为 **raw 线性镜像** 或 MTD 设备, **通常无需** `mkpt`.

**语法**:

```text
pfs mtd mkfs -d <path> -t <type> [-o <offset>] [-s <size>] [-w] [-q] [-v]
pfs mtd mkvol -d <path> -t <type> -s <srcdir> [-o <offset>] [--erase-size <bytes>] [--label <id>] [-w] [-q] [-v]
pfs mtd mkfs help | -h | --h
```

| 项 | 约定 |
|----|------|
| `-d` | 块设备或 **Flash 镜像路径** (**必填**) |
| `-t` | **必填**; 见下表 (`PFSMTD_*`) |
| `-o` | 字节偏移; 默认 `0` (整镜像) |
| `-s` | **mkfs**: 卷容量; 省略则用设备容量 `- part_offset`. **mkvol**: 宿主源目录 (**必填**) |
| `-w` | 破坏性写 (**必填**); `--force` 等价 |
| `--erase-size` | 仅 **mkvol** 且 `-t jffs2`; 见 §3.9.1 |
| `--label` | 仅 **mkvol**; 传给 `pfsmtd_mkvol_opts.volume_id`; 各 FS 是否采纳见下表 |
| `-v` | 格式化/灌入后 mount 探测 |

**`-t` 取值** (库 `PFSMTD_*`):

| 用户 | 说明 | 按 FS 说明 |
|------|------|------------|
| `squashfs` / `romfs` | 只读压缩/ROM 根; **mkvol** 可灌树 | §3.9.2 |
| `ubifs` / `jffs2` / `yaffs` / `yaffs2` | 可写 Flash 类 (库为写子集) | jffs2 见 §3.9.1; ubifs/yaffs 见 §3.9.4 |
| `erofs` / `cramfs` | 只读; **mkvol** 非空目录 `EOPNOTSUPP` | §3.9.3 |

**与块设备 `mkfs` 分界**: 块 FS (`fat32`/`exfat`/…) 用 §3.6–§3.7; Flash 镜像 **不归** `image create`, 用本节 `mtd mkfs`/`mkvol`.

**示例**:

```bash
# 空 jffs2 (16MiB raw 镜像须预先存在且可写, 或整设备)
truncate -s 16M flash.jffs2
pfs mtd mkfs -d flash.jffs2 -t jffs2 -s 16M -w
pfs probe -d flash.jffs2

# mkfs + 灌目录 (自测可省略 --erase-size; 对齐真实 MTD 见 §3.9.1)
pfs mtd mkvol -d flash.jffs2 -t jffs2 -s ./rootfs -w -v
pfs mtd mkvol -d flash.jffs2 -t jffs2 -s ./rootfs --erase-size 64K -w

# squashfs 打包
pfs mtd mkvol -d root.sqsh -t squashfs -s ./rootfs -w
```

成功时打印 `fs=... part_offset=... vol_bytes=...` (`mkvol` 另含 `src_dir=`).

#### 3.9.1 `jffs2` (`--erase-size`)

| 项 | 约定 |
|----|------|
| 适用命令 | 仅 **`mtd mkvol -t jffs2`** (`mkfs` 无此选项) |
| `--erase-size <bytes>` | 擦除块字节数, 写入 `pfsmtd_mkvol_opts.jffs2_erase_size` |
| 校验 | `>= 512` 且为 **2 的幂**; 否则用法错误 |
| 省略 / `0` | **线性追加** 布局 (pfs 自测默认); 扫描不依赖擦除块对齐 |
| 实机 MTD | 宜与 NAND/NOR **擦除块大小一致** (如 `64K` / `128K` / `256K`), 便于与 Linux `mtd` 行为对齐 |

`mtd mkfs -t jffs2` 仅写空卷 (全 `0xFF` + 根目录 inode); 灌目录须 `mtd mkvol`. `--label` 当前 jffs2 栈**忽略**.

```bash
# 64KiB 擦除块 (常见 NOR)
pfs mtd mkvol -d flash.jffs2 -t jffs2 -s ./rootfs --erase-size 65536 -w -v
pfs mtd mkvol -d flash.jffs2 -t jffs2 -s ./rootfs --erase-size 64K -w
```

#### 3.9.2 `squashfs` / `romfs`

| 项 | 约定 |
|----|------|
| **mkvol** | 自 `srcdir` **一次打包** 整棵目录树 (只读根文件系统) |
| **mkfs** | 空卷 (无文件内容) |
| `-s <size>` (mkfs) / `vol_bytes` | squashfs/romfs **打包布局暂不参与** `vol_bytes` (库分派注释) |
| `--label` | **romfs**: 写入 superblock 卷名 (UTF-8, 可省略). **squashfs**: 当前栈**忽略** |

```bash
pfs mtd mkvol -d root.sqsh -t squashfs -s ./rootfs -w
pfs mtd mkvol -d root.romfs -t romfs -s ./rootfs --label ROOT -w -v
```

#### 3.9.3 `erofs` / `cramfs`

| 项 | 约定 |
|----|------|
| **mkfs** | 空只读卷 |
| **mkvol** | `srcdir` **须为空目录** (仅走 `mkfs`); 非空树返回 **`EOPNOTSUPP`** |
| 灌树替代 | 用 **`squashfs`** 或 **`romfs`** (`§3.9.2`) |
| `--label` | 当前栈**忽略** |

```bash
mkdir -p empty && pfs mtd mkvol -d empty.erofs -t erofs -s empty -w
# pfs mtd mkvol -d x.erofs -t erofs -s ./rootfs -w   # 非空 -> 失败
```

#### 3.9.4 `ubifs` / `yaffs` / `yaffs2`

| 项 | 约定 |
|----|------|
| **mkvol** | `mkfs` + 自 `srcdir` 灌入 (可写 Flash 写子集) |
| **mkfs** | 空卷; `-s <size>` 限制格式化范围 |
| `-t yaffs` / `yaffs2` | 解析为同一 `PFSMTD_YAFFS` |
| 专属 CLI 选项 | 无 (无 `--erase-size` 等); `--label` 当前栈**忽略** |
| **ubifs** 实机 | 常需先 `mtd ubi create` 等 UBI 层 (见工具规划); 裸 linear 镜像多用于 pfs 自测 |

### 3.10 挂载后文件操作 (`ls` / `cat` / `mkdir` / `rm` / `rmdir` / `mv` / `cp`)

须 session **mount**; 实现 `pfs_cmd_vfs.c`. **只读**命令默认 RDONLY open + `mount_probe`; **写**命令须 **`-w`** (RDWR open + RDWR mount), 否则报 `write requires -w`.

**公共约定**:

| 项 | 约定 |
|----|------|
| `-d <path>` | 块设备或镜像 (**必填**) |
| `-o <offset>` | 分区字节偏移; 默认 `0`; 非 raw 容器且存在分区表时 session **自动取首分区** (同 `probe fs`) |
| `vfs_path` | 卷内 UTF-8 绝对路径, **须以 `/` 开头** |
| `-r` / `-w` | 默认只读; 写子命令须 `-w` |
| `-q` / `-v` | 少输出 / 详细 (`ls` 的 `-v` 为类型前缀) |
| `--force` | 与 `-w` 等价 (兼容别名) |
| 只读 FS | ISOFS/UDF 等: 读命令可用; 写命令失败 (`EROFS` 等) |

**`ls`** — 列目录项 (含 `.` / `..`):

```text
pfs ls -d <path> [vfs_path] [-o <offset>] [-q] [-v] [-r|-w]
pfs ls help | -h | --h
```

| 项 | 约定 |
|----|------|
| `vfs_path` | 省略时默认 `/` |
| `-v` | 每行前加类型前缀: `d` 目录 / `-` 普通文件 / `?` 未知 |

**`cat`** — 读文件写 stdout (二进制原样):

```text
pfs cat -d <path> <vfs_path> [-o <offset>] [-q]
```

**`mkdir`** — 建单级目录 (不递归父路径):

```text
pfs mkdir -d <path> <vfs_path> [-o <offset>] [-w] [-w]
```

**`rm`** — 删**文件** (`pfs_unlink`; 非目录):

```text
pfs rm -d <path> <vfs_path> [-o <offset>] [-w] [-w]
```

**`rmdir`** — 删**空**目录:

```text
pfs rmdir -d <path> <vfs_path> [-o <offset>] [-w] [-w]
```

**`mv`** — 重命名或同卷移动:

```text
pfs mv -d <path> <old_vfs_path> <new_vfs_path> [-o <offset>] [-w]
```

**`cp`** — 拷贝**单文件** (不递归目录). 路径: **`/…`** 卷内 VFS; **`@…`** 宿主机 (类似 curl `@file`). 一侧 `@` 即可与卷互拷; 卷内→卷内两端均为 `/`.

```text
pfs cp -d <path> <src> <dst> [-o <offset>] [-w]
```

| 方向 | 示例 | `-w` |
|------|------|------|
| 卷内 | `cp … /a /b` | 要 |
| 宿主→卷 | `cp … @./local /remote` | 要 |
| 卷→宿主 | `cp … /remote @./local` | 不要 |

**示例**:

```bash
pfs cp -d disk.vhdx @./payload.bin /data/payload.bin -w
pfs cp -d disk.vhdx /data/readme.txt @./readme.txt
pfs cp -d disk.img /a.txt /b.txt -w
```

**其它 VFS 示例**:

```bash
pfs ls -d disk.vhdx /
pfs cat -d disk.vhdx /readme.txt
pfs mkdir -d disk.img /newdir -w
pfs mv -d disk.img /b.txt /c.txt -w
pfs rm -d disk.img /c.txt -w
```

**限制** (相对常见 shell 工具):

| 项 | 说明 |
|----|------|
| `cp` / `rm` | 仅单文件; **无** `-r` 递归目录; 宿主路径须 `@` 前缀 |
| `mkdir` | **无** `-p` 递归建父目录 |
| `rm` | 不删目录; 删目录树用 `rmdir` (须已空) 或后续 `remove` API 扩展 |

---

## 4 参考工作流

制盘三层正交: **容器 (blkio)** → **分区表** → **文件系统**.

```text
pfs image create disk.vhdx
pfs mkpt -d disk.vhdx -w                  # 默认 MBR; 或 mkpt gpt / -p gpt
pfs mkfs -d disk.vhdx -w                  # 自动探测首分区; 默认 exfat
pfs probe disk.vhdx
```

灌目录可用 `mkvol` 替代 `mkfs`:

```bash
pfs mkvol -d disk.vhdx -o 1048576 -t fat32 -s ./rootfs -w
```

裸卷 (无分区表): 跳过 `mkpt`, 直接 `mkfs -d … -w` 或 `mkvol … -w`.

**术语对照**:

| 用户说法 | 工具命令 |
|----------|----------|
| 做分区 | `mkpt` (默认 MBR) / `mkpt gpt` / `mkpt -p gpt` |
| 格式化 / 做 FS | `mkfs` |
| 做可用盘 / 灌文件 | `mkvol` |

**当前可用 (P0 制盘)**: `image create` / `mkpt` / `mkfs` / `mkvol` / `probe` / `dump`.

**挂载后文件 (P1)**: `ls` / `cat` / `mkdir` / `rm` / `rmdir` / `mv` / `cp` (见 §3.10).

**Flash (MTD)**: `mtd mkfs` / `mtd mkvol` (见 §3.9); `mtd ubi` 未实现.

**制盘后浏览/改文件** (裸卷示例):

```bash
pfs mkfs -d disk.img -t fat32 -w
pfs ls -d disk.img /
pfs mkdir -d disk.img /data -w
pfs mkvol -d disk2.img -t fat32 -s ./rootfs -w
pfs cat -d disk2.img /somefile.txt
```

---

## 5 待实现命令

路由已注册 (`pfs_tool_dispatch.c`); 未实现命令退出码 `1` 并打印 `not implemented yet`.

### 5.1 一期 P0 (制盘主路径)

`mkpt` / `mkfs` / `mkvol` 见 §3.5–§3.7; `dump` 见 §3.8.

### 5.2 二期 P1 (挂载后文件操作)

§3.10 中 `ls` / `cat` / `mkdir` / `rm` / `rmdir` / `mv` / `cp` **已实现**; 下表为**仍待实现**:

| 命令 | 语法 (规划) | API |
|------|-------------|-----|
| `access` | `access -d <path> <vfs_path>` | `pfs_access` |
| `info fs` | `info fs -d <path>` | `pfs_ctx_fs_type` / sbi 摘要 |

### 5.3 三期 P2 (镜像 / 设备信息)

| 命令 | 状态 | 说明 |
|------|------|------|
| `image create` | **已实现** | 见 §3.4; 最简 `pfs image create <path>` |
| `image info` | 未实现 | 容器类型与容量 |
| `info dev` | 未实现 | 扇区大小、容量 (偏物理/raw) |

**示例 (规划)**:

```bash
pfs image info disk.vhdx
pfs info dev -d /dev/sdb
```

### 5.4 四期 P3 (MTD)

session 须 `PFSMTD_REGISTER_FS_ALL` (probe / mkfs 校验已注册).

| 命令 | 状态 | API |
|------|------|-----|
| `mtd mkfs` | **已实现** | `pfsmtd_mkfs` (见 §3.9) |
| `mtd mkvol` | **已实现** | `pfsmtd_mkvol` (见 §3.9) |
| `mtd ubi` | 未实现 | `pfsmtd_ubi_*` |

Flash 镜像 (squashfs / jffs2 / …) **不归** `image` 子命令, **不归** 块设备 `mkfs -t`.

### 5.5 明确不做

| 项 | 归属 |
|----|------|
| doc_id / TID / `a`/`all` 批量语法 | **pfs_test** |
| 长期 mount 守护进程 | 非 portfs 定位 |
| 单一 `format` 混做分区+FS | 须显式 `mkpt` + `mkfs` 或 `mkvol` |
| 完整 fsck | API 未就绪; 日后可 `check` |

---

## 6 与 `pfs_test` 对照

| 维度 | **pfs** 工具 | **pfs_test** |
|------|--------------|--------------|
| argv | 稳定子命令 + POSIX 风格选项 | `1.2.1` / TID / `a`/`all` |
| 用途 | 运维 / 制盘 / 手测 | 回归 harness |
| 破坏性 | 显式 `-w` (`mkpt` / `mkfs` / `mkvol` 等) | harness flags |
| 文档 | 本文 | [pfs_test.md](pfs_test.md) |

---

## 7 实现分期速查

| 阶段 | 命令 | 实现文件 |
|------|------|----------|
| — | `help`, `version` | `pfs_tool_dispatch.c`, `pfs_tool_print.c` |
| P0 | `probe`, `probe part`, `probe fs`, `mkpt`, `mkfs`, `mkvol`, `dump` | `pfs_cmd_fs.c`, `pfs_cmd_part.c`, `pfs_cmd_blkio.c` |
| P1 | `ls`, `cat`, `mkdir`, `rm`, `rmdir`, `mv`, `cp` | `pfs_cmd_vfs.c` (**已实现**) |
| P1 | `access`, `info fs` | `pfs_cmd_vfs.c` (**未实现**) |
| P2 | `image create` | `pfs_cmd_blkio.c` |
| P2 | `image info`, `info dev` | `pfs_cmd_blkio.c` |
| P3 | `mtd mkfs`, `mtd mkvol` | `pfs_cmd_mtd.c` |
| P3 | `mtd ubi` | `pfs_cmd_mtd.c` |

---

## 8 外部参考: 阿里 PolarDB PFS 工具

> **性质**: 第三方 CLI 命令行格式摘录, **非** portfs `pfs` 实现; 仅供命名对照与命令设计参考.
> **真源**: [ApsaraDB/PolarDB-FileSystem](https://github.com/ApsaraDB/PolarDB-FileSystem) — [PFS_Tools-CN.md](https://github.com/ApsaraDB/PolarDB-FileSystem/blob/master/docs/PFS_Tools-CN.md) / [PFS_Tools-EN.md](https://github.com/ApsaraDB/PolarDB-FileSystem/blob/master/docs/PFS_Tools-EN.md)
> **说明**: 官方称该工具**仅供调试测试**; 运行时依赖 `pfsdaemon` (安装于 `/usr/local/polarstore/pfsd/`).

### 8.1 总形态

```text
pfs [-H hostid] [-C disk] <command> [options] <pbdpath>
```

| 项 | 约定 |
|----|------|
| `-C disk` | 操作本地块设备 (如 `nvme1n1`) |
| `-H hostid` | 集群内 host 标识, 须唯一 (可选) |
| `<pbdpath>` | 路径格式 `/pbdname/path`, 例 `/nvme1n1/mydir/myfile` |
| 权限 | 多数操作需 `sudo` |

**示例**:

```bash
sudo pfs -C disk mkfs nvme1n1
sudo pfs -C disk ls /nvme1n1/
sudo pfs -C disk touch /nvme1n1/hello.txt
```

### 8.2 文件系统级命令 (对磁盘/PBD)

| 命令 | 功能 | 主要选项 |
|------|------|----------|
| `mkfs` | 格式化创建 PFS | `-u` 最大写实例数; `-l` journal 大小; `-f` 强制 |
| `growfs` | 扩容后格式化新 chunk | `-o` 扩前 chunk 数; `-n` 扩后 chunk 数; `-f` 强制 |
| `info` | 打印元数据使用情况 | 无 |
| `dumpfs` | dump superblock / chunk header / metaobj | `-m`, `-t`, `-c`, `-o` |
| `dumple` | 读取 journal 中 log entry | `-a`, `-t`, `-b`, `-d`, `-i` |

```bash
sudo pfs -C disk mkfs -u 10 nvme1n1
sudo pfs -C disk growfs -o 1 -n 3 nvme1n1
sudo pfs -C disk info nvme1n1
sudo pfs -C disk dumpfs nvme1n1
sudo pfs -C disk dumple -a nvme1n1
```

### 8.3 文件/目录通用命令

| 命令 | 功能 | 主要选项 |
|------|------|----------|
| `stat` | 查看属性 | 无 |
| `rm` | 删除文件或目录 | `-r` 删目录 |
| `rename` | 重命名/移动 | 无 |
| `du` | 统计磁盘用量 | `-a` 含文件; `-d` 深度 |
| `cp` | 拷贝文件或目录 | `-r` 拷目录 |

### 8.4 文件命令

| 命令 | 功能 | 主要选项 |
|------|------|----------|
| `touch` | 创建文件 | 无 |
| `write` | 写文件 (从 stdin) | `-o` 偏移; `-l` 长度 |
| `read` | 读文件 | `-o` 偏移; `-l` 长度 |
| `truncate` | 调整文件大小 (不分配物理块) | `-l` 新长度 |
| `fallocate` | 预分配物理块 | `-o` 偏移; `-l` 长度 |
| `map` | 显示文件 block 索引表 | `-o` 偏移 |

```bash
echo "012345" | sudo pfs -C disk write -o 2 -l 3 /nvme1n1/myfile
sudo pfs -C disk read -o 2 -l 10 /nvme1n1/myfile
sudo pfs -C disk truncate -l 104857600 /nvme1n1/mydir/myfile
```

### 8.5 目录命令

| 命令 | 功能 | 主要选项 |
|------|------|----------|
| `mkdir` | 创建目录 | `-p` 递归建父目录 |
| `ls` | 列目录 | 无 |
| `tree` | 打印目录树 | `-v` 详细信息 |
| `rmdir` | 删除空目录 | 无 |

### 8.6 与 portfs `pfs` 对照 (命名冲突备忘)

| 维度 | PolarDB `pfs` | portfs `pfs` (本文 §1–§7) |
|------|---------------|---------------------------|
| 定位 | PolarDB 分布式块存储专有 FS | 通用镜像/块设备上的 FAT/NTFS/ext4 等 |
| 设备指定 | `-C disk` + PBD 路径 `/nvme1n1/...` | `-d <path>` + 可选 `-o <offset>` |
| 子命令形态 | 单层动词 (`mkfs`, `ls`, `mkdir` …) | 动词组 (`probe part`, `mkpt gpt`, `image create` …) |
| 重叠子命令名 | `mkfs`, `ls`, `mkdir`, `rm`, `cp`, `stat` | `mkfs` 等见 §3; `ls`/`cat`/… 见 §3.10; **无** `growfs`/`dumpfs`/`dumple`/`fallocate`/`map` |
| 特有命令 | `growfs`, `dumpfs`, `dumple`, `fallocate`, `map` | `probe`, `mkpt`, `image`, `mtd`, `mkvol` 等 |
