# klua 文档

> `klua_doc/klb/klua/` — 代码: `klb/inc/klua/`, `klb/src_c/klua/`

## 子目录

| 路径 | 内容 |
|------|------|
| [design/](design/) | 分层总览, 生命周期, 预加载, env 扩展, C API, require 清单, 协程约定 |
| [k/](k/) | L3 **k\*** C→Lua API |

## 设计

| 文件 | 说明 |
|------|------|
| [design/layers.md](design/layers.md) | **八层总览**与选型表 |
| [design/k-bindings.md](design/k-bindings.md) | **L3 k\*** 机制与域对照 |
| [design/lifecycle.md](design/lifecycle.md) | `klua_env` 生命周期与 loop |
| [design/c-env-api.md](design/c-env-api.md) | L1/L2 C 头导读 (`inc/klua/`) |
| [design/env-extension.md](design/env-extension.md) | env 框架扩展 (`klua_ex_*`, LPC, 两阶段注册) |
| [design/preload.md](design/preload.md) | `_PRELOAD`, `klua_loadlib`, plugins |
| [design/require-guide.md](design/require-guide.md) | §0～§6 库清单 |
| [design/coroutine.md](design/coroutine.md) | 为何用 `kco` 而非 `coroutine` |

## k* API 文档

全表见 [k/readme.md](k/readme.md). 网络类在 [k/net/](k/net/).

| 分组 | 入口 |
|------|------|
| 基础/平台 | [k/readme.md](k/readme.md) § 基础 |
| 网络 | [k/net/](k/net/) |

## 关联文档

| 入口 | 说明 |
|------|------|
| [klbcore 脚本库](../klbcore/readme.md) | L6 `klbcore.*` (非 k*) |
| [klbapp 启动](../klbapp/design/startup.md) | `klb_app_main` 与 klua 预加载 |
| [klbapp plugins](../klbapp/design/plugins.md) | 动态库注入预加载 |
| [klbui](../klbui/readme.md) | 脚本 UI (`klbcore.klbui` + `kgui`) |
| [api 总览](../api/overview.md) | klb C API 模块索引 |

## 查阅顺序 (ai)

1. [design/layers.md](design/layers.md) — 分层总览
2. L3: [design/k-bindings.md](design/k-bindings.md) → [k/readme.md](k/readme.md)
3. `klb/inc/klua/`
4. `klb/src_c/klua/`
5. 设计技能 **klb-klua-design**, **klb-klua-env-design**, **klbcore-design**
