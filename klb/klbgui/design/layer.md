# 图层

> `klua_doc/klb/klbgui/design/layer.md` — 头文件: `klb/inc/klbgui/klbui_layer.h`, `klb/inc/klbutil/klb_canvas.h`; 实现: `klbuiex_render.c`, `klbuiex_*layer.c`

画布契约 [canvas.md](canvas.md). API [../api/klbui_layer.md](../api/klbui_layer.md).

## UI 图层栈（自顶向下）

```
        *-----------------*
        |   mouse         |  ------- 图形驱动 / 输入
        *-----------------*

        *-----------------*
        |   tip           |  ------- 独立画布 (KLB_CANVAS_LAYER_tip)
        *-----------------*

        *-----------------*
        |   wait          |  ------- 等待遮罩 (KLB_CANVAS_LAYER_wait)
        *-----------------*

        *-----------------*
        |   udata         |  ------- 用户自定义 (KLB_CANVAS_LAYER_udata)
        *-----------------*

        *-----------------*
        |   message box   |  ---*
        *-----------------*     |
        |   popup         |  ---|--- 主画布或多独立画布
        *-----------------*     |
        |   modal         |  ---*
        *-----------------*
```

modal / popup / msgbox 可在 **单主画布** 上绘制，也可在 **多图层模式** 下使用独立 `klb_canvas_t`（见下节）。

## 两套对齐关系

| 维度 | 窗口样式 (`klb_wnd.h`) | 画布图层 (`klb_canvas_layer_type_e`) | 扩展 |
|------|------------------------|--------------------------------------|------|
| popup | `KLB_WND_STYLE_LAYER_POPUP` | `KLB_CANVAS_LAYER_popup` | render |
| msgbox | `KLB_WND_STYLE_LAYER_MSGBOX` | `KLB_CANVAS_LAYER_msgbox` | render |
| tip | `KLB_WND_STYLE_LAYER_TIP` | `KLB_CANVAS_LAYER_tip` | `klbuiex_tip` |
| udata | `KLB_WND_STYLE_LAYER_UDATA` | `KLB_CANVAS_LAYER_udata` | `klbuiex_udatalayer` |
| wait | `KLB_WND_STYLE_LAYER_WAIT` | `KLB_CANVAS_LAYER_wait` | `klbuiex_waitlayer` |
| main | （默认） | `KLB_CANVAS_LAYER_main` | 主画布 |

`klb_gui_set_wnd_layer_type(p_wnd, layer_type)`（`klb_gui_in.h`）设置窗口对应画布层类型。

## 单画布 vs 多画布

| 模式 | 行为 |
|------|------|
| 单图层 | main 上绘制 modal/popup/msgbox；tip 等仍可用独立层 |
| 多图层 | `klbuiex_render` 为 popup/msgbox 等分配子画布；`klb_canvas_refresh_layer` 合成 |

查询：`klb_gui_is_multi_canvas_layer(p_gui)` → `klbuiex_render_is_multi_layer`。

`klb_gui_attach_canvas` 时框架尝试为 tip / udata / wait 扩展挂接子画布（`klb_gui.c`）。

## udata / wait 图层 API

公开于 `klbui_layer.h`，由 `klbuiex_udatalayer` / `klbuiex_waitlayer` 实现。详见 [../api/klbui_layer.md](../api/klbui_layer.md).

要点：

- **udata**：绑定顶层窗口 path 或 `klb_wnd_t*`；可 `move` / `show`
- **wait**：绑定窗口 + `klb_gui_wait(p_gui, true)` — 启用后 **丢弃外设消息**；可用等待层窗口定时绘图

## 关联

| 主题 | 入口 |
|------|------|
| 画布刷新 | [canvas.md](canvas.md) § 多图层 |
| render 扩展 | [extension.md](extension.md) |
| modal/popup API | [../api/klb_gui.md](../api/klb_gui.md) |
