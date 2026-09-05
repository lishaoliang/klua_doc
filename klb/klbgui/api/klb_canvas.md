# 画布 C API 导读

> `klua_doc/klb/klbgui/api/klb_canvas.md` — 头文件: [klb/inc/klbutil/klb_canvas.h](https://gitee.com/klua/klb/blob/trunk/inc/klbutil/klb_canvas.h), [klb/inc/klbgui/klb_wnd_ex.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd_ex.h)

架构 [../design/canvas.md](../design/canvas.md). 分层 [../design/layers.md](../design/layers.md).

## 头文件

| 头文件 | 说明 |
|--------|------|
| [klb/inc/klbutil/klb_canvas.h](https://gitee.com/klua/klb/blob/trunk/inc/klbutil/klb_canvas.h) | 画布对象、`vtable`、图层与刷新 |
| [klb/inc/klbgui/klb_wnd_ex.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd_ex.h) | 窗口侧扩充绘图 API 与 opt 编号 |

实现：[klb/src_c/klbutil/klb_canvas.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbutil/klb_canvas.c), [klb/src_c/klbutil/klb_canvas_argb8888.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbutil/klb_canvas_argb8888.c); [klb/src_c/klbgui/klb_wnd_ex.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_wnd_ex.c).

## klb_canvas.h — 软实现与包装 API

### 生命周期

| API | 说明 |
|-----|------|
| `klb_canvas_create(w, h, color_fmt)` | 软实现创建画布 |
| `klb_canvas_destroy` | 销毁 |

平台硬实现由产品创建并填充 `vtable`，通常不经 `create`。

### 绘制与资源（经 vtable）

| API | vtable 成员 |
|-----|-------------|
| `klb_canvas_set_draw_color` / `get_draw_color` | `set_draw_color` / `get_draw_color` |
| `klb_canvas_set_font_height` / `get_font_height` | `set_font_height` / `get_font_height` |
| `klb_canvas_load_font` / `unload_font` | `load_font` / `unload_font` |
| `klb_canvas_load_image` / `image_size` / `clear_image` | `load_image` / `image_size` / `clear_image` |
| `klb_canvas_draw_clear` | `draw_clear` |
| `klb_canvas_draw_point` / `draw_points` | `draw_point` / `draw_points` |
| `klb_canvas_draw_line` / `draw_lines` | `draw_line` / `draw_lines` |
| `klb_canvas_draw_rect` / `draw_rects` | `draw_rect` / `draw_rects` |
| `klb_canvas_draw_fill_rect` / `draw_fill_rects` | `draw_fill_rect` / `draw_fill_rects` |
| `klb_canvas_draw_text` / `text_size` | `draw_text` / `text_size` |
| `klb_canvas_draw_image` | `draw_image` |

### 刷新

| API | 说明 |
|-----|------|
| `klb_canvas_refresh_rect` / `refresh_rects` | 单区域 / 多区域刷到显存 |
| `klb_canvas_refresh` | 多画布数组合并刷新（旧接口） |
| `klb_canvas_refresh_layer` | **推荐**：`klb_canvas_layer_t[]` + `refresh_opt` |

### 子层与布局

| API | 说明 |
|-----|------|
| `klb_canvas_malloc(p_main, idx, rsv, layer_type)` | 主画布派生子层 |
| `klb_canvas_move` | 移动画布屏幕位置 |
| `klb_canvas_resize` | 在固定 `mem_len` 内改宽高 |
| （vtable `free`） | 释放子层 |

### 扩展

| API | 说明 |
|-----|------|
| `klb_canvas_draw_opt1`～`draw_opt8` | 自定义绘图 |
| `klb_canvas_ioctrl_opt8` | 设备 ioctrl |

GUI 直通：`klb_gui_canvas_ioctrl_opt8(p_gui, …)`。

## klb_canvas_vtable_t — 平台须实现

分组：锁 (`lock`/`unlock`)、颜色与字体、图集、基础图元、文本与图片、`draw_copy`、刷新三件套 (`refresh_rect`/`refresh_rects`/`refresh`/`refresh_layer`)、`move`/`resize`/`malloc`/`free`、扩展 `draw_opt*` / `ioctrl_opt8`。

未实现的 vtable 项由 [klb/src_c/klbutil/klb_canvas.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbutil/klb_canvas.c) 包装层判空返回失败。

## klb_wnd_ex.h — 窗口扩充绘图

面向嵌入式/跨平台；内部走 `klb_wnd_draw_opt*` + 画布 `draw_opt`。

### 宏与常量

| 宏 | 说明 |
|----|------|
| `KLB_CANVAS_COLOR_ZERO` | FB 透明色（视频控件打洞） |
| `KLB_CANVAS_PATH_TMPIMAGE` | 临时图片路径 key |
| `KLB_CANVAS_DRAW_OPT_*` | 100～512 绘图 opt 编号 |
| `KLB_CANVAS_IOCTRL_OPT_IMAGE` / `TMPIMAGE` | 图片信息 ioctrl |

### draw_opt 编号（节选）

| opt | 值 | 用途 |
|-----|-----|------|
| `KLB_CANVAS_DRAW_OPT_LINE` | 100 | 线段（含线宽） |
| `KLB_CANVAS_DRAW_OPT_LINES` | 101 | 多线段/折线 |
| `KLB_CANVAS_DRAW_OPT_RECT` / `RECTS` | 110/111 | 空心矩形 |
| `KLB_CANVAS_DRAW_OPT_FILL_RECTS` | 120 | 填充矩形 |
| `KLB_CANVAS_DRAW_OPT_IMAGE_*` | 130～133 | 缩放/色键/九宫格 |
| `KLB_CANVAS_DRAW_OPT_TEXT` / `TEXT_LINES` | 150/151 | 文本 / 多行 |

### 主要函数

| API | 说明 |
|-----|------|
| `klb_wndex_draw_line` / `draw_lines` / `draw_lines2` | 线段 |
| `klb_wndex_draw_rect` / `draw_rects` / `draw_rects2` | 边框 |
| `klb_wndex_draw_fill_rects` / `draw_fill_rects2` | 填充 |
| `klb_wndex_draw_image_resize` 等 | 图片绘制变体 |
| `klb_wndex_draw_text` / `draw_text_lines` | 文本 |

## 查阅顺序

1. [../design/canvas.md](../design/canvas.md)
2. [klb/inc/klbutil/klb_canvas.h](https://gitee.com/klua/klb/blob/trunk/inc/klbutil/klb_canvas.h)
3. [klb/src_c/klbutil/klb_canvas.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbutil/klb_canvas.c)
4. P0a 后端 [wlua/readme.md](../../../wlua/readme.md)
