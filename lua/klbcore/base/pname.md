## 协议名常量

> **require**: `klbcore.base.pname` | 代码: [klb/bin/klbcore/base/pname.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/base/pname.lua)
> **文档样板**: Lua API 四层 — [k-bindings.md](../../../klb/klua/design/k-bindings.md) § Lua API 文档

网络协议 **字符串常量** 表; 与 knet 命名对齐. 无函数 API.

### 导出 API

| 常量 | 值 | 说明 |
|------|-----|------|
| `UNKOWN` | `'UNKOWN'` | 未知 |
| `MNP` / `MNPS` | `'MNP'` / `'MNPS'` | MNP (TLS) |
| `RTMP` | `'RTMP'` | RTMP |
| `RTSP` | `'RTSP'` | RTSP |
| `HTTP` / `HTTPS` | `'HTTP'` / `'HTTPS'` | HTTP (TLS) |
| `HTTPMNP` / `HTTPFLV` | `'HTTP-MNP'` / `'HTTP-FLV'` | HTTP 扩展 |
| `WS` / `WSS` | `'WS'` / `'WSS'` | WebSocket (TLS) |
| `WSMNP` / `WSFLV` | `'WS-MNP'` / `'WS-FLV'` | WebSocket 扩展 |

### 伪代码

源码: [klb/bin/klbcore/base/pname.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/base/pname.lua)

```lua
--[[
-- @file   pname.lua
-- @brief  协议名字符串常量 (from knet)
--   \n require("klbcore.base.pname")
-- @note 无函数 API; 只读常量表
--]]

local pname = {}

pname.UNKOWN = 'UNKOWN'
pname.MNP = 'MNP'
pname.MNPS = 'MNPS'
pname.RTMP = 'RTMP'
pname.RTSP = 'RTSP'
pname.HTTP = 'HTTP'
pname.HTTPS = 'HTTPS'
pname.HTTPMNP = 'HTTP-MNP'
pname.HTTPFLV = 'HTTP-FLV'
pname.WS = 'WS'
pname.WSS = 'WSS'
pname.WSMNP = 'WS-MNP'
pname.WSFLV = 'WS-FLV'

return pname
```

### 示例

```lua
local pname = require("klbcore.base.pname")

if schema == pname.RTSP then
	-- RTSP 分支
end
```

### 注意

- 只读常量; **`UNKOWN`** 拼写与源码一致 (非 `UNKNOWN`)
- 协议栈实现见 **klb-net-design** / **klb-mnp-smp-design**
