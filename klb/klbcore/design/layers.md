# klbcore 分层与域分配

> `klua_doc/klb/klbcore/design/layers.md` — 代码: `klb/bin/klbcore/`

## 结论

klbcore 是 klua **L6 纯 Lua 库**. 与 **C 同名模块** (klbnet, klbgui) **必须分文档阅读**: 脚本包在 `klb/bin/klbcore/`; C 在 `klb/inc/` + `klb/src_c/`.

## 层次图 (相对 klua)

```
L7  产品脚本              bin/*lua/
L6  klbcore (本节)        klb/bin/klbcore/     require klbcore.*
L3  k* C 绑定             klua_k*.c            require("krtsp") 等
L1  C klbnet / klbgui     klb/src_c/klbnet/…   C API / 内部对象
```

脚本 **不** 直接调用 `klb_netconn_*`; 经 **k*** 或本层封装.

## 域模块表

| klbcore 目录 | require | 脚本设计 | C / k* 对照 |
|--------------|---------|----------|-------------|
| `klbui/` | `klbcore.klbui` | [klbui/readme.md](../../klbui/readme.md) | **klb-gui-design**; `kgui` |
| `net/` | `klbcore.net.http_mime` | [net-rtsp.md](net-rtsp.md) | **klb-net-design** |
| `klbrtsp/` | `klbcore.klbrtsp` | [net-rtsp.md](net-rtsp.md) | **klb-net-design**; `krtsp` |
| `klbsmp/` | `klbcore.klbsmp` | **klb-mnp-smp-design** § 脚本层 | `ksmp`, C klbsmp |
| `util/`, `base/` | `klbcore.util.*` 等 | **klbcore-design** | 各 k* |
| `help/` | `klbcore.help.*` | **klbcore-design** § 示例 | 桩与 demo |

**无** `klbmnp` klbcore 包 (对称缺口, 见 **klb-mnp-smp-design**).

## 设计技能分工 (C vs 脚本)

| 技能 | 范围 |
|------|------|
| **klbcore-design** | klbcore 横切: path, kthread, klbui 脚本正文 |
| **klbcore-net-design** | 仅脚本 `net/`, `klbrtsp/` |
| **klb-net-design** | 仅 C `klbnet` + `klua_net` k* (**不含** klbcore 脚本) |
| **klb-gui-design** | 仅 C klbgui + kgui (**不含** klbui 脚本正文) |
| **klb-mnp-smp-design** | mnp/smp 规范 C + klbsmp 脚本 |

用户说「记录 net 设计」: **C** → **klb-net-design**; **脚本 net/rtsp** → **klbcore-net-design**.

## package.path

```lua
local base = kenv.base_path() .. '/klbcore/'
package.path = package.path .. ';' .. base .. '?.lua;' .. base .. '?/init.lua'
```

详 [require-guide.md](../../klua/design/require-guide.md) §4, **klbcore-design** § require.

## 我该改哪一层

| 需求 | 改哪里 | 文档 |
|------|--------|------|
| RTSP 会话脚本 | `klbcore/klbrtsp/` | [net-rtsp.md](net-rtsp.md) |
| `krtsp` 绑定 | `klua_net/` | **klb-net-design** |
| socket / netmulti C | `klbnet/` | **klb-net-design** |
| 声明式 UI 脚本 | `klbcore/klbui/` | [klbui/readme.md](../../klbui/readme.md) |
| SMP RPC 脚本 | `klbcore/klbsmp/` | **klb-mnp-smp-design** |

## 相关文档

| 文档 | 内容 |
|------|------|
| [net-rtsp.md](net-rtsp.md) | 脚本 HTTP / RTSP |
| [klua/design/layers.md](../../klua/design/layers.md) | klua 八层 |
| [klua/design/require-guide.md](../../klua/design/require-guide.md) | require 清单 |
