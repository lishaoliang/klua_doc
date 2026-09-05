## H.26x 文件读取 (kh26x)

> **require**: `kh26x` | 代码: [klb/src_c/klua/klua_format/klua_kh26x.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua_format/klua_kh26x.c)
> **文档样板**: k* Lua API 四层 — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

读取 **H.264/H.265** 裸流文件, 按帧返回 **`klb_buf_t*`** (lightuserdata). **P2** 模块.

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `load(path)` | h26x 对象 | 打开文件 |

#### h26x 对象

| 方法 | 返回 | 说明 |
|------|------|------|
| `read()` | cobj1, cobj2 | 读帧; 两路缓冲 (见 C 实现) |
| `size()` | integer | 帧总数 |
| `close()` | — | 关闭 |

### 伪代码

桩: [klb/bin/klbcore/help/k/kh26x.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/help/k/kh26x.lua)

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   kh26x.lua
-- @brief  C kh26x — 读取 h264/h265 文件
--   \n require("kh26x")
--   C: klua_format/klua_kh26x.c
--]]

local kh26x = {}


-- @brief 加载 h26x 文件
-- @param [in] path[string]			文件路径
-- @return [userdata|table] h26x 对象
kh26x.load = function (path)
	local h = {}

	-- @return [lightuserdata|nil] cobj1, [lightuserdata|nil] cobj2  (klb_buf_t*)
	h.read = function ()
		return nil, nil
	end

	h.size = function ()
		return 0
	end

	h.close = function ()
		return
	end

	return h
end


return kh26x
```

### 示例

```lua
local kh26x = require("kh26x")

local f = kh26x.load("test.h264")
print("frames", f:size())

local b1, b2 = f:read()
-- 处理 klb_buf_t* ...
f:close()
```

### 注意

- 返回 **C 缓冲指针**; 须按 **klbformat** / 媒体栈约定释放或转交
- 与 **klbnet** 媒体发送配合时见 **klb-net-design**
- 裁剪/可用性以产品 `klua_loadlib` 为准
