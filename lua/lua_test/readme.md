# lua_test 手测

> `klua_doc/lua/lua_test/` — 记名 **lua test** = 本体系; 规则 **klua-test** / **klua-test-design**

## 运行

```bash
cd bin
./klua test.lua list
./klua test.lua 1.1.1
./klua test.lua klbui.custom.chrome
./klua test.lua 2.1.2
./klua test.lua kpfs.probe.all
./klua test.lua 3.1.1
./klua test.lua klb.kco.fork
./klua test.lua 1.2.1
./klua test.lua klbui.require
```

启动时会打印 `[lua_test env]` (os/arch, HOME, cwd, tmp_root).

Windows:

```text
cd d:\work\opendemo\bin
klua.exe test.lua 2.1.2
klua.exe test.lua pfs.probe
```

**双入口** (等价): 章号 `x.y.z` 或语义 id `klb.kco.fork` / `kpfs.probe.all` / `klbui.custom.chrome`; 第 2 章兼容别名 `pfs.*`.

## 批量

```bash
./klua test.lua a          # all registered cases
./klua test.lua 1.x        # chapter 1 klbui (1.1.1 ..)
./klua test.lua 1.1.x      # section 1.1 (UI custom)
./klua test.lua 2.x        # chapter 2 (2.1.1 .. 2.6.20)
./klua test.lua 2.1.x      # section 2.1 only
./klua test.lua 3.x        # chapter 3 klb (3.1.1 ..)
./klua test.lua 3.1.x      # section 3.1 kco
```

| 过滤 | 说明 |
|------|------|
| `a` / `all` | 全部已登记用例 |
| `1.x` | 第 1 章 klbui (现行 1.1.1 .. 1.1.3) |
| `1.1.x` / `1.1` | 节 1.1 (UI custom) |
| `2.x` | 第 2 章全部 |
| `2.1.x` / `2.1` | 节 2.1 |
| `3.x` | 第 3 章 klb |
| `3.1.x` / `3.1` | 节 3.1 kco |

`batch_ok=false` 在 `list` 标 `[SINGLE_ONLY]` (提示重流程); 批量 `a`/`N.x`/`N.M.x` 仍会执行. 详 **klua-test-design** § 批量.

参数经 `ksys.get_args()` 传递: `[1]` klua 路径, `[2]` test.lua, `[3]` 用例 id 或批量过滤, `[4]…` 用例参数.

## 章索引

| 章 | 子项目 | 文档 | 源码 |
|----|--------|------|------|
| **1** | klbui | [klbui/readme.md](klbui/readme.md) | `klua_run/lua_test/`klbui/` |
| **2** | kpfs | 见下表 | `klua_run/lua_test/`pfs/` |
| **3** | klb k* | [klb/readme.md](klb/readme.md) | `klua_run/lua_test/`klb/` |

章号 **冻结**: 1=klbui / 2=kpfs / 3=klb; 新子项目从 **4** 追加; 章内只追加节, 禁止插入.

### 第 1 章 klbui

枢纽 [klbui/readme.md](klbui/readme.md) (脚本 UI `klbcore.klbui` + `kgui`; **1.1** 可运行自定义内容; 1.2+ 条文 **待实现**).

| 节 | 手测文档 | 说明 |
|----|----------|------|
| 1.1 | [klbui/klbui_custom.md](klbui/klbui_custom.md) | 自定义内容 (UI) |
| 1.2 | [klbui/klbui_parse.md](klbui/klbui_parse.md) | 解析 parse |
| 1.3 | [klbui/klbui_select.md](klbui/klbui_select.md) | 选择器 select |
| 1.4 | [klbui/klbui_css.md](klbui/klbui_css.md) | CSS |
| 1.5 | [klbui/klbui_kview.md](klbui/klbui_kview.md) | kview |

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

### 第 3 章 klb

枢纽 [klb/readme.md](klb/readme.md) (k* C→Lua; 现行 `3.1.1` `klb.kco.fork`).

### 已实现用例

| doc_id | 语义 id (主) | 别名 | 文档 |
|--------|--------------|------|------|
| `1.1.1` | `klbui.custom.chrome` | | [klbui/klbui_custom.md](klbui/klbui_custom.md) |
| `1.1.2` | `klbui.custom.widgets` | | [klbui/klbui_custom.md](klbui/klbui_custom.md) |
| `1.1.3` | `klbui.custom.types` | | [klbui/klbui_custom.md](klbui/klbui_custom.md) |
| `2.1.1` | `kpfs.version` | `pfs.version` | [kpfs/kpfs_probe.md](kpfs/kpfs_probe.md) |
| `2.1.2` | `kpfs.probe.all` | `pfs.probe.all`, `pfs.probe` | [kpfs/kpfs_probe.md](kpfs/kpfs_probe.md) |
| `2.1.3` | `kpfs.probe.part` | `pfs.probe.part` | [kpfs/kpfs_probe.md](kpfs/kpfs_probe.md) |
| `2.1.4` | `kpfs.probe.fs` | `pfs.probe.fs` | [kpfs/kpfs_probe.md](kpfs/kpfs_probe.md) |
| `2.2.1` | `kpfs.image` | `pfs.image` | [kpfs/kpfs_make.md](kpfs/kpfs_make.md) |
| `2.2.2` | `kpfs.image.vhd` | `pfs.image.vhd` | [kpfs/kpfs_make.md](kpfs/kpfs_make.md) |
| `2.2.3` | `kpfs.image.fixed` | `pfs.image.fixed` | [kpfs/kpfs_make.md](kpfs/kpfs_make.md) |
| `2.2.4` | `kpfs.image.vhd_fixed` | `pfs.image.vhd_fixed` | [kpfs/kpfs_make.md](kpfs/kpfs_make.md) |
| `2.2.5` | `kpfs.mkpt` | `pfs.mkpt` | [kpfs/kpfs_make.md](kpfs/kpfs_make.md) |
| `2.2.6` | `kpfs.mkfs` | `pfs.mkfs` | [kpfs/kpfs_make.md](kpfs/kpfs_make.md) |
| `2.2.7` | `kpfs.mkvol` | `pfs.mkvol` | [kpfs/kpfs_make.md](kpfs/kpfs_make.md) |
| `3.1.1` | `klb.kco.fork` | `klb.kco_fork` | [klb/readme.md](klb/readme.md) |

kpfs 全文索引与规划条数见 [kpfs/readme.md](kpfs/readme.md).

新增用例: 先写文档 `### x.y.z`, 再 `registry.lua` + 源码; 见 **klua-test-design**.

