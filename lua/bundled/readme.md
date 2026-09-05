# bundled 第三方 Lua 库

> `klua_doc/lua/bundled/` — 类别: ② 第三方开源 (C 预加载) | 代码: [klb/src_c/klua/*-x.x/](https://gitee.com/klua/klb/tree/trunk/src_c/klua/*-x.x/)

经 `klua_loadlib` 写入 `registry._PRELOAD`, Lua 侧 `require("短名")` 加载. 详 [klb/klua/design/preload.md](../../klb/klua/design/preload.md).

各库文档采用与 [klua/](../klua/readme.md) k* 相同的 **四层结构**: 导出 API → 伪代码 → 示例 → 注意.

## 库清单

| require | 版本 (vendor) | 文档 | 裁剪宏 | 源码目录 |
|---------|---------------|------|--------|----------|
| `cjson`, `cjson.safe` | 2.1.0 | [cjson.md](cjson.md) | — | `lua-cjson-2.1.0/` |
| `lfs` | 1.6.3 | [lfs.md](lfs.md) | — | `luafilesystem-2.0/` |
| `LuaXML_lib` | 130610 | [luaxml.md](luaxml.md) | — | `LuaXML_130610/` |
| `lpeg` | 1.0.2 | [lpeg.md](lpeg.md) | `__KLB_NO_LPEG__` | `lpeg-1.0.2/` |
| `zlib` | lua-zlib 1.3 | [zlib.md](zlib.md) | `__KLB_NO_ZLIB__` | `lua-zlib-1.3/` |
| `lsqlite3` | lsqlite3 + sqlite3 | [lsqlite3.md](lsqlite3.md) | `__KLB_NO_SQLITE__` | `lsqlite3/` |

## 相关

| 文档 | 内容 |
|------|------|
| [index-by-require.md](../index-by-require.md) | 全库索引 |
| [klb/klua/design/require-guide.md](../../klb/klua/design/require-guide.md) | §1 bundled |
| [ksys.md](../klua/ksys.md) | klb JSON 传参 (`pack_json`, 非 cjson) |
| 技能 **klb-vendor-subagents** | vendor 形态与升级边界 |
