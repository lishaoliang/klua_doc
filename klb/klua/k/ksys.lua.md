## 系统控制与序列化

> 代码: `klb/src_c/klua/klua_util/klua_ksys.c`

### 伪代码

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   ksys.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  C ksys
--   \n require("ksys")
--   \n C导出文件: ./klb/src_c/klua/klua_util/klua_ksys.c
-- @version 0.1
--]]

local ksys = {}


-- @brief 退出当前环境 lua_State(klua_env_t)
-- @return 无
ksys.exit = function ()
	return
end


-- @brief 获取是否退出当前环境
-- @return [boolean] true.退出; false.未退出
ksys.is_exit = function ()
	return false
end


-- @brief 将函数参数打包成一个字符串(二进制)
-- @param [in] [...]		任意类型
-- @return [string]	打包后的字符串(二进制)
-- @note eg. local s = ksys.pack_string('a', true, {a=1})
ksys.pack_string = function (...)
	return ''
end


-- @brief 将打包的字符串(二进制) 解包
-- @param [in] s[string]	打包的字符串
-- @return [...] 任意类型
-- @note eg. local a, b, c = ksys.unpack('')
ksys.unpack = function (s)
	return ...
end


-- @brief 将函数参数打包成JSON字符串
-- @param [in] [...]		任意类型
-- @return [string]	打包后的字符串
-- @note eg. local s = ksys.pack_json('a', true, {a=1})
ksys.pack_json = function (...)
	return '{}'
end


-- @brief 将打包的JSON字符串解包
-- @param [in] s[string]	打包的字符串
-- @return [...] 任意类型
ksys.unpack_json = function (s)
	return ...
end


-- @brief 获取 当前环境 lua_State(klua_env_t) 的全局参数
-- @return [...] 任意类型
-- @note 来源于 kthread.start() 的第4个参数开始
--  eg. local a, b, c = ksys.get_args()
ksys.get_args = function ()
	return ...
end


return ksys
```

### 注意

- `pack_string` / `unpack` 为 klb 二进制序列化 (luaseri); 跨 LPC/线程传参常用
- `get_args` 与 [kenv](kenv.lua.md) `get_args` 相同
- `exit` 触发 [lifecycle](../design/lifecycle.md) 中 env 退出流程
