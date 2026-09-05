## klpc 脚本扩展

> **require**: `klbcore.base.klpcex` | 代码: [klb/bin/klbcore/base/klpcex.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/base/klpcex.lua) | C 绑定 **`klpc`** 见 [klua/klpc.md](../../klua/klpc.md)
> **文档样板**: Lua API 四层 — [k-bindings.md](../../../klb/klua/design/k-bindings.md) § Lua API 文档

跨 **`klua_env`** 调用本地模块方法的 **便捷封装**; 内部 `klpc.new()` → `co_call` → `close`.

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `call(mo_name, ...)` | ... | 协程内调用目标 env 模块方法 |

### 伪代码

源码: [klb/bin/klbcore/base/klpcex.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/base/klpcex.lua)

```lua
--[[
-- Copyright (c) 2022, GNU GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file  klpcex.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief klpc extented
--   \n require("klbcore.base.klpcex")
--   \n C 绑定: klpc (klua_klpc.c)
-- @version 0.1
--]]

local klpcex = {}


-- @brief 调用本地模块提供的方法 (可跨线程 Lua 环境)
-- @param [in] mo_name[string]		模块名称
-- @param [in] ...					参数数据
-- @return [...]					模块回复的数据
-- @note 仅在 kco 协程中使用; 内部 klpc.new → co_call → close
klpcex.call = function (mo_name, ...)
	return ...
end


return klpcex
```

### 示例

```lua
local kco = require("kco")
local klpcex = require("klbcore.base.klpcex")

kco.fork(function ()
	local ok, data = klpcex.call('web.handler', 'get_config', 'id')
	print(ok, data)
end)
```

### 注意

- **须在 `kco` 协程内**; 底层为 [klpc](../../klua/klpc.md) `co_call`
- 跨线程只传 **可序列化值** — **klbcore-design** § kthread
- 完整 klpc API (注册/广播等) → [klua/klpc.md](../../klua/klpc.md)
