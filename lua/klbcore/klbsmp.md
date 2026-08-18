## SMP 脚本层

> **require**: `klbcore.klbsmp` | 代码: `bin/klbcore/klbsmp/` | C 绑定 **`ksmp`** 见 [klua/readme.md](../klua/readme.md) 网络节
> **文档样板**: Lua API 四层 — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

SMP 协议 **纯 Lua 封装**; 底层 **`ksmp`** / **`kurl`**. 协议与 C 栈见 **klb-mnp-smp-design**.

### 导出 API

#### 模块级 (init.lua)

| 函数 | 返回 | 说明 |
|------|------|------|
| `new_client([cfg])` | client 对象 | 新建 SMP 客户端 |
| `connect(host, port)` | client 对象 | 快捷连接 |
| `new_listen([cfg])` | listen 对象 | 新建监听 |
| `listen(port [, cfg])` | listen 对象 | 快捷监听 |
| `new_client_rpc([cfg])` | rpc client 对象 | 新建 RPC 客户端 |
| `connect_rpc(host, port)` | rpc client 对象 | 快捷 RPC 连接 |
| `co_post_rpc(url, ...)` | string | 一次性 POST-RPC |
| `co_notify_rpc(url, ...)` | string | 一次性 NOTIFY-RPC |
| `co_call_rpc(url, ...)` | string, ... | 一次性 CALL-RPC |
| `new_listen_rpc([cfg])` | rpc listen 对象 | 新建 RPC 监听 |
| `listen_rpc(port [, cfg])` | rpc listen 对象 | 快捷 RPC 监听 |

#### 常量

| 名 | 值 | 说明 |
|----|-----|------|
| `RPC_LUA` / `RPC_JSON` | `'LUA'` / `'JSON'` | 载荷编码 |
| `RPC_POST` / `RPC_NOTIFY` / `RPC_REQUEST` / `RPC_RESPONSE` | 字符串 | RPC 方法 |

#### client 对象 (`smpclienter`)

| 方法 | 说明 |
|------|------|
| `connect(host, port)` | 连接; `0` 成功 |
| `disconnect()` | 断开 |
| `send_text` / `send_binary` / `send_media` | 发送 (实现中) |
| `co_recv_text` / `co_recv_binary` / `co_recv_media` | 协程接收 (实现中) |

#### listen 对象 (`smplistener`)

| 方法 | 说明 |
|------|------|
| `open(port)` | 开始监听 |
| `close()` | 关闭 |
| `co_accept()` | 协程接受连接 → `smpserver` 对象 |

#### rpc client 对象 (`smpclientrpcer`)

| 方法 | 说明 |
|------|------|
| `connect(host, port)` / `connect(url)` | 连接 |
| `disconnect()` | 断开 |
| `post(...)` / `notify(...)` | 发送 RPC |
| `co_call(...)` | 协程 CALL |
| `co_wait()` | 等待完成 |

#### rpc listen 对象

| 方法 | 说明 |
|------|------|
| `open(port)` / `close()` | 监听控制 |
| `co_accept()` | 接受连接 → `smpserverpcer` 对象 |

### 伪代码

源码: `bin/klbcore/klbsmp/init.lua` 及 `client/`、`serve/` 子模块

```lua
--[[
-- Copyright (c) 2025, GNU LESSER GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file   init.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  klbsmp init.lua
--   \n require("klbcore.klbsmp")
--   \n 源码: bin/klbcore/klbsmp/
-- @note SMP 协议脚本层; C 绑定 ksmp
-- @version 0.1
--]]

local klbsmp = {}


-- @brief 新建 SMP 客户端
-- @param [in] cfg[table]			[可选] 客户端配置
-- @return [table] client 对象
klbsmp.new_client = function (cfg)
	return {}
end


-- @brief 新建客户端并连接
-- @param [in] host[string]			主机
-- @param [in] port[number(int)]	端口
-- @return [table] client 对象
klbsmp.connect = function (host, port)
	local client = klbsmp.new_client()
	client:connect(host, port)
	return client
end


-- @brief 新建监听模块
-- @param [in] cfg[table]			[可选] 监听配置
-- @return [table] listen 对象
klbsmp.new_listen = function (cfg)
	return {}
end


-- @brief 新建监听并开始监听端口
-- @param [in] port[number(int)]	端口
-- @param [in] cfg[table]			[可选] 监听配置
-- @return [table] listen 对象
klbsmp.listen = function (port, cfg)
	local l = klbsmp.new_listen(cfg)
	l:open(port)
	return l
end


-- @brief 新建 RPC 客户端
-- @param [in] cfg[table]			[可选] 配置 (含 rpctype)
-- @return [table] rpc client 对象
klbsmp.new_client_rpc = function (cfg)
	return {}
end


-- @brief 新建 RPC 客户端并连接
-- @param [in] host[string]			主机
-- @param [in] port[number(int)]	端口
-- @return [table] rpc client 对象
klbsmp.connect_rpc = function (host, port)
	local client = klbsmp.new_client_rpc()
	client:connect_rpc(host, port)
	return client
end


-- @brief 一次性 POST-RPC (须在 kco 协程内)
-- @param [in] url[string]			eg. 'smprpc://user:pass@127.0.0.1:3457'
-- @param [in] ...					RPC 参数
-- @return [string] 'success'
klbsmp.co_post_rpc = function (url, ...)
	return 'success'
end


-- @brief 一次性 NOTIFY-RPC (须在 kco 协程内)
-- @param [in] url[string]			目标 URL
-- @param [in] ...					RPC 参数
-- @return [string] 'success'
klbsmp.co_notify_rpc = function (url, ...)
	return 'success'
end


-- @brief 一次性 CALL-RPC (须在 kco 协程内)
-- @param [in] url[string]			目标 URL
-- @param [in] ...					RPC 参数
-- @return [string] 'success', ...	回应数据
klbsmp.co_call_rpc = function (url, ...)
	return 'success', ...
end


-- @brief 新建 RPC 监听模块
-- @param [in] cfg[table]			[可选] 配置
-- @return [table] rpc listen 对象
klbsmp.new_listen_rpc = function (cfg)
	return {}
end


-- @brief 新建 RPC 监听并开始监听
-- @param [in] port[number(int)]	端口
-- @param [in] cfg[table]			[可选] 配置
-- @return [table] rpc listen 对象
klbsmp.listen_rpc = function (port, cfg)
	local l = klbsmp.new_listen_rpc(cfg)
	l:open(port)
	return l
end


-- 数据组织方式
klbsmp.RPC_LUA = 'LUA'
klbsmp.RPC_JSON = 'JSON'

-- RPC 方法
klbsmp.RPC_POST = 'POST'
klbsmp.RPC_NOTIFY = 'NOTIFY'
klbsmp.RPC_REQUEST = 'REQUEST'
klbsmp.RPC_RESPONSE = 'RESPONSE'


return klbsmp
```

