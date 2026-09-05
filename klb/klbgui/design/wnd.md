# 窗口体系

> `klua_doc/klb/klbgui/design/wnd.md` — 头文件: [klb/inc/klbgui/klb_wnd.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd.h); 实现: [klb/src_c/klbgui/klb_wnd.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_wnd.c)

API [../api/klb_gui.md](../api/klb_gui.md) § klb_wnd. 分层 [layers.md](layers.md). 事件 [event.md](event.md). 盒模型 [css.md](css.md).

## 结论

`klb_wnd_t` 是 klbgui **② 窗口基类**：树形链表 + vtable + 样式/状态位；**不携带 CSS 字段**（体积最小化）。具体控件在 `ctrl[]` 柔性数组扩展子类。path 寻址与 type 创建由 **wndhash** 扩展负责（见 [wndhash.md](wndhash.md)）。

## 窗口树

```
klb_gui_t
  └── 顶层窗口 (KLB_WND_STYLE_TOP)
        ├── p_child → 子窗口链
        │     p_next / p_prev 兄弟链表
        └── ...
```

| 字段 | 说明 |
|------|------|
| `p_parent` / `p_child` | 父子 |
| `p_prev` / `p_next` | 兄弟双向链 |
| `p_gui` | 所属 GUI |
| `ctrl[]` | 控件私有数据；`KLB_WIDGETS_PTR` / `KLB_WND_PTR` 互转 |

扩展开发可 `klb_wnd_push_child` 自行挂子树（2023-4）。销毁：`klb_wnd_destroy_tree` / `KLB_FREE_WND`。

## 坐标

| 字段 | 含义 |
|------|------|
| `pos.rect_in_parent` | 相对父窗口；**含 margin** 的实际占位 |
| `pos.rect_in_canvas` | 相对画布（屏幕）；绘制与命中测试用 |

左上角为原点。`klb_wnd_move` / `resize` 改 `rect_in_parent`；`klb_wnd_update_canvas_rect` 标记需重算画布坐标。与 CSS 盒模型关系见 [css.md](css.md).

## vtable (`klb_wnd_vtable_t`)

| 回调 | 职责 |
|------|------|
| `destroy` | 释放控件体 + 窗口 |
| `on_control` | 控件内部消息处理 |
| `on_command` | 外部绑定（`klb_wnd_bind_command`） |
| `on_paint` | 自定义绘制（优先于默认 draw） |
| `on_set` / `on_get` | map 形式属性读写 |

## 消息分发（非完整冒泡）

简化模型：**当前聚焦窗口** + **其顶层激活窗口** 响应键鼠等消息。

| 样式 | 作用 |
|------|------|
| `KLB_WND_STYLE_PEEK_EVENT` | 父窗口需读取部分子窗口事件 |
| `KLB_WND_STYLE_NOFOCUS` | 不参与聚焦 |
| `KLB_WND_STYLE_NOCOMMAND` | `bind_command` 不生效 |

框架可 `klb_gui_drop_msg_dispatch(p_gui, true)` 终止本轮消息分发（[klb/src_c/klbgui/klb_gui_in.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_gui_in.h)）。`klb_gui_wait` 启用时丢弃外设消息（见 [layer.md](layer.md)）。

调用链：`klb_wnd_call_control` → `on_control`；`klb_wnd_call_command` → `on_command`；`call_control_and_command` 两者依次。

## 样式 (`klb_wnd_style_e`)

| 标记 | 说明 |
|------|------|
| `TOP` | 顶层窗口（modal/popup 等须设） |
| `PEEK_EVENT` / `NOFOCUS` / `NOCOMMAND` | 见上 |
| `FOCUS_WITHOUT_REDRAW` | 聚焦不重绘 |
| `FOCUS_CONTINUE` | 继续向下找焦点 |
| `FOCUS_DELAY` | 聚焦后延时触发 `KLBUI_focusdelay` + tip |
| `TICKER` / `TICKER_TOPMOST` | 控件定时器；见 [ticker.md](ticker.md) |
| `LAYER_POPUP` / `MSGBOX` / `UDATA` / `WAIT` / `TIP` | 图层标记；见 [layer.md](layer.md) |

