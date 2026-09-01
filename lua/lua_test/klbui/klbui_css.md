# 第 1 章 § 1.4 CSS

> API: [klbui.md](../../klbcore/klbui.md) (`default_css` / `global_css` / `style`); 语法 [klbui_css.md](../../klbcore/css/klbui_css.md) | 枢纽: [readme.md](readme.md)
> 源码规划: `klua_run/lua_test/`klbui/css/ch1_s4_{z}.lua` → `lua_test.klbui.css.ch1_s4_{z}`

约定见 [readme.md](readme.md). 本节省 CSS 生效路径 (行内 `style` / `css.type` / `has_global_css`); 控件键语义见 1.5 与各 `klbui_k*.md`.

`default_css` / `global_css` **须在控件创建前** 设置.

---

## 1.4 CSS

### 1.4.1 行内 style

- `klbui.css.style`

| 项 | 值 |
|----|-----|
| doc_id | `1.4.1` |
| CLI | `1.4.1` / `klbui.css.style` |
| 模块 | `lua_test.klbui.css.ch1_s4_1` |
| 状态 | **待实现** |

步骤: 1. require (失败 skip); 2. parse `kview` `"/home"`, `style`=`{ title = "via-style" }`; 3. [`kgui.get`](../../klua/kgui.md)(`"/home"`, `"title"`)

预期: get 得到 `"via-style"` (或 seri 解包后的等价首值)

失败: style 未写入控件

---

### 1.4.2 css.type

- `klbui.css.type`

| 项 | 值 |
|----|-----|
| doc_id | `1.4.2` |
| CLI | `1.4.2` / `klbui.css.type` |
| 模块 | `lua_test.klbui.css.ch1_s4_2` |
| 状态 | **待实现** |

步骤: 1. parse 时传入 `css = { type = { kview = { title = "via-type" } } }`; 2. `kgui.get("/home", "title")`

预期: title 为 `"via-type"`

失败: `csser.css` 未按 type 选择器生效

---

### 1.4.3 has_global_css

- `klbui.has_global_css`

| 项 | 值 |
|----|-----|
| doc_id | `1.4.3` |
| CLI | `1.4.3` / `klbui.has_global_css` |
| 模块 | `lua_test.klbui.css.ch1_s4_3` |
| 状态 | **待实现** |

步骤: 1. [`klbui.has_global_css`](../../klbcore/klbui.md)(`"kview"`); 2. 对未注册 type (如 `"kdialog"`) 再查一次

预期: `"kview"` 为 `true` (embed 已 `init_globalcss`); `"kdialog"` 为 `false`

失败: 已注册 type 报不支持; 或未注册 type 报支持
