# 第 2 章 § 2.4 文件 (VFS open/read/write)

> API: [kpfs.md](../../kpfs/kpfs.md) ([`require("kpfs")`](../../kpfs/kpfs.md), mount), [kpfs_vfs.md](../../kpfs/kpfs_vfs.md) (`kpfs.vfs`, file userdata) | 枢纽: [readme.md](readme.md) | 对照 pfs_test 章 1.3–1.6

约定见 [readme.md](readme.md). 须先 §2.3 [`kpfs.mount`](../../kpfs/kpfs.md); 盘符路径 `"T1:/…"`. 写操作须 RDWR mount 或 [`remount`](../../kpfs/kpfs.md).

节内共用: 各条可复用同一 **预置卷** (§2.2 制盘 + §2.3 mount exfat) 于 `paths.case_dir("2.4.x")`; 测试路径前缀 `/kpfs_lt_*` (对齐 pfs_test `/pfs_t_*`).

---

## 2.4 文件

### 2.4.1 写不存在文件 (CREAT)

- `pfs.vfs.open`

前置: [`mount`](../../kpfs/kpfs.md)("T", …, "rw") 或 [`remount`](../../kpfs/kpfs.md)("T", "rw").

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

### 2.4.4 覆盖写已存在文件

- `pfs.vfs.write`

步骤: 1. 预置 `T1:/kpfs_lt_over.txt` 已知内容; 2. [`vfs.open`](../../kpfs/kpfs_vfs.md)(..., "w") 写新数据; 3. `close`; 4. 再 `open` 读回

预期: 覆盖成功; 内容为最后一次写入

失败: 追加误写; 旧内容残留

---

### 2.4.5 打开不存在路径 (负例)

- `pfs.vfs.open.enoent`

步骤: 1. [`vfs.open`](../../kpfs/kpfs_vfs.md)("T1:/kpfs_lt_nope.txt", "r")

预期: 返回 `rc, msg` (非 userdata); `rc ~= 0`

失败: 误创建文件; 崩溃

---

### 2.4.6 `open` 模式 `r+` / `a` / `a+`

- `pfs.vfs.open.mode`

步骤: 1. `f =` [`vfs.open`](../../kpfs/kpfs_vfs.md)("T1:/kpfs_lt_mode.txt", "r+") 读写; 2. 另测 `"a"` 追加 (写后 `tell` 在末尾); 3. `"a+"` 可读可追加

预期: 各模式打开成功; 追加模式定位在 EOF; 读写语义正确

失败: 模式解析错; 追加覆盖首部

---

### 2.4.7 分块 `read(n)`

- `pfs.vfs.read.chunk`

步骤: 1. 预置 ≥8KiB 已知内容; 2. `f:read(1024)` 循环至 EOF; 3. 拼接比对

预期: 分块读与一次读内容一致; 短读为实际长度

失败: 块边界错乱; 提前 EOF

---

### 2.4.8 空文件 (CREAT 不写即 close)

- `pfs.vfs.empty`

步骤: 1. [`vfs.open`](../../kpfs/kpfs_vfs.md)("T1:/kpfs_lt_empty.txt", "w"); 2. 直接 `close`; 3. [`vfs.access`](../../kpfs/kpfs_vfs.md)(..., "f"); 4. `open` 读长度为 0; 5. [`unlink`](../../kpfs/kpfs_vfs.md)

预期: 0 字节文件存在且可删; 行为稳定

失败: 损坏目录项; 无法删除

---

### 2.4.9 `truncate` 扩展

- `pfs.vfs.truncate.extend`

步骤: 1. 文件长度 S; 2. [`truncate`](../../kpfs/kpfs_vfs.md)(S * 2); 3. `seek` 到扩展区再 `write`; 4. 读回校验

预期: 扩展后可写; 原 S 字节内容不变

失败: 扩展崩溃; 长度与内容不一致

---

### 2.4.10 非法 seek / seek 到 EOF 再读

- `pfs.vfs.seek.bad`

步骤: 1. 对短文件 `seek` 超大负偏移或越界 (以实现为准); 2. `seek(0,"end")` 后 `read` 应得 `""` 且 `eof` 为真

预期: 非法 seek 返回 `rc, msg`; EOF seek+read 不崩溃

失败: 非法 seek 成功写坏; EOF 后仍读出数据

---

### 2.4.11 `f:error()` 流错误查询

- `pfs.vfs.error`

步骤: 1. 正常读写后 `f:error()`; 2. 触发已知错误 (如只读 mount 下写) 后再 `f:error()`

预期: 无错时 `rc == 0`; 有错时 `rc ~= 0` 且 `msg` 非空

失败: 错误 latch 不更新; 误报 OK

---

### 2.4.12 `fsync` 持久化 (umount 周期)

- `pfs.vfs.fsync`

步骤: 1. 写数据后 [`fsync`](../../kpfs/kpfs_vfs.md); 2. [`umount`](../../kpfs/kpfs.md)("T"); 3. 再 [`mount`](../../kpfs/kpfs.md) 读回

预期: `fsync` `rc == 0`; remount 后数据仍在

失败: sync 后数据丢失

注: 标 **UMOUNT_FLOW**; **禁止**进 `a`/`2.1.x`.

---
