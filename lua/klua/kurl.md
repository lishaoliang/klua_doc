## URL 解析

> **require**: `kurl` | 代码: [klb/src_c/klua/klua_net/klua_kurl.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua_net/klua_kurl.c)
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

基于 `http_parser` 解析 URL 字符串; **只读**, 无网络 IO. 被 **klbsmp** / **klbrtsp** 等脚本层用于连接串解析.

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `parse(url)` | table | 解析成功返回字段表; 失败返回 `{}` |

**`parse` 结果表** (仅 URL 中存在的字段会出现):

| 字段 | 类型 | 说明 |
|------|------|------|
| `schema` | string | 方案, eg. `http`, `rtsp`, `smprpc` |
| `host` | string | 主机名或 IP |
| `port` | string | 端口 (字符串形式) |
| `path` | string | 路径, 含前导 `/` |
| `query` | string | 查询串 (不含 `?`) |
| `fragment` | string | 片段 (不含 `#`) |
| `userinfo` | string | 用户名密码, eg. `user:pass` |

### 伪代码

桩: [klb/bin/klbcore/help/k/kurl.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/help/k/kurl.lua)

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   kurl.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  C kurl
--   \n require("kurl")
--   \n C导出文件: ./klb/src_c/klua/klua_net/klua_kurl.c
-- @version 0.1
--]]

local kurl = {}


-- @brief 解析 URL 字符串
-- @param [in] url[string]			完整 URL
-- @return [table]					成功: 含已解析字段; 失败: {}
-- @note eg. kurl.parse("http://user:pass@127.0.0.1:8080/path?q=1#frag")
--   仅存在的组件写入表; 解析失败返回空表 {}
kurl.parse = function (url)
	return {
		schema = "http",					-- [可选] string    方案
		host = "127.0.0.1",					-- [可选] string    主机
		port = "8080",						-- [可选] string    端口
		path = "/path",						-- [可选] string    路径
		query = "q=1",						-- [可选] string    查询 (无 ?)
		fragment = "frag",					-- [可选] string    片段 (无 #)
		userinfo = "user:pass",				-- [可选] string    用户信息
	}
end


return kurl
```

### 示例

```lua
local kurl = require("kurl")

local u = kurl.parse("smprpc://127.0.0.1:3457/rpc")
if u.host then
	print(u.schema, u.host, u.port)
end

local u2 = kurl.parse("rtsp://192.168.1.1:554/live/stream")
print(u2.host, u2.port or "554")
```

### 注意

- **无** IO; 仅字符串解析. 网络连接见 **klb-net-design** / **klbcore-net-design**
- 解析失败返回 **空表 `{}`**, 非 `nil`; 调用方须检查 `u.host` 等键
- `port` 为 **string** (与 C `http_parser` 切片一致); 数值比较前 `tonumber`
- **klbsmp** `connect(url)`、`klbrtsp` 等内部使用本模块 — 见 [klbsmp.md](../klbcore/klbsmp.md)
- C 栈与传输: **klb-net-design**; 本模块为 L3 工具绑定
