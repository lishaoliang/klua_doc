## 通用工具 (util)

> 代码: `bin/klbcore/util/` | 架构 **klbcore-design** § 通用模块
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

源码: `bin/klbcore/util/stringex.lua`

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

源码: `bin/klbcore/util/tableex.lua`

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

源码: `bin/klbcore/util/xmlparser.lua`

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

源码: `bin/klbcore/util/printex.lua`

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
