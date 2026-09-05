## JSON 编解码 (lua-cjson)

> **require**: `cjson`, `cjson.safe` | 代码: [klb/src_c/klua/lua-cjson-2.1.0/lua_cjson.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-cjson-2.1.0/lua_cjson.c)
> **文档样板**: bundled Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — 同 [ksys.md](../klua/ksys.md)
> **上游**: [lua-cjson](https://github.com/openresty/lua-cjson) 2.1.0 (MIT)

### 导出 API

| 符号 | 返回 | 说明 |
|------|------|------|
| `encode(value)` | string | Lua 值 → JSON 文本 |
| `decode(json)` | value | JSON 文本 → Lua 值 |
| `encode_sparse_array(convert, ratio, safe)` | boolean/integer×3 | 稀疏数组编码策略 (get/set) |
| `encode_max_depth(n)` | integer | 编码嵌套深度上限 (默认 1000) |
| `decode_max_depth(n)` | integer | 解码嵌套深度上限 (默认 1000) |
| `encode_number_precision(n)` | integer | 浮点输出精度 1–14 (默认 14) |
| `encode_keep_buffer(flag)` | boolean | 复用编码缓冲 |
| `encode_invalid_numbers(mode)` | string | `"off"` / `"on"` / `"null"` |
| `decode_invalid_numbers(flag)` | boolean | 是否接受 NaN/Infinity 等 |
| `new()` | table | 独立配置的 cjson 实例 |
| `null` | lightuserdata | JSON `null` 占位 (表内不可存 `nil`) |
| `_VERSION` | string | `"2.1.0"` |

**`cjson.safe`**: 与上表相同 API; `encode` / `decode` 失败时返回 `nil, err` (不抛异常).

### 伪代码

桩: [klb/bin/klbcore/help/lib3d/cjson.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/help/lib3d/cjson.lua)

```lua
--[[
-- Copyright (c) 2022, GNU GENERAL PUBLIC LICENSE Version 3
-- @file   cjson.lua
-- @brief  内置库 require("cjson") / require("cjson.safe")
--   \n C: ./klb/src_c/klua/lua-cjson-2.1.0/lua_cjson.c
-- @version 2.1.0
--]]

local cjson = {}


-- @brief JSON null 占位 (decode 时 JSON null → cjson.null)
cjson.null = nil	-- lightuserdata


-- @brief Lua 值编码为 JSON 字符串
-- @param [in] value[any]		table / array / string / number / boolean / cjson.null
-- @return [string] json
-- @note eg. cjson.encode({ a = 1, list = { 1, 2 } })
cjson.encode = function (value)
	return '{}'
end


-- @brief JSON 字符串解码为 Lua 值
-- @param [in] json[string]	JSON 文本
-- @return [any] value		通常为 table; JSON null → cjson.null
-- @note cjson: 失败抛异常; 可用 pcall(cjson.decode, s)
-- @note cjson.safe: 成功 value; 失败 nil, err
cjson.decode = function (json)
	return {}
end


-- @brief 稀疏数组编码策略 (传参 get/set, 省略则只读当前值)
-- @param [in] convert[boolean]	[可选] true 转 object; false 报错
-- @param [in] ratio[number(int)]	[可选] 稀疏阈值比, 默认 2
-- @param [in] safe[number(int)]	[可选] index ≤ safe 时仍用 array, 默认 10
-- @return [boolean] convert, [number(int)] ratio, [number(int)] safe
cjson.encode_sparse_array = function (convert, ratio, safe)
	return false, 2, 10
end


-- @brief 编码嵌套深度上限
-- @param [in] n[number(int)]	[可选] 1..; 省略则返回当前值
-- @return [number(int)] depth
cjson.encode_max_depth = function (n)
	return 1000
end


-- @brief 解码嵌套深度上限
-- @param [in] n[number(int)]	[可选] 1..; 省略则返回当前值
-- @return [number(int)] depth
cjson.decode_max_depth = function (n)
	return 1000
end


-- @brief 浮点数 JSON 输出有效位数
-- @param [in] n[number(int)]	[可选] 1..14
-- @return [number(int)] precision
cjson.encode_number_precision = function (n)
	return 14
end


-- @brief 是否复用内部编码缓冲
-- @param [in] flag[boolean|string]	[可选] true/"on" 或 false/"off"
-- @return [boolean] keep_buffer
cjson.encode_keep_buffer = function (flag)
	return true
end


-- @brief 编码时对 NaN/Infinity 的处理
-- @param [in] mode[string]	[可选] "off" | "on" | "null"
-- @return [string] mode
cjson.encode_invalid_numbers = function (mode)
	return 'off'
end


-- @brief 解码时是否接受非法数字
-- @param [in] flag[boolean|string]	[可选] true/"on" 或 false/"off"
-- @return [boolean] allow
cjson.decode_invalid_numbers = function (flag)
	return true
end


-- @brief 创建独立配置的 cjson 模块表 (互不影响 encode_* 选项)
-- @return [table] cjson 实例 (含上述 API)
cjson.new = function ()
	return {}
end


return cjson
```

### 示例

```lua
local cjson = require("cjson")
local safe = require("cjson.safe")

-- 基本编解码
local js = cjson.encode({ name = "demo", tags = { "a", "b" } })
local t = cjson.decode(js)

-- JSON null
local o = cjson.decode('{"k":null}')
if o.k == cjson.null then
	print("k is null")
end

-- 安全版 (网络/外部输入)
local data, err = safe.decode('not json')
if not data then
	print("decode failed:", err)
end

-- 独立配置实例
local cj = cjson.new()
cj.encode_sparse_array(true, 2, 10)
local sparse = cj.encode({ [100] = "x" })
```

### 注意

- 默认编入 klb; 无 `__KLB_NO_*__` 裁剪宏; 预加载见 [preload.md](../../klb/klua/design/preload.md)
- JSON `null` 解码为 `cjson.null` (lightuserdata), **不是** `nil`; 写入表时用 `cjson.null`
- `cjson.decode` 失败抛 Lua 异常; 外部数据优先 `cjson.safe` 或 `pcall`
- 与 klb 自带 JSON 序列化 [ksys](../klua/ksys.md) `pack_json` / `unpack_json` 用途不同 (后者为 env 传参)
- 协议: `klua_doc/klb/licenses/LICENSE-lua-cjson-2.1.0`
