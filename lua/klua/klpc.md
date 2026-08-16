## LPC 本地过程调用

> **require**: `klpc` | 代码: `klb/src_c/klua/klua_base/klua_klpc.c`
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

### 导出 API

#### 模块函数

| 函数 | 返回 | 说明 |
|------|------|------|
| `new_module(name)` | module userdata | 注册 LPC 服务端 |
| `new()` | lpc userdata | 创建 LPC 客户端 |
| `post(mo_name, ...)` | boolean | 投递消息 |

#### module userdata 方法

| 方法 | 返回 | 说明 |
|------|------|------|
| `close()` | — | 关闭模块 |
| `status()` | boolean | 是否在运行 |
| `co_recv()` | `src, ...` | **须协程**; 收消息 |
| `response(name, ...)` | boolean | 回复请求 |
| `notify(name, ...)` | boolean | 单向通知 |

#### lpc userdata 方法

| 方法 | 返回 | 说明 |
|------|------|------|
| `close()` | — | 关闭客户端 |
| `status()` | boolean, integer | 状态 + 状态码 |
| `post(mo_name, ...)` | boolean | 投递消息 |
| `co_call(mo_name, ...)` | `...` | **须协程**; 同步 RPC |
| `co_recv_notify()` | `...` | **须协程**; 收 notify |

### 伪代码

桩: `klb/bin/klbcore/help/k/klpc.lua`

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   klpc.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  C klpc, Local Procedure Call Protocol
--   \n require("klpc")
--   \n C导出文件: ./klb/src_c/klua/klua_base/klua_klpc.c
--   \n 本地夸线程通信
--   \n 定义模块, 访问模块
-- @version 0.1
--]]

local klpc = {}


-- @brief 新建一个导出模块(newmetatable)
-- @param [in]	name[string]		模块名称(不可重复)
-- @return module对象
-- @note 
klpc.new_module = function (name)
	local mo = {}
	
	-- @brief 关闭
	-- @return 无
	-- @note 显示关闭, 可提前释放非Lua相关的资源(内存, 文件句柄等)
	--		不显示关闭, 则需要等待gc才释放
	mo:close = function ()
		return
	end

	-- @brief 获取当前状态
	-- @return 	b[boolean]					是否正常
	--			status[number(int)]			状态码; 0.正常; 非0.错误码
	mo:status = function ()
		return true, 0
	end
	
	-- @brief 读取(LPC)消息
	-- @return [string]name		请求(call/post)来源名称
	--			[任意]...		消息列表
	-- @note 仅在协程中使用
	--		eg. local src, msg = mo:co_recv()
	--			...
	--			mo:response(src, '123', true)
	mo:co_recv = function ()
		local name = 'efse2'
		return name, ...
	end

	-- @brief 回复数据
	-- @param [in]	name[string]	目标名称
	-- @param [in]	...[任意]		消息
	-- @return 无
	-- @note 由 co_recv 函数获取后, 回应 response
	mo:response = function (name, ...)
		return
	end
	
	-- @brief 通知消息
	-- @param [in]	name[string]	目标名称
	-- @param [in]	...[任意]		消息
	-- @return 无
	-- @note 
	mo:notify = function (name, ...)
		return
	end
	
	return mo
end


-- @brief 新建一个lpc对象
-- @return lpc对象
klpc.new = function ()
	local lpc = {}

	-- @brief 关闭
	-- @return 无
	-- @note 显示关闭, 可提前释放非Lua相关的资源(内存, 文件句柄等)
	--		不显示关闭, 则需要等待gc才释放
	lpc:close = function ()
		return
	end

	-- @brief 获取当前状态
	-- @return 	b[boolean]					是否正常
	--			status[number(int)]			状态码; 0.正常; 非0.错误码
	lpc:status = function ()
		return true, 0
	end

	-- @brief 向某个模块post消息(不等待返回)
	-- @param [in]	mo_name[string]	目标模块名称
	-- @return [boolean] 是否成功
	lpc:post = function (mo_name, ...)
		return true
	end	

	-- @brief 向某个模块发送消息,并等待返回数据
	-- @param [in]	mo_name[string]	目标模块名称
	-- @param [in]	...[任意类型]	请求数据
	-- @return [...] 对方回复的数据
	-- @note 仅在协程中使用
	lpc:co_call = function (mo_name, ...)
		return ...
	end

	-- @brief 接收 'notify' 通知消息
	-- @return [...] 通知消息
	-- @note 需要初始化时开启通知接收, 仅在协程中使用
	lpc:co_recv_notify = function ()
		return ...
	end
	
	return lpc
end


-- @brief 向某个模块post消息(不等待返回)
-- @param [in]	mo_name[string]	目标模块名称
-- @return [boolean] 是否成功
klpc.post = function (mo_name, ...)
	return true
end


return klpc
```

### 示例

```lua
local kco = require("kco")
local klpc = require("klpc")

-- 服务端: 注册模块并在协程中处理
local mo = klpc.new_module("demo.svc")
kco.fork(function ()
    while mo:status() do
        local src, cmd, arg = mo:co_recv()
        if cmd == "ping" then
            mo:response(src, "pong", arg)
        end
    end
end)

-- 客户端: 同步 RPC (须协程)
local lpc = klpc.new()
kco.fork(function ()
    local ok, st = lpc:status()
    if ok then
        local reply = lpc:co_call("demo.svc", "ping", 123)
        print("reply", reply)
    end
end)

-- 单向投递 (无需协程)
klpc.post("demo.svc", "log", "hello")
```

### 注意

- 服务端: `klpc.new_module(name)` 注册模块; 在 `kco.fork` 内 `mo:co_recv()` 循环处理
- 客户端: `klpc.new()` 或模块级 `klpc.post`; 同步 RPC 用 `lpc:co_call` (**须协程**)
- `module:status()` 仅返回运行 boolean; `lpc:status()` 返回 boolean + 状态码
- 跨 **klua_env** (含 `kthread` worker) 经 C 消息队列; env 扩展 `_KLUA_EX_LPC_`
- 与 [kco](kco.md) 成对使用; 见 [coroutine 设计](../guide/coroutine.md)
