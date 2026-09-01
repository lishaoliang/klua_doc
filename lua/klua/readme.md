# klua 扩展 (k*)

> `klua_doc/lua/klua/` — 类别: ④ klua 扩展 (L3 C→Lua) | 代码: `klb/src_c/klua/**/klua_k*.c`

C 经 `klua_loadlib` 注册到 `registry._PRELOAD`, Lua 侧 `require("kxxx")`. 机制与 **Lua API 文档** 四层见 [klb/klua/design/k-bindings.md](../../klb/klua/design/k-bindings.md) (导出 API 简略、伪代码详注; 样板 [kgui.md](kgui.md) / [kpfs.md](../kpfs/kpfs.md)).

## 查阅顺序

1. 本目录 `kxxx.md` (Lua 调用面)
2. 域 C 技能 (**klb-net-design** / **klb-gui-design** …)
3. `klua_kxxx.c` 源码

## 基础 / 平台

| require | 文档 | C 源 | 状态 |
|---------|------|------|------|
| `kco` | [kco.md](kco.md) | `klua_base/klua_kcoro.c` | 已有 |
| `klpc` | [klpc.md](klpc.md) | `klua_base/klua_klpc.c` | 已有 |
| `kgui` | [kgui.md](kgui.md) | `klua_base/klua_kgui.c` | 已有 (**文档样板**) |
| `kkpa` | [kkpa.md](kkpa.md) | `klua_base/klua_kpackage.c` | 已有 (P2) |
| `kenv` | [kenv.md](kenv.md) | `klua_util/klua_kenv.c` | 已有 |
| `ksys` | [ksys.md](ksys.md) | `klua_util/klua_ksys.c` | 已有 |
| `krand` | [krand.md](krand.md) | `klua_util/klua_krand.c` | 已有 |
| `kos` | [kos.md](kos.md) | `klua_platform/klua_kos.c` | 已有 |
| `ktime` | [ktime.md](ktime.md) | `klua_platform/klua_ktime.c` | 已有 |
| `kthread` | [kthread.md](kthread.md) | `klua_multithread/klua_kthread.c` | 已有 |
| `klist` | [klist.md](klist.md) | `klua_multithread/klua_klist.c` | 已有 |
| `kmcache` | [kmcache.md](kmcache.md) | `klua_multithread/klua_kmcache.c` | 已有 |
| `kh26x` | [kh26x.md](kh26x.md) | `klua_format/klua_kh26x.c` | 已有 (P2) |

## 网络 (k*; 待定)

| require | 文档 | C 源 | 状态 |
|---------|------|------|------|
| `krtsp` | [krtsp.md](krtsp.md) | `klua_net/klua_krtsp.c` | 已有 |
| `ksmp` | [ksmp.md](ksmp.md) | `klua_net/klua_ksmp.c` | 已有 |
| `kurl` | [kurl.md](kurl.md) | `klua_net/klua_kurl.c` | 已有 |
| `kmnp` | — | `klua_net/klua_kmnp.c` | 空库桩 |
| `khttp_flv` | — | `klua_net/klua_khttp_flv.c` | 空库桩 |
| `khttp_mnp` | — | `klua_net/klua_khttp_mnp.c` | 空库桩 |
| `kws_flv` | — | `klua_net/klua_kws_flv.c` | 空库桩 |
| `kws_mnp` | — | `klua_net/klua_kws_mnp.c` | 空库桩 |
| `krtp` | — | `klua_net/klua_krtp.c` | 空库桩 |
| `kws_rtp` | — | `klua_net/klua_kws_rtp.c` | 空库桩 |

**未进 `klua_loadlib_all`**: `krtp`, `kws_rtp` (须产品自行 `klua_loadlib`); `ksip` 无 open 入口.

L6 封装: [klbcore/readme.md](../klbcore/readme.md) + **klbcore-net-design**. C 栈: **klb-net-design**. 清单: [require-guide.md](../../klb/klua/design/require-guide.md) §2.

**已迁 backup (旧栈 multiplex)**: `ktcp`, `kudp`, `khttp`, `kws` — 绑定 `backup/klua_net/`; 归档 API 文档 `backup/klua_doc/klb/klua/k/net/`.

## guide

| 文件 | 说明 |
|------|------|
| [guide/coroutine.md](guide/coroutine.md) | 为何用 `kco` 而非 `coroutine` |

## 相关 (C 侧)

| 文档 | 内容 |
|------|------|
| [klb/klua/design/layers.md](../../klb/klua/design/layers.md) | 八层总览 |
| [klb/klua/design/lifecycle.md](../../klb/klua/design/lifecycle.md) | `klua_env` 生命周期 |
| [klb/klua/design/preload.md](../../klb/klua/design/preload.md) | 预加载链 |
| [klb/klua/api/klua_env.md](../../klb/klua/api/klua_env.md) | L1/L2 C API |
| [klb/klua/design/require-guide.md](../../klb/klua/design/require-guide.md) | require 全量清单 (含 src_packages, 待定) |
