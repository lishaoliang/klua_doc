# 第 1 章 § 1.2 解析 (parse)

> API: [klbui.md](../../klbcore/klbui.md) ([`require("klbcore.klbui")`](../../klbcore/klbui.md)), [kgui.md](../../klua/kgui.md) | 枢纽: [readme.md](readme.md)
> 源码规划: `klua_run/lua_test/`klbui/parse/ch1_s2_{z}.lua` → `lua_test.klbui.parse.ch1_s2_{z}`

约定见 [readme.md](readme.md). 本节省窗口树创建; **不** 测选择器/CSS 键语义 (见 1.3 / 1.4).

---

## 1.2 解析

### 1.2.1 模块 require

- `klbui.require`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.1` |
| CLI | `1.2.1` / `klbui.require` |
| 模块 | `lua_test.klbui.parse.ch1_s2_1` |
| 状态 | **待实现** |

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
| 模块 | `lua_test.klbui.parse.ch1_s2_2` |
| 状态 | **待实现** |

步骤: 1. require 同 1.2.1 (失败 skip); 2. [`klbui.parse`](../../klbcore/klbui.md)({ `path`=`"/home"`, `type`=`"kview"`, `pos`=`{0,0,320,240}` }); 3. [`kgui.get_kwnd`](../../klua/kgui.md)(`"/home"`)

预期: `dialog.path == "/home"`; `get_kwnd` 非 `nil`

失败: `append`/`parse` 失败; `get_kwnd` 为 `nil`

注: 顶层用 **kview** (已注册); **不用** `kdialog`.

---

### 1.2.3 解析子控件

- `klbui.parse.child`

| 项 | 值 |
|----|-----|
| doc_id | `1.2.3` |
| CLI | `1.2.3` / `klbui.parse.child` |
| 模块 | `lua_test.klbui.parse.ch1_s2_3` |
| 状态 | **待实现** |

步骤: 1. parse 根 `kview` `"/home"` + `child` 一项 `kstatic` (`pos` 非负); 2. 读子项回写的 `path`; 3. `kgui.get_kwnd` 根与子

预期: 根与子 `get_kwnd` 均非 `nil`; 子 `path` 以 `"/home/"` 为前缀

失败: 子 type 未注册导致 `append` 失败; 子路径未回写
