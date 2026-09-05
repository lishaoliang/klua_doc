## 内置协程 (coroutine)

> **require**: `coroutine` | 代码: [klb/src_c/klua/lua-5.4.6/src/lcorolib.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-5.4.6/src/lcorolib.c) (`luaopen_coroutine`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.2](https://www.lua.org/manual/5.4/manual.html#6.2)

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `create(f)` | thread | 新建协程 (未运行) |
| `resume(co [, ...])` | ok, `...` | 启动/恢复协程 |
| `yield(...)` | `...` | 挂起当前协程 |
| `status(co)` | string | `"running"`/`"suspended"`/`"normal"`/`"dead"` |
| `running()` | thread, boolean | 当前协程; 第二值表主线程 |
| `wrap(f)` | function | 包装为可多次调用的函数 |
| `isyieldable()` | boolean | 当前上下文可否 yield |
| `close(co)` | ok, `...` | 关闭协程 (Lua 5.4+) |

### 伪代码

真源: `lcorolib.c` — `co_funcs[]`

```lua
--[[
-- @file   coroutine.lua (伪代码)
-- @brief  Lua 5.4 coroutine 库
--   \n 代码: ./klb/src_c/klua/lua-5.4.6/src/lcorolib.c
--]]

local coroutine = {}


-- @brief 创建协程 (不立即执行)
-- @param [in] f[function]	协程体
-- @return [thread] co
coroutine.create = function (f)
	return {}
end


-- @brief 恢复/启动协程
-- @param [in] co[thread]
-- @param [in] [...]	[可选] 传给协程或 yield 返回值
-- @return [boolean] ok, [...] 返回值或错误
coroutine.resume = function (co, ...)
	return true, ...
end


-- @brief 挂起 (仅协程内)
-- @return [...] 传给 resume 的返回值
coroutine.yield = function (...)
	return ...
end


-- @brief 协程状态
-- @return [string] "running"|"suspended"|"normal"|"dead"
coroutine.status = function (co)
	return 'suspended'
end


-- @brief 当前运行的协程
-- @return [thread|nil] co, [boolean] ismain
coroutine.running = function ()
	return nil, true
end


-- @brief 包装协程为函数 (内部 resume)
-- @return [function] 每次调用 resume 一次
coroutine.wrap = function (f)
	return function () end
end


-- @brief 当前是否可 yield
-- @return [boolean]
coroutine.isyieldable = function ()
	return false
end


-- @brief 关闭协程 (未结束则运行到结束或 error)
-- @return [boolean] ok, [...]
coroutine.close = function (co)
	return true
end

return coroutine
```

### 示例

```lua
-- 仅作 Lua VM 语义说明; klb 业务勿用
local co = coroutine.create(function ()
	print("start")
	local v = coroutine.yield(42)
	print("after yield", v)
end)

local ok, n = coroutine.resume(co)
print(ok, n)  -- true  42

coroutine.resume(co, "hello")
```

### 注意

- **klb 业务脚本禁止使用内置 `coroutine`**; IO/网络须用 **`kco`** → [kco.md](../klua/kco.md), [guide/coroutine.md](../klua/guide/coroutine.md)
- klua env / kthread 与内置协程栈模型不同; 混用易导致不可预期行为
- 库仍随 `luaL_openlibs` 注册, 供 VM 内部与调试; 新代码请 `require("kco")`
