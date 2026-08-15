# k* Lua API 文档

> `klua_doc/klb/klua/k/` — C 绑定: `klb/src_c/klua/**/klua_k*.c`

L3 **C→Lua** 模块. 机制见 [design/k-bindings.md](../design/k-bindings.md).

## 查阅顺序

1. 本目录 `kxxx.lua.md` (Lua 调用面)
2. 域 C 技能 (**klb-net-design** / **klb-gui-design** …)
3. `klua_kxxx.c` 源码

## 基础 / 平台

| require | 文档 | C 源 | 状态 |
|---------|------|------|------|
| `kco` | [kco.lua.md](kco.lua.md) | `klua_base/klua_kcoro.c` | 已有 |
| `klpc` | [klpc.lua.md](klpc.lua.md) | `klua_base/klua_klpc.c` | 已有 |
| `kgui` | [kgui.lua.md](kgui.lua.md) | `klua_base/klua_kgui.c` | 已有 |
| `kkpa` | [kkpa.lua.md](kkpa.lua.md) | `klua_base/klua_kpackage.c` | 桩 |
| `kenv` | [kenv.lua.md](kenv.lua.md) | `klua_util/klua_kenv.c` | 已有 |
| `ksys` | [ksys.lua.md](ksys.lua.md) | `klua_util/klua_ksys.c` | 已有 |
| `krand` | [krand.lua.md](krand.lua.md) | `klua_util/klua_krand.c` | 已有 |
| `kos` | [kos.lua.md](kos.lua.md) | `klua_platform/klua_kos.c` | 已有 |
| `ktime` | [ktime.lua.md](ktime.lua.md) | `klua_platform/klua_ktime.c` | 已有 |
| `kthread` | [kthread.lua.md](kthread.lua.md) | `klua_multithread/klua_kthread.c` | 已有 |
| `klist` | — | `klua_multithread/klua_klist.c` | P2 |
| `kmcache` | — | `klua_multithread/klua_kmcache.c` | P2 |
| `kh26x` | — | `klua_format/klua_kh26x.c` | P2 |

## 网络 (`k/net/`)

| require | 文档 | C 源 | 状态 |
|---------|------|------|------|
| `krtsp` | [net/krtsp.lua.md](net/krtsp.lua.md) | `klua_net/klua_krtsp.c` | 桩 |
| `ksmp` | [net/ksmp.lua.md](net/ksmp.lua.md) | `klua_net/klua_ksmp.c` | 桩 |
| `kurl` | [net/kurl.lua.md](net/kurl.lua.md) | `klua_net/klua_kurl.c` | 桩 |
| `kmnp` | [net/kmnp.lua.md](net/kmnp.lua.md) | `klua_net/klua_kmnp.c` | 空库桩 |
| `khttp_flv` | [net/khttp_flv.lua.md](net/khttp_flv.lua.md) | `klua_net/klua_khttp_flv.c` | 空库桩 |
| `khttp_mnp` | [net/khttp_mnp.lua.md](net/khttp_mnp.lua.md) | `klua_net/klua_khttp_mnp.c` | 空库桩 |
| `kws_flv` | [net/kws_flv.lua.md](net/kws_flv.lua.md) | `klua_net/klua_kws_flv.c` | 空库桩 |
| `kws_mnp` | [net/kws_mnp.lua.md](net/kws_mnp.lua.md) | `klua_net/klua_kws_mnp.c` | 空库桩 |
| `krtp` | [net/krtp.lua.md](net/krtp.lua.md) | `klua_net/klua_krtp.c` | 空库桩 |
| `kws_rtp` | [net/kws_rtp.lua.md](net/kws_rtp.lua.md) | `klua_net/klua_kws_rtp.c` | 空库桩 |

**未进 `klua_loadlib_all`**: `krtp`, `kws_rtp` (须产品自行 `klua_loadlib`); `ksip` 无 open 入口.

L6 封装: [klbcore/design/net-rtsp.md](../../klbcore/design/net-rtsp.md). C 栈: **klb-net-design**.

**已迁 backup (旧栈 multiplex)**: `ktcp`, `kudp`, `khttp`, `kws` — 绑定 `klb/backup/klua_net/`; API 文档 `klb/backup/klua_doc/klb/klua/k/net/`.
