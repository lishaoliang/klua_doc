## 字符串 (string)

> **require**: `string` | 代码: [klb/src_c/klua/lua-5.4.6/src/lstrlib.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-5.4.6/src/lstrlib.c) (`luaopen_string`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.4](https://www.lua.org/manual/5.4/manual.html#6.4)

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `byte(s [, i [, j]])` | int / `...` | 字节值 |
| `char(...)` | string | 字节 → 字符串 |
| `len(s)` | integer | 长度 (字节) |
| `sub(s, i [, j])` | string | 子串 (负索引自尾) |
| `lower(s)` / `upper(s)` | string | 大小写 |
| `reverse(s)` | string | 反转 |
| `rep(s, n [, sep])` | string | 重复 |
| `find(s, pattern [, init [, plain]])` | start, end / 无 | 查找 |
| `match(s, pattern [, init])` | captures / 无 | 首次匹配捕获 |
| `gmatch(s, pattern)` | iterator | 全局模式迭代 |
| `gsub(s, pattern, repl [, n])` | s, n | 全局替换 |
| `format(fmt, ...)` | string | `printf` 风格格式化 |
| `dump(f [, strip])` | string | 函数二进制 dump |
| `pack(fmt, ...)` | string | 二进制打包 (fmt 同 struct) |
| `packsize(fmt)` | integer | 格式占用字节 |
| `unpack(fmt, s [, pos])` | `...`, nextpos | 二进制解包 |

字符串可通过 **string metatable** 用 `:` 语法调用上表函数 (如 `"abc":upper()`).

### 伪代码

真源: `lstrlib.c` — `strlib[]`

```lua
--[[
-- @file   string.lua (伪代码)
-- @brief  Lua 5.4 string 库
--]]

local string = {}


-- @brief s[i] 或 s[i..j] 的字节值
string.byte = function (s, i, j)
	return 0
end


-- @brief 字节码 → 字符串
string.char = function (...)
	return ''
end


-- @brief 字节长度 (#s 等价)
string.len = function (s)
	return 0
end


-- @brief 子串; j 缺省为 -1
string.sub = function (s, i, j)
	return ''
end


string.lower = function (s) return s end
string.upper = function (s) return s end
string.reverse = function (s) return s end
string.rep = function (s, n, sep) return s end


-- @brief Lua 模式查找; plain=true 则 plain 搜索
-- @return [number(int)] start, [number(int)] end; 未找到则无返回
string.find = function (s, pattern, init, plain)
	return 1, 3
end


-- @brief 首次匹配捕获
string.match = function (s, pattern, init)
	return ...
end


-- @brief 全局模式迭代
string.gmatch = function (s, pattern)
	return function () end
end


-- @brief 全局替换; repl 可为 string/table/function
-- @return [string] 结果, [number(int)] 替换次数
string.gsub = function (s, pattern, repl, n)
	return s, 0
end


-- @brief 格式化 (%s %d %q %f …; Lua 5.4 支持 %@ 等扩展)
string.format = function (fmt, ...)
	return ''
end


-- @brief 函数二进制 dump (load/loadstring 可还原)
-- @param [in] strip[boolean]		[可选] 剥离调试信息
-- @return [string]
string.dump = function (f, strip)
	return ''
end


-- @brief 二进制 struct 打包 (如 "<i4", ">f")
string.pack = function (fmt, ...)
	return ''
end


-- @brief 格式 fmt 占用字节数
string.packsize = function (fmt)
	return 0
end


-- @brief 从 s[pos] 解包
-- @return [...] values, [number(int)] nextpos
string.unpack = function (fmt, s, pos)
	return ..., pos
end

return string
```

### 示例

```lua
local s = string.format("pi=%.2f", math.pi)
print(s)

local name, ver = string.match("Lua 5.4", "(%a+)%s+([%d%.]+)")
print(name, ver)

-- 二进制
local bin = string.pack("<I4", 0x12345678)
local n, pos = string.unpack("<I4", bin)
print(string.format("0x%x", n))

for w in string.gmatch("a,b,c", "[^,]+") do
	print(w)
end
```

### 注意

- Lua **模式** 非 PCRE; 复杂正则用 bundled **`lpeg`** → [lpeg.md](../bundled/lpeg.md)
- JSON 序列化用 **`cjson`** 或 klb **`ksys.pack_json`** → [cjson.md](../bundled/cjson.md), [ksys.md](../klua/ksys.md)
- UTF-8 字符操作见 **`utf8`** → [utf8.md](utf8.md); `string.len` 为**字节**长度
