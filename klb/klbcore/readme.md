# klbcore 文档

> `klua_doc/klb/klbcore/` — 代码: `klb/bin/klbcore/`

klbcore 是 **纯 Lua 运行时库** (klua 分层 **L6**). **非** C 预加载; 入口脚本须配置 `package.path`. 与 C 模块 **klbnet** / **klbgui** 同名不同层.

## 子目录

| 路径 | 内容 |
|------|------|
| [design/](design/) | 分层, 域模块分配, net/rtsp 脚本 |

## 设计

| 文件 | 说明 |
|------|------|
| [design/layers.md](design/layers.md) | **分层总览**, 域技能对照 (C vs 脚本) |
| [design/net-rtsp.md](design/net-rtsp.md) | `klbcore.net.http_mime`, `klbcore.klbrtsp` |

## 域模块 (源码)

| 目录 | require | 设计文档 |
|------|---------|----------|
| `klbui/` | `klbcore.klbui` | [klbui/readme.md](../klbui/readme.md) |
| `net/` | `klbcore.net.http_mime` | [design/net-rtsp.md](design/net-rtsp.md) |
| `klbrtsp/` | `klbcore.klbrtsp` | 同上 |
| `klbsmp/` | `klbcore.klbsmp` | 技能 **klb-mnp-smp-design** § klbcore 脚本层 |
| `util/`, `base/`, `help/` | `klbcore.util.*` 等 | 技能 **klbcore-design** |

## 关联

| 入口 | 说明 |
|------|------|
| [klua/design/require-guide.md](../klua/design/require-guide.md) | §4 klbcore 清单 |
| [klua/design/layers.md](../klua/design/layers.md) | L6 klbcore |
| C **klbnet** / k* | 技能 **klb-net-design** (≠ 本目录) |

## 查阅顺序 (ai)

1. [design/layers.md](design/layers.md)
2. 域设计 (net/rtsp → [design/net-rtsp.md](design/net-rtsp.md); ui → [klbui](../klbui/readme.md))
3. `klb/bin/klbcore/` 源码
4. **klbcore-design**, **klbcore-net-design**
