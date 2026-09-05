## 系统控制与序列化

> **require**: `ksys` | 代码: [klb/src_c/klua/klua_util/klua_ksys.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua_util/klua_ksys.c)
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `version()` | string, integer, integer | klb 版本串/号/日期 |
| `exit()` | — | 退出当前 env |
| `is_exit()` | boolean | 是否已退出 |
| `pack_string(...)` | string | luaseri 二进制打包 |
| `unpack(s)` | `...` | luaseri 解包 |
| `pack_json(...)` | string | JSON 打包 |
| `unpack_json(s)` | `...` | JSON 解包 |
| `get_args()` | `...` | env 全局参数 (同 `kenv.get_args`) |

### 伪代码

桩: [klb/bin/klbcore/help/k/ksys.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/help/k/ksys.lua)

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


-- @brief klb 库版本
-- @return [string] 版本串, [number(int)] 版本号, [number(int)] 日期
ksys.version = function ()
    return "0.0.0", 0, 0
end


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

### 示例

```lua
local ksys = require("ksys")

local ver, num, date = ksys.version()
print("klb", ver, num, date)

-- 二进制序列化 (LPC/线程传参)
local bin = ksys.pack_string("a", true, { a = 1 })
local a, b, c = ksys.unpack(bin)

-- JSON 序列化
local js = ksys.pack_json("x", { k = "v" })
local x, t = ksys.unpack_json(js)

-- 读取 kthread.start 传入参数
local arg1, arg2 = ksys.get_args()
```

### 注意

- `pack_string` / `unpack` 为 klb 二进制序列化 (luaseri); 跨 LPC/线程传参常用
- `get_args` 与 [kenv](kenv.md) `get_args` 相同
- `exit` 触发 [lifecycle](../../klb/klua/design/lifecycle.md) 中 env 退出流程
