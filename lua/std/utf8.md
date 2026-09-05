## UTF-8 (utf8)

> **require**: `utf8` | 代码: [klb/src_c/klua/lua-5.4.6/src/lutf8lib.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-5.4.6/src/lutf8lib.c) (`luaopen_utf8`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.5](https://www.lua.org/manual/5.4/manual.html#6.5)

### 导出 API

| 符号 | 返回 | 说明 |
|------|------|------|
| `len(s [, i [, j]])` | n / `nil, pos` | 字符个数; 非法 UTF-8 返回 nil + 错误位置 |
| `offset(s, n [, i])` | integer | 第 n 个字符的字节偏移 |
| `codepoint(s [, i [, j]])` | code / `...` | Unicode 码点 |
| `char(...)` | string | 码点 → UTF-8 字符串 |
| `codes(s)` | iterator | 迭代 `(pos, code)` |
| `charpattern` | string | 匹配单个 UTF-8 字符的模式 (只读) |

### 伪代码

真源: `lutf8lib.c` — `funcs[]`

```lua
--[[
-- @file   utf8.lua (伪代码)
-- @brief  Lua 5.4 utf8 库
--]]

local utf8 = {}

utf8.charpattern = '[\0-\x7F\xC2-\xF4][\x80-\xBF]*'


-- @brief UTF-8 字符数 (非字节数)
-- @return [number(int)] n; 非法时 nil, [number(int)] errpos
utf8.len = function (s, i, j)
	return 0
end


-- @brief 第 n 个字符的字节索引 (n=0 表 s:sub(i) 长度)
-- @param [in] i[number(int)]	[可选] 起始字节, 默认 1
utf8.offset = function (s, n, i)
	return 1
end


-- @brief 码点 (可多个)
utf8.codepoint = function (s, i, j)
	return 0
end


-- @brief 码点 → UTF-8
utf8.char = function (...)
	return ''
end


-- @brief 迭代每个字符的 pos 与 code
-- @return [function] for pos, code in utf8.codes(s) do … end
utf8.codes = function (s)
	return function () end
end

return utf8
```

### 示例

```lua
local s = "中文ABC"
print(utf8.len(s))           -- 5 个字符
print(#s)                    -- 字节数更大

for p, c in utf8.codes(s) do
	print(p, string.format("U+%04X", c))
end

local sub = string.sub(s, utf8.offset(s, 2))  -- 从第 2 字符到末尾
```

### 注意

- **`string.len` / `#s`** 为字节长度; 显示/截断多字节文本须配合 **`utf8`**
- `utf8.charpattern` 可用于 `string.gmatch` 按字符切分
- 规范化 (NFC/NFD) 等高级 Unicode 操作不在标准库内
