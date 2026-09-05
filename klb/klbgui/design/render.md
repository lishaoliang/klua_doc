# 渲染与重绘

> `klua_doc/klb/klbgui/design/render.md` — 实现: [klb/src_c/klbgui/extensions/klbuiex_render.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_render.c), [klb/src_c/klbgui/extensions/klbuiex_redraw.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_redraw.c); 头文件: [klb/src_c/klbgui/extensions/klbuiex_render.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_render.h), [klb/src_c/klbgui/extensions/klbuiex_redraw.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_redraw.h)

扩展总览 [extension.md](extension.md). 画布 [canvas.md](canvas.md). 窗口 [wnd.md](wnd.md).

## 结论

UI 绘制分 **记录脏区**（`klbuiex_redraw`）与 **合成刷新**（`klbuiex_render`）。`klb_gui_loop_once` 消息处理结束后 step 6 调用 `klbuiex_render_redraw_and_refresh`。2025-6 图形渲染从 GUI 核心迁入上述两扩展。

## 模块分工

| 扩展 | 职责 |
|------|------|
| `KLBUIEX-redraw` | 记录 `klb_wnd_update` 标记的窗口；剔除父子重复；特殊场景改全量重绘 |
| `KLBUIEX-render` | 按 modal/popup/msgbox 栈绘制；多画布 attach；`refresh_layer` 合成 |

`klb_gui_create` 立即激活两者（`p_redraw` / `p_render` 缓存于 `klb_gui_t`）。

## redraw 流程

来源 [klb/src_c/klbgui/extensions/klbuiex_redraw.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_redraw.h) @note：

1. 控件或框架调用 `klb_wnd_update` → `klbuiex_redraw_push`
2. 同一 `loop_once` 内消息处理完毕后再批量重绘
3. 记录列表中若存在父子（祖孙）关系，**只保留父窗口**
4. 遇 modal/popup/msgbox 等特殊场景 → `klbuiex_redraw_all` 放弃增量列表

| API | 说明 |
|-----|------|
| `klbuiex_redraw_push` | 加入待重绘窗 |
| `klbuiex_redraw_all` | 标记全量（modal 压栈等路径会调） |
| `klbuiex_redraw_clear` | 清空记录 |
| `klbuiex_redraw_need_repaint` | 查询是否有待绘 |

## render 流程

### 单图层 vs 多图层

| 模式 | `is_multi_layer` | 行为 |
|------|------------------|------|
| 单图层 | false | modal/popup/msgbox 均在 **主画布** 上 `klb_wnd_draw` |
| 多图层 | true | popup/msgbox 各持独立子画布；`refresh_layer` 合成 |

`klb_gui_attach_canvas` → `klbuiex_render_try_attach_canvas`：尝试 `malloc` popup/msgbox 子层。查询：`klb_gui_is_multi_canvas_layer` → `klbuiex_render_is_multi_layer`。

tip/udata/wait 子画布由各自扩展 attach（[layer.md](layer.md)）。

### 绘制次序（全量 `redraw_all`）

自底向上：

1. modal 栈 `p_modal_wnd[0..modal_num-1]`
2. popup 栈 `p_popup_wnd[0..popup_num-1]`
3. `p_msg_box`
4. tip 层（独立扩展，`klbuiex_tip_redraw`）

多图层 popup/msgbox 在各自 `p_popup_canvas[i]` / `p_msgbox_canvas` 上绘制后再合成。

### 与 loop 衔接

```
klb_gui_loop_once
  step 2  处理消息队列（可能 klb_wnd_update / redraw_push）
  step 3  focusdelay → tip
  step 4  扩展 cb_loop_once（含 wndticker）
  step 5  clear_async
  step 6  klbuiex_render_redraw_and_refresh   ← 本模块
```

`klb_gui_set_redraw_full_event(true)` 时，鼠标移动等也可触发 step 6 前的完整重绘路径（经 `klbuiex_util.is_redraw_full`）。

### popup/msgbox 多画布

压栈时：

- `klbuiex_render_popup_wnd(render, idx, p_top)` — 更新 popup 子画布位置
- `klbuiex_render_msgbox_wnd(render, p_top)` — 同上 msgbox

## 窗口侧入口

| 调用 | 效果 |
|------|------|
| `klb_wnd_update` | redraw_push |
| `klb_gui_update` | 全树 redraw_all |
| modal/popup 压栈 | clear_msg + redraw_all + render  reposition |

绘制 API 见 [draw.md](draw.md)；底层契约见 [canvas.md](canvas.md).

## 关联

| 主题 | 入口 |
|------|------|
| 图层栈 | [layer.md](layer.md) |
| modal 栈 | [wnd.md](wnd.md) § modal |
| 扩展注册 | [extension.md](extension.md) |
