# 第 2 章 § 2.4 文件 (VFS open/read/write)

> API: [kpfs.md](../../kpfs/kpfs.md) ([`require("kpfs")`](../../kpfs/kpfs.md), mount), [kpfs_vfs.md](../../kpfs/kpfs_vfs.md) (`kpfs.vfs`, file userdata) | 枢纽: [readme.md](readme.md) | 对照 pfs_test 章 1.3–1.6

约定见 [readme.md](readme.md). 须先 §2.3 [`kpfs.mount`](../../kpfs/kpfs.md); 盘符路径 `"T1:/…"`. 写操作须 RDWR mount 或 [`remount`](../../kpfs/kpfs.md).

节内共用: 各条可复用同一 **预置卷** (§2.2 制盘 + §2.3 mount exfat) 于 `paths.case_dir("2.4.x")`; 测试路径前缀 `/kpfs_lt_*` (对齐 pfs_test `/pfs_t_*`).

---

## 2.4 文件

### 2.4.1 写不存在文件 (CREAT)

- `pfs.vfs.open`

前置: [`mount`](../../kpfs/kpfs.md)("T", …, 0x02) RDWR.

步骤: 1. 确保 `T1:/kpfs_lt_new.txt` 不存在; 2. [`vfs.open`](../../kpfs/kpfs_vfs.md)(..., "w") 写已知数据; 3. `close`; 4. 再 `open` 读回

预期: 新建并写入成功; 内容一致

失败: 创建/写入失败; 读回错乱

---

### 2.4.2 读存在文件

- `pfs.vfs.read`

步骤: 1. 预置 `T1:/kpfs_lt_read.txt` 已知内容; 2. [`vfs.open`](../../kpfs/kpfs_vfs.md)(..., "r") 顺序 `read` 至 EOF; 3. `close`

预期: 内容一致; `eof` 在 EOF 后为真

失败: 内容不符; 提前 EOF

---

### 2.4.3 seek / tell / truncate

- `pfs.vfs.seek`

步骤: 1. `f =` [`vfs.open`](../../kpfs/kpfs_vfs.md)("T1:/kpfs_lt_seek.bin", "w+"); 2. 写入固定长度; `tell` 校验; [`seek`](../../kpfs/kpfs_vfs.md)(0,"set"); `read` 比对; 3. [`truncate`](../../kpfs/kpfs_vfs.md)(4) 后读尾; 4. `flush`/`fsync`; `close`

预期: seek/tell/truncate 行为正确; 缩短后旧尾不可读

失败: 错位写; truncate 后崩溃

---
