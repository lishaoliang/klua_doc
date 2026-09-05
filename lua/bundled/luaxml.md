## XML 解析 (LuaXML)

> **require**: `LuaXML_lib` | 代码: [klb/src_c/klua/LuaXML_130610/LuaXML_lib.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/LuaXML_130610/LuaXML_lib.c)
> **文档样板**: bundled Lua API 四层 — 同 [ksys.md](../klua/ksys.md)
> **上游**: LuaXml / Gerald Franz (MIT)

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `load(filename)` | table | 读文件并解析为 XML 树 |
| `eval(xml_string)` | table | 解析 XML 字符串 |
| `encode(text)` | string | 对文本做实体转义 |
| `registerCode(decoded, encoded)` | — | 注册自定义实体对 |

**解析结果 table 约定**

- 元表 `__index` / `__tostring` 依赖全局 `xml` (纯 Lua 层); C 层只建树
- 索引 `0`: 标签名 (string)
- 索引 `1..n`: 子节点 (嵌套 table 或文本 string)
- 其他 string 键: 属性名 → 属性值

### 伪代码

```lua
--[[
-- @file   LuaXML_lib.lua
-- @brief  require("LuaXML_lib")
--   \n C: ./klb/src_c/klua/LuaXML_130610/LuaXML_lib.c
-- @note 模块名必须为 LuaXML_lib (非 LuaXML)
--]]

local LuaXML = {}


-- @brief 从文件加载并解析
-- @param [in] filename[string]
-- @return [table] root	XML 树; 失败抛异常
-- @note 内部 fread → eval
LuaXML.load = function (filename)
	return { [0] = 'root' }
end


-- @brief 解析 XML 字符串
-- @param [in] xml[string|lightuserdata]
-- @return [table] root
LuaXML.eval = function (xml)
	return { [0] = 'root' }
end


-- @brief 文本实体编码 (默认 & < > " ')
-- @param [in] text[string]
-- @return [string] encoded
LuaXML.encode = function (text)
	return text
end


-- @brief 注册额外实体 (decoded → encoded)
-- @param [in] decoded[string]	如 "©"
-- @param [in] encoded[string]	如 "&copy;"
LuaXML.registerCode = function (decoded, encoded)
	return
end


return LuaXML
```

### 示例

```lua
local LuaXML = require("LuaXML_lib")

local xml = [[
<root id="1">
	<item>alpha</item>
	<item>beta</item>
</root>
]]

local tree = LuaXML.eval(xml)
print(tree[0])		-- "root"
print(tree.id)		-- "1" (属性)
print(tree[1][0])	-- "item"
print(tree[1][1])	-- "alpha"

local safe = LuaXML.encode('<tag & "')
-- &lt;tag &amp; &quot;

-- 文件
-- local doc = LuaXML.load("config.xml")
```

### 注意

- 默认编入; 无裁剪宏; require 名 **`LuaXML_lib`** (大小写敏感)
- 轻量解析器, **非** DOM/XPath 完整实现; 复杂 XML 建议评估其他方案
- `eval` 依赖全局 `xml` 做 metatable; 纯 C 预加载环境需自行提供 `xml` 表 (见 klbcore 封装)
- klbcore 纯 Lua 封装 `klbcore.util.LuaXml` → **klbcore-design** § 通用模块 (与 C 模块分层不同)
- 内置实体: `&` → `&amp;`, `<` → `&lt;`, `>` → `&gt;`, `"` → `&quot;`, `'` → `&apos;`
