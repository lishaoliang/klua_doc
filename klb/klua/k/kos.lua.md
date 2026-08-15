## 平台信息

> 代码: `klb/src_c/klua/klua_platform/klua_kos.c`

### 伪代码

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

### 注意

- 只读字段, 编译期/启动期确定
- 用于脚本内平台分支; 路径/IO 仍推荐 [kenv](kenv.lua.md) `base_path`
