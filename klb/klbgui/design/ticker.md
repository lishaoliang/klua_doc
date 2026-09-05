# 控件定时器 (wndticker)

> `klua_doc/klb/klbgui/design/ticker.md` — 头文件: [klb/src_c/klbgui/extensions/klbuiex_wndticker.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_wndticker.h); 实现: [klb/src_c/klbgui/extensions/klbuiex_wndticker.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_wndticker.c)

扩展 [extension.md](extension.md). 事件 [event.md](event.md) § onticker. 窗口样式 [wnd.md](wnd.md).

## 结论

**控件定时器** 由扩展 **`KLBUIEX-wndticker`** 驱动：带 `KLB_WND_STYLE_TICKER` 的窗口在顶层窗 **激活显示** 时周期性收到 **`KLBUI_onticker`**。与 GUI 级 `klb_gui_get/set_ticker_interval` 联动（默认 **500** ms）。[klb/inc/klbgui/klbui_timer.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_timer.h) 为空头，逻辑均在本扩展。

## 窗口样式

| 样式 | 条件 |
|------|------|
| `KLB_WND_STYLE_TICKER` | 所属顶层窗处于激活即触发 |
| `KLB_WND_STYLE_TICKER_TOPMOST` | 须为激活栈 **最顶层** 才触发 |

窗口 **onload** 后入 ticker 列表；unload 移除。实现见 [klb/inc/klbgui/klb_wnd.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd.h) @history 2025-1。

## 与 modal/popup/msgbox

压栈/出栈时 wndticker 维护分栈列表：

| API | 时机 |
|-----|------|
| `klbuiex_wndticker_modal` / `modal_end` | modal 压栈/出栈 |
| `klbuiex_wndticker_popup` / `popup_end` | popup |
| `klbuiex_wndticker_msgbox` / `msgbox_end` | messagebox |
| `set_modal_num` / `set_popup_num` / `set_msgbox_num` | 同步栈深 |

各栈独立 `klb_nlist_t*` 存待 tick 窗口（`KLBUI_MODAL_WND_MAX=16`，`KLBUI_POPUP_WND_MAX=4`）。

## GUI API

| API | 说明 |
|-----|------|
| `klb_gui_get_ticker_interval` / `set_ticker_interval` | 全局 tick 间隔（ms） |
| `klb_gui_get_tick_count` | 当前 GUI 滴答 |

扩展内部：`klbuiex_wndticker_set_interval` / `get_interval`；在 `cb_loop_once` 中比对 `loop_tc` 派发 `KLBUI_onticker`。

## 事件

| 常量 | 说明 |
|------|------|
| `KLBUI_ontimer` | 0x570 区间 |
| `KLBUI_onticker` | 0x571；控件定时刷新 |

Lua 字符串名见 `klbcore/klbui/event.lua`（**klbcore-design**）。

## 关联

| 主题 | 入口 |
|------|------|
| loop 顺序 | [extension.md](extension.md) § loop |
| modal 栈 | [wnd.md](wnd.md) |
