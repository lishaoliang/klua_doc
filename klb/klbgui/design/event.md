# 消息与事件

> `klua_doc/klb/klbgui/design/event.md` — 头文件: [klb/inc/klbgui/klb_msg.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_msg.h), [klb/inc/klbgui/klbui_event.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_event.h); 消息队列: [klb/src_c/klbgui/klb_gui_in.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_gui_in.h)

API [../api/klb_msg.md](../api/klb_msg.md). 分层 [layers.md](layers.md).

## 结论

对话中 **消息 ≈ 事件**（同一套 UI 交互描述）。分两层理解：

| 层 | 头文件 | 内容 |
|----|--------|------|
| 底层输入 | [klb/inc/klbgui/klb_msg.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_msg.h) | Windows 风格 `KLB_WM_*`、`KLB_VK_*`；投递到 GUI 消息队列 |
| 框架事件 | [klb/inc/klbgui/klbui_event.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_event.h) | H5 风格命名 `KLBUI_*`；窗口生命周期与控件回调 |

仅当区分「外设原始消息」与「窗口事件常量」时分开说明。

## 消息队列 (`klb_gui`)

```
平台 loop / wlua
  → klb_gui_push_msg(p_gui, msg, x1, y1, x2, y2, lparam, wparam)
  → p_msg_list (klb_msg_t 链表, 带 mutex)
  → klb_gui_loop_once 内分发 → KLBUI_* 事件
```

| API | 说明 |
|-----|------|
| `klb_gui_push_msg` | 投递消息 |
| `klb_gui_clear_msg` | 清空队列 |

`klb_msg_t` 字段：`msg`, `pt1`, `pt2`, `lparam`, `wparam`（定义于 [klb/src_c/klbgui/klb_gui_in.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_gui_in.h)）。

**等待状态**：`klb_gui_wait` 启用后 GUI **丢弃**键鼠等外设消息（见 [layer.md](layer.md) § waitlayer）。

## 窗口生命周期

来源 [klb/inc/klbgui/klbui_event.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_event.h) @note：

```
create / append / push_child
  → modal / popup / messagebox 压栈
  → KLBUI_onload
  → KLBUI_onpredraw          // 首次绘制前最后调整布局
  → klb_wnd_draw (框架内部)
  → KLBUI_onpaint
  → click / focus / …        // 用户交互
  → modal_end / popup_end / messagebox_end
  → KLBUI_onunload
  → destroy
```

- `create`/`destroy` 一般各执行一次
- `onpredraw`：部分控件须在 **全部 CSS 设置完成** 后布局（亦见 `KLBUI_onparsewindow`）

## 事件分区 ([klb/inc/klbgui/klbui_event.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_event.h))

| 区间 | 示例 |
|------|------|
| 0x401～0x411 | 键鼠：`KLBUI_click`, `mousedown`, `mousedrag`, `mousewheel`… |
| 0x450 | `KLBUI_outwindow` — popup 外点击 |
| 0x500～0x520 | 系统：`onabort`, `onerror`, `onpredraw`, `onpaint` |
| 0x570～0x571 | 定时器：`ontimer`, `onticker` |
| 0x580～0x581 | Lua 解析：`onparsewindow`, `onparsedialog` |
| 0x601～0x604 | 控件：`onload`, `onunload`, `onresize`, `onchange` |
| 0x700～0x710 | 焦点：`focusin`, `focus`, `blur`, `focusdelay` |
| 0x861 | `layout` — 控件内部重布局 |
| 0x3A00+ | `KLBUI_event_ctrl` — 组件私有事件 |
| 0x4000+ | `KLBUI_event_user` — 用户自定义 |

键鼠事件参数约定见头文件 @note（`pt1` 坐标、`lparam` 按键等）。

## 与 Lua / klbuiex 的关系

| C | 脚本 |
|----|------|
| C | [klb/inc/klbgui/klbui_event.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_event.h) / `KLBUI_*` | `klbcore/klbui/event.lua` 字符串名 |
| `klb_gui_bind_command` | `commands[path].click` 等 |
| `klb_gui_loop_once` | env / `kgui` 主循环 |

Lua 事件名与 C 常量对照见 [lua/klua/kgui.md](../../../lua/klua/kgui.md).

## 关联

| 主题 | 入口 |
|------|------|
| GUI 主循环 | [extension.md](extension.md) § loop |
| 聚焦 / focusdelay / tip | [wnd.md](wnd.md), [tip.md](tip.md) |
| onticker | [ticker.md](ticker.md) |
| 底层 WM 宏 | [../api/klb_msg.md](../api/klb_msg.md) |
| klua env GUI | [../../klua/design/env-extension.md](../../klua/design/env-extension.md) |
