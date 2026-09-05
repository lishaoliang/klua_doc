## RTSP 传输 (krtsp)

> **require**: `krtsp` | 代码: [klb/src_c/klua/klua_net/klua_krtsp.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua_net/klua_krtsp.c), `klua_krtsp_serve.c`
> **文档样板**: k* Lua API 四层 — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

RTSP 协议 **C 传输层** (L3); 脚本封装见 [klbrtsp.md](../klbcore/klbrtsp.md). C 栈 **klb-net-design**.

### 导出 API

#### 模块级

| 函数 | 返回 | 说明 |
|------|------|------|
| `connect(host, port)` | client / `nil` | RTSP 客户端连接 |
| `listen(port)` | listen | 监听 RTSP |
| `new_serve(listen_socket)` | serve | 由 accept 侧 socket 构造 serve 对象 |

#### client (RTSP 客户端 userdata)

| 方法 | 说明 |
|------|------|
| `disconnect()` | 关闭 |
| `send_text(text)` | 发送 RTSP 文本报文 |
| `co_recv()` | **须 kco 协程**; `"text"`+body 或 `"media"`+lightuserdata |
| `free_media(ptr)` | 释放 `co_recv` 得到的媒体缓冲 |
| `dump_media(ptr)` | 调试打印媒体头 (stdout) |

#### serve (RTSP 会话 userdata)

| 方法 | 说明 |
|------|------|
| `disconnect()` | 关闭 |
| `send_text(text)` / `send_media(ptr)` | 发送 |
| `co_recv()` | **须协程**; 同 client |

#### listen

| 方法 | 说明 |
|------|------|
| `close()` | 停止监听 |
| `co_accept()` | **须协程**; 接受连接 (内部可配合 `new_serve`) |

### 伪代码

真源: `klua_krtsp.c`, `klua_krtsp_serve.c`

```lua
--[[
-- @file   krtsp.lua (伪代码)
-- @brief  require("krtsp")
--   C: klua_net/klua_krtsp.c, klua_krtsp_serve.c
--]]

local krtsp = {}


-- @brief 连接 RTSP 服务
-- @param [in] host[string]
-- @param [in] port[number(int)]
-- @return [userdata|nil] client
krtsp.connect = function (host, port)
	return nil
end


-- @brief 监听 RTSP 端口
-- @return [userdata] listen
krtsp.listen = function (port)
	return {}
end


-- @brief 由 listen 接受的 socket 构造 serve
-- @param [in] listen_socket[lightuserdata]
-- @return [userdata] serve
krtsp.new_serve = function (listen_socket)
	return {}
end


return krtsp
```

**client**:

```lua
function client:disconnect()
	return
end

function client:send_text(text)
	return 0
end

-- @return [string] typ, [string|lightuserdata] payload
function client:co_recv()
	return "text", ""
end

-- @param [in] ptr[lightuserdata]	co_recv 返回的 media 指针
function client:free_media(ptr)
	return
end

function client:dump_media(ptr)
	return
end
```

**serve** / **listen**: 同 ksmp 族; listen 仅 `close` / `co_accept`.

### 示例

```lua
local kco = require("kco")
local krtsp = require("krtsp")

kco.fork(function ()
	local c = krtsp.connect("127.0.0.1", 554)
	if nil == c then
		return
	end
	while true do
		local typ, data = c:co_recv()
		if "media" == typ then
			c:dump_media(data)
			c:free_media(data)
		elseif "text" == typ then
			print(data)
		end
	end
end)
```

### 注意

- **`co_recv` / `co_accept`** 须在 **`kco` 协程**内
- 收到 **`media`** 后须 **`free_media`**, 否则泄漏 `klb_buf_t`
- 报文级 RTSP 握手/SDP 见 **klbcore.klbrtsp** — [klbrtsp.md](../klbcore/klbrtsp.md)
- **`dump_media`** 仅调试; 生产勿依赖 stdout 输出
