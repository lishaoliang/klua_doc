# Lua 脚本文档

> `klua_doc/lua/` — 按 `require` 形态归类的 **Lua 脚本侧** 文档真源. C API 与 klua 机制见 [klb/](../klb/readme.md).

## 分类导航

| 类别 | 目录 | 加载方式 |
|------|------|----------|
| ① 标准 Lua 5.4 | [std/](std/readme.md) | `luaL_openlibs` |
| ② 第三方开源 (bundled) | [bundled/](bundled/readme.md) | C 预加载 `require("cjson")` 等 |
| ③ 纯 Lua 脚本库 klbcore | [klbcore/](klbcore/readme.md) | `package.path` → `klbcore.*` |
| ④ klua 扩展 (k*) | [klua/](klua/readme.md) | C 预加载 `require("kco")` 等 |
| ⑤ kpfs 扩展 | [kpfs/](kpfs/readme.md) | plugins 注入 `require("kpfs")` |
| ⑥ **lua test** 手测 | [lua_test/](lua_test/readme.md) | `klua_run/test.lua` + `lua_test.*` |

## 快速入口

| 需求 | 入口 |
|------|------|
| 按 `require` 名查文档 | [index-by-require.md](index-by-require.md) |
| k* C→Lua API | [klua/readme.md](klua/readme.md) |
| kpfs Lua API | [kpfs/readme.md](kpfs/readme.md) |
| 为何用 kco 不用 coroutine | [klua/guide/coroutine.md](klua/guide/coroutine.md) |
| klbcore 脚本库 | [klbcore/readme.md](klbcore/readme.md) |
| bundled 第三方 | [bundled/readme.md](bundled/readme.md) |
| klua 八层架构 (C 侧) | [klb/klua/design/layers.md](../klb/klua/design/layers.md) |
| require 全量清单 | [klb/klua/design/require-guide.md](../klb/klua/design/require-guide.md) |

## 加载顺序小结

1. `luaL_openlibs` — ① [std](std/readme.md)
2. `cb_pre_load` — ② [bundled](bundled/readme.md) + ④ [klua](klua/readme.md) + plugins ⑤
3. 入口脚本配置 `package.path` — ③ [klbcore](klbcore/readme.md)

预加载机制详 [klb/klua/design/preload.md](../klb/klua/design/preload.md).

## 迁移状态

| 来源                                           | 目标             | 状态                        |
| -------------------------------------------- | -------------- | ------------------------- |
| `klb/klua/k/*.lua.md` 等                      | `lua/klua/`    | 已迁入; **klb/ 下 stub 已删除**  |
| `pfs/kpfs.md`                                | `lua/kpfs/`    | 已迁入; stub 已删除             |
| `klb/klbcore/` (已删)                          | `lua/klbcore/` | 已迁入; `klb/klbcore/` 目录已删除 |
| C 侧文档 (`klb/klua/design/`, `klbgui/design/`) | —              | 保留; **Lua API 链到 `lua/`** |
