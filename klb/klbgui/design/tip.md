# TIP 提示

> `klua_doc/klb/klbgui/design/tip.md` — 实现: [klb/src_c/klbgui/extensions/klbuiex_tip.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_tip.c), [klb/src_c/klbgui/shwnd/klbshw_tip.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/shwnd/klbshw_tip.c), [klb/src_c/klbgui/subviews/klbwnd_tip.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/subviews/klbwnd_tip.c); 头文件: [klb/src_c/klbgui/extensions/klbuiex_tip.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_tip.h)

窗口字段 [wnd.md](wnd.md). 共享窗 [shwnd.md](shwnd.md). 图层 [layer.md](layer.md). 事件 [event.md](event.md).

## 结论

TIP 分三层：**窗口数据**（`klb_wnd_tip_t`）→ **扩展**（`klbuiex_tip` + 独立 tip 画布）→ **共享 UI 模板**（`/klbui/tip`）。2025-7 机制重写；始终使用 **`KLB_CANVAS_LAYER_tip`** 独立图层。

## 窗口侧数据

```c
typedef struct klb_wnd_tip_t_ {
    sds title;      // 静态 tip
    sds dynamic;    // 动态 tip（运行期变更）
} klb_wnd_tip_t;
```

| API | 说明 |
|-----|------|
| `klb_wnd_set_tip` / `get_tip` | 静态文本 |
| `klb_wnd_set_tip_dynamic` / `get_tip_dynamic` | 动态文本 |
| `klb_wnd_dyntip` / `is_dyntip` | `KLB_WND_STATUS_TIP_DYNAMIC` |
| `klb_wnd_tip_update` | 标记需刷新（框架择时） |

控件须 **`KLB_WND_STYLE_FOCUS_DELAY`** 才走延时 tip 流程。

## 显示流程

```
鼠标移入 → 聚焦 p_focus
  → focus_tc 记录
  → focusdelay = true

klb_gui_loop_once step 3:
  若 focusdelay_tc 已过且非 wait:
    发 KLBUI_focusdelay
    取 tip: dynamic 优先，否则 static
    klbuiex_tip_set_title → move(鼠标x, 控件底+1) → show
```

聚焦窗隐藏则取消 focusdelay（2025 bugfix）。`klb_gui_update_tip` 可强制更新标题。

## klbuiex_tip 扩展

| API | 说明 |
|-----|------|
| `klbuiex_tip_try_attach_canvas` | 主画布 `malloc` tip 子层 |
| `klbuiex_tip_get_canvas` | tip 画布 |
| `klbuiex_tip_show` / `move` / `set_title` | 显示控制 |
| `klbuiex_tip_redraw` | 重绘 tip 层 |
| `klbuiex_tip_set_dirty` / `get_dirty` | 局部脏区 |

`klb_gui_attach_canvas` 时框架为 tip 扩展挂子画布（与 udata/wait 并列，[layer.md](layer.md)）。

## 共享 tip 窗

`klbui_shwnd_get_tip` 创建 path **`/klbui/tip`** 的顶层模板（[shwnd.md](shwnd.md)）。`klbshw_tip_set_title` / `klbshw_tip_layout` 供扩展内部布局。

## 关联

| 主题 | 入口 |
|------|------|
| 聚焦延时 | [wnd.md](wnd.md) § 聚焦 |
| 画布图层 | [canvas.md](canvas.md) § tip |
| 渲染合成 | [render.md](render.md) |
