# 共享窗口 (shwnd)

> `klua_doc/klb/klbgui/design/shwnd.md` — 头文件: [klb/inc/klbgui/klbui_shwnd.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_shwnd.h); 实现: [klb/src_c/klbgui/extensions/klbuiex_shwnd.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_shwnd.c), [klb/src_c/klbgui/shwnd/klbshw_tip.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/shwnd/klbshw_tip.c)

扩展 [extension.md](extension.md) § shwnd. CSS [css.md](css.md). tip 详 [tip.md](tip.md).

## 结论

**共享窗口** 按 **path 关键字** 全局唯一，由框架管理生命周期；用于跨页面复用同一套 UI 模板（CSS map + 顶层窗体）。扩展名 **`KLBUIEX-shwnd`**；对外 API 在 [klb/inc/klbgui/klbui_shwnd.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_shwnd.h)。

## 对外 API

| API | 说明 |
|-----|------|
| `klb_gui_push_shwnd(p_gui, path, p_top_wnd)` | 登记共享顶层窗；path 冲突失败 |
| `klb_gui_get_shwnd(p_gui, path)` | 获取 |
| `klb_gui_shwnd_css_set` / `get` | 按 path 读写 CSS map |

约束（头文件 @note）：

1. push 后框架管理销毁
2. 同 path **唯一**
3. **`/klbui` 路径保留**给框架内置 UI

## 扩展实现

`klbuiex_shwnd_t` 持 `klb_hlist_t*` 存所有共享顶层窗。`KLBUI_EX_MSG_clear` / `quit` 时 `klb_wnd_destroy_tree` 释放。

`klb_gui_create` 末尾：

- **仍启用**：`klbui_shwnd_get_tip(p_gui)` → path `KLBSHW_tip` = `"/klbui/tip"`
- **已注释**：calendar / combomenu / decimal / messagebox 等 shwnd

## 内置 tip 共享窗

| 路径 | 源文件 | 说明 |
|------|--------|------|
| `/klbui/tip` | [klb/src_c/klbgui/shwnd/klbshw_tip.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/shwnd/klbshw_tip.c) | 框架 tip 浮层 UI 模板 |
| — | [klb/src_c/klbgui/subviews/klbwnd_tip.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/subviews/klbwnd_tip.c) | tip 子视图辅助 |

`klbshw_tip_layout`：按 max_w/max_h 重算 tip 窗大小。与 `klbuiex_tip` 扩展配合显示（[tip.md](tip.md)）。

## 与 globalcss / default 关系

| 机制 | 用途 |
|------|------|
| shwnd | 整窗模板 + path 级 CSS |
| `klb_gui_globalcss_*` | 跨控件全局 CSS map |
| `klb_gui_default_css_*` | 主题默认值 → [default.md](default.md) |

Lua：`klbui.shwnd_css` 对应 `klb_gui_shwnd_css_*`（**klbcore-design**）。

## 关联

| 主题 | 入口 |
|------|------|
| tip 显示流程 | [tip.md](tip.md) |
| CSS map | [css.md](css.md), [../api/klbui_css.md](../api/klbui_css.md) |
