# 按 require 查文档

> `klua_doc/lua/index-by-require.md` — 脚本侧 API 索引真源. 全量清单含 §5～§6 见 [require-guide.md](../klb/klua/design/require-guide.md).

## ① 标准 Lua 5.4

详 [std/readme.md](std/readme.md). 代码: `klb/src_c/klua/lua-5.4.6/src/linit.c`.

| require | 说明 | klb 注意 |
|---------|------|----------|
| `_G`, `package`, `table`, `io`, `os`, `string`, `math`, `utf8`, `debug` | Lua 5.4 标准库 | — |
| `coroutine` | 内置协程 | **业务勿用**; 用 **`kco`** → [klua/kco.md](klua/kco.md) |

## ② bundled 第三方

详 [bundled/readme.md](bundled/readme.md).

| require | 文档 |
|---------|------|
| `cjson`, `cjson.safe` | [bundled/cjson.md](bundled/cjson.md) |
| `lfs` | [bundled/lfs.md](bundled/lfs.md) |
| `LuaXML_lib` | [bundled/luaxml.md](bundled/luaxml.md) |
| `lpeg` | [bundled/lpeg.md](bundled/lpeg.md) |
| `zlib` | [bundled/zlib.md](bundled/zlib.md) |
| `lsqlite3` | [bundled/lsqlite3.md](bundled/lsqlite3.md) |

## ③ klbcore 纯 Lua

详 [klbcore/readme.md](klbcore/readme.md). 路径根: `klb/bin/klbcore/`.

| require | 文档 |
|---------|------|
| `klbcore.*` (模块总览) | [klbcore/readme.md](klbcore/readme.md) |
| klbui 控件/CSS | [klbcore/css/](klbcore/css/) |

## ④ klua k* (C 预加载)

详 [klua/readme.md](klua/readme.md).

| require | 文档 |
|---------|------|
| `kco` | [klua/kco.md](klua/kco.md) |
| `klpc` | [klua/klpc.md](klua/klpc.md) |
| `kgui` | [klua/kgui.md](klua/kgui.md) |
| `kenv` | [klua/kenv.md](klua/kenv.md) |
| `ksys` | [klua/ksys.md](klua/ksys.md) |
| `krand` | [klua/krand.md](klua/krand.md) |
| `kos` | [klua/kos.md](klua/kos.md) |
| `ktime` | [klua/ktime.md](klua/ktime.md) |
| `kthread` | [klua/kthread.md](klua/kthread.md) |
| `kurl`, `kmnp`, `ksmp`, `krtsp` | 待定; 见 [klua/readme.md](klua/readme.md) 网络节、**klb-net-design** |
| `khttp_flv`, `khttp_mnp`, `kws_flv`, `kws_mnp` | 同上 |
| `krtp`, `kws_rtp` | 同上 (未进 `klua_loadlib_all`) |

`kpa_*` 扩展包 (**待定**, 无独立 Lua 文档): 见 [require-guide.md](../klb/klua/design/require-guide.md) §3.

## ⑤ kpfs (plugins)

枢纽 [kpfs/readme.md](kpfs/readme.md). 代码: `portfs/src_klua/`.

| require | 文档 | 说明 |
|---------|------|------|
| `kpfs` | [kpfs/kpfs.md](kpfs/kpfs.md) | mount / remount / umount |
| `kpfs.disk` | [kpfs/kpfs_disk.md](kpfs/kpfs_disk.md) | 块设备 mkpt / mkfs / mkvol |
| `kpfs.probe` | [kpfs/kpfs_probe.md](kpfs/kpfs_probe.md) | 容器/分区/FS 探测 |
| `kpfs.image` | [kpfs/kpfs_image.md](kpfs/kpfs_image.md) | 虚拟磁盘 create / info |
| `kpfs.mtd` | [kpfs/kpfs_mtd.md](kpfs/kpfs_mtd.md) | MTD 格式化与 ubi / raw |
| `kpfs.vfs` | [kpfs/kpfs_vfs.md](kpfs/kpfs_vfs.md) | 盘符路径文件/目录 |
