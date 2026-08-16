## 随机数

> **require**: `krand` | 代码: `klb/src_c/klua/klua_util/klua_krand.c`
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `rand([max])` | `n, r` | 随机整数与范围上限 |
| `rand_string([len])` | string | 随机字母数字串 |
| `rand_integer([len])` | integer | 随机整型数值 |

### 伪代码

桩: `klb/bin/klbcore/help/k/krand.lua`

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   krand.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  C krand
--   \n require("krand")
--   \n C导出文件: ./klb/src_c/klua/klua_util/klua_krand.c
-- @version 0.1
--]]

local krand = {}



-- @brief 随机值
-- @param [in]  	max[number(int)]	[可选](默认0x7fff)值最大
-- @return [number(int)] 随机值
--			[number(int)] 范围[1, N]
-- @note eg. 1. local n, r = krand.rand(10)		1 <= n <= r, r = 10
-- 		2. local n, r = krand.rand()			1 <= n <= r, r = 0x7fff
krand.rand = function (max)
	return 5, 9
end


-- @brief 随机字符串
-- @param [in]  	len[number(int)]	字符个数
-- @return [string] 字符串
krand.rand_string = function (len)
	return '123456'
end


-- @brief 随机整型数值
-- @param [in]  	len[number(int)]	字符个数: eg. 3 ==> 123
-- @return [number(int)] 整型数值
krand.rand_integer = function (len)
	return 123456
end


return krand
```

### 示例

```lua
local krand = require("krand")

local n, r = krand.rand(10)
print("rand", n, r)  -- 1 <= n <= r, r = 10

local s = krand.rand_string(16)
print("str", s)

local id = krand.rand_integer(6)
print("id", id)
```

### 注意

- `rand` 返回两个值: 随机数与范围上限 N
- `klbui` 生成虚拟控件 path 等场景常用 `krand.rand_string`
