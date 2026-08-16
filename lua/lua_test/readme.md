# lua_test 手测

> `klua_doc/lua/lua_test/` — 记名 **lua test** = 本体系; 规则 **klua-test** / **klua-test-design**

## 运行

```bash
cd bin
./klua test.lua list
./klua test.lua 1.1.1
./klua test.lua klb.kco.fork
./klua test.lua 2.1.2
./klua test.lua kpfs.probe.all
```

启动时会打印 `[lua_test env]` (os/arch, HOME, cwd, tmp_root).

Windows:

```text
cd d:\work\opendemo\bin
klua.exe test.lua 2.1.2
klua.exe test.lua pfs.probe
```

**双入口** (等价): 章号 `x.y.z` 或语义 id `klb.kco.fork` / `kpfs.probe.all`; 第 2 章兼容别名 `pfs.*`.

## 批量

```bash
./klua test.lua a          # 全部 batch_ok
./klua test.lua 1.x        # 第 1 章
./klua test.lua 1.1.x      # 节 1.1
./klua test.lua 2.1.x      # 节 2.1 (章 2 禁 2.x, 节 2.1 允许)
```

| 过滤 | 说明 |
|------|------|
| `a` / `all` | 所有 `batch_ok` 用例 |
| `1.x` | 第 1 章可批量 |
| `1.1.x` / `1.1` | 节内可批量 |
| `2.x` | **跳过** (第 2 章整章禁批量) |
| `2.1.x` | 节 2.1 可批量 |

`batch_ok=false` (**SINGLE_ONLY**) 仅精确 id 单跑; 批量时打印 `[SKIP]`. 详 **klua-test-design** § 批量.

参数经 `ksys.get_args()` 传递: `[1]` klua 路径, `[2]` test.lua, `[3]` 用例 id 或批量过滤, `[4]…` 用例参数.

## 章索引

| 章 | 子项目 | 文档 | 源码 |
|----|--------|------|------|
| **1** | klb k* | [klb/readme.md](klb/readme.md) | `bin/lua_test/klb/` |
| **2** | kpfs | 见下表 | `bin/lua_test/pfs/` |

### 第 2 章 kpfs

枢纽 [kpfs/readme.md](kpfs/readme.md) (对照 [pfs_test.md](../../pfs/pfs_test.md), 批量与章号独立).

| 节 | 手测文档 | 说明 |
|----|----------|------|
| 2.1 | [kpfs/kpfs_probe.md](kpfs/kpfs_probe.md) | 探测 probe |
| 2.2 | [kpfs/kpfs_make.md](kpfs/kpfs_make.md) | 制盘 (image / mkpt / mkfs / mkvol) |
| 2.3 | [kpfs/kpfs_mount.md](kpfs/kpfs_mount.md) | 盘符 mount / umount / remount |
| 2.4 | [kpfs/kpfs_file.md](kpfs/kpfs_file.md) | 文件 open/read/write/seek |
| 2.5 | [kpfs/kpfs_path.md](kpfs/kpfs_path.md) | 路径 mkdir/opendir/access |
| 2.6 | [kpfs/kpfs_mtd.md](kpfs/kpfs_mtd.md) | MTD (对照 pfs_test 章 5) |

### 已实现用例

| doc_id | 语义 id (主) | 别名 | 文档 |
|--------|--------------|------|------|
| `1.1.1` | `klb.kco.fork` | `klb.kco_fork` | [klb/readme.md](klb/readme.md) |
| `2.1.2` | `kpfs.probe.all` | `pfs.probe.all`, `pfs.probe` | [kpfs/kpfs_probe.md](kpfs/kpfs_probe.md) |

kpfs 全文索引与规划条数见 [kpfs/readme.md](kpfs/readme.md).

新增用例: 先写文档 `### x.y.z`, 再 `registry.lua` + 源码; 见 **klua-test-design**.

## 目录

| 路径 | 说明 |
|------|------|
| `bin/test.lua` | **唯一入口** |
| `bin/lua_test/` | 用例实现 (`registry`, `batch`, `paths`, `klb/`, `pfs/`) |
| `bin/tmp/` | Windows 运行时临时根 (`kenv.base_path() .. "tmp"`) |

## 临时目录

| 宿主           | 路径                | 说明                             |
| ------------ | ----------------- | ------------------------------ |
| Windows      | `<base_path>tmp/` | `kenv.base_path()` = klua 所在目录 |
| Linux / WSL  | `~/tmp/`          | `HOME/tmp`                     |
| arm64 (QEMU) | `/tmp/`           | 分区约 **256MiB**; 单镜像建议 ≤32MiB   |

环境变量 **`LUA_TEST_TMP`** 可覆盖上述规则. arm64 检测: `LUA_TEST_PROFILE=arm64` 或 `HOME=/tmp/app`.

用例工作目录: `<tmp_root>/lua_test/<doc_id>/` (`paths.case_dir`, 如 `2_1_1`).

## 部署

Linux `build` stage/deploy 与 `copy_win.sh` 须同步 **`bin/lua_test/`** 树 (与 `klbcore/` 同级). `test.lua` 由 `copy_bin_lua_files` 复制.

## 与其它测试边界

| 项 | 路径 | 区别 |
|----|------|------|
| k* 桩烟测 | `klbcore/help/k_test/` | 历史; 新用例优先 **lua_test** |
| pfs C 回归 | `pfs_test` | C 控制台, 章号 **独立**, 非 Lua |
| UI 演示 | `bin/sample/` | 产品 demo |
