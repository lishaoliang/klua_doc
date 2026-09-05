# klbui_layout C API（桩）

> `klua_doc/klb/klbgui/api/klbui_layout.md` — 头文件（规划）: [klb/inc/klbgui/klbui_layout.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_layout.h)  
> 状态: **桩文档**；头文件与实现未落地

设计 [../design/layout.md](../design/layout.md). 窗口 [../design/wnd.md](../design/wnd.md).

## 结论

BoxFlow 布局 C API 规划挂 klbgui 核心；具体符号以实现为准。本文仅列占位，**非**现行 API。

## 规划符号（待定）

| API | 说明 |
|-----|------|
| `klbui_layout_flow(klb_wnd_t* p_container)` | 对单容器执行 BoxFlow |
| `klbui_layout_flow_tree(klb_wnd_t* p_root)` | 深度优先；仅 `layout:flow` 节点 |
| `klb_wnd_mark_layout_dirty(klb_wnd_t* p_wnd)` | 标记 flow 祖先需 relayout |

## 相关既有 API

| API | 说明 |
|-----|------|
| `klb_wnd_move` / `klb_wnd_resize` | 布局结果写入 |
| `klb_wnd_call_control(..., KLBUI_layout, ...)` | 现有 layout 事件 |
| `klb_gui_css_map` `layout` 键 | [klb/src_c/klbgui/klbui_css_std_function.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klbui_css_std_function.c) 递归触发 |

## 实现状态

未开始。落地后同步本文件与 [klb_gui.md](klb_gui.md) 交叉链接。
