# 第 2 章 § 2.1 探测 (probe)

> API: [kpfs.md](../../kpfs/kpfs.md) ([`require("kpfs")`](../../kpfs/kpfs.md)), [kpfs_probe.md](../../kpfs/kpfs_probe.md) (`kpfs.probe`) | 枢纽: [readme.md](readme.md) | 对照 pfs CLI `probe`

约定见 [readme.md](readme.md). 探测只读, **不** [`kpfs.mount`](../../kpfs/kpfs.md). 固定返回 `info, rc, msg` (`info` 恒为表; `rc~=0` 时为 `{}`).

---

## 2.1 探测

### 2.1.1 模块版本

- `pfs.version`

步骤: 1. [`require("kpfs")`](../../kpfs/kpfs.md) (失败 skip); 2. [`kpfs.version()`](../../kpfs/kpfs.md)

预期: 返回非空字符串 (现行 `"0.1"`)

失败: require 失败未 skip; 返回空

---

### 2.1.2 空镜像 `probe.all`

- `pfs.probe.all`

步骤: 1. [`require("kpfs")`](../../kpfs/kpfs.md) (失败 skip); 2. `paths.case_dir("2.1.2")` 下 `image.create` dynamic vhdx (Win 4MiB / arm64 2MiB); 3. [`probe.all`](../../kpfs/kpfs_probe.md)(image_path)

预期: 打印 `container`, `parts` 数量; 输出 `lua_test.pfs.probe PASS`

失败: create 失败; `probe.all` 的 `rc ~= 0` 或 `info` 为空表 `{}`

注: 本例 **仅** create 空镜像, 不测分区表/FS.

---

### 2.1.3 分区表 `probe.part`

- `pfs.probe.part`

步骤: 1. `image.create`; 2. [`disk.mkpt`](../../kpfs/kpfs_disk.md)({ scheme = "gpt", fs = "exfat" }); 3. [`probe.part`](../../kpfs/kpfs_probe.md)(image_path)

预期: `scheme == "gpt"`, `part_count >= 1`; `parts[]` **无** `fs_type` / `mount_ok` / `mount_err`

失败: part 探测失败; 误含 FS 字段

注: 含 `mkpt`, 单跑时毁写盘头; 批量 `2.1.x` 须各 case 独立镜像路径.

---

### 2.1.4 首分区 FS `probe.fs`

- `pfs.probe.fs`

步骤: 1. `image.create` → `mkpt` → `mkfs` (exfat, `mkfs` 须 `force`); 2. [`probe.fs`](../../kpfs/kpfs_probe.md)(image_path)

预期: 首分区或 `bare_fs` 的 `mount_ok == true`

失败: 已格式化卷探测 mount 失败

注: 有分区表时 **仅** 试探首分区; 裸盘读 `bare_fs`.

---