**client 对象** (`smpclienter`):

```lua
-- @brief 连接到目标
-- @return [number(int)] 0 成功; 1 连接失败
function client:connect(host, port)
	return 0
end

-- @brief 断开连接
function client:disconnect()
	return
end

function client:send_text(...)
	-- @note 桩; 随 ksmp C 层补齐
	return
end

function client:send_binary(...)
	-- @note 桩; 随 ksmp C 层补齐
	return
end

function client:send_media(...)
	-- @note 桩; 随 ksmp C 层补齐
	return
end

-- @brief 协程接收文本 (须在 kco 协程内)
-- @return [string] 文本; 失败 ""
function client:co_recv_text()
	return ""
end

-- @brief 协程接收二进制 (须在 kco 协程内)
-- @return [string] 二进制串; 失败 ""
function client:co_recv_binary()
	return ""
end

-- @brief 协程接收媒体 (须在 kco 协程内)
-- @return [lightuserdata] 媒体指针等; 桩
function client:co_recv_media()
	return nil
end
```

**listen 对象** (`smplistener`):

```lua
-- @brief 开始监听端口
function listen:open(port)
	return
end

-- @brief 关闭监听
function listen:close()
	return
end

-- @brief 协程接受新连接 → smpserver 对象
function listen:co_accept()
	return {}
end
```

**rpc client 对象** (`smpclientrpcer`):

```lua
-- @brief 按 host/port 连接 RPC
-- @return [number(int)] 0 成功; 1 失败
function rpc_client:connect_rpc(host, port)
	return 0
end

-- @brief 按 URL 连接 RPC (schema smprpc/smprpcs)
function rpc_client:connect(url)
	return 0
end

function rpc_client:disconnect()
	return
end

-- @brief 发送 POST-RPC
-- @param [in] ...					RPC 参数 (LUA 直传或 JSON 编码)
-- @return [number(int)] 0 成功; 非 0 失败
function rpc_client:post(...)
	return 0
end

-- @brief 发送 NOTIFY-RPC
-- @param [in] ...					RPC 参数
-- @return [number(int)] 0 成功; 非 0 失败
function rpc_client:notify(...)
	return 0
end

-- @brief 协程 CALL-RPC (须在 kco 协程内)
-- @param [in] ...					RPC 参数
-- @return [...] 对方回复数据
function rpc_client:co_call(...)
	return ...
end

-- @brief 协程等待 RPC 发送完成 (须在 kco 协程内)
-- @return 无
function rpc_client:co_wait()
	return
end
```

**rpc listen 对象**:

```lua
-- @brief 开始 RPC 监听端口
function rpc_listen:open(port)
	return
end

-- @brief 关闭 RPC 监听
function rpc_listen:close()
	return
end

-- @brief 协程接受 RPC 连接 → smpserverpcer 对象
-- @return [table] smpserverpcer; 失败 nil
function rpc_listen:co_accept()
	return {}
end
```

### 示例

```lua
local kco = require("kco")
local klbsmp = require("klbcore.klbsmp")

-- 服务端
kco.fork(function ()
	local l = klbsmp.listen(3456)
	while true do
		local srv = l:co_accept()
		if srv then
			-- 处理连接…
		end
	end
end)

-- RPC 一次性调用
local msg, data = klbsmp.co_call_rpc(
	'smprpc://127.0.0.1:3457',
	'get_status'
)
```

### 注意

- IO 须在 **`kco` 协程**内 (`co_accept` / `co_call_rpc` 等)
- **`ksmp`** C 绑定仍为 **待定** 文档; 以源码与 **klb-mnp-smp-design** 为准
- 客户端 `send_*` / `co_recv_*` 部分为桩, 随 C 层补齐
- 测试脚本: `bin/klbcore/klbsmp/test/test_*.lua`
