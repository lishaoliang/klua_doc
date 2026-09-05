## 解析表达式 (LPeg)

> **require**: `lpeg` | 代码: [klb/src_c/klua/lpeg-1.0.2/](https://gitee.com/klua/klb/tree/trunk/src_c/klua/lpeg-1.0.2/)
> **文档样板**: bundled Lua API 四层 — 同 [ksys.md](../klua/ksys.md)
> **上游**: [LPeg](http://www.inf.puc-rio.br/~roberto/lpeg/) 1.0.2 (MIT); 可选脚本 `re.lua` (未预加载)

### 导出 API

**构造**

| 函数 | 说明 |
|------|------|
| `P(v)` | 模式: 字符串 / 字符 / 模式 / 函数 / table |
| `S(set)` | 字符集合 |
| `R(range)` | 字符范围 `"az"` 或 `"09","az"` |
| `locale([cat])` | 区域类 (如 `"alpha"`) |
| `V(name)` | 语法变量 (配合 `lpeg.P{ ... }`) |

**捕获**

| 函数 | 说明 |
|------|------|
| `C(p)` | 捕获匹配 |
| `Cc(...)` | 捕获常量 |
| `Cg(p [, name])` | 命名组 |
| `Ct(p)` | 捕获为 table |
| `Cs(p)` | 替换捕获 |
| `Cf(p, fn)` | 折叠捕获 |
| `Cb(name)` | 反向引用 |
| `Carg(n)` | 额外参数捕获 |
| `Cp()` | 位置捕获 |
| `Cmt(p, fn)` | 运行时匹配时间函数 |

**运算 (亦可用运算符)**

| 运算符 | 函数 | 说明 |
|--------|------|------|
| `p1 * p2` | 序列 | 连接 |
| `p1 + p2` | `lp_choice` | 有序选择 |
| `p ^ 0` | 重复 0+ | Kleene star |
| `p ^ 1` | 重复 1+ | |
| `p ^ -1` | 可选 | |
| `-p` | `lp_not` | 否定前瞻 |
| `#p` | `lp_and` | 与前瞻 |
| `p1 - p2` | 差集 | |
| `p / fn` | 除捕获 | 语义动作 |

**其他**

| 函数 | 返回 | 说明 |
|------|------|------|
| `match(p, subject [, init [, ...]])` | captures | 对 subject 匹配 |
| `type(v)` | string | `"pattern"` 等 |
| `version()` | string | LPeg 版本 |
| `setmaxstack(n)` | — | 回溯栈上限 |
| `B(p)` | pattern | 反向匹配 |
| `ptree(p)` / `pcode(p)` | — | 调试打印 |

### 伪代码

```lua
--[[
-- @file   lpeg.lua
-- @brief  require("lpeg") — LPeg 1.0.2
--   \n C: ./klb/src_c/klua/lpeg-1.0.2/lptree.c
--]]

local lpeg = {}


-- @brief 构造模式 (字符串/单字节/模式/函数/语法表)
-- @param [in] v[any]
-- @return [pattern] userdata
lpeg.P = function (v)
	return v
end


-- @brief 字符集合
-- @param [in] set[string]			eg. " \t\n" 或 "abc"
-- @return [pattern]
lpeg.S = function (set)
	return set
end


-- @brief 字符范围 (可多个范围串)
-- @param [in] range[string]			eg. "az", "09"
-- @return [pattern]
lpeg.R = function (range)
	return range
end


-- @brief 区域类
-- @param [in] cat[string]			[可选] eg. "alpha", "alnum"
-- @return [pattern]
lpeg.locale = function (cat)
	return cat
end


-- @brief 语法变量 (配合 lpeg.P{ ... })
-- @param [in] name[string]
-- @return [pattern]
lpeg.V = function (name)
	return name
end


-- @brief 捕获匹配 / 常量 / 命名组 / table / 替换 / 折叠
lpeg.C = function (p) return p end
lpeg.Cc = function (...) return ... end
lpeg.Cg = function (p, name) return p end
lpeg.Ct = function (p) return p end
lpeg.Cs = function (p) return p end
lpeg.Cf = function (p, fn) return p end


-- @brief 反向引用 / 额外参数 / 位置 / 匹配时间函数
lpeg.Cb = function (name) return name end
lpeg.Carg = function (n) return n end
lpeg.Cp = function () return 0 end
lpeg.Cmt = function (p, fn) return p end


-- @brief 反向匹配
lpeg.B = function (p) return p end


-- @brief 对 subject 执行匹配
-- @param [in] p[pattern]
-- @param [in] subject[string]
-- @param [in] init[number(int)]		[可选] 起始位置, 默认 1
-- @param [in] ...[any]				[可选] Carg 额外参数
-- @return [any] 捕获结果; 失败 nil
-- @note eg. lpeg.match(lpeg.P("hello"), "say hello")
lpeg.match = function (p, subject, init, ...)
	return nil
end


-- @brief 是否 pattern / 版本 / 回溯栈上限 / 调试
-- @return [string] "pattern" 或 nil
lpeg.type = function (v)
	return "pattern"
end

lpeg.version = function ()
	return "1.0.2"
end

lpeg.setmaxstack = function (n)
	return
end

lpeg.ptree = function (p)
	return
end

lpeg.pcode = function (p)
	return
end


-- pattern 组合运算符 (metamethods): * 序列, + 选择, ^ 重复, - 差集, # 与前瞻, -p 非前瞻, /fn 语义动作


return lpeg
```

### 示例

```lua
local lpeg = require("lpeg")

-- 简单解析
local num = lpeg.R("09") ^ 1
local ws = lpeg.S(" \t") ^ 0
local pat = ws * num * ws
local n = lpeg.match(pat, "  42  ")
print(n)	-- "42"

-- 捕获为 table
local pair = lpeg.Cg(lpeg.R("az") ^ 1, "key") * "=" * lpeg.Cg(lpeg.R("09") ^ 1, "val")
local t = lpeg.match(pair, "x=10")
-- t.key == "x", t.val == "10"

-- 语法 (递归)
local g = lpeg.P {
	"Exp",
	Exp = lpeg.V("Num") + "(" * lpeg.V("Exp") * ")",
	Num = lpeg.R("09") ^ 1,
}
```

### 注意

- 裁剪: `__KLB_NO_LPEG__` ([klb/clip.mk](https://gitee.com/klua/klb/blob/trunk/clip.mk) → `no-lpeg`); 未定义时默认编入
- pattern 为 userdata, 用 `*` / `+` / `^` 等组合 (metamethods)
- vendor 内 `re.lua` 为 PEG 语法糖, **未** 经 `klua_loadlib` 预加载; 需要时自行 `loadfile`
- 协议: `klua_doc/klb/licenses/LICENSE-lpeg-1.0.2`
