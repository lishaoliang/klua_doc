# 底层消息 API 导读

> `klua_doc/klb/klbgui/api/klb_msg.md` — 头文件: [klb/inc/klbgui/klb_msg.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_msg.h)

架构 [../design/event.md](../design/event.md). 框架事件常量见 [klb/inc/klbgui/klbui_event.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_event.h)（同 design 文档）.

## klb_msg.h

Windows `winuser.h` 风格键值与鼠标消息号的 **klb 拷贝**，供平台输入层与 GUI 消息队列使用。2023-1 起自定义 UI 回调统一称 **event**（见 `klbui_event.h`），本头保留底层 `WM`/`VK` 定义。

### 虚拟键 (`KLB_VK_*`)

范围约 `0x01`～`0xFF`。常用：

| 宏 | 说明 |
|----|------|
| `KLB_VK_LBUTTON` / `RBUTTON` / `MBUTTON` | 鼠标键 |
| `KLB_VK_BACK` / `TAB` / `RETURN` / `ESCAPE` | 编辑键 |
| `KLB_VK_SHIFT` / `CONTROL` / `MENU` | 修饰键 |
| `KLB_VK_LEFT`～`DOWN` / `HOME` / `END` | 方向与导航 |
| `KLB_VK_F1`～`F24` | 功能键 |
| `KLB_VK_NUMPAD0`～`DIVIDE` | 小键盘 |

完整列表见头文件。

### 鼠标消息 (`KLB_WM_*`)

| 宏 | 值 | 说明 |
|----|-----|------|
| `KLB_WM_MOUSEMOVE` | 0x0200 | 移动 |
| `KLB_WM_LBUTTONDOWN` / `UP` / `DBLCLK` | 0x0201～0x0203 | 左键 |
| `KLB_WM_RBUTTONDOWN` / `UP` / `DBLCLK` | 0x0204～0x0206 | 右键 |
| `KLB_WM_MBUTTONDOWN` / `UP` / `DBLCLK` | 0x0207～0x0209 | 中键 |
| `KLB_WM_MOUSEHWHEEL` | 0x020E | 横向滚轮 |
| `KLB_WM_USER` | 0x0400 | 私有消息起点 |

## klb_msg_t（内部结构）

定义于 [klb/src_c/klbgui/klb_gui_in.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_gui_in.h)（非对外头）：

```c
typedef struct klb_msg_t_
{
    int         msg;
    klb_point_t pt1;
    klb_point_t pt2;
    int         lparam;
    int         wparam;
} klb_msg_t;
```

## GUI 投递 API (`klb_gui.h`)

| API | 说明 |
|-----|------|
| `klb_gui_push_msg(p_gui, msg, x1, y1, x2, y2, lparam, wparam)` | 分配并入队 |
| `klb_gui_clear_msg(p_gui)` | 清空队列 |

平台侧（如 wlua/SDL）将 SDL 事件映射为 `KLB_WM_*` 后 `push_msg`；框架 `loop_once` 内转换为 `KLBUI_*` 分发给窗口树。

## 查阅顺序

1. [../design/event.md](../design/event.md)
2. [klb/inc/klbgui/klb_msg.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_msg.h)
3. [klb/inc/klbgui/klbui_event.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_event.h)
4. [klb/src_c/klbgui/klb_gui.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_gui.c) — 消息分发
