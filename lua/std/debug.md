## 调试 (debug)

> **require**: `debug` | 代码: `klb/src_c/klua/lua-5.4.6/src/ldblib.c` (`luaopen_debug`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.10](https://www.lua.org/manual/5.4/manual.html#6.10)

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `traceback([thread,] [msg [, level]])` | string | 栈回溯 |
| `getinfo(thread, f [, what])` | table / 无 | 激活记录信息 |
| `getlocal(thread, level, index)` | name, value | 局部变量 |
| `setlocal(thread, level, index, value)` | name | 改局部变量 |
| `getupvalue(f, index)` | name, value | 上值 |
| `setupvalue(f, index, value)` | name | 改上值 |
| `upvalueid(f, index)` | id | 上值唯一 id |
| `upvaluejoin(f1, n1, f2, n2)` | 无 | 共享上值 |
| `getmetatable(value)` | metatable | 调试版 (可穿透 __metatable) |
| `setmetatable(value, table)` | value | 调试版设元表 |
| `getuservalue(u [, index])` | value | userdata 关联值 |
| `setuservalue(u, value [, index])` | userdata | 设 userdata 值 |
| `gethook([thread])` | hook, mask, count | 钩子信息 |
| `sethook([thread,] hook, mask [, count])` | 无 | 设调试钩子 |
| `getregistry()` | table | registry 表 |
| `debug()` | 无 | 进入交互调试 (需外部支持) |
| `setcstacklimit(limit)` | oldlimit | C 栈深度上限 (Lua 5.4+) |

### 伪代码

真源: `ldblib.c` — `dblib[]`

```lua
--[[
-- @file   debug.lua (伪代码)
-- @brief  Lua 5.4 debug 库
--]]

local debug = {}


-- @brief 栈回溯字符串
-- @param [in] thread[thread]	[可选]
-- @param [in] msg[string]	[可选] 前缀
-- @param [in] level[number(int)]	[可选] 跳过层数
-- @return [string]
debug.traceback = function (thread, msg, level)
	return ''
end


-- @brief 激活记录 (what: "nSluftLr")
-- @return [table] { source, short_src, linedefined, … }
debug.getinfo = function (thread, f, what)
	return {}
end


-- @brief 读/写栈上局部变量 (level: 1=当前函数)
debug.getlocal = function (thread, level, index)
	return '', nil
end

debug.setlocal = function (thread, level, index, value)
	return ''
end


-- @brief 函数上值
debug.getupvalue = function (f, index)
	return '', nil
end

debug.setupvalue = function (f, index, value)
	return ''
end


-- @brief 上值唯一 id / 共享上值
debug.upvalueid = function (f, index)
	return {}
end

debug.upvaluejoin = function (f1, n1, f2, n2)
	return
end


-- @brief 调试版 get/set metatable (可穿透 __metatable)
debug.getmetatable = function (value)
	return {}
end

debug.setmetatable = function (value, table)
	return value
end


-- @brief userdata 关联 Lua 值
debug.getuservalue = function (u, index)
	return nil
end

debug.setuservalue = function (u, value, index)
	return u
end


-- @brief 调试钩子
-- @return hook[function|nil], mask[string], count[number(int)]
debug.gethook = function (thread)
	return nil, '', 0
end

debug.sethook = function (thread, hook, mask, count)
	return
end


-- @brief 进入交互调试 (需外部支持)
debug.debug = function ()
	return
end


-- @brief registry 表 (预加载、全局元表等)
debug.getregistry = function ()
	return {}
end


-- @brief 设置 C 调用栈限制; 返回旧限制
debug.setcstacklimit = function (limit)
	return 0
end

return debug
```

### 示例

```lua
local function inner()
	error("fail")
end

local function outer()
	inner()
end

local ok, err = pcall(outer)
print(debug.traceback(err, 2))

local info = debug.getinfo(outer, "Sl")
print(info.source, info.linedefined)
```

### 注意

- 生产环境慎用 `sethook` / 改上值 / `setlocal`; 可能影响性能与安全
- klua 业务调试优先日志与 **`ksys`** / 测试脚本; C 侧见 **klb-klua-env-design**
- `debug.getregistry()` 可看到 `_PRELOAD` 等内部结构, 勿随意修改
