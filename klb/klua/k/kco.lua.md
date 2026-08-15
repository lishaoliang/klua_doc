## 协程


### 伪代码

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   kco.lua
-- @brief  C kco
--   \n require("kco")
--   \n C导出文件: ./klb/src_c/klua/klua_base/klua_kcoro.c
--   \n 协程(coroutine)
--   \n 与lua自带的协程库冲突, 使用本库替代即可!
-- @version 0.1
--]]

local kco = {}

-- @brief fork(创建)一个新协程(coroutine)
-- @param [in]	func[function]		协程入口函数
-- @param [in]	...[任意]			参数列表
-- @return 无
-- @note eg. kco.fork(function (...) print('123456', ...) end, 'a', 'b')
kco.fork = function (func, ...)
	return
end


-- @brief 等待一段时间后, 启动一个新的协程(coroutine)
-- @param [in]	tc[number(int)]		等待时间(单位毫秒)
-- @param [in]	func[function]		协程入口函数
-- @param [in]	...[任意]			参数列表
-- @return 无
-- @note eg. kco.timeout(1000, function (...) print('123456', ...) end, 'a', 1)
kco.timeout = function (tc, func, ...)
	return
end


-- @brief 协程休眠一段时间
-- @param [in]	tc[number(int)]		休眠时间(单位毫秒)
-- @return 无
-- @note eg. kco.co_sleep(1000)
--  仅协程中使用
kco.co_sleep = function (tc)
	return
end

return kco
```

### 注意

- **禁止**使用 Lua 内置 `coroutine`; 见 [coroutine 设计](../design/coroutine.md)
- `kco.co_sleep` **仅**在协程内调用; 会 `yield`, 由 `klua_env_loop_once` 唤醒
- `kco.fork` / `kco.timeout` 可在主线程或协程外调用
- 与 `klpc:co_call` / `mo:co_recv` 配合做模块间异步 RPC; 见 [klpc](klpc.lua.md)
- `ktime.sleep` 阻塞 C 线程, 与 `kco.co_sleep` 语义不同; 见 [ktime](ktime.lua.md)

