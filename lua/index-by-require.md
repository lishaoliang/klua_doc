# 按 require 查文档

> `klua_doc/lua/index-by-require.md` — 脚本侧 API 索引真源. 全量清单含 §5～§6 见 [require-guide.md](../klb/klua/design/require-guide.md).

## ① 标准 Lua 5.4

详 [std/readme.md](std/readme.md). 代码: [klb/src_c/klua/lua-5.4.6/src/linit.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-5.4.6/src/linit.c).

| require / 全局 | 文档 | klb 注意 |
|----------------|------|----------|
| `_G` (base) | [std/base.md](std/base.md) | — |
| `package` | [std/package.md](std/package.md) | klbcore path; k* 在 preload |
| `table` | [std/table.md](std/table.md) | — |
| `io` | [std/io.md](std/io.md) | — |
| `os` | [std/os.md](std/os.md) | 慎用 `os.execute`; env 退出用 `kenv`/`ksys` |
| `string` | [std/string.md](std/string.md) | 模式/JSON 见 bundled |
| `math` | [std/math.md](std/math.md) | 业务随机见 `krand` |
| `utf8` | [std/utf8.md](std/utf8.md) | — |
| `debug` | [std/debug.md](std/debug.md) | 生产慎用 |
| `coroutine` | [std/coroutine.md](std/coroutine.md) | **业务勿用**; 用 **`kco`** → [klua/kco.md](klua/kco.md) |

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

详 [klbcore/readme.md](klbcore/readme.md). 路径根: [klb/bin/klbcore/](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/) (源 [klb/bin/klbcore/](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/)).

| require | 文档 |
|---------|------|
| `klbcore.*` (模块总览) | [klbcore/readme.md](klbcore/readme.md) |
| `klbcore.klbui` | [klbcore/klbui.md](klbcore/klbui.md) |
| klbui 控件/CSS | [klbcore/css/](klbcore/css/) |
| `klbcore.klbrtsp` | [klbcore/klbrtsp.md](klbcore/klbrtsp.md) |
| `klbcore.klbsmp` | [klbcore/klbsmp.md](klbcore/klbsmp.md) |
| `klbcore.net.http_mime` | [klbcore/net/http_mime.md](klbcore/net/http_mime.md) |
| `klbcore.util.*` | [klbcore/util.md](klbcore/util.md) |
| `klbcore.base.pname` | [klbcore/base/pname.md](klbcore/base/pname.md) |
| `klbcore.base.klpcex` | [klbcore/base/klpcex.md](klbcore/base/klpcex.md) |

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
| `kmcache` | [klua/kmcache.md](klua/kmcache.md) |
| `kurl` | [klua/kurl.md](klua/kurl.md) |
| `ksmp` | [klua/ksmp.md](klua/ksmp.md) |
| `krtsp` | [klua/krtsp.md](klua/krtsp.md) |
| `klist` | [klua/klist.md](klua/klist.md) |
| `kkpa` | [klua/kkpa.md](klua/kkpa.md) |
| `kh26x` | [klua/kh26x.md](klua/kh26x.md) |
| `kmnp` | 空库; 见 [klua/readme.md](klua/readme.md) 网络节、**klb-net-design** |
| `khttp_flv`, `khttp_mnp`, `kws_flv`, `kws_mnp` | 同上 |
| `krtp`, `kws_rtp` | 同上 (未进 `klua_loadlib_all`) |

**src_packages** 扩展 (**待定**): 现行 **klbwui**; 见 [require-guide.md](../klb/klua/design/require-guide.md) §3. 旧 `kpa_*` 已归档 backup.

## ⑤ kpfs (plugins)

枢纽 [kpfs/readme.md](kpfs/readme.md). 代码: [portfs/src_klua/](https://gitee.com/klua/portfs/tree/trunk/src_klua/).

| require | 文档 | 说明 |
|---------|------|------|
| `kpfs` | [kpfs/kpfs.md](kpfs/kpfs.md) | mount / remount / umount |
| `kpfs.disk` | [kpfs/kpfs_disk.md](kpfs/kpfs_disk.md) | 块设备 mkpt / mkfs / mkvol |
| `kpfs.probe` | [kpfs/kpfs_probe.md](kpfs/kpfs_probe.md) | 容器/分区/FS 探测 |
| `kpfs.image` | [kpfs/kpfs_image.md](kpfs/kpfs_image.md) | 虚拟磁盘 create / info |
| `kpfs.mtd` | [kpfs/kpfs_mtd.md](kpfs/kpfs_mtd.md) | MTD 格式化与 ubi / raw |
| `kpfs.vfs` | [kpfs/kpfs_vfs.md](kpfs/kpfs_vfs.md) | 盘符路径文件/目录 |
