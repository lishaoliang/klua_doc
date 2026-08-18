# 第 2 章 § 2.6 kpfs.mtd

> API: [kpfs_mtd.md](../../kpfs/kpfs_mtd.md) ([`require("kpfs.mtd")`](../../kpfs/kpfs_mtd.md)), [kpfs_disk.md](../../kpfs/kpfs_disk.md) (对比: **禁止** [`disk.mkfs`](../../kpfs/kpfs_disk.md) 格式化 MTD FS) | 枢纽: [readme.md](readme.md) | 对照 pfs_test 章 5

约定见 [readme.md](readme.md). [`require("kpfs.mtd")`](../../kpfs/kpfs_mtd.md) 失败 skip. `ubi.create`/`raw.create` 须 `force = true`. **禁止** 用 [`kpfs.disk.mkfs`](../../kpfs/kpfs_disk.md) 格式化 MTD FS. `mkfs`/`mkvol` **无** `force` opts.

对齐 pfs_test 章 5: **`5.x.1` = mkfs**, **`5.x.2` = mkvol**; 本节 Lua 编号独立, 语义对照见下表.

| pfs_test | kpfs Lua | 说明 |
|----------|----------|------|
| 5.1.1 | 2.6.2 | squashfs mkfs |
| 5.1.2 | 2.6.7 | squashfs mkvol |
| 5.2.1 | 2.6.4 | ubifs mkfs |
| 5.2.2 | 2.6.9 | ubifs mkvol |
| 5.3.1 | 2.6.11 | jffs2 mkfs |
| 5.3.2 | 2.6.5 | jffs2 mkvol |
| 5.4.1 | 2.6.13 | yaffs mkfs |
| 5.4.2 | 2.6.14 | yaffs mkvol |
| 5.5.1 | 2.6.15 | erofs mkfs |
| 5.5.2 | 2.6.16 | erofs mkvol |
| 5.6.1 | 2.6.17 | cramfs mkfs |
| 5.6.2 | 2.6.18 | cramfs mkvol |
| 5.7.1 | 2.6.8 | romfs mkfs |
| 5.7.2 | 2.6.3 | romfs mkvol |
| (CLI) | 2.6.1 | `mtd raw create` |
| (CLI) | 2.6.6 | `mtd ubi create` |

---

## 2.6 MTD

### 2.6.1 `raw.create`

- `pfs.mtd.raw`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md)({ path = "raw.bin", size = "4M", force = true }); 2. 校验文件存在且 size 512 对齐

预期: `rc == 0`; 文件大小符合 (arm64 可用 2M)

失败: 非对齐 size 仍成功; 文件缺失

---

### 2.6.2 `mkfs` squashfs

- `pfs.mtd.mkfs`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md) 空镜像; 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)("flash.img", { fs = "squashfs" }); 3. [`kpfs.mount`](../../kpfs/kpfs.md)("M", flash_path) + [`probe.fs`](../../kpfs/kpfs_probe.md) 或 VFS 只读访问

预期: `mkfs` `rc == 0`; 可 mount 且 FS 类型为 squashfs

失败: mkfs 失败; 误用 [`kpfs.disk.mkfs`](../../kpfs/kpfs_disk.md)

注: 默认 `fs = "squashfs"`.

---

### 2.6.3 `mkvol` romfs

- `pfs.mtd.mkvol`

步骤: 1. case 目录建 `tree/` (含 `hello.txt`); 2. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 3. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)("flash.img", { fs = "romfs", src = tree }); 4. mount 只读 + [`vfs.open`](../../kpfs/kpfs_vfs.md) 读 `hello.txt`

预期: `mkvol` `rc == 0`; 源树文件可按路径读出

失败: mkvol 失败; 打包内容缺失

---

### 2.6.4 `mkfs` ubifs

- `pfs.mtd.mkfs.ubifs`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)({ path = …, fs = "ubifs" }); 3. mount + 核对 FS 类型

预期: `rc == 0`; `fs_type` 为 ubifs

失败: ubifs mkfs 失败; mount 类型不符

---

### 2.6.5 `mkvol` jffs2

- `pfs.mtd.mkvol.jffs2`

步骤: 1. 建 `tree/`; 2. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)({ path = …, fs = "jffs2", src = tree, jffs2_erase_size = 65536 }); 3. mount + 读回源文件

预期: `mkvol` 成功; 内容可读

失败: erase_size 非法; mkvol 失败; 灌入缺失

---

### 2.6.6 `ubi.create`

- `pfs.mtd.ubi`

步骤: 1. [`mtd.ubi.create`](../../kpfs/kpfs_mtd.md)({ path = "ubi.img", force = true }) (默认 LEB 128KiB × 64); 2. 显式 `leb_size` / `leb_count` 再建一份

预期: 默认与显式参数均 `rc == 0`

