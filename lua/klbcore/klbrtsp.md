## RTSP 脚本层

> **require**: `klbcore.klbrtsp` | 代码: [klb/bin/klbcore/klbrtsp](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/klbrtsp) | C 绑定 **`krtsp`** 见 [klua/readme.md](../klua/readme.md) 网络节
> **文档样板**: Lua API 四层 — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

RTSP 协议 **纯 Lua 封装**; 底层 **`krtsp`** / **`kurl`**. C 栈见 **klb-net-design**; 脚本设计 **klbcore-net-design**.

### 导出 API

#### 模块级 (init.lua)

| 函数 | 返回 | 说明 |
|------|------|------|
| `new_client([cfg])` | client 对象 | 新建 RTSP 客户端 |
| `new_listen([cfg])` | listen 对象 | 新建 RTSP 监听 |

#### client 对象 (`rtspclienter`)

| 方法 | 说明 |
|------|------|
| `co_connect(url)` | 协程连接并完成 OPTIONS/DESCRIBE/SETUP/PLAY 握手; `0` 成功 |
| `co_recv()` | 协程接收; `media` 消息自动 dump/free |
| `disconnect()` | 断开 |

连接后对象字段: `url`, `session`, `video`, `audio` (SDP 解析结果).

#### listen 对象 (`rtsplistener`)

| 方法 | 说明 |
|------|------|
| `open(port)` / `close()` | 监听控制 |
| `co_accept()` | 协程接受连接 → `rtspserver` 对象 |

#### 子模块

| require | 文件 | 说明 |
|---------|------|------|
| `klbcore.klbrtsp.client.rtspsdper` | `client/rtspsdper.lua` | SDP/报文 pack/parse |
| `klbcore.klbrtsp.serve.rtspserver` | `serve/rtspserver.lua` | 会话服务 |
| `klbcore.klbrtsp.rtspcode` | `rtspcode.lua` | 状态码 |

### 伪代码

源码: [klb/bin/klbcore/klbrtsp/init.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/klbrtsp/init.lua) 及 `client/`、`serve/` 子模块

```lua
--[[
-- Copyright (c) 2025, GNU LESSER GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file   init.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  klbrtsp init.lua
--   \n require("klbcore.klbrtsp")
--   \n 源码: bin/klbcore/klbrtsp/
-- @note RTSP 协议脚本层; C 绑定 krtsp
-- @version 0.1
--]]

local klbrtsp = {}


-- @brief 新建 RTSP 客户端
-- @param [in] cfg[table]			[可选] 客户端配置
-- @return [table] client 对象
klbrtsp.new_client = function (cfg)
	return {
		_client = nil,
		cseq = 1,
		url = '',
		session = '',
		video = {},
		audio = {},
	}
end


-- @brief 新建 RTSP 监听模块
-- @param [in] cfg[table]			[可选] 监听配置
-- @return [table] listen 对象
klbrtsp.new_listen = function (cfg)
	return {
		_listen = nil,
	}
end


return klbrtsp
```

**client 对象** (`rtspclienter`):

```lua
-- @brief 协程连接并完成 OPTIONS/DESCRIBE/SETUP/PLAY 握手
-- @param [in] url[string]			eg. 'rtsp://127.0.0.1:554/live'
-- @return [number(int)] 0 成功; 1 连接失败
-- @note 须在 kco 协程内; 成功后 video/audio 为 SDP 解析结果
function client:co_connect(url)
	return 0
end


-- @brief 协程接收数据
-- @return 无
-- @note msg=='media' 时内部 dump_media/free_media
function client:co_recv()
	return
end


-- @brief 断开连接
function client:disconnect()
	return
end
```

**listen 对象** (`rtsplistener`):

```lua
-- @brief 开始监听端口 (会先 close)
-- @param [in] port[number(int)]	端口
function listen:open(port)
	return
end


-- @brief 关闭监听
function listen:close()
	return
end


-- @brief 协程接受新连接 → rtspserver 对象
-- @return [table] rtspserver; 失败 nil
function listen:co_accept()
	return {}
end
```

**rtspsdper** (`require("klbcore.klbrtsp.client.rtspsdper")`):

```lua
local rtspsdper = {}

-- @brief 打包 OPTIONS 请求
-- @param [in] url[string]			RTSP URL
-- @param [in] cseq[number(int)]		序列号
-- @return [string] RTSP 报文
rtspsdper.pack_OPTIONS = function (url, cseq)
	return ""
end

-- @brief 打包 DESCRIBE / SETUP / PLAY / GET_PARAMETER / TEARDOWN (参数同 pack_OPTIONS + session 等)
rtspsdper.pack_DESCRIBE = function (url, cseq)
	return ""
end

-- @brief 解析 RTSP 响应报文
-- @param [in] txt[string]			原始报文
-- @return [table]					{ status, headers, body, ... }
rtspsdper.parse = function (txt)
	return {}
end

-- @brief 解析 SDP 文本
-- @param [in] txt[string]			SDP 正文
-- @return [table]					{ video = {}, audio = {}, ... }
rtspsdper.parse_sdp = function (txt)
	return { video = {}, audio = {} }
end

return rtspsdper
```

**rtspserver** (`require("klbcore.klbrtsp.serve.rtspserver")`):

```lua
-- @brief 新建 RTSP 会话服务对象
-- @param [in] conn[userdata]		accept 得到的连接
-- @param [in] cfg[table]			[可选] 服务配置
-- @return [table] rtspserver 对象
rtspserver.new = function (conn, cfg)
	return {}
end

-- @brief 断开会话
function server:disconnect()
	return
end

-- @brief 协程处理 RTSP 会话 (须在 kco 协程内)
function server:co_do_session()
	return
end

-- @brief 发送媒体数据
-- @param [in] ptr[lightuserdata]	媒体指针
function server:send_media(ptr)
	return
end
```

**rtspcode** (`require("klbcore.klbrtsp.rtspcode")`):

```lua
local rtspcode = {}

-- @brief RTSP 状态码 → 描述字符串
-- @param [in] code[number(int)]		eg. 200, 404
-- @return [string] 描述; 未知为 ""
rtspcode.code_tostring = function (code)
	return ""
end

return rtspcode
```

### 示例

```lua
local kco = require("kco")
local klbrtsp = require("klbcore.klbrtsp")

kco.fork(function ()
	local c = klbrtsp.new_client()
	local rc = c:co_connect('rtsp://127.0.0.1:554/live')
	if 0 == rc then
		while true do
			c:co_recv()
		end
	end
	c:disconnect()
end)
```

### 注意

- **`co_connect` / `co_recv` / `co_accept`** 须在 **`kco` 协程**内
- **`krtsp`** C 绑定文档 **待定**; 以 **klb-net-design** 与源码为准
- 客户端 `co_connect` 当前自动完成 DESCRIBE/SETUP/PLAY; TEARDOWN 需自行调用
- 与 C **`klbrtsp`** 模块同名但分层不同 — 见 **klbcore-design** § 对话记名