## 状态 (`klb_wnd_status_e`)

| 标记 | 说明 |
|------|------|
| `HIDE` | 隐藏（对应 CSS visibility） |
| `INPUT` / `CHECK` / `DISABLE` | 输入 / 选中 / 不使能 |
| `TOPMOST` | 当前激活栈最顶层（框架设置） |
| `FOCUS` | 鼠标聚焦 |
| `TIP_DYNAMIC` | 需重算动态 tip |
| `RESIZE` | 尺寸变更待控件处理布局 |
| `CANVAS_RECT` | 需重算 `rect_in_canvas` |

`show`/`hide` 与 `klb_wnd_update` 配合：`show` 附带刷新标记；`hide` 仅改状态。

## 聚焦

| 项 | 说明 |
|----|------|
| `p_focus` | 当前聚焦控件 |
| `p_focus_top` | 聚焦所在顶层窗 |
| `focus_tc` | 聚焦时刻（ms tick） |
| `focusdelay` / `focusdelay_tc` | 延时 tip 标志与间隔 |

`klb_gui_create` 默认 `focusdelay_tc = 300` ms；`klb_gui_set_focusdelay` 可改。延时到期：发 `KLBUI_focusdelay` → 显示 tip（见 [tip.md](tip.md)）。

API：`klb_gui_get_focus` / `get_focus_top`。

## modal / popup / messagebox 栈

定义于 [klb/src_c/klbgui/klb_gui_in.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_gui_in.h)；API 见 [../api/klb_gui.md](../api/klb_gui.md).

| 类型 | 栈深上限 | 画布层（多图层时） |
|------|----------|-------------------|
| modal | `KLBUI_MODAL_WND_MAX` = **16** | 单图层时 main；多图层见 [render.md](render.md) |
| popup | `KLBUI_POPUP_WND_MAX` = **4** | `KLB_CANVAS_LAYER_popup` |
| messagebox | 1（`p_msg_box`） | `KLB_CANVAS_LAYER_msgbox` |

流程概要：

```
klb_gui_modal(path) / popup / messagebox
  → wndhash_find 顶层窗
  → clear_msg（切换绘制上下文，防残留消息）
  → 压栈 p_modal_wnd[] / p_popup_wnd[] / p_msg_box
  → klb_gui_set_wnd_layer_type
  → do_push_stack_top_wnd → onload → onpredraw → 绘制
  → wndticker_modal/popup/msgbox（定时器窗表）
  → render_popup_wnd / render_msgbox_wnd（多画布 reposition）
```

结束：`modal_end` / `popup_end` / `messagebox_end` → `onunload` → 出栈。绘制次序自底向上：modal 栈 → popup 栈 → msgbox（见 [klb/src_c/klbgui/extensions/klbuiex_render.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_render.c) `redraw_all`）。

须 **`KLB_WND_STYLE_TOP`** 且尚未在栈中。popup 外点击 → `KLBUI_outwindow`（[event.md](event.md)）。

## 建议尺寸

`klb_wnd_suggestw` / `suggesth`：控件自报测量值。**禁止**作为 BoxFlow 布局输入（见 [layout.md](layout.md)）。

## 扩展开发要点

| API | 用途 |
|-----|------|
| `klb_wnd_push_child` | 挂子窗口 |
| `klb_wnd_bind_paint` | 替换绘制 |
| `klb_wnd_draw_*` / `draw_opt*` | 基础/扩展绘图 → [draw.md](draw.md) |
| `klb_wnd_set` / `get` | map 属性 |
| `klb_wnd_update` | 标记脏区 → [render.md](render.md) § redraw |

## 关联

| 主题 | 入口 |
|------|------|
| path / type 工厂 | [wndhash.md](wndhash.md) |
| 绘制与刷新 | [draw.md](draw.md), [render.md](render.md) |
| 图层 | [layer.md](layer.md) |
| tip | [tip.md](tip.md) |
| 定时器 | [ticker.md](ticker.md) |
