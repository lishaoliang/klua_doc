# 第 2 章 § 2.6 kpfs.mtd

> API: [kpfs_mtd.md](../../kpfs/kpfs_mtd.md) ([`require("kpfs.mtd")`](../../kpfs/kpfs_mtd.md)), [kpfs_disk.md](../../kpfs/kpfs_disk.md) (对比: **禁止** [`disk.mkfs`](../../kpfs/kpfs_disk.md) 格式化 MTD FS) | 枢纽: [readme.md](readme.md) | 对照 pfs_test 章 5

约定见 [readme.md](readme.md). [`require("kpfs.mtd")`](../../kpfs/kpfs_mtd.md) 失败 skip. `ubi.create`/`raw.create` 须 `force = true`. **禁止** 用 [`kpfs.disk.mkfs`](../../kpfs/kpfs_disk.md) 格式化 MTD FS.

对齐 pfs_test 章 5: **`5.x.1` = mkfs**, **`5.x.2` = mkvol**; 本节 Lua 编号独立, 语义对照见下表.

| pfs_test | kpfs Lua | 说明 |
|----------|----------|------|
| 5.1.1 | 2.6.2 | squashfs mkfs |
| 5.1.2 | 2.6.3 | squashfs mkvol |
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

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md) 空镜像; 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)({ path = …, fs = "squashfs", force = true })

预期: `rc == 0`

失败: mkfs 失败; 误用 [`kpfs.disk.mkfs`](../../kpfs/kpfs_disk.md)

注: 默认 `fs = "squashfs"`.

---

### 2.6.3 `mkvol` romfs

- `pfs.mtd.mkvol`

步骤: 1. case 目录建 `tree/`; 2. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 3. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)({ path = …, fs = "romfs", src = tree, force = true })

预期: `mkvol` 成功

失败: mkvol 失败; 源树缺失

---

### 2.6.4 `mkfs` ubifs

- `pfs.mtd.mkfs.ubifs`

步骤: 1. [`mtd.raw.create`](../../kpfs/kpfs_mtd.md); 2. [`mtd.mkfs`](../../kpfs/kpfs_mtd.md)({ fs = "ubifs", force = true })

预期: `rc == 0`

失败: ubifs mkfs 失败

---

### 2.6.5 `mkvol` jffs2

- `pfs.mtd.mkvol.jffs2`

步骤: 1. 建 `tree/`; 2. [`mtd.mkvol`](../../kpfs/kpfs_mtd.md)({ fs = "jffs2", src = tree, jffs2_erase_size = 65536, force = true })

预期: `mkvol` 成功

失败: erase_size 非法; mkvol 失败

---

### 2.6.6 `ubi.create`

- `pfs.mtd.ubi`

步骤: 1. [`mtd.ubi.create`](../../kpfs/kpfs_mtd.md)({ path = "ubi.img", force = true }) (默认 LEB 128KiB × 64); 2. 可选显式 `leb_size` / `leb_count`

预期: `rc == 0`

失败: 参数非法; 创建失败

---
