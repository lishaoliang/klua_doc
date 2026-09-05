## 平台信息

> **require**: `kos` | 代码: [klb/src_c/klua/klua_platform/klua_kos.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua_platform/klua_kos.c)
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

### 导出 API

只读字段 (无函数):

| 字段 | 类型 | 说明 |
|------|------|------|
| `os` | string | 操作系统名 |
| `arch` | string | CPU 架构 |

### 伪代码

桩: [klb/bin/klbcore/help/k/kos.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/help/k/kos.lua)

```lua
--[[
-- Copyright(c) 2020, LGPL All Rights Reserved
-- @file   kos.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  C kos
--   \n require("kos")
--   \n C导出文件: ./klb/src_c/klua/klua_platform/klua_kos.c
-- @version 0.1
--]]

local kos = {}

-- @name   kos.os
-- @export 操作系统名称
kos.os = 'linux' -- 'linux', 'windows', 'apple', 'unix'

-- @name   kos.arch
-- @export 芯片架构
kos.arch = 'amd64'

return kos
```

### 示例

```lua
local kos = require("kos")

print("os", kos.os)
print("arch", kos.arch)

if kos.os == "windows" then
    -- Windows 分支
elseif kos.os == "linux" then
    -- Linux 分支
end
```

### 注意

- 只读字段, 编译期/启动期确定
- 用于脚本内平台分支; 路径/IO 仍推荐 [kenv](kenv.md) `base_path`
