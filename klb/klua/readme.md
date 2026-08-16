# klua 文档

> `klua_doc/klb/klua/` — 代码: `klb/inc/klua/`, `klb/src_c/klua/` — **C 运行时与机制**. Lua 脚本 API 真源: [lua/klua/](../../lua/klua/readme.md).

## 子目录

| 路径 | 内容 |
|------|------|
| [design/](design/) | 分层总览, 生命周期, 预加载, env 扩展, require 清单 |
| [api/](api/) | C API 说明 |

## 设计 (C / 机制)

| 文件 | 说明 |
|------|------|
| [design/layers.md](design/layers.md) | **八层总览**与选型表 |
| [design/k-bindings.md](design/k-bindings.md) | **L3 k\*** 注册机制 (Lua API → [lua/klua/](../../lua/klua/readme.md)) |
| [design/lifecycle.md](design/lifecycle.md) | `klua_env` 生命周期与 loop |
| [design/env-extension.md](design/env-extension.md) | env 框架扩展 (`klua_ex_*`, LPC, 两阶段注册) |
| [design/preload.md](design/preload.md) | `_PRELOAD`, `klua_loadlib`, plugins |
| [design/require-guide.md](design/require-guide.md) | §0～§6 库清单 |

## API

| 文件 | 说明 |
|------|------|
| [api/klua_env.md](api/klua_env.md) | L1/L2 C 头导读 (`inc/klua/`) |

## Lua 脚本 API (在 lua/)

| 分组 | 入口 |
|------|------|
| k* C→Lua | [lua/klua/readme.md](../../lua/klua/readme.md) |
| 协程 (kco) | [lua/klua/guide/coroutine.md](../../lua/klua/guide/coroutine.md) |
| 按 require 索引 | [lua/index-by-require.md](../../lua/index-by-require.md) |

## 关联文档

| 入口 | 说明 |
|------|------|
| [lua/klbcore](../../lua/klbcore/readme.md) | L6 脚本库; 架构 **klbcore-design** |
| [klbapp 启动](../klbapp/design/startup.md) | `klb_app_main` 与 klua 预加载 |
| [klbapp plugins](../klbapp/design/plugins.md) | 动态库注入预加载 |
| [klbgui](../klbgui/readme.md) | C klbgui; Lua UI [lua/klbcore/](../../lua/klbcore/readme.md) |
| [klbapp](../klbapp/readme.md) | 应用壳 |

## 查阅顺序 (ai)

1. [design/layers.md](design/layers.md) — 分层总览
2. C API: [api/klua_env.md](api/klua_env.md)
3. L3 机制: [design/k-bindings.md](design/k-bindings.md) → Lua API [lua/klua/readme.md](../../lua/klua/readme.md)
4. `klb/inc/klua/`
5. `klb/src_c/klua/`
6. 设计技能 **klb-klua-design**, **klb-klua-env-design**, **klbcore-design**