失败: 参数非法; 创建失败

---

### 2.6.7 `mkvol` squashfs

- `pfs.mtd.mkvol.squashfs`

步骤: 1. 建 `tree/`; 2. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 3. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)({ fs = "squashfs", src = tree }); 4. mount 只读 + 读回

预期: `mkvol` `rc == 0`; 源树可按路径读出

失败: mkvol 失败; 内容缺失

---

### 2.6.8 `mkfs` romfs

- `pfs.mtd.mkfs.romfs`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)({ fs = "romfs" }); 3. mount 只读 + 核对类型

预期: `rc == 0`; 空 romfs 可 mount

失败: mkfs 失败; 类型不符

---

### 2.6.9 `mkvol` ubifs

- `pfs.mtd.mkvol.ubifs`

步骤: 1. 建 `tree/`; 2. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)({ fs = "ubifs", src = tree }); 3. mount + 读回

预期: `mkvol` 成功; 内容可读

失败: mkvol 失败; 灌入缺失

---

### 2.6.10 `disk.mkfs` 格式化 MTD FS (禁止)

- `pfs.mtd.disk_mkfs.neg`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 2. [`kpfs.disk.mkfs`](../../kpfs/kpfs_disk.md)({ path = …, fs = "squashfs" })

预期: `rc ~= 0`; 卷未被 disk.mkfs 格式化

失败: disk.mkfs 误成功

---

### 2.6.11 `mkfs` jffs2

- `pfs.mtd.mkfs.jffs2`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)({ fs = "jffs2" }); 3. mount + 核对类型

预期: `rc == 0`; FS 类型为 jffs2

失败: mkfs 失败

---

### 2.6.12 多分区 `mkfs` opts 数组

- `pfs.mtd.mkfs.multi`

步骤: 1. 准备含多 MTD 分区的镜像或裸盘; 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)(path, { { fs = "squashfs" }, { fs = "jffs2", size = "4M" } })

预期: 各分区项 `rc == 0`; 可按序 mount 探测

失败: 某分区 mkfs 失败; index 错位

---

### 2.6.13 `mkfs` yaffs

- `pfs.mtd.mkfs.yaffs`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)({ fs = "yaffs2" }); 3. mount + 核对类型

预期: `rc == 0`; FS 类型为 yaffs/yaffs2

失败: mkfs 失败

---

### 2.6.14 `mkvol` yaffs

- `pfs.mtd.mkvol.yaffs`

步骤: 1. 建 `tree/`; 2. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)({ fs = "yaffs2", src = tree }); 3. mount + 读回

预期: `mkvol` 成功; 内容可读

失败: mkvol 失败

---

### 2.6.15 `mkfs` erofs

- `pfs.mtd.mkfs.erofs`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)({ fs = "erofs" }); 3. 只读 mount + 核对类型

预期: `rc == 0`; FS 类型为 erofs

失败: mkfs 失败

---

### 2.6.16 `mkvol` erofs (空树)

- `pfs.mtd.mkvol.erofs`

步骤: 1. 空 `tree/`; 2. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)({ fs = "erofs", src = tree }); 3. 只读 mount

预期: 空树 `rc == 0` (等同空卷 mkfs); 非空树可 `PFS_EOPNOTSUPP` (以实现为准)

失败: 空树 mkvol 失败

注: 非空灌树优先 squashfs/romfs; 本例仅空树.

---

### 2.6.17 `mkfs` cramfs

- `pfs.mtd.mkfs.cramfs`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)({ fs = "cramfs" }); 3. 只读 mount + 核对类型

预期: `rc == 0`; FS 类型为 cramfs

失败: mkfs 失败

---

### 2.6.18 `mkvol` cramfs (空树)

- `pfs.mtd.mkvol.cramfs`

步骤: 1. 空 `tree/`; 2. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)({ fs = "cramfs", src = tree }); 3. 只读 mount

预期: 空树 `rc == 0`; 非空树可 `PFS_EOPNOTSUPP` (以实现为准)

失败: 空树 mkvol 失败

---

### 2.6.19 `raw.create` 非对齐 size (负例)

- `pfs.mtd.raw.neg`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md)({ path = "bad.bin", size = 1000, force = true }) (非 512 对齐)

预期: `rc ~= 0`

失败: 非对齐 size 仍创建成功

---

### 2.6.20 `mkvol` 多分区 + `label`

- `pfs.mtd.mkvol.multi`

步骤: 1. 建 `tree1/`、`tree2/`; 2. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)(path, { { fs = "squashfs", src = tree1, label = "V1" }, { fs = "jffs2", src = tree2, label = "V2", jffs2_erase_size = 65536 } }); 3. 各分区 mount 读回

预期: 多分区 mkvol `rc == 0`; 各源树可读

失败: 某分区失败; label 未生效 (若可探测)

---
