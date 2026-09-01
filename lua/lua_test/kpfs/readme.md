# 第 2 章 kpfs (Lua 手测)

> `klua_doc/lua/lua_test/kpfs/` | API: [kpfs](../../kpfs/readme.md) | 源码: `klua_run/lua_test/`pfs/`
> 约定 **klua-test-design**; C 回归对照 [pfs_test.md](../../../pfs/pfs_test.md) (**pfs-test-design**)

章号 **2.x** 与 `pfs_test` 章 1–5 **编号空间独立**; 下文「对照」仅指能力/步骤语义, 非 doc_id 共用.

**组织原则**: 按 **手测流程** 分节 (probe → 制盘 → mount → 文件 → 路径), **非** 照搬 API 子模块目录.

**源码目录** (`klua_run/lua_test/`pfs/`): 节子目录 `probe` `make` `mount` `file` `path` `mtd`; 用例 `ch2_s{M}_{z}.lua`（`2.M.z`）; 公共 `make/mount/mtd/common.lua`; 章共享 `test_disk.lua`. 详 **klua-test-design** § 用例目录.

---

## 运行

```bash
cd bin
./klua test.lua 2.1.2
./klua test.lua kpfs.probe.all
./klua test.lua 2.1.x      # 节 2.1 可批量 (章 2 禁 2.x)
```

Windows: `klua.exe test.lua 2.1.2`. 双入口: doc_id 或语义 id (`kpfs.*`; 兼容 `pfs.*`).

## 批量 (第 2 章)

| 命令 | 允许 | 省略 |
|------|------|------|
| `a` / `all` | 全章 `batch_ok` 条 | `batch_ok=false` (**SINGLE_ONLY**) |
| `2.x` | — | **整章禁批量** |
| `2.1.x` | 节 2.1 可批量条 | 破坏性 / mount 周期条 |

对齐 **pfs-test-design**: 破坏性写 / 显式 `umount` 周期 → 仅精确 doc_id; **禁止**假设进 `2.x` 章批量.

## 流程与 API / pfs_test 对照

| 本节 | 手测文档 | 阶段 | 主要 API | 对照 |
|------|----------|------|----------|------|
| 2.1 | [kpfs_probe.md](kpfs_probe.md) | 探测 | [`kpfs.probe`](../../kpfs/kpfs_probe.md) | CLI `probe` |
| 2.2 | [kpfs_make.md](kpfs_make.md) | 制盘 | [`kpfs.image`](../../kpfs/kpfs_image.md), [`mkpt`](../../kpfs/kpfs.md)/[`mkfs`](../../kpfs/kpfs.md)/[`mkvol`](../../kpfs/kpfs.md) | pfs_test 章 3 / 4.4; CLI `image`/`mkpt`/`mkfs`/`mkvol` |
| 2.3 | [kpfs_mount.md](kpfs_mount.md) | 盘符 mount | [`kpfs.mount`](../../kpfs/kpfs.md)/[`umount`](../../kpfs/kpfs.md)/[`remount`](../../kpfs/kpfs.md) | pfs_test 章 1.1; Lua 独有能力 |
| 2.4 | [kpfs_file.md](kpfs_file.md) | 文件 IO | [`kpfs.vfs`](../../kpfs/kpfs_vfs.md) (file) | pfs_test 章 1.3–1.6 |
| 2.5 | [kpfs_path.md](kpfs_path.md) | 路径 path / opendir | [`kpfs.vfs`](../../kpfs/kpfs_vfs.md) (path 级 / dir) | pfs_test 章 1.7–1.8 |
| 2.6 | [kpfs_mtd.md](kpfs_mtd.md) | MTD 制盘 | [`kpfs.mtd`](../../kpfs/kpfs_mtd.md) | pfs_test 章 5 |

## 公共约定

