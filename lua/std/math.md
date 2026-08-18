## 数学 (math)

> **require**: `math` | 代码: `klb/src_c/klua/lua-5.4.6/src/lmathlib.c` (`luaopen_math`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.7](https://www.lua.org/manual/5.4/manual.html#6.7)

### 导出 API

#### 常量

| 字段 | 说明 |
|------|------|
| `pi` | π |
| `huge` | 最大浮点 |
| `maxinteger` | 最大整数 |
| `mininteger` | 最小整数 |

#### 函数

| 函数 | 返回 | 说明 |
|------|------|------|
| `abs(x)` | number | 绝对值 |
| `acos`/`asin`/`atan`/`cos`/`sin`/`tan` | number | 三角 |
| `atan(y [, x])` | number | 两参时为 `atan2` |
| `ceil`/`floor` | integer | 取整 |
| `deg`/`rad` | number | 角度 ↔ 弧度 |
| `exp`/`log`/`sqrt` | number | 指数/对数/平方根 |
| `fmod`/`modf` | number | 取模 / 拆分整数与小数 |
| `max(...)` / `min(...)` | number | 最大/最小 |
| `tointeger(x)` | integer / `nil` | 可转整数则返回 |
| `type(x)` | string / `nil` | `"integer"`/`"float"`/`nil` |
| `ult(m, n)` | boolean | 无符号整数 `<` |
| `random([m [, n]])` | number | 伪随机 (独立 RNG 状态) |
| `randomseed(x [, y])` | 无 | 设随机种子 |

#### 兼容函数 (`LUA_COMPAT_MATHLIB`, klb 默认启用)

| 函数 | 说明 |
|------|------|
| `atan2(y, x)` | 同 `atan(y, x)` |
| `cosh`/`sinh`/`tanh` | 双曲 |
| `pow(x, y)` | 幂 (推荐 `x^y`) |
| `frexp`/`ldexp` | 浮点分解/组合 |
| `log10(x)` | 以 10 为底 |

业务随机串/ID 亦可 **`krand`** → [krand.md](../klua/krand.md).

### 伪代码

真源: `lmathlib.c` — `mathlib[]`, `randfuncs[]`

```lua
--[[
-- @file   math.lua (伪代码)
-- @brief  Lua 5.4 math 库
--]]

local math = {}

math.pi = 3.141592653589793
math.huge = math.huge
math.maxinteger = 9223372036854775807
math.mininteger = -9223372036854775807


math.abs = function (x) return 0 end
math.acos = function (x) return 0 end
math.asin = function (x) return 0 end
math.atan = function (y, x) return 0 end
math.cos = function (x) return 0 end
math.sin = function (x) return 0 end
math.tan = function (x) return 0 end
math.deg = function (x) return 0 end
math.rad = function (x) return 0 end
math.exp = function (x) return 0 end
math.log = function (x, base) return 0 end
math.max = function (...) return 0 end
math.min = function (...) return 0 end
math.floor = function (x) return 0 end
math.ceil = function (x) return 0 end
math.sqrt = function (x) return 0 end
math.fmod = function (x, y) return 0 end
math.modf = function (x) return 0, 0 end
math.ult = function (m, n) return false end

-- LUA_COMPAT_MATHLIB (klb 默认启用)
math.atan2 = function (y, x) return 0 end
math.cosh = function (x) return 0 end
math.sinh = function (x) return 0 end
math.tanh = function (x) return 0 end
math.pow = function (x, y) return 0 end
math.frexp = function (x) return 0, 0 end
math.ldexp = function (m, e) return 0 end
math.log10 = function (x) return 0 end


-- @brief 随机整数 [1,n] 或 [m,n]; 无参返回 [0,1) 浮点
math.random = function (m, n)
	return 0
end


-- @brief 设种子 (Lua 5.4 可传 x,y 两个整数)
math.randomseed = function (x, y)
end


-- @brief 可精确转 integer 则返回, 否则 nil
math.tointeger = function (x)
	return 0
end


-- @brief "integer"|"float"|nil
math.type = function (x)
	return 'integer'
end

return math
```

### 示例

```lua
math.randomseed(os.time())

local n = math.random(1, 100)
print("dice", n)

local i = math.tointeger("42")
local f = math.type(3.14)  -- "float"

local r = math.deg(math.pi)  -- 180
```

### 注意

- klb 编译 Lua 5.4 时 **`LUA_COMPAT_5_3`** 开启 → 含 **`LUA_COMPAT_MATHLIB`** 废弃 API (见上表)
- `math.random` 使用库内独立 RNG; 与 **`krand`** (C 层) 种子无关
- 需要密码学强度随机时勿仅用 `math.random`
