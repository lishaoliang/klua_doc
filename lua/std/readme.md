# 标准 Lua 5.4

> `klua_doc/lua/std/` — 类别: ① 标准 Lua | 代码: `klb/src_c/klua/lua-5.4.6/src/linit.c` (`luaL_openlibs`)

`klua_env_doinit` 第一步调用 `luaL_openlibs`, 注册下列全局库.

各库文档采用与 [klua/](../klua/readme.md) k* 相同的 **四层结构**: 导出 API → 伪代码 → 示例 → 注意. 权威语义以 [Lua 5.4 参考手册](https://www.lua.org/manual/5.4/) 为准; 导出清单以 bundled `lua-5.4.6/src/l*.c` 为准.

## 库清单

| 全局 / require | 文档 | 说明 | klb 注意 |
|----------------|------|------|----------|
| `_G` (base) | [base.md](base.md) | `print`, `type`, `pcall`, `load`… | — |
| `package` | [package.md](package.md) | `require`, path, preload | klbcore 依赖 path; k* 在 preload |
| `coroutine` | [coroutine.md](coroutine.md) | 内置协程 | **业务勿用**; 用 **`kco`** → [kco.md](../klua/kco.md) |
| `table` | [table.md](table.md) | 表操作 | — |
| `io` | [io.md](io.md) | 文件/标准 IO | — |
| `os` | [os.md](os.md) | 时间, 环境, execute 等 | 慎用 `os.execute`; 退出 env 用 `kenv`/`ksys` |
| `string` | [string.md](string.md) | 字符串/二进制 pack | 复杂模式用 `lpeg`; JSON 用 `cjson` |
| `math` | [math.md](math.md) | 数学/随机 | 兼容 API 默认开启; 业务随机见 `krand` |
| `utf8` | [utf8.md](utf8.md) | UTF-8 | 字符数 ≠ 字节 `#s` |
| `debug` | [debug.md](debug.md) | 调试 | 生产慎用 hook/改栈 |

## 相关

| 文档 | 内容 |
|------|------|
| [index-by-require.md](../index-by-require.md) | 全库索引 |
| [klb/klua/design/require-guide.md](../../klb/klua/design/require-guide.md) | §0～§6 完整清单 |
| [klb/klua/design/layers.md](../../klb/klua/design/layers.md) | L0 VM 层 |
| [Lua 5.4 manual](https://www.lua.org/manual/5.4/) | 标准语义权威参考 |
