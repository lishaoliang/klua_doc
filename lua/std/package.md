## 模块加载 (package)

> **require**: `package` | 代码: `klb/src_c/klua/lua-5.4.6/src/loadlib.c` (`luaopen_package`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.3](https://www.lua.org/manual/5.4/manual.html#6.3)

### 导出 API

| 符号 | 返回 | 说明 |
|------|------|------|
| `require(modname)` | module | 加载模块 (全局函数, 非 `package` 表字段) |
| `loadlib(name, funcname)` | func / `nil, msg` | 动态加载 C 库符号 (低层) |
| `searchpath(name, path [, sep [, rep]])` | filename / `nil, msg` | 在 path 模板中搜索 |
| `path` | string | Lua 模块搜索路径 (`LUA_PATH` / 默认) |
| `cpath` | string | C 模块搜索路径 (`LUA_CPATH`) |
| `config` | string | 路径分隔符等配置 (只读拼接串) |
| `loaded` | table | 已加载模块缓存 (`package.loaded`) |
| `preload` | table | 预加载 C  opener (`registry._PRELOAD`) |
| `searchers` | array | require 搜索器链 (preload → Lua → C → Croot) |

klb 预加载 (bundled + k* + plugins) 写入 `package.preload`; 详 [preload.md](../../klb/klua/design/preload.md).

### 伪代码

真源: `loadlib.c` — `pk_funcs[]`, `ll_funcs[]`, `createsearcherstable`

```lua
--[[
-- @file   package.lua (伪代码)
-- @brief  Lua 5.4 package 库
--   \n 代码: ./klb/src_c/klua/lua-5.4.6/src/loadlib.c
--]]

local package = {}

package.path = '?.lua;?/init.lua'
package.cpath = ''
package.config = '/\n;\n?\n!\n-\n'
package.loaded = {}		-- 已 require 的模块
package.preload = {}		-- modname -> opener; klb 在此注册 cjson/kco/kpfs 等
package.searchers = {}		-- 1=preload, 2=Lua文件, 3=C库, 4=Croot


-- @brief 加载并返回模块 (全局函数)
-- @param [in] modname[string]	如 "kco", "klbcore.init", "kpfs"
-- @return [any] module 表或返回值
-- @note 首次加载后写入 package.loaded[modname]
require = function (modname)
	return {}
end


-- @brief 在 path 模板中查找 Lua/C 模块文件
-- @param [in] name[string]	模块名 (点转路径)
-- @param [in] path[string]	package.path 或 package.cpath
-- @param [in] sep[string]	[可选] 目录分隔, 默认 package.config 首字符
-- @param [in] rep[string]	[可选] 点名替换, 默认 "?"
-- @return [string|nil] filename, [string|nil] err
package.searchpath = function (name, path, sep, rep)
	return 'kco.lua'
end


-- @brief 动态加载 C 库函数 (低层; 一般用 require)
-- @return [function|nil] f, [string|nil] err
package.loadlib = function (name, funcname)
	return function () end
end

return package
```

### 示例

```lua
-- klbcore 典型: 扩展 package.path 后 require 纯 Lua 模块
local kenv = require("kenv")
package.path = kenv.base_path() .. "/klbcore/?.lua;" .. package.path

local kco = require("kco")          -- C 预加载 (package.preload)
local ui = require("klbcore.klbui") -- Lua 文件搜索

-- 查看是否已加载
if package.loaded["kpfs"] then
	print("kpfs already loaded")
end
```

### 注意

- **klbcore** 依赖 `package.path`; 入口常用 `kenv.base_path()` 拼接 → [kenv.md](../klua/kenv.md)
- k* / bundled / kpfs 由 C 侧 `klua_loadlib` 写入 **`package.preload`**, 非 `package.path`
- 全量 require 清单 → [require-guide.md](../../klb/klua/design/require-guide.md)
- 工作区脚本 **`require("…")` 须带括号** → **coding-lua** § require
