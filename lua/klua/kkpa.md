## 键值包文件 (kkpa)

> **require**: `kkpa` | 代码: `klb/src_c/klua/klua_base/klua_kpackage.c`
> **文档样板**: k* Lua API 四层 — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

简单 **K:V 打包文件** 读写 (资源包/配置包). **P2** 模块, API 以源码为准.

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `open_w(path)` | kpa 对象 | 创建/打开可写包 |
| `open_r(path)` | kpa 对象 | 打开只读包 |

#### kpa 对象 (写)

| 方法 | 返回 | 说明 |
|------|------|------|
| `write(k, v)` | rc | 写入键值 |
| `write_file(k, path)` | rc | 键对应文件内容 |
| `close()` | — | 关闭 (亦可 GC) |

#### kpa 对象 (读)

| 方法 | 返回 | 说明 |
|------|------|------|
| `size()` | integer | 条目数 |
| `read(idx)` | k, v | 按 **1-based** 序号读 |
| `close()` | — | 关闭 |

### 伪代码

桩: `klb/bin/klbcore/help/k/kkpa.lua`

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   kkpa.lua
-- @brief  C kkpa / require("kkpa")
--   C: klua_base/klua_kpackage.c
--]]

local kkpa = {}


-- @brief 以写方式打开包文件
-- @param [in] path[string]
-- @return [userdata|table] kpa
function kkpa.open_w(path)
	local kpa = {}

	kpa.close = function ()
		return
	end

	-- @return [number(int)] 0 成功; 非 0 错误码
	kpa.write = function (k, v)
		return 0
	end

	kpa.write_file = function (k, fpath)
		return 0
	end

	return kpa
end


-- @brief 以读方式打开包文件
function kkpa.open_r(path)
	local kpa = {}

	kpa.close = function ()
		return
	end

	kpa.size = function ()
		return 0
	end

	-- @param [in] idx[number(int)]	1..size()
	-- @return [string|nil] k, [string|nil] v
	kpa.read = function (idx)
		return "", ""
	end

	return kpa
end


return kkpa
```

### 示例

```lua
local kkpa = require("kkpa")

local w = kkpa.open_w("bundle.kpa")
w:write("version", "1")
w:write_file("logo", "logo.png")
w:close()

local r = kkpa.open_r("bundle.kpa")
for i = 1, r:size() do
	local k, v = r:read(i)
	print(k, v)
end
r:close()
```

### 注意

- 记名 **kkpa** / require **`kkpa`**; 与 `kpa_*` 扩展包 (**待定**) 不同
- 显式 **`close()`** 可提前释放文件句柄; 否则依赖 GC
- 读序号 **1-based**, 与 Lua 惯例一致
