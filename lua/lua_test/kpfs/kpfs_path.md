# 第 2 章 § 2.5 路径 (VFS path / opendir)

> API: [kpfs.md](../../kpfs/kpfs.md) ([`require("kpfs")`](../../kpfs/kpfs.md), mount), [kpfs_vfs.md](../../kpfs/kpfs_vfs.md) (`kpfs.vfs`, path 级 / dir userdata) | 枢纽: [readme.md](readme.md) | 对照 pfs_test 章 1.3.4 / 1.7 / 1.8

约定见 [readme.md](readme.md). 须先 §2.3 [`kpfs.mount`](../../kpfs/kpfs.md); 盘符路径 `"T1:/…"`. 写操作须 RDWR mount 或 [`remount`](../../kpfs/kpfs.md).

节内共用: 同 §2.4, 预置卷于 `paths.case_dir("2.5.x")`; 测试路径前缀 `/kpfs_lt_*`.

---

## 2.5 路径

### 2.5.1 `mkdir` 三级目录写文件

- `pfs.vfs.path.mkdir`

步骤: 1. [`vfs.mkdir`](../../kpfs/kpfs_vfs.md) 建齐 `T1:/d1/d2/`; 2. §2.4 [`open`](../../kpfs/kpfs_vfs.md)("T1:/d1/d2/kpfs_lt_w.txt", "w") 写入; 3. 读回

预期: 父目录存在时可新建并写入

失败: 父目录缺失却成功; `open` 不递归建目录

---

### 2.5.2 `opendir` 目录枚举

- `pfs.vfs.path.opendir`

步骤: 1. 预置 `T1:/kpfs_lt_dir/` 下至少 1 文件; 2. `d =` [`vfs.opendir`](../../kpfs/kpfs_vfs.md)("T1:/kpfs_lt_dir"); 3. 循环 [`readdir`](../../kpfs/kpfs_vfs.md) 至 `nil`; 4. `d:close()`

预期: 枚举到预置文件名; `ent.type` 为 `"file"`/`"dir"`

失败: 漏项; `readdir` 崩溃

---

### 2.5.3 path 级 `access` / `rename` / `unlink`

- `pfs.vfs.path`

步骤: 1. [`vfs.access`](../../kpfs/kpfs_vfs.md)("T1:/", "f"); 2. [`mkdir`](../../kpfs/kpfs_vfs.md)("T1:/kpfs_lt_x"); 3. [`rename`](../../kpfs/kpfs_vfs.md) 为 `kpfs_lt_y`; 4. 建文件后 [`unlink`](../../kpfs/kpfs_vfs.md); 5. [`rmdir`](../../kpfs/kpfs_vfs.md)

预期: 各步 `rc == 0`

失败: 目录 rename 跨 ctx 返回 PFS_EISDIR; rmdir 非空目录误成功

---
