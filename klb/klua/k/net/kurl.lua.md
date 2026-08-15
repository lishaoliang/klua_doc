# kurl

> `require("kurl")` — 代码: `klb/src_c/klua/klua_net/klua_kurl.c`

URL 解析 (http_parser). **无** 网络 IO.

## 库函数

| 函数 | 说明 |
|------|------|
| `parse(url)` | 解析 URL, 返回字段表 (schema/host/port/path/query/fragment/userinfo) |

```lua
local t = require("kurl").parse("http://127.0.0.1:8080/path?q=1")
```
