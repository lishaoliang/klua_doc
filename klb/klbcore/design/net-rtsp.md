# klbcore 脚本 net / RTSP

> `klua_doc/klb/klbcore/design/net-rtsp.md` — 代码: `klb/bin/klbcore/net/`, `klb/bin/klbcore/klbrtsp/`

## 结论

`klbcore.klbrtsp` 与 `klbcore.net.http_mime` 是 **L6 纯 Lua** 网络相关封装. **非** C `klbnet`. C 传输与 k* 绑定见技能 **klb-net-design**.

**已迁 backup**: `klbcore.net.httpc` (依赖旧 `khttp` multiplex) — `klb/backup/klbcore/net/`; 绑定 `klb/backup/klua_net/`.

## 分层

```
产品脚本
    ↓ require
klbcore.klbrtsp / net.http_mime (本节)
    ↓ require krtsp, kco
klua_net (k* 绑定)          → klb-net-design
    ↓
klb_netmulti_loop_once + kco yield
```

IO 型调用须在 **kco 协程**内 (同 SMP 脚本, 见 **klb-mnp-smp-design** § kco).

## klbcore.net (`net/`)

| 模块 | 文件 | 说明 |
|------|------|------|
| `klbcore.net.http_mime` | `http_mime.lua` | MIME 辅助 |

## klbcore.klbrtsp (`klbrtsp/`)

| 入口 / 文件 | 说明 |
|-------------|------|
| `init.lua` | `require("klbcore.klbrtsp")`; `new_client`, `new_listen` |
| `client/rtspclienter.lua` | RTSP 客户端 |
| `client/rtspsdper.lua` | SDP 辅助 |
| `serve/rtsplistener.lua` | 监听 |
| `serve/rtspserver.lua` | 会话服务 |
| `serve/rtspserve_parser.lua` | 请求解析 |
| `rtspcode.lua` | 状态码等 |

C 对照: `require("krtsp")` (`klua_krtsp.c`); C 协议 `klb/src_c/klbnet/klbrtsp/` (**开发调整中**).

## 与 klbsmp 分界

| 包 | 路径 | 设计 |
|----|------|------|
| `klbcore.klbsmp` | `klbsmp/` | **klb-mnp-smp-design** § klbcore 脚本层 |
| `klbcore.klbrtsp` | `klbrtsp/` | 本节 |
| `klbcore.net.http_mime` | `net/` | 本节 |

## 与 C 模块记名

| 口语 | 脚本侧 (本节) | C / k* 侧 |
|------|---------------|-----------|
| rtsp | `klbcore.klbrtsp` | `krtsp`, `klbrtsp` C |

## 相关

| 文档 | 内容 |
|------|------|
| [layers.md](layers.md) | klbcore 域分配 |
| [klua/design/require-guide.md](../../klua/design/require-guide.md) | §2 k*, §4 klbcore |
| [klua/design/coroutine.md](../../klua/design/coroutine.md) | kco |

设计技能: **klbcore-net-design** (脚本); **klb-net-design** (C).
