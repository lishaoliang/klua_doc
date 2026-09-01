# 图层 API 导读

> `klua_doc/klb/klbgui/api/klbui_layer.md` — 头文件: `klb/inc/klbgui/klbui_layer.h`

架构 [../design/layer.md](../design/layer.md). 画布图层类型见 [../design/canvas.md](../design/canvas.md).

## klbui_layer.h

用户自定义图层与等待图层的高层 API；底层由 `klbuiex_udatalayer` / `klbuiex_waitlayer` 实现。

### 用户自定义图层 (udata)

| API | 说明 |
|-----|------|
| `klb_gui_udatalayer_bind(p_gui, path)` | 绑定顶层窗口路径，如 `/aaa` |
| `klb_gui_udatalayer_bind_wnd(p_gui, p_top)` | 绑定窗口；`p_top=NULL` 取消 |
| `klb_gui_udatalayer_move(p_gui, x, y)` | 移动图层 |
| `klb_gui_udatalayer_show(p_gui, show)` | 显示/隐藏 |

对应画布类型：`KLB_CANVAS_LAYER_udata`。窗口样式：`KLB_WND_STYLE_LAYER_UDATA`。

### 等待图层 (wait)

| API | 说明 |
|-----|------|
| `klb_gui_waitlayer_bind(p_gui, path)` | 绑定顶层窗口路径 |
| `klb_gui_waitlayer_bind_wnd(p_gui, p_top)` | 绑定窗口；`NULL` 取消 |
| `klb_gui_waitlayer_move(p_gui, x, y)` | 移动图层及窗口 |
| `klb_gui_wait(p_gui, wait)` | 启用等待：丢弃所有外设消息；若已绑定等待层窗口则由其定时绘图 |

对应画布类型：`KLB_CANVAS_LAYER_wait`。窗口样式：`KLB_WND_STYLE_LAYER_WAIT`。

### 相关 GUI API（非本头文件）

| API | 说明 |
|-----|------|
| `klb_gui_is_multi_canvas_layer` | 是否多独立画布模式 |
| `klb_gui_attach_canvas` | 挂主画布并尝试挂接 tip/udata/wait 子画布 |

## 查阅顺序

1. [../design/layer.md](../design/layer.md)
2. `klb/inc/klbgui/klbui_layer.h`
3. `klb/src_c/klbgui/extensions/klbuiex_udatalayer.c`, `klbuiex_waitlayer.c`