| 项 | 约定 |
|----|------|
| 插件 | [`require("kpfs")`](../../kpfs/kpfs.md) 失败 → skip (未部署 libkpfs) |
| 工作目录 | `paths.case_dir(doc_id)` → `<tmp_root>/lua_test/<doc_id>/` |
| 镜像尺寸 | ≤ `paths.max_image_bytes()`; arm64 建议 ≤32MiB |
| 破坏性写 | `force = true`; 标 **SINGLE_ONLY** |
| 非格式化用例 | **禁止** [`mkpt`](../../kpfs/kpfs.md)/[`mkfs`](../../kpfs/kpfs.md)/[`mkvol`](../../kpfs/kpfs.md)/[`mtd.mkfs`](../../kpfs/kpfs_mtd.md) (主题即格式化除外; 同 **pfs-test-design**) |
| 盘符路径 | `mount` 后 `"T1:/path"`; 须先 §2.3 [`kpfs.mount`](../../kpfs/kpfs.md) 再 §2.4/§2.5 [`kpfs.vfs`](../../kpfs/kpfs_vfs.md) |
| 固定盘 M/N/P | `klua_run/tmp/test_disk/disks/`; `lua_test.pfs.test_disk` (`SLOT_LAYOUT`: M GPT×4, N MBR×2, P GPT×2); mount 盘名 **M**/**N**/**P** |
| MTD FS | **禁止** 走 [`kpfs.disk.mkfs`](../../kpfs/kpfs_disk.md); 须 [`kpfs.mtd.*`](../../kpfs/kpfs_mtd.md) (§2.6) |

## 节索引与用例

| 节 | 文档 | 条数 | 已实现 |
|----|------|------|--------|
| 2.1 | [kpfs_probe.md](kpfs_probe.md) | 4 | `2.1.1`–`2.1.4` |
| 2.2 | [kpfs_make.md](kpfs_make.md) | 7 | `2.2.1`–`2.2.7` |
| 2.3 | [kpfs_mount.md](kpfs_mount.md) | 12 | `2.3.1`–`2.3.12` |
| 2.4 | [kpfs_file.md](kpfs_file.md) | 12 | `2.4.1`–`2.4.12` |
| 2.5 | [kpfs_path.md](kpfs_path.md) | 12 | `2.5.1`–`2.5.12` |
| 2.6 | [kpfs_mtd.md](kpfs_mtd.md) | 20 | `2.6.1`–`2.6.20` |

新增: 先写本节 `### x.y.z` 条文 → `klua_run/lua_test/`pfs/` → `registry.lua` → 更新本页 § 已实现.

### 已实现

| doc_id | 语义 id (主) | 别名 | 文档 |
|--------|--------------|------|------|
| `2.1.1` | [`kpfs.version`](../../kpfs/kpfs.md) | `pfs.version` | [kpfs_probe.md](kpfs_probe.md) |
| `2.1.2` | [`kpfs.probe.all`](../../kpfs/kpfs_probe.md) | `pfs.probe.all`, `pfs.probe` | [kpfs_probe.md](kpfs_probe.md) |
| `2.1.3` | [`kpfs.probe.part`](../../kpfs/kpfs_probe.md) | `pfs.probe.part` | [kpfs_probe.md](kpfs_probe.md) |
| `2.1.4` | [`kpfs.probe.fs`](../../kpfs/kpfs_probe.md) | `pfs.probe.fs` | [kpfs_probe.md](kpfs_probe.md) |
| `2.2.1` | [`kpfs.image`](../../kpfs/kpfs_image.md) | `pfs.image` | [kpfs_make.md](kpfs_make.md) |
| `2.2.2` | [`kpfs.image.vhd`](../../kpfs/kpfs_image.md) | `pfs.image.vhd` | [kpfs_make.md](kpfs_make.md) |
| `2.2.3` | [`kpfs.image.fixed`](../../kpfs/kpfs_image.md) | `pfs.image.fixed` | [kpfs_make.md](kpfs_make.md) |
| `2.2.4` | [`kpfs.image.vhd_fixed`](../../kpfs/kpfs_image.md) | `pfs.image.vhd_fixed` | [kpfs_make.md](kpfs_make.md) |
| `2.2.5` | [`kpfs.mkpt`](../../kpfs/kpfs_disk.md) | `pfs.mkpt` | [kpfs_make.md](kpfs_make.md) |
| `2.2.6` | [`kpfs.mkfs`](../../kpfs/kpfs_disk.md) | `pfs.mkfs` | [kpfs_make.md](kpfs_make.md) |
| `2.2.7` | [`kpfs.mkvol`](../../kpfs/kpfs_disk.md) | `pfs.mkvol` | [kpfs_make.md](kpfs_make.md) |
