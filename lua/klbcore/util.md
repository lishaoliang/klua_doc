## 通用工具 (util)

> 代码: [klb/bin/klbcore/util](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/util) | 架构 **klbcore-design** § 通用模块
> **文档样板**: Lua API 四层 — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

小工具模块; 按需 `require("klbcore.util.*")`.

### 模块一览

| require | 文档节 | 说明 |
|---------|--------|------|
| `klbcore.util.stringex` | [§ stringex](#stringex) | 字符串扩展 |
| `klbcore.util.tableex` | [§ tableex](#tableex) | 表扩展 |
| `klbcore.util.xmlparser` | [§ xmlparser](#xmlparser) | JSON 式 table ↔ XML |
| `klbcore.util.printex` | [§ printex](#printex) | JSON 行打印 |
| `klbcore.util.LuaXml` | — | xmlparser 依赖; 见 [bundled/luaxml.md](../bundled/luaxml.md) |
| `klbcore.util.http_mime` | [§ http_mime](#http_mime) | 按扩展名查 MIME |
| `klbcore.util.klpcex` | [§ klpcex](#klpcex) | LPC 脚本扩展 |
| `klbcore.util.pname` | [§ pname](#pname) | 协议名常量 |

---

### stringex

> **require**: `klbcore.util.stringex` | 代码: `util/stringex.lua`

#### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `url_encode(s)` | string | URL 编码 |
| `url_decode(s)` | string | URL 解码 |
| `trim(s)` | string | 去首尾空白 |
| `cmp_ignore_case(s1, s2)` | boolean | 忽略大小写比较 |
| `find_ignore_case(s1, s2)` | boolean | 忽略大小写子串查找 |
| `join(...)` | string | 多值拼接为字符串 |

#### 伪代码

源码: [klb/bin/klbcore/util/stringex.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/stringex.lua)

```lua
--[[
-- Copyright (c) 2022, GNU LESSER GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file   stringex.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  string extented
--   \n require("klbcore.util.stringex")
-- @version 0.1
--]]

local stringex = {}


-- @brief URL 编码
-- @param [in] s[string]			原始字符串
-- @return [string] 编码结果
stringex.url_encode = function (s)
	return ''
end


-- @brief URL 解码
-- @param [in] s[string]			编码字符串
-- @return [string] 解码结果
stringex.url_decode = function (s)
	return ''
end


-- @brief 去除前后空白
-- @param [in] s[string]			原始字符串
-- @return [string] 修剪结果
stringex.trim = function (s)
	return ''
end


-- @brief 忽略大小写比较
-- @param [in] s1[string]			字符串 1
-- @param [in] s2[string]			字符串 2
-- @return [boolean] true 相等
stringex.cmp_ignore_case = function (s1, s2)
	return false
end


-- @brief 忽略大小写子串查找
-- @param [in] s1[string]			被查找串
-- @param [in] s2[string]			子串
-- @return [boolean] true 找到
stringex.find_ignore_case = function (s1, s2)
	return false
end


-- @brief 多值拼接为字符串
-- @param [in] ...					任意值 (tostring)
-- @return [string] 拼接结果
stringex.join = function (...)
	return ''
end


return stringex
```

#### 示例

```lua
local stringex = require("klbcore.util.stringex")

local q = stringex.url_encode('a b=c')   -- 'a+b%3Dc'
local t = stringex.trim('  hi  ')       -- 'hi'
local s = stringex.join('a', 1, true)   -- 'a1true'
```

---

### tableex

> **require**: `klbcore.util.tableex` | 代码: `util/tableex.lua`

#### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `is_array(t)` | boolean | 是否为数组 (连续数字键) |
| `is_empty(t)` | boolean | 空表或非 table |
| `is_not_empty(t)` | boolean | 非空 table |
| `copy(src)` | table | 浅层复制 (仅 number/string/boolean 与嵌套 table) |

#### 伪代码

源码: [klb/bin/klbcore/util/tableex.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/tableex.lua)

```lua
--[[
-- Copyright (c) 2022, GNU LESSER GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file   tableex.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  table extented
--   \n require("klbcore.util.tableex")
-- @version 0.1
--]]

local tableex = {}


-- @brief 判定 table 是否为数组 (连续数字键 1..#t)
-- @param [in] t[table]				table 对象
-- @return [boolean] true 是数组
tableex.is_array = function (t)
	return false
end


-- @brief 判定 table 是否为空
-- @param [in] t[table]				table 对象; 非 table 视为空
-- @return [boolean] true 为空
tableex.is_empty = function (t)
	return true
end


-- @brief 判定 table 不为空
-- @param [in] t[table]				table 对象
-- @return [boolean] true 不为空
tableex.is_not_empty = function (t)
	return false
end


-- @brief 复制 table (仅 number/string/boolean 与嵌套 table)
-- @param [in] src[table]			原始 table; nil → {}
-- @return [table] 新 table
tableex.copy = function (src)
	return {}
end


return tableex
```

#### 示例

```lua
local tableex = require("klbcore.util.tableex")

if tableex.is_array({ 1, 2, 3 }) then
	local dup = tableex.copy({ a = 1, b = { c = 2 } })
end
```

---

### xmlparser

> **require**: `klbcore.util.xmlparser` | 代码: `util/xmlparser.lua`

受限 **JSON 式 table** 与 XML 互转; 规则参考 [json2xml 工具](http://web.chacuo.net/charsetjson2xml).

#### 导出 API

| 名 | 说明 |
|----|------|
| `cfg` | 配置 table: `boolean`/`number`/`string`/`array`/`object` 是否写 `type` 属性 |
| `to_xml(t [, root])` | table → LuaXml 对象; 默认根 `root` |
| `to_xml_str(t [, root])` | table → XML 字符串 |
| `to_table(x)` | LuaXml 对象或 XML 字符串 → table |

转换约定: 数组元素键 **`item`**; 根节点默认 **`root`**.

#### 伪代码

源码: [klb/bin/klbcore/util/xmlparser.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/xmlparser.lua)

```lua
--[[
-- Copyright (c) 2022, GNU LESSER GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file   xmlparser.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  xml parser (受限 JSON 式 table ↔ XML)
--   \n require("klbcore.util.xmlparser")
-- @note 数组键 item; 根节点默认 root
-- @version 0.1
--]]

local xmlparser = {}


-- @brief 转换配置: 是否在 XML 写 type 属性
xmlparser.cfg = {
	['boolean'] = true,
	['number'] = true,
	['string'] = false,
	['array'] = false,
	['object'] = false,
}


-- @brief 将 table 转换为 LuaXml 对象
-- @param [in] t[table]				标准 table
-- @param [in] root[string]			[可选] 根节点名; 默认 'root'
-- @return [table] LuaXml 对象
xmlparser.to_xml = function (t, root)
	return {}
end


-- @brief 将 table 转换为 XML 字符串
-- @param [in] t[table]				标准 table
-- @param [in] root[string]			[可选] 根节点名; 默认 'root'
-- @return [string] XML 字符串
xmlparser.to_xml_str = function (t, root)
	return ''
end


-- @brief 将 XML 转换为 table
-- @param [in] x[table,string]		LuaXml 对象或 XML 字符串
-- @return [table] {} 标准 table
xmlparser.to_table = function (x)
	return {}
end


return xmlparser
```

#### 示例

```lua
local xmlparser = require("klbcore.util.xmlparser")

local t = {
	funcname = 'get_video',
	result = { code = 0, fmt = 'h264' },
}

local xml = xmlparser.to_xml_str(t)
local back = xmlparser.to_table(xml)
```

#### 注意

- 仅支持 **类 JSON 结构**; 任意 XML 勿用
- 依赖 **`klbcore.util.LuaXml`** (bundled `LuaXML_lib` 封装)

---

### printex

> **require**: `klbcore.util.printex` | 代码: `util/printex.lua`

模块导出 **单一函数**: 将参数序列化为 JSON 后 `print`.

#### 导出 API

| 调用 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `printex(...)` | 任意参数 | 无 | table 递归序列化; 其他 `tostring`; 输出一行 JSON |

#### 伪代码

源码: [klb/bin/klbcore/util/printex.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/printex.lua)

```lua
--[[
-- Copyright (c) 2022, GNU LESSER GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file   printex.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  print extented (JSON 行输出)
--   \n require("klbcore.util.printex")
-- @note 依赖 cjson.safe; 模块导出单一函数
-- @version 0.1
--]]

-- @brief 将参数序列化为 JSON 后 print
-- @param [in] ...					任意参数; table 递归; 其他 tostring
-- @return 无
local printex = function (...)
	print('')
end

return printex
```

#### 示例

```lua
local printex = require("klbcore.util.printex")

printex({ ok = true, n = 3 })
-- 输出一行 JSON
```

#### 注意

- 依赖 **`cjson.safe`**
- 调试辅助; 生产日志请用项目 logger

---

### http_mime

> **require**: `klbcore.util.http_mime` | 代码: [klb/bin/klbcore/util/http_mime.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/http_mime.lua)

按文件扩展名返回 **Content-Type**; 静态 HTTP 服务常用. 架构 **klbcore-design** § 通用模块.

#### 导出 API

模块导出 **单一函数** (非 table):

| 调用 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `http_mime(filename)` | 文件名或路径 | string | MIME 类型; 未知为 `application/octet-stream` |

内置扩展含 `html`、`css`、`js`、`json`、`png`、`jpg`、`wasm`、`mp4` 等 (见源码 `my_mime` 表).

#### 伪代码

源码: [klb/bin/klbcore/util/http_mime.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/http_mime.lua)

```lua
--[[
-- @file   http_mime.lua
-- @brief  按扩展名查 Content-Type
--   \n require("klbcore.util.http_mime")
-- @note 模块导出单一函数 (非 table); 未知 → application/octet-stream
--]]

local my_mime = {
	html = 'text/html',
	htm = 'text/html',
	css = 'text/css',
	js = 'text/javascript',
	json = 'application/json',
	png = 'image/png',
	jpg = 'image/jpeg',
	jpeg = 'image/jpeg',
	wasm = 'application/wasm',
	mp4 = 'video/mpeg4',
	-- ... 完整表见源码 my_mime
}

-- @brief 由文件名扩展名查 MIME
-- @param [in] filename[string]		文件名或路径, eg. 'index.html'
-- @return [string] MIME 类型; 未知 'application/octet-stream'
local http_mime = function (filename)
	local ext = string.match(filename, '[^.]*$')
	if nil ~= ext then
		ext = string.lower(stringex.trim(ext))
		local mime = my_mime[ext]
		if nil ~= mime then
			return mime
		end
	end
	return 'application/octet-stream'
end

return http_mime
```

#### 示例

```lua
local http_mime = require("klbcore.util.http_mime")

local path = '/static/app.js'
local ctype = http_mime(path)   -- 'text/javascript'

-- 配合 HTTP 响应头
local header = 'Content-Type: ' .. ctype
```

#### 注意

- 仅查 **最后一个 `.` 后缀**; 无扩展名 → `application/octet-stream`
- 扩展名 **大小写不敏感**
- 旧 `klbcore.net.httpc` 已迁 `backup/klbcore/net/`; 本模块可独立使用

---

### klpcex

> **require**: `klbcore.util.klpcex` | 代码: [klb/bin/klbcore/util/klpcex.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/klpcex.lua) | C 绑定 **`klpc`** 见 [klua/klpc.md](../klua/klpc.md)

跨 **`klua_env`** 调用本地模块方法的 **便捷封装**; 内部 `klpc.new()` → `co_call` → `close`.

#### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `call(mo_name, ...)` | ... | 协程内调用目标 env 模块方法 |

#### 伪代码

源码: [klb/bin/klbcore/util/klpcex.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/klpcex.lua)

```lua
--[[
-- Copyright (c) 2022, GNU GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file  klpcex.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief klpc extented
--   \n require("klbcore.util.klpcex")
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

#### 示例

```lua
local kco = require("kco")
local klpcex = require("klbcore.util.klpcex")

kco.fork(function ()
	local ok, data = klpcex.call('web.handler', 'get_config', 'id')
	print(ok, data)
end)
```

#### 注意

- **须在 `kco` 协程内**; 底层为 [klpc](../klua/klpc.md) `co_call`
- 跨线程只传 **可序列化值** — **klbcore-design** § kthread
- 完整 klpc API (注册/广播等) → [klua/klpc.md](../klua/klpc.md)

---

### pname

> **require**: `klbcore.util.pname` | 代码: [klb/bin/klbcore/util/pname.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/pname.lua)

网络协议 **字符串常量** 表; 与 knet 命名对齐. 无函数 API.

#### 导出 API

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

#### 伪代码

源码: [klb/bin/klbcore/util/pname.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/util/pname.lua)

```lua
--[[
-- @file   pname.lua
-- @brief  协议名字符串常量 (from knet)
--   \n require("klbcore.util.pname")
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

#### 示例

```lua
local pname = require("klbcore.util.pname")

if schema == pname.RTSP then
	-- RTSP 分支
end
```

#### 注意

- 只读常量; **`UNKOWN`** 拼写与源码一致 (非 `UNKNOWN`)
- 协议栈实现见 **klb-net-design** / **klb-mnp-smp-design**
