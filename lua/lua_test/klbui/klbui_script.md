# 第 1 章 § 1.2 脚本层 (parse / select / css)

> API: [klbui.md](../../klbcore/klbui.md) ([`require("klbcore.klbui")`](../../klbcore/klbui.md)), [kgui.md](../../klua/kgui.md) | 枢纽: [readme.md](readme.md)
> 源码: `klua_run/lua_test/`klbui/script/ch1_s2_{z}.lua` → `lua_test.klbui.script.ch1_s2_{z}`

约定见 [readme.md](readme.md). 本节为 **P3 脚本层** 逻辑条 (`klua`, 可批量): 窗口树 / 选择器 / CSS 生效路径. **不** 测控件私有键 (见 1.3+). **不** 开 SDL.

`default_css` / `global_css` **须在控件创建前** 设置. `select` 返回函数 `f(s)`; 结果表含 `_wnds` (匹配数组) 与 `path()`.

旧节号 `1.3.*` / `1.4.*` (select / css 独立节) **已收回**, 不留兼容.

## lua 对照

| doc_id | 语义 id | lua `@brief` | 文件 |
|--------|---------|--------------|------|
| `1.2.1` | `klbui.require` | 1.2.1 require kgui / klbui | `script/ch1_s2_1.lua` |
| `1.2.2` | `klbui.parse.kview` | 1.2.2 parse kview root | `script/ch1_s2_2.lua` |
| `1.2.3` | `klbui.parse.child` | 1.2.3 parse kview + kstatic child | `script/ch1_s2_3.lua` |
| `1.2.4` | `klbui.select.name` | 1.2.4 select by name | `script/ch1_s2_4.lua` |
| `1.2.5` | `klbui.select.type` | 1.2.5 select by type | `script/ch1_s2_5.lua` |
| `1.2.6` | `klbui.select.id` | 1.2.6 select by id | `script/ch1_s2_6.lua` |
| `1.2.7` | `klbui.css.style` | 1.2.7 inline style title | `script/ch1_s2_7.lua` |
| `1.2.8` | `klbui.css.type` | 1.2.8 css.type title | `script/ch1_s2_8.lua` |
| `1.2.9` | `klbui.has_global_css` | 1.2.9 has_global_css | `script/ch1_s2_9.lua` |

---

## 1.2 脚本层

### 解析

### 1.2.1 模块 require

- `klbui.require`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.1` |
| CLI | `1.2.1` / `klbui.require` |
| 模块 | `lua_test.klbui.script.ch1_s2_1` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. [`require("kgui")`](../../klua/kgui.md) (失败 skip); 2. [`require("klbcore.klbui")`](../../klbcore/klbui.md) (失败 skip); 3. 检查 `type(klbui.parse) == "function"` 且 `type(kgui.append) == "function"`

预期: 两模块均可 require; `parse` / `append` 为函数

失败: require 失败未 skip; 导出不是函数

---

### 1.2.2 解析 kview 根

- `klbui.parse.kview`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.2` |
| CLI | `1.2.2` / `klbui.parse.kview` |
| 模块 | `lua_test.klbui.script.ch1_s2_2` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. require 同 1.2.1 (失败 skip); 2. [`klbui.parse`](../../klbcore/klbui.md)({ `path`=`"/home"`, `type`=`"kview"`, `pos`=`{0,0,320,240}` }); 3. [`kgui.get_kwnd`](../../klua/kgui.md)(`"/home"`)

预期: `dialog.path == "/home"`; `get_kwnd` 非 `nil`

失败: `append`/`parse` 失败; `get_kwnd` 为 `nil`

注: 本节顶层用 **kview**. **不** 测 `kdialog` type (见后续控件节).

---

### 1.2.3 解析子控件

- `klbui.parse.child`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.3` |
| CLI | `1.2.3` / `klbui.parse.child` |
| 模块 | `lua_test.klbui.script.ch1_s2_3` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. parse 根 `kview` `"/home"` + `child` 一项 `kstatic` (`pos` 非负); 2. 读子项回写的 `path`; 3. `kgui.get_kwnd` 根与子

预期: 根与子 `get_kwnd` 均非 `nil`; 子 `path` 以 `"/home/"` 为前缀

失败: 子 type 未注册导致 `append` 失败; 子路径未回写

---

### 选择器

### 1.2.4 按 name

- `klbui.select.name`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.4` |
| CLI | `1.2.4` / `klbui.select.name` |
| 模块 | `lua_test.klbui.script.ch1_s2_4` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. require `klbcore.klbui` (失败 skip); 2. 构造 dialog (`kview` 根 `name`=`"root"` + `kstatic` 子 `name`=`"lab1"`); 3. `f = klbui.select(dialog)`; 4. `r = f("lab1")`

预期: `#r._wnds == 1` 且 `r._wnds[1].name == "lab1"`

失败: 未命中或命中多项

注: 本条 **可不** `parse` (只扫 table).

---

### 1.2.5 按 type

- `klbui.select.type`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.5` |
| CLI | `1.2.5` / `klbui.select.type` |
| 模块 | `lua_test.klbui.script.ch1_s2_5` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. 同 1.2.4 构造树 (根 `kview` + 子 `kstatic`); 2. `klbui.select(dialog, true)`; 3. `f(":kstatic")`

预期: 多选下 `_wnds` 仅 `kstatic` (至少 1 项); `f(":kview")` 含根

失败: type 选择混入其它 type

---

### 1.2.6 按 id

- `klbui.select.id`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.6` |
| CLI | `1.2.6` / `klbui.select.id` |
| 模块 | `lua_test.klbui.script.ch1_s2_6` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. 子控件 `id`=`"s1"`; 2. `f("#s1")`

预期: `#_wnds == 1` 且 `id == "s1"`

失败: `#` 规则未命中

---

### CSS

### 1.2.7 行内 style

- `klbui.css.style`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.7` |
| CLI | `1.2.7` / `klbui.css.style` |
| 模块 | `lua_test.klbui.script.ch1_s2_7` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. require (失败 skip); 2. parse `kview` `"/home"`, `style`=`{ title = "via-style" }`; 3. [`kgui.get`](../../klua/kgui.md)(`"/home"`, `"title"`)

预期: get 得到 `"via-style"` (或 seri 解包后的等价首值)

失败: style 未写入控件

---

### 1.2.8 css.type

- `klbui.css.type`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.8` |
| CLI | `1.2.8` / `klbui.css.type` |
| 模块 | `lua_test.klbui.script.ch1_s2_8` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. parse 时传入 `css = { type = { kview = { title = "via-type" } } }`; 2. `kgui.get("/home", "title")`

预期: title 为 `"via-type"`

失败: `csser.css` 未按 type 选择器生效

---

### 1.2.9 has_global_css

- `klbui.has_global_css`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.9` |
| CLI | `1.2.9` / `klbui.has_global_css` |
| 模块 | `lua_test.klbui.script.ch1_s2_9` |
| 状态 | **已实现** |
| registry | `batch_ok=true` |

步骤: 1. [`klbui.has_global_css`](../../klbcore/klbui.md)(`"kview"`); 2. 对未注册 type `"knotexist"` 再查一次

预期: `"kview"` 为 `true` (embed 已 `init_globalcss`); `"knotexist"` 为 `false`

失败: 已注册 type 报不支持; 或未注册 type 报支持

注: 不用 `kdialog` 当反例 (`kdialog` 现行已注册).
