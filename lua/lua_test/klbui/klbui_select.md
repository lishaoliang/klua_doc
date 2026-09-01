# 第 1 章 § 1.3 选择器 (select)

> API: [klbui.md](../../klbcore/klbui.md) (`klbui.select`) | 枢纽: [readme.md](readme.md)
> 源码规划: `klua_run/lua_test/`klbui/select/ch1_s3_{z}.lua` → `lua_test.klbui.select.ch1_s3_{z}`

约定见 [readme.md](readme.md). 选择器走 Lua table (`name` / `#id` / `.class` / `:type` / `*`); 本节省检索, **不** 测 `set`/`get` 键语义 (见 1.5).

`select` 返回函数 `f(s)`; 结果表含 `_wnds` (匹配数组) 与 `path()`.

---

## 1.3 选择器

### 1.3.1 按 name

- `klbui.select.name`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.1` |
| CLI | `1.3.1` / `klbui.select.name` |
| 模块 | `lua_test.klbui.select.ch1_s3_1` |
| 状态 | **待实现** |

步骤: 1. require `klbcore.klbui` (失败 skip); 2. 构造 dialog (`kview` 根 `name`=`"root"` + `kstatic` 子 `name`=`"lab1"`); 3. `f = klbui.select(dialog)`; 4. `r = f("lab1")`

预期: `#r._wnds == 1` 且 `r._wnds[1].name == "lab1"`

失败: 未命中或命中多项

注: 本条 **可不** `parse` (只扫 table).

---

### 1.3.2 按 type

- `klbui.select.type`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.2` |
| CLI | `1.3.2` / `klbui.select.type` |
| 模块 | `lua_test.klbui.select.ch1_s3_2` |
| 状态 | **待实现** |

步骤: 1. 同 1.3.1 构造树 (根 `kview` + 子 `kstatic`); 2. `klbui.select(dialog, true)`; 3. `f(":kstatic")`

预期: 多选下 `_wnds` 仅 `kstatic` (至少 1 项); `f(":kview")` 含根

失败: type 选择混入其它 type

---

### 1.3.3 按 id

- `klbui.select.id`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.3` |
| CLI | `1.3.3` / `klbui.select.id` |
| 模块 | `lua_test.klbui.select.ch1_s3_3` |
| 状态 | **待实现** |

步骤: 1. 子控件 `id`=`"s1"`; 2. `f("#s1")`

预期: `#_wnds == 1` 且 `id == "s1"`

失败: `#` 规则未命中
