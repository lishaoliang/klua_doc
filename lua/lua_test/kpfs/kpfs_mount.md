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

### 2.3.5 显式 `"rw"` mount

- `pfs.mount.rw`

步骤: 1. 制盘; 2. [`kpfs.mount`](../../kpfs/kpfs.md)("T", image_path, "rw"); 3. §2.4 [`vfs.open`](../../kpfs/kpfs_vfs.md)("T1:/kpfs_lt_rw.txt", "w") 写一字节; 4. [`umount`](../../kpfs/kpfs.md)("T")

预期: mount `rc == 0`; 写成功

失败: 读写挂载失败; 写被拒绝

---

### 2.3.6 按分区 mode 数组 mount

- `pfs.mount.part_mode`

前置: 固定盘 **N** (MBR×2, `bin/tmp/test_disk/disks/n.vhd`); 须 `test_disk setup --partition` 已写分区表; 用例内 `disk.mkfs` 格式化 `N1`/`N2` exfat (见 `mount_common.prepare_slot_dual_exfat`).

步骤: 1. [`kpfs.mount`](../../kpfs/kpfs.md)("N", image_path, { "r", "rw" }); 2. `N1:` 只读下写失败; `N2:` 写成功; 3. [`umount`](../../kpfs/kpfs.md)("N")

预期: 各分区按 mode 独立生效; mount `rc == 0`

失败: 分区 mode 串扰; 单槽失败导致全盘不可用

---

### 2.3.7 mount `opts` (裸盘 offset / sector_size)

- `pfs.mount.opts`

前置: **无分区表** 裸 exfat 镜像 (`mkfs` 于 offset 0).

步骤: 1. [`kpfs.mount`](../../kpfs/kpfs.md)("T", bare_path, { offset = 0 }); 2. 或 `mount("T", bare_path, "r", { sector_size = 512 })`; 3. [`vfs.access`](../../kpfs/kpfs_vfs.md)("T1:/", "f"); 4. [`umount`](../../kpfs/kpfs.md)("T")

预期: opts 三参/四参形态均可 mount; VFS 可访问

失败: offset/sector_size 无效却 mount 成功; 访问错乱

---

### 2.3.8 remount 单分区 `{ index, mode }`

- `pfs.remount.part`

前置: 固定盘 **N** (MBR×2); 分区表由 setup 写入; 用例内 mkfs 双分区 exfat; 全盘先只读 mount.

步骤: 1. [`kpfs.remount`](../../kpfs/kpfs.md)("N", { index = 2, mode = "rw" }); 2. 仅 `N2:` 可写; `N1:` 仍只读; 3. [`umount`](../../kpfs/kpfs.md)("N")

预期: 单槽 remount `rc == 0`; 未指定分区模式不变

失败: index 错分区; 全盘被切换

---

### 2.3.9 `umount` 单分区 / 批量

- `pfs.umount.part`

前置: 固定盘 **N** (MBR×2); mkfs 后双分区均已 mount.

步骤: 1. [`kpfs.umount`](../../kpfs/kpfs.md)("N", 1); 2. `N1:` 访问失败、`N2:` 仍可访问; 3. [`kpfs.umount`](../../kpfs/kpfs.md)("N", { 2 }); 4. 或整盘 [`umount`](../../kpfs/kpfs.md)("N")

预期: 单分区/批量卸载后对应槽不可访问; 其余槽不受影响直至卸载

失败: 误卸全盘; 卸载后仍可 VFS 访问

---

### 2.3.10 重复 mount 同盘名

- `pfs.mount.eexist`

步骤: 1. 已成功 [`mount`](../../kpfs/kpfs.md)("T", image_a); 2. 再次 [`mount`](../../kpfs/kpfs.md)("T", image_b) (同盘名)

预期: 第二次 `rc ~= 0` (`PFS_EEXIST` 或约定负码); 原挂载仍有效

失败: 二次 mount 覆盖; 盘符状态错乱

---

### 2.3.11 mount 后 `probe.all` 对照

- `pfs.mount.probe_ok`

步骤: 1. 制盘 (mkpt + mkfs); 2. [`kpfs.mount`](../../kpfs/kpfs.md)("T", …); 3. [`kpfs.probe.all`](../../kpfs/kpfs_probe.md)(image_path); 4. 比对 `parts[].mount_ok` 与 mount 结果; 5. [`umount`](../../kpfs/kpfs.md)("T")

预期: 已格式化分区 `mount_ok == true`; `fs_type` 与 mkfs 一致

失败: mount 成功但 probe 显示 `mount_ok == false`

---

### 2.3.12 损坏卷 mount 失败

- `pfs.mount.bad_vol`

步骤: 1. `image.create` 全零或随机数据镜像 (无 FS); 2. [`kpfs.mount`](../../kpfs/kpfs.md)("T", bad_path)

预期: `rc ~= 0`; 盘名未登记或 mount 后 VFS 不可用

失败: 误挂载成功; 访问崩溃

注: 标 **MANUAL** / **SINGLE_ONLY**; 毁写盘头.

---