## 目录

| 路径 | 说明 |
|------|------|
| `klua_run/test.lua` | **唯一入口** |
| `klua_run/lua_test/`` | 框架 (`registry`, `batch`, `paths`, `util/`) |
| `klua_run/lua_test/`klbui/` | **第 1 章** klbui 用例 (1.1 custom 已实现; 1.2+ 待实现) |
| `klua_run/lua_test/`pfs/` | **第 2 章** kpfs 用例 |
| `klua_run/lua_test/`klb/` | **第 3 章** klb k* 用例 |
| `klua_run/tmp/` | Windows 运行时临时根 (`kenv.base_path() .. "tmp"`) |

### 用例源码布局（章 = 子项目目录）

| 章 | 根目录 | 命名 |
|----|--------|------|
| **1** | `klbui/` | 节子目录 `custom` `parse` `select` `css` `kview`；用例 `ch1_s{M}_{z}.lua`；`common.lua` / `ui.lua` / `pref.lua`；1.1 模板 `custom/page_tmpl.lua` |
| **2** | `pfs/` | 节子目录 `probe`…`mtd`；用例 `ch2_s{M}_{z}.lua`；公共 `make/mount/mtd/common.lua`；`test_disk.lua` |
| **3** | `klb/` | 推荐 `ch3_s{M}_{z}.lua`（`3.M.z`）；现行 `kco_fork.lua` = `3.1.1` |

示例: `2.3.8` → `pfs/mount/ch2_s3_8.lua` → `mod` `lua_test.pfs.mount.ch2_s3_8`. 详 **klua-test-design** § 用例目录.

## 临时目录

| 宿主           | 路径                | 说明                             |
| ------------ | ----------------- | ------------------------------ |
| Windows      | `<base_path>tmp/` | `kenv.base_path()` = klua 所在目录 |
| Linux / WSL  | `~/tmp/`          | `HOME/tmp`                     |
| arm64 (QEMU) | `/tmp/`           | 分区约 **256MiB**; 单镜像建议 ≤32MiB   |

环境变量 **`LUA_TEST_TMP`** 可覆盖上述规则. arm64 检测: `LUA_TEST_PROFILE=arm64` 或 `HOME=/tmp/app`.

用例工作目录: `<tmp_root>/lua_test/<doc_id>/` (`paths.case_dir`, 如 `2_1_1`).

## 部署

Linux `build` stage/deploy 与 `copy_win.sh` 须同步 **`klua_run/lua_test/``** 树 (与 `klbcore/` 同级). `test.lua` 由 `copy_bin_lua_files` 复制.

## 与其它测试边界

| 项 | 路径 | 区别 |
|----|------|------|
| k* 桩烟测 | `klbcore/help/k_test/` | 历史; 新用例优先 **lua_test** |
| pfs C 回归 | `pfs_test` | C 控制台, 章号 **独立**, 非 Lua |
| UI 演示 | `klua_run/sample/` | 产品 demo; **非** 第 1 章 klbui 手测 |
