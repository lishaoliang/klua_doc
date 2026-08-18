## 表操作 (table)

> **require**: `table` | 代码: `klb/src_c/klua/lua-5.4.6/src/ltablib.c` (`luaopen_table`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.6](https://www.lua.org/manual/5.4/manual.html#6.6)

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `concat(list [, sep [, i [, j]]])` | string | 连接数组部分 |
| `insert(list [, pos], value)` | 无 | 插入元素 (pos 默认 `#list+1`) |
| `remove(list [, pos])` | any | 移除并返回 pos 处元素 (默认末尾) |
| `move(a1, f, e, t [, a2])` | table | 区间搬移 (a2 默认同 a1) |
| `pack(...)` | table | 可变参打包为 `{1=..., n=#}` |
| `unpack(list [, i [, j]])` | `...` | 解包数组段 |
| `sort(list [, comp])` | 无 | 原地排序 (1..n) |

### 伪代码

真源: `ltablib.c` — `tab_funcs[]`

```lua
--[[
-- @file   table.lua (伪代码)
-- @brief  Lua 5.4 table 库
--]]

local table = {}


-- @brief 连接 list[i..j] 为字符串
-- @param [in] sep[string]	[可选] 分隔符, 默认 ""
-- @param [in] i[number(int)]	[可选] 起始, 默认 1
-- @param [in] j[number(int)]	[可选] 结束, 默认 #list
-- @return [string]
table.concat = function (list, sep, i, j)
	return ''
end


-- @brief 插入 value; pos 缺省为 #list+1
-- @param [in] pos[number(int)]	[可选]
-- @param [in] value[any]
table.insert = function (list, pos, value)
end


-- @brief 移除 pos 处元素并返回; pos 缺省为 #list
-- @return [any] 被移除值
table.remove = function (list, pos)
	return nil
end


-- @brief 将 a1[f..e] 复制到 a2 起始于 t (可同表重叠搬移)
-- @param [in] a2[table]	[可选] 目标表, 默认 a1
-- @return [table] a2
table.move = function (a1, f, e, t, a2)
	return a2
end


-- @brief 可变参 → { n = 个数, 1..n = 值 }
-- @return [table]
table.pack = function (...)
	return { n = 0 }
end


-- @brief list[i..j] 解包为可变参
-- @return [...]
table.unpack = function (list, i, j)
	return ...
end


-- @brief 对 list[1..#list] 原地排序
-- @param [in] comp[function]	[可选] (a,b) -> boolean, 默认 <
table.sort = function (list, comp)
end

return table
```

### 示例

```lua
local t = { 3, 1, 4 }
table.sort(t)
print(table.concat(t, ","))  -- 1,3,4

local a = { 10, 20, 30, 40 }
table.move(a, 1, 2, 3)       -- 10,20 移到 3,4
-- a => { _, _, 10, 20 }

local p = table.pack("a", "b")
local x, y = table.unpack(p, 1, p.n)
```

### 注意

- `table.unpack` 与全局别名 `unpack` 等价 (Lua 5.4 仍保留在 table 库)
- kpfs / klbcore 返回表须保证内嵌子表键稳定 (空则 `{}`) → **coding-lua** § 返回 table
