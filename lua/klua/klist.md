## 跨线程链表 (klist)

> **require**: `klist` | 代码: `klb/src_c/klua/klua_multithread/klua_klist.c`
> **文档样板**: k* Lua API 四层 — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

进程内 **FIFO 链表**, 按命名路径共享; 元素为 **`klb_obj_t*`** (lightuserdata). 适合 worker 线程生产、主线程消费.

### 导出 API

#### 模块级

| 函数 | 说明 |
|------|------|
| `new(path)` | 创建/获取命名 klist 对象 |
| `push(path, cobj)` | 全局路径入队 |
| `pop(path)` | 全局路径出队 |

#### klist 对象 (`new` 返回)

| 方法 | 返回 | 说明 |
|------|------|------|
| `push(cobj)` | — | 入队 |
| `pop()` | lightuserdata / 无 | 出队 `klb_obj_t*` |
| `size()` | integer | 队列长度 |
| `clear()` | — | 清空 |

### 伪代码

桩: `klb/bin/klbcore/help/k/klist.lua`

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   klist.lua
-- @brief  C klist
--   \n require("klist")
--   \n C: klua_multithread/klua_klist.c
-- @note 数据只能取出一次; 适合 A 线程生产 B 线程消费
--]]

local klist = {}


-- @brief 按路径 push (全局命名队列)
-- @param [in] path[string]			eg. "/aaa/bbb"
-- @param [in] cobj[lightuserdata]	klb_obj_t*
klist.push = function (path, cobj)
	return
end


-- @brief 按路径 pop
-- @return [lightuserdata|nil] klb_obj_t*
klist.pop = function (path)
	return nil
end


-- @brief 创建命名 klist 对象
-- @param [in] path[string]
-- @return [userdata|table] klist 对象
klist.new = function (path)
	local o = {}

	o.push = function (cobj)
		return
	end

	o.pop = function ()
		return nil
	end

	o.size = function ()
		return 0
	end

	o.clear = function ()
		return
	end

	return o
end


return klist
```

### 示例

```lua
local klist = require("klist")

local q = klist.new("/media/queue")
-- worker: q:push(cobj)
-- main:   local obj = q:pop()
```

### 注意

- 元素须为 C **`klb_obj_t*`**; Lua 不负责生命周期, 消费方须按协议释放
- 与 **`kthread`** 配合; 见 [kthread.md](kthread.md)
- 同路径 `new` 返回同一逻辑队列 (C 注册表)
