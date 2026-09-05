# 窗口绘制 API

> `klua_doc/klb/klbgui/design/draw.md` — 头文件: [klb/inc/klbgui/klb_wnd.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd.h), [klb/inc/klbgui/klb_wnd_ex.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klb_wnd_ex.h); 实现: [klb/src_c/klbgui/klb_wnd.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_wnd.c), [klb/src_c/klbgui/klb_wnd_ex.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_wnd_ex.c)

画布契约 [canvas.md](canvas.md). API 详表 [../api/klb_canvas.md](../api/klb_canvas.md) § klb_wnd_ex. 刷新 [render.md](render.md).

## 结论

控件绘制走 **`klb_wnd_draw_*`** 包装 → 窗口当前 **`klb_canvas_t`**（`klb_wnd_get_canvas`，勿缓存指针）。自定义绘制可绑 `on_paint` 或走 **`klb_wnd_ex`** 扩充 opt（100～512，面向 ARM/嵌入式）。

## 绘制路径

```
klb_wnd_on_paint (若绑定)
  或 vtable 默认 draw
    → klb_wnd_draw_* / klb_wnd_draw_opt*
      → klb_canvas_* / vtable.draw_opt*
```

`klb_wnd_get_canvas` @note：框架可能按 modal/popup/图层切换画布。

## 基础 API (`klb_wnd.h`)

### 颜色与字体

`klb_wnd_set_draw_color` / `set_font_height` 等 → 转发当前画布。

### 图元（`*` 与 `*2` 两套）

| 系列 | 颜色参数 |
|------|----------|
| `klb_wnd_draw_clear` / `draw_line` / … | `uint32_t*` 可 NULL 用当前色 |
| `klb_wnd_draw_clear2` / `draw_line2` / … | 直接 `uint32_t color` |

支持：点、线、矩形、填充、文本、`draw_image` / `image_size`、`text_size`。

### 扩展槽

`klb_wnd_draw_opt1`～`opt8`：opt 编号 + 参数指针，由画布 vtable `draw_opt*` 实现。

## 扩充 API (`klb_wnd_ex.h`, 2026)

### 常量

| 宏 | 说明 |
|----|------|
| `KLB_CANVAS_COLOR_ZERO` | FB 透明色（视频打洞） |
| `KLB_CANVAS_PATH_TMPIMAGE` | 临时图 key `~/tmpimage` |

### draw_opt 编号（100～512）

| opt | 用途 |
|-----|------|
| 100/101 | 线段 / 多线段（含线宽） |
| 110/111 | 空心矩形 / 多个 |
| 120 | 填充矩形 |
| 130～133 | 图片缩放 / 色键 / 九宫格 / 九宫格+色键 |
| 150/151 | 文本 / 多行自动换行 |

对应函数：`klb_wndex_draw_line`、`draw_fill_rects2`、`draw_image_scale9` 等（见 [../api/klb_canvas.md](../api/klb_canvas.md)）。

### ioctrl

| opt | 用途 |
|-----|------|
| 20 | 图片信息 |
| 21 | 临时图片信息 |

GUI 直通：`klb_gui_canvas_ioctrl_opt8`。

## 辅助绘制 ([klb/inc/klbgui/klbui_util.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_util.h))

`klbuiutil_remove_margin/padding/border`：从 rect 扣除盒模型区域。

`klbuiutil_draw_rectangle`、三角箭头、`+`/`-`/`X` 等简易符号 — 控件常用，非 canvas 契约一部分。

## 与 CSS 绘制

带 CSS 控件在 `on_paint` 或 vtable draw 内读取 `klbuicssex_*` 属性；margin 区域须自行覆盖（[css.md](css.md)）。

## 关联

| 主题 | 入口 |
|------|------|
| 脏区与刷新 | [render.md](render.md) |
| P0 画布 vtable | [canvas.md](canvas.md) |
