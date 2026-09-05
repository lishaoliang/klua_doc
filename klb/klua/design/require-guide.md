# require 库清单

> `klua_doc/klb/klua/design/require-guide.md` — 代码: [klb/src_c/klua/klua.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua.c) (`klua_loadlib_all`)

对话问「Lua 有哪些库」→ **本节 §0～§4** 为 klb 默认范畴; §5～§6 视产品而定.  
**按 require 查文档**: [lua/index-by-require.md](../../../lua/index-by-require.md).  
§0/§1 详文: [lua/std/readme.md](../../../lua/std/readme.md), [lua/bundled/readme.md](../../../lua/bundled/readme.md).

## §0 Lua 5.4 标准库

> 详 [lua/std/readme.md](../../../lua/std/readme.md). 代码: [klb/src_c/klua/lua-5.4.6/src/linit.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-5.4.6/src/linit.c) (`luaL_openlibs`)

| 全局名 | 说明 | klb 注意 |
|--------|------|----------|
| `_G` | base (print, type, error…) | — |
| `package` | require, path, loaded | klbcore 依赖 path 配置 |
| `coroutine` | Lua 内置协程 | **业务勿用**; 用 **`kco`** |
| `table` | 表操作 | — |
| `io` | 文件/标准 IO | — |
| `os` | 时间, 环境, execute 等 | 嵌入式慎用 `os.execute` |
| `string` | 字符串 | — |
| `math` | 数学 | — |
| `utf8` | UTF-8 | — |
| `debug` | 调试 | — |

## §1 bundled 第三方 (C 预加载)

> 详 [lua/bundled/readme.md](../../../lua/bundled/readme.md).

| require | 文档 |
|---------|------|
| `cjson`, `cjson.safe` | [cjson](../../../lua/bundled/cjson.md) |
| `lfs` | [lfs](../../../lua/bundled/lfs.md) |
| `LuaXML_lib` | [luaxml](../../../lua/bundled/luaxml.md) |
| `lpeg` | [lpeg](../../../lua/bundled/lpeg.md) |
| `zlib` | [zlib](../../../lua/bundled/zlib.md) |
| `lsqlite3` | [lsqlite3](../../../lua/bundled/lsqlite3.md) |

## §2 klb 自有 k* (C 预加载, 默认全注册)

| 分类 | require | 源码目录 | API 文档 |
|------|---------|----------|----------|
| 协程/IPC | `kco`, `klpc` | `klua_base/` | [kco](../../../lua/klua/kco.md), [klpc](../../../lua/klua/klpc.md) |
| GUI/包 | `kgui`, `kkpa` | `klua_base/` | [kgui](../../../lua/klua/kgui.md); kkpa 待定 |
| 环境/系统 | `kenv`, `ksys`, `krand`, `kos`, `ktime` | `klua_util/`, `klua_platform/` | [kenv](../../../lua/klua/kenv.md), [ksys](../../../lua/klua/ksys.md), [krand](../../../lua/klua/krand.md), [kos](../../../lua/klua/kos.md), [ktime](../../../lua/klua/ktime.md) |
| 多线程/容器 | `kthread`, `klist`, `kmcache` | `klua_multithread/` | [kthread](../../../lua/klua/kthread.md), [kmcache](../../../lua/klua/kmcache.md) |
| 网络 | `kurl`, `kmnp`, `ksmp`, `krtsp`, `khttp_flv`, `khttp_mnp`, `kws_flv`, `kws_mnp` | `klua_net/` | 待定; **klb-net-design**; [k/readme.md](../../../lua/klua/readme.md) |
| 格式 | `kh26x` | `klua_format/` | 待补充 |

**未进 `klua_loadlib_all`**: 如 `krtp`, `kws_rtp` (须产品自行 `klua_loadlib`).

## §3 src_packages 扩展

> **待定** / **评估中**; 逐项 `no-<name>` 可裁. 详 **klb-klua-design** § src_packages.

| 扩展 | 路径 | 裁剪 |
|------|------|------|
| **klbwui** | `src_packages/klbwui/` | `no-wui`（依赖 klbgui） |

**已迁 backup**（不编入、不预加载）: `kpa_mgui`、`kpa_http`、`kpa_ws`、`kpa_mnp`、`kpa_flv`、`kpa_sip`、`kpa_rtsp` → `backup/src_packages/kpa_*/`.

## §4 klbcore 纯 Lua (须 package.path)

| require 前缀 | 路径 | 说明 |
|--------------|------|------|
| `klbcore.klbui` | `klbui/` | 声明式 UI; 见 [lua/klbcore/readme.md](../../../lua/klbcore/readme.md) § klbui |
| `klbcore.klbsmp` | `klbsmp/` | **klb-mnp-smp-design** § klbcore 脚本层 |
| `klbcore.klbrtsp` | `klbrtsp/` | RTSP 脚本层; **klbcore-net-design** |
| `klbcore.net.http_mime` | `http_mime.lua` | **klbcore-net-design** |

**已迁 backup**: `klbcore.net.httpc` (`backup/klbcore/net/`); 依赖旧 `khttp` multiplex 绑定.

| require 前缀 | 路径 | 说明 |
|--------------|------|------|
| `klbcore.util.*` | `util/` | **klbcore-design** § 通用模块 |
| `klbcore.base.*` | `base/` | **klbcore-design** |
| `klbcore.help.*` | `help/` | [klb/bin/klbcore/help/](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/help/) (示例/桩) |

路径根: [klb/bin/klbcore/](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/) (部署常拷至产品 `bin/klbcore`).

## §5 产品层追加

产品可在 `cb_pre_load` 中追加自有 C 预加载或 `require` 路径. 见 **klb-app-design**.

## §6 其它来源

| 来源 | 说明 |
|------|------|
| app **plugins** dll | `klbappex_pre_open` 注入 C 预加载 (如 `kpfs` → [lua/kpfs/readme.md](../../../lua/kpfs/readme.md)) |
| 产品 `bin/*lua/` | 业务脚本, `package.path` |

## 加载顺序小结

1. `luaL_openlibs` — §0
2. `cb_pre_load` — 通常 `klua_loadlib_all` + plugins → §1～§3
3. 入口脚本配置 `package.path` → §4

预加载机制详 [preload.md](preload.md); 环境生命周期 [lifecycle.md](lifecycle.md).
