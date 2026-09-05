## HTTP MIME 映射

> **require**: `klbcore.net.http_mime` | 代码: [klb/bin/klbcore/net/http_mime.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/net/http_mime.lua)
> **文档样板**: Lua API 四层 — [k-bindings.md](../../../klb/klua/design/k-bindings.md) § Lua API 文档

按文件扩展名返回 **Content-Type**; 静态 HTTP 服务常用. 架构 **klbcore-net-design**.

### 导出 API

模块导出 **单一函数** (非 table):

| 调用 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `http_mime(filename)` | 文件名或路径 | string | MIME 类型; 未知为 `application/octet-stream` |

内置扩展含 `html`、`css`、`js`、`json`、`png`、`jpg`、`wasm`、`mp4` 等 (见源码 `my_mime` 表).

### 伪代码

源码: [klb/bin/klbcore/net/http_mime.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/net/http_mime.lua)

```lua
--[[
-- @file   http_mime.lua
-- @brief  按扩展名查 Content-Type
--   \n require("klbcore.net.http_mime")
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
	-- … 完整表见源码 my_mime
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

### 示例

```lua
local http_mime = require("klbcore.net.http_mime")

local path = '/static/app.js'
local ctype = http_mime(path)   -- 'text/javascript'

-- 配合 HTTP 响应头
local header = 'Content-Type: ' .. ctype
```

### 注意

- 仅查 **最后一个 `.` 后缀**; 无扩展名 → `application/octet-stream`
- 扩展名 **大小写不敏感**
- **`klbcore.net.httpc`** 尚未落盘 (设计见 **klbcore-net-design**); 本模块可独立使用
