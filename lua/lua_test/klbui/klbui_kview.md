# 第 1 章 § 1.5 kview

> API: [klbui.md](../../klbcore/klbui.md), [kgui.md](../../klua/kgui.md) (`set`/`get`); C `klb/src_packages/klbwui/embed_widgets/klbui_view.c` + `embed_wnd/klbwnd_view.c` | 枢纽: [readme.md](readme.md)
> 源码规划: `klua_run/lua_test/`klbui/kview/ch1_s5_{z}.lua` → `lua_test.klbui.kview.ch1_s5_{z}`

约定见 [readme.md](readme.md). 本节省 **kview** 已绑定键 (`title`, `background-color`, `margin` 等); CSS 语法通则见 1.4.

现行 bind: `margin` / `padding` / `background-color` / `background-image` / `border-width` / `border-color` / `scale9-image` / `title`.

---

## 1.5 kview

### 1.5.1 title 读写

- `klbui.kview.title`

| 项 | 值 |
|----|-----|
| doc_id | `1.5.1` |
| CLI | `1.5.1` / `klbui.kview.title` |
| 模块 | `lua_test.klbui.kview.ch1_s5_1` |
| 状态 | **待实现** |

步骤: 1. require (失败 skip); 2. parse `kview` `"/v"`, `title`=`"hello"`; 3. `kgui.get("/v", "title")` 应为 `"hello"`; 4. `kgui.set("/v", "title", "world")`; 5. 再 get

预期: 两轮 get 分别为 `"hello"` / `"world"`

失败: set/get 空或与写入不一致

---

### 1.5.2 background-color

- `klbui.kview.background`

| 项 | 值 |
|----|-----|
| doc_id | `1.5.2` |
| CLI | `1.5.2` / `klbui.kview.background` |
| 模块 | `lua_test.klbui.kview.ch1_s5_2` |
| 状态 | **待实现** |

步骤: 1. parse `kview` `"/v"`; 2. `kgui.set("/v", "background-color", {255, 10, 20, 30})`; 3. `kgui.get("/v", "background-color")`

预期: set `rc == 0`; get 非空 (表或等价 ARGB, 以 seri 解包为准)

失败: set 非 0; get 为空

注: **不** 比对像素; 只验绑定读写.

---

### 1.5.3 margin

- `klbui.kview.margin`

| 项 | 值 |
|----|-----|
| doc_id | `1.5.3` |
| CLI | `1.5.3` / `klbui.kview.margin` |
| 模块 | `lua_test.klbui.kview.ch1_s5_3` |
| 状态 | **待实现** |

步骤: 1. parse `kview` `"/v"`; 2. `kgui.set("/v", "margin", {1, 2, 3, 4})` (上右下左, 与 CSS 文档一致); 3. get `"margin"` 或分项 `margin-top` 等

预期: set `rc == 0`; get 能读回所写分量

失败: margin bind 未生效
