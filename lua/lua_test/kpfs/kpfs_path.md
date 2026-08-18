# 第 2 章 § 2.5 路径 (VFS path / opendir)

> API: [kpfs.md](../../kpfs/kpfs.md) ([`require("kpfs")`](../../kpfs/kpfs.md), mount), [kpfs_vfs.md](../../kpfs/kpfs_vfs.md) (`kpfs.vfs`, path 级 / dir userdata) | 枢纽: [readme.md](readme.md) | 对照 pfs_test 章 1.7–1.8 / 1.10

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

### 2.5.4 `copy` 卷内单文件

- `pfs.vfs.copy`

步骤: 1. 预置 `T1:/kpfs_lt_src.txt`; 2. [`vfs.copy`](../../kpfs/kpfs_vfs.md)("T1:/kpfs_lt_src.txt", "T1:/kpfs_lt_dst.txt"); 3. 读 dst 比对; 4. 源文件仍存在

预期: `rc == 0`; 内容一致; 源未删除

失败: copy 失败; 源被删; 内容不一致

---

### 2.5.5 `copy` 宿主↔卷 (`@` 路径)

- `pfs.vfs.copy.host`

步骤: 1. case 目录建宿主 `local.bin`; 2. [`vfs.copy`](../../kpfs/kpfs_vfs.md)("@./local.bin", "T1:/kpfs_lt_host.bin"); 3. 反向 `copy`("T1:/kpfs_lt_host.bin", "@./local_out.bin"); 4. 比对字节

预期: 宿主→卷、卷→宿主均 `rc == 0`; 内容一致

失败: `@` 路径解析错; 写卷侧只读 mount 未拒绝

---

### 2.5.6 `copy` 跨分区

- `pfs.vfs.copy.cross`

前置: 固定盘 **N** (MBR×2); setup 分区表 + 用例内 mkfs 双分区 exfat; mount `N` RW.

步骤: 1. `N1:` 预置源文件; 2. [`vfs.copy`](../../kpfs/kpfs_vfs.md)("N1:/kpfs_lt_a.txt", "N2:/kpfs_lt_b.txt"); 3. 读 `N2:` 比对

预期: 跨分区 copy `rc == 0`; 源保留

失败: copy 失败; 目标分区无文件

---

### 2.5.7 `remove` (文件 / 空目录)

- `pfs.vfs.remove`

步骤: 1. 建文件后 [`vfs.remove`](../../kpfs/kpfs_vfs.md); 2. 再 `access` 应失败; 3. 建空目录后 `remove`; 4. `access` 应失败

预期: 文件与空目录均可 `remove`; 删除后不可访问

失败: 与 `unlink`/`rmdir` 语义不一致; 残留可访问

---

### 2.5.8 path 负例 (mkdir / rmdir / rename / unlink)

- `pfs.vfs.path.neg`

步骤: 1. 重复 [`mkdir`](../../kpfs/kpfs_vfs.md) 同路径 → `PFS_EEXIST`; 2. 缺父目录 mkdir → `PFS_ENOENT`; 3. 非空目录 [`rmdir`](../../kpfs/kpfs_vfs.md) → `PFS_ENOTEMPTY`; 4. 对目录 [`unlink`](../../kpfs/kpfs_vfs.md) 失败; 5. rename 到已存在目标 → `PFS_EEXIST` (或约定行为)

预期: 各负例 `rc ~= 0`; 卷树不被破坏

失败: 负例误成功; 双路径并存

---

### 2.5.9 跨分区 `rename`

- `pfs.vfs.rename.cross`

前置: 固定盘 **N** (MBR×2); mkfs 后双分区 mount RW.

步骤: 1. 普通文件 `rename`("N1:/kpfs_lt_f.txt", "N2:/kpfs_lt_f.txt") 成功; 2. 目录跨分区 `rename` → `PFS_EISDIR`

预期: 文件可跨分区移动 (copy+unlink 源); 目录跨分区拒绝

失败: 目录被移走; 文件 rename 后双份存在

---

### 2.5.10 `opendir` 负例与 `direrror`

- `pfs.vfs.opendir.neg`

步骤: 1. 对不存在路径、普通文件路径 [`opendir`](../../kpfs/kpfs_vfs.md) 失败; 2. 合法目录枚举后 [`direrror`](../../kpfs/kpfs_vfs.md) 应为 `rc == 0`

预期: 负例返回 `rc, msg`; 正常枚举后 `direrror` 无错

失败: 负例误成功; EOF 后 `direrror < 0`

---

### 2.5.11 根目录与子目录枚举

- `pfs.vfs.opendir.full`

步骤: 1. [`opendir`](../../kpfs/kpfs_vfs.md)("T1:/") 枚举 (含 `.`/`..` 若 FS 暴露); 2. 预置子目录 ≥3 文件 + ≥2 子目录; 3. `opendir` 子目录枚举齐全; 4. `ent.type` 与 [`access`](../../kpfs/kpfs_vfs.md) 交叉

预期: 根与子目录项齐全; 类型正确

失败: 漏项; `type` 与实体不符

---

### 2.5.12 UTF-8 / 长度边界文件名

- `pfs.vfs.name.utf8`

步骤: 1. 建含非 ASCII 名文件 (如 `kpfs_lt_测试.txt`); 2. `opendir` 枚举可见; 3. `open` 读写信; 4. 可选测接近 FS 最大名长 (exFAT 255 UTF-16 单元)

预期: UTF-8 名可建可读; 超长名被拒绝 (`PFS_ENAMETOOLONG` 或约定)

失败: 乱码名; 超长名截断致碰撞

---
