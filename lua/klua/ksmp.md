## SMP 传输 (ksmp)

> **require**: `ksmp` | 代码: `klb/src_c/klua/klua_net/klua_ksmp.c`, `klua_ksmp_serve.c`
> **文档样板**: k* Lua API 四层 — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

SMP 协议 **C 传输层** (L3); 脚本封装见 [klbsmp.md](../klbcore/klbsmp.md). C 栈 **klb-net-design** / **klb-mnp-smp-design**.

### 导出 API

#### 模块级

| 函数 | 返回 | 说明 |
|------|------|------|
| `connect(host, port)` | client / `nil` | SMP 客户端连接 |
| `connect_rpc(host, port)` | rpc_client | SMP-RPC 客户端 (失败时 userdata 内无有效连接) |
| `listen(port)` | listen | 监听 SMP 连接 |
| `listen_rpc(port)` | rpc_listen | 监听 SMP-RPC 连接 |

#### client (SMP 客户端 userdata)

| 方法 | 说明 |
|------|------|
| `disconnect()` | 关闭连接 |
| `send_text(text)` | 发送文本; 返回错误码 |
| `co_recv()` | **须 kco 协程**; 收 `"text"`+body 或 `"media"`+lightuserdata |

#### rpc_client (SMP-RPC 客户端 userdata)

| 方法 | 说明 |
|------|------|
| `disconnect()` | 关闭 |
| `post(rpctype, ...)` / `notify(rpctype, ...)` | 发送 RPC; 返回错误码 |
| `co_recv()` | **须协程**; 收 POST/NOTIFY 载荷 (解包为多返回值) |
| `co_call(rpctype, ...)` | **须协程**; CALL 并等待 RESPONSE |
| `co_wait()` | **须协程**; 等待发送缓冲刷入 socket |
| `status()` | 当前 RPC 状态表 |

#### serve (accepted SMP 连接)

| 方法 | 说明 |
|------|------|
| `disconnect()` | 关闭 |
| `send_text(text)` / `send_media(ptr)` | 发送 |
| `co_recv()` | **须协程**; 同 client |

#### listen / rpc_listen

| 方法 | 说明 |
|------|------|
| `close()` | 停止监听 |
| `co_accept()` | **须协程**; 接受连接 → serve / serverpc |

#### serverpc (accepted RPC 连接)

| 方法 | 说明 |
|------|------|
| `disconnect()` | 关闭 |
| `post` / `notify` / `response` | 发送 RPC |
| `co_recv()` | **须协程** |
| `status()` | 状态表 |

### 伪代码

真源: `klua_ksmp.c`, `klua_ksmp_serve.c` — `luaL_Reg`

```lua
--[[
-- @file   ksmp.lua (伪代码)
-- @brief  require("ksmp")
--   C: klua_net/klua_ksmp.c, klua_ksmp_serve.c
--]]

local ksmp = {}


-- @brief 连接 SMP 服务端
-- @param [in] host[string]			主机或 IP
-- @param [in] port[number(int)]	端口
-- @return [userdata|nil] client	成功; 失败 nil
ksmp.connect = function (host, port)
	return nil
end


-- @brief 连接 SMP-RPC 服务端
-- @return [userdata] rpc_client
ksmp.connect_rpc = function (host, port)
	return {}
end


-- @brief 监听 SMP 端口
-- @param [in] port[number(int)]
-- @return [userdata] listen
ksmp.listen = function (port)
	return {}
end


-- @brief 监听 SMP-RPC 端口
ksmp.listen_rpc = function (port)
	return {}
end


return ksmp
```

**client**:

```lua
-- @brief 关闭
function client:disconnect()
	return
end

-- @brief 发送文本
-- @return [number(int)] 0 成功; 非 0 错误码
function client:send_text(text)
	return 0
end

-- @brief 协程接收 (须 kco 协程内)
-- @return [string] "text"|"media", [string|lightuserdata] body 或媒体指针
function client:co_recv()
	return "text", ""
end
```

**rpc_client**:

```lua
-- @brief POST-RPC
-- @param [in] rpctype[number(int)]	KLB_MNP_RPC_LUA / KLB_MNP_RPC_JSON
-- @param [in] ...					载荷 (LUA 序列化或 JSON 串)
-- @return [number(int)] 错误码
function rpc_client:post(rpctype, ...)
	return 0
end

function rpc_client:notify(rpctype, ...)
	return 0
end

-- @brief 协程收 POST/NOTIFY
-- @return [...] 解包后的 RPC 参数
function rpc_client:co_recv()
	return ...
end

-- @brief 协程 CALL 并等待 RESPONSE
-- @return [...] 响应数据; 发送失败 nil
function rpc_client:co_call(rpctype, ...)
	return ...
end

-- @brief 协程等待写缓冲清空
function rpc_client:co_wait()
	return
end

-- @brief RPC 状态
-- @return [table] { rpctype, method, sequence, ... }
function rpc_client:status()
	return {}
end
```

**listen**:

```lua
function listen:close()
	return
end

-- @return [userdata] serve; 无连接时 yield
function listen:co_accept()
	return {}
end
```

**serve** / **serverpc**: 方法同上表; `co_recv` 语义与 client 族一致.

### 示例

```lua
local kco = require("kco")
local ksmp = require("ksmp")

kco.fork(function ()
	local c = ksmp.connect("127.0.0.1", 3456)
	if nil == c then
		return
	end
	c:send_text("hello")
	local typ, body = c:co_recv()
	if "text" == typ then
		print(body)
	end
	c:disconnect()
end)
```

### 注意

- **`co_recv` / `co_accept` / `co_call` / `co_wait`** 须在 **`kco` 协程**内 — [kco.md](kco.md)
- 媒体消息 `co_recv` 第二值为 **lightuserdata** (`klb_buf_t*`); 用毕须释放 (协议层约定)
- 脚本侧高级 API (**klbcore.klbsmp**) 封装本模块; 见 [klbsmp.md](../klbcore/klbsmp.md)
- RPC 类型/方法常量见 **klb-mnp-smp-design**; C **klbnet** 见 **klb-net-design**
