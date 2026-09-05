## 基础库 (_G)

> **全局**: `_G` (base) | 代码: [klb/src_c/klua/lua-5.4.6/src/lbaselib.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-5.4.6/src/lbaselib.c) (`luaopen_base`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.1](https://www.lua.org/manual/5.4/manual.html#6.1)

### 导出 API

`luaL_openlibs` 将下列函数写入全局环境 (亦可通过 `_G` 访问). 只读字段: `_G._G` (全局表自身), `_G._VERSION` (如 `"Lua 5.4"`).

| 函数 | 返回 | 说明 |
|------|------|------|
| `assert(v [, msg])` | `v` 或 无 (失败) | 条件为假则 `error` |
| `collectgarbage([opt [, arg]])` | 依 opt | GC 控制 (`collect`/`count`/`step`/…) |
| `dofile([filename])` | `...` | 加载并执行文件 |
| `error(msg [, level])` | 无 | 抛出错误 |
| `getmetatable(obj)` | metatable / `nil` | 取元表 |
| `ipairs(t)` | iterator, t, 0 | 整数键迭代 |
| `load(chunk [, name [, mode [, env]]])` | function / `nil, err` | 编译 chunk |
| `loadfile([filename [, mode [, env]]])` | function / `nil, err` | 编译文件 |
| `next(t [, index])` | index, value / 无 | 表下一键值 |
| `pairs(t)` | iterator, t, `nil` | 全键迭代 (含 `__pairs`) |
| `pcall(f [, ...])` | ok, `...` | 保护调用 |
| `print(...)` | 无 | 输出 (经 `tostring`) |
| `warn(...)` | 无 | 警告 (可配 `@on`/`@off`) |
| `rawequal(v1, v2)` | boolean | 原始相等 |
| `rawget(t, index)` | value | 绕过 `__index` 读 |
| `rawlen(v)` | integer | 原始长度 (string/table) |
| `rawset(t, index, value)` | t | 绕过 `__newindex` 写 |
| `select(index, ...)` | `...` | 选取可变参 |
| `setmetatable(t, mt)` | t | 设元表 |
| `tonumber(v [, base])` | number / `nil` | 转数值 |
| `tostring(v)` | string | 转字符串 (含 `__tostring`) |
| `type(v)` | string | 类型名 |
| `xpcall(f, msgh [, ...])` | ok, `...` | 带错误处理器的保护调用 |

### 伪代码

真源: `lbaselib.c` — `base_funcs[]`

```lua
--[[
-- @file   base.lua (伪代码, 非独立模块)
-- @brief  Lua 5.4 基础库 — 全局函数
--   \n 代码: ./klb/src_c/klua/lua-5.4.6/src/lbaselib.c
--]]

-- @brief 断言; v 为假则 error(msg)
-- @param [in] v[any]
-- @param [in] msg[any]	[可选] 错误信息
-- @return [any] v
assert = function (v, msg)
	return v
end

-- @brief 垃圾回收控制
-- @param [in] opt[string]	[可选] "collect"|"count"|"step"|"setpause"|"setstepmul"|"isrunning"
-- @param [in] arg[number(int)]	[可选] 部分 opt 的参数
-- @return [any] 依 opt 而定
collectgarbage = function (opt, arg)
	return 0
end

-- @brief 加载并执行文件
-- @param [in] filename[string]	[可选] 省略则从 stdin
-- @return [...] chunk 返回值
dofile = function (filename)
	return ...
end

-- @brief 抛出错误 (可带栈 level)
-- @param [in] msg[any]
-- @param [in] level[number(int)]	[可选] 栈层级, 默认 1
error = function (msg, level)
end

-- @brief 取元表 (尊重 __metatable)
-- @return [table|nil]
getmetatable = function (obj)
	return {}
end

-- @brief 整数索引迭代 (1..n)
-- @return [function] iterator, [table] t, [number(int)] 0
ipairs = function (t)
	return function () end, t, 0
end

-- @brief 编译字符串或 loader 返回的 chunk
-- @param [in] chunk[string|function]
-- @param [in] name[string]	[可选] chunk 名
-- @param [in] mode[string]	[可选] "b"|"t"|"bt"
-- @param [in] env[table]	[可选] 运行环境
-- @return [function|nil] f, [string|nil] err
load = function (chunk, name, mode, env)
	return function () end
end

-- @brief 编译文件
-- @param [in] filename[string]	[可选]
-- @param [in] mode[string]	[可选]
-- @param [in] env[table]	[可选]
-- @return [function|nil] f, [string|nil] err
loadfile = function (filename, mode, env)
	return function () end
end

-- @brief 表迭代下一键 (配合 for)
-- @param [in] t[table]
-- @param [in] index[any]	[可选] 上一键; 省略则从首键
-- @return [any] key, [any] value; 结束无返回
next = function (t, index)
	return nil, nil
end

-- @brief 全键迭代 (调用 __pairs 若存在)
-- @return [function] iterator, [table] t, [nil]
pairs = function (t)
	return next, t, nil
end

-- @brief 保护模式调用
-- @return [boolean] ok, [...] 返回值或错误对象
pcall = function (f, ...)
	return true, ...
end

-- @brief 打印 (tab 分隔, 末尾换行)
print = function (...)
end

-- @brief 输出警告 (Lua 5.4+)
warn = function (...)
end

-- @brief 原始相等 (不触发 __eq)
rawequal = function (v1, v2)
	return false
end

-- @brief 原始读表
rawget = function (t, index)
	return nil
end

-- @brief 原始长度 (# 语义, 不触发 __len)
rawlen = function (v)
	return 0
end

-- @brief 原始写表
rawset = function (t, index, value)
	return t
end

-- @brief 选取可变参 (index 可为 "#" 表示个数)
select = function (index, ...)
	return ...
end

-- @brief 设置元表 (禁止改 protected 元表)
setmetatable = function (t, mt)
	return t
end

-- @brief 转 number
-- @param [in] base[number(int)]	[可选] 2–36 进制
tonumber = function (v, base)
	return 0
end

-- @brief 转 string
tostring = function (v)
	return ''
end

-- @brief 类型名 ("nil"|"number"|"string"|"boolean"|"table"|"function"|"thread"|"userdata")
type = function (v)
	return 'nil'
end

-- @brief 保护调用 + 错误处理器 msgh
xpcall = function (f, msgh, ...)
	return true, ...
end
```

### 示例

```lua
-- 全局函数, 无需 require
print("Lua", _VERSION)

local ok, err = pcall(function ()
	error("boom")
end)
assert(not ok)

local t = { a = 1, b = 2 }
for k, v in pairs(t) do
	print(k, v)
end

local f, err = load("return 1 + 2", "=tmp", "t")
if f then
	print(f())  -- 3
end
```

### 注意

- klb 内 bundled `lpeg`/`cjson` 等经 `require` 加载; 基础库在 `klua_env_doinit` 第一步 `luaL_openlibs` 即就绪
- `load`/`loadfile`/`dofile` 在沙箱场景须限制 `env` 与路径
- 业务协程用 **`kco`**, 勿依赖全局 `coroutine` → [kco.md](../klua/kco.md)
