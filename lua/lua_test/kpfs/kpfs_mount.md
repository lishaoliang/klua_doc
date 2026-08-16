# 第 2 章 § 2.3 盘符 mount

> API: [kpfs.md](../../kpfs/kpfs.md) ([`require("kpfs")`](../../kpfs/kpfs.md), mount/umount/remount), [kpfs_probe.md](../../kpfs/kpfs_probe.md) (`kpfs.probe`), [kpfs_vfs.md](../../kpfs/kpfs_vfs.md) (`kpfs.vfs`) | 枢纽: [readme.md](readme.md) | 对照 pfs_test 章 1.1

约定见 [readme.md](readme.md). 须先 §2.2 制盘 (image + mkpt + mkfs exfat). [`mount`](../../kpfs/kpfs.md) 后盘符路径 `"T1:/…"`; VFS 见 §2.4 / §2.5.

---

## 2.3 盘符 mount

### 2.3.1 盘符 mount / umount

- `pfs.mount`

前置: 本 case 工作目录内 `image.create` → `mkpt` (force) → `mkfs` (exfat, force).

步骤: 1. [`kpfs.mount`](../../kpfs/kpfs.md)("T", image_path) 默认只读; 2. 校验 `rc == 0`; 3. 可选 [`kpfs.probe.all`](../../kpfs/kpfs_probe.md) 对照 `parts[].mount_ok`; 4. [`kpfs.umount`](../../kpfs/kpfs.md)("T")

预期: mount/umount 均 `rc == 0`; 输出 PASS

失败: mount 失败; umount 后仍占用盘名

---

### 2.3.2 `remount` 只读↔读写

- `pfs.remount`

步骤: 1. 制盘后 [`kpfs.mount`](../../kpfs/kpfs.md)("T", …) 只读; 2. [`kpfs.remount`](../../kpfs/kpfs.md)("T", "rw"); 3. 可选 §2.4 [`vfs.open`](../../kpfs/kpfs_vfs.md)(..., "w") 写一字节; 4. [`kpfs.remount`](../../kpfs/kpfs.md)("T", "r"); 5. [`kpfs.umount`](../../kpfs/kpfs.md)("T")

预期: remount 往返 `rc == 0`; 只读阶段写失败 (若测)

失败: remount 无效; 只读下仍可写

---

### 2.3.3 只读挂载下写

- `pfs.vfs.ro_write`

步骤: 1. [`mount`](../../kpfs/kpfs.md)("T", …) 默认只读; 2. [`vfs.open`](../../kpfs/kpfs_vfs.md)(..., "w") 或 `"w+"`

预期: 打开失败 (`rc, msg`) 或写返回错误; 卷未被改写

失败: 只读卷被改写

---

### 2.3.4 显式 `umount` 流程

- `pfs.vfs.umount_flow`

步骤: 1. RDWR mount; 2. [`open`](../../kpfs/kpfs_vfs.md) 读文件不 `close`; 3. [`umount`](../../kpfs/kpfs.md)("T") (预期 busy 或约定行为); 4. 另测: `close` 后 umount 成功; 5. 再 mount 验证文件仍可读

预期: 有未关闭句柄时 umount 失败或约定行为; 无句柄时成功; 不崩溃

失败: umount 后旧句柄仍可写; 卷损坏

注: 标 UMOUNT_FLOW; **禁止**进 `a`/`2.1.x`.

---
