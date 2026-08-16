# 第 2 章 § 2.2 制盘 (容器 / 分区 / FS)

> API: [kpfs.md](../../kpfs/kpfs.md) ([`require("kpfs")`](../../kpfs/kpfs.md), mount), [kpfs_disk.md](../../kpfs/kpfs_disk.md) (`kpfs.disk`, mkpt/mkfs/mkvol), [kpfs_image.md](../../kpfs/kpfs_image.md) (`kpfs.image`), [kpfs_probe.md](../../kpfs/kpfs_probe.md) (`kpfs.probe`) | 枢纽: [readme.md](readme.md) | 对照 pfs_test 章 3 / 章 4.4, pfs CLI `image`/`mkpt`/`mkfs`/`mkvol`

约定见 [readme.md](readme.md). [`image.create`](../../kpfs/kpfs_image.md) **无** `force` opts (脚本即破坏性); [`mkpt`](../../kpfs/kpfs_disk.md)/[`mkfs`](../../kpfs/kpfs_disk.md)/[`mkvol`](../../kpfs/kpfs_disk.md) 同 probe/image 三种形态、**无** `force`. MTD FS **禁止** 本节 `mkfs`, 走 §2.6.

---

## 2.2 制盘

### 2.2.1 Dynamic VHDX

- `pfs.image`

步骤: 1. [`image.create`](../../kpfs/kpfs_image.md)(path, { type = "vhdx", size = "4M", layout = "dynamic" }); 2. [`image.info`](../../kpfs/kpfs_image.md)(path); 3. 比对 `container`, `sector_size`, `size`

预期: `create`/`info` 成功; `container == "vhdx"`, `sector_size == 512`

失败: create 失败; info 字段不符

---

### 2.2.2 Dynamic VHD

- `pfs.image.vhd`

步骤: 1. [`image.create`](../../kpfs/kpfs_image.md)(path, { type = "vhd", size = "4M", layout = "dynamic" }); 2. [`image.info`](../../kpfs/kpfs_image.md) 校验 `container == "vhd"`

预期: 创建成功; info 与参数一致

失败: vhd create 不支持或失败

---

### 2.2.3 Fixed VHDX

- `pfs.image.fixed`

步骤: 1. [`image.create`](../../kpfs/kpfs_image.md)(path, { type = "vhdx", size = "8M", layout = "fixed" }); 2. [`image.info`](../../kpfs/kpfs_image.md) 校验 `size`

预期: fixed 镜像创建成功; `size` 与请求一致 (arm64 可用 2M)

失败: fixed 布局失败; 尺寸不符

注: `qcow2` 仅 `dynamic`.

---

### 2.2.4 Fixed VHD

- `pfs.image.vhd_fixed`

步骤: 1. [`image.create`](../../kpfs/kpfs_image.md)(path, { type = "vhd", size = "8M", layout = "fixed" }); 2. [`image.info`](../../kpfs/kpfs_image.md) 校验 `container == "vhd"`, `size`

预期: fixed 镜像创建成功; `size` 与请求一致 (arm64 可用 2M)

失败: fixed 布局失败; 尺寸不符

---

### 2.2.5 写分区表 `mkpt`

- `pfs.mkpt`

步骤: 1. [`image.create`](../../kpfs/kpfs_image.md) 空 vhdx; 2. [`disk.mkpt`](../../kpfs/kpfs_disk.md)({ path = …, scheme = "gpt", fs = "exfat" }); 3. [`kpfs.probe.part`](../../kpfs/kpfs_probe.md) 读回

预期: `mkpt` `rc == 0`; [`probe.part`](../../kpfs/kpfs_probe.md) 显示 `scheme == "gpt"`, `part_count >= 1`; `parts[]` 无 `fs_type`

失败: mkpt 失败; 分区表不符

---

### 2.2.6 空卷格式化 `mkfs`

- `pfs.mkfs`

步骤: 1. [`image.create`](../../kpfs/kpfs_image.md) + `mkpt`; 2. [`disk.mkfs`](../../kpfs/kpfs_disk.md)({ path = …, fs = "exfat" }); 3. [`kpfs.probe.fs`](../../kpfs/kpfs_probe.md) 读回首分区

预期: `mkfs` 成功; [`probe.fs`](../../kpfs/kpfs_probe.md) 显示 `mount_ok == true`

失败: mkfs 失败; 探测类型不符

注: **不支持** `opts.label`; MTD FS 走 §2.6.

---

### 2.2.7 目录树制卷 `mkvol`

- `pfs.mkvol`

步骤: 1. case 目录建 `tree/` (含 `hello.txt`); 2. [`image.create`](../../kpfs/kpfs_image.md); 3. [`disk.mkvol`](../../kpfs/kpfs_disk.md)({ path = …, fs = "exfat", src = tree, label = "LT" }); 4. [`probe.fs`](../../kpfs/kpfs_probe.md) 或 §2.3 [`mount`](../../kpfs/kpfs.md) + §2.4 [`vfs.access`](../../kpfs/kpfs_vfs.md)

预期: `mkvol` 成功; 卷可探测或 VFS 可读

失败: mkvol 失败; 内容不可访问

---
