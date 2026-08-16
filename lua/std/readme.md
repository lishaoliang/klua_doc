# 标准 Lua 5.4

> `klua_doc/lua/std/` — 类别: ① 标准 Lua | 代码: `klb/src_c/klua/lua-5.4.6/src/linit.c` (`luaL_openlibs`)

`klua_env_doinit` 第一步调用 `luaL_openlibs`, 注册下列全局库.

## 库清单

| 全局名 | 说明 | klb 注意 |
|--------|------|----------|
| `_G` | base (print, type, error…) | — |
| `package` | require, path, loaded | klbcore 依赖 path 配置 |
| `coroutine` | Lua 内置协程 | **业务勿用**; 用 **`kco`** → [klua/kco.md](../klua/kco.md), [guide/coroutine.md](../klua/guide/coroutine.md) |
| `table` | 表操作 | — |
| `io` | 文件/标准 IO | — |
| `os` | 时间, 环境, execute 等 | 嵌入式慎用 `os.execute` |
| `string` | 字符串 | — |
| `math` | 数学 | — |
| `utf8` | UTF-8 | — |
| `debug` | 调试 | — |

## 相关

| 文档 | 内容 |
|------|------|
| [index-by-require.md](../index-by-require.md) | 全库索引 |
| [klb/klua/design/require-guide.md](../../klb/klua/design/require-guide.md) | §0～§6 完整清单 |
| [klb/klua/design/layers.md](../../klb/klua/design/layers.md) | L0 VM 层 |
