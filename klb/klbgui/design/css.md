# C 侧 CSS 模型

> `klua_doc/klb/klbgui/design/css.md` — 头文件: `klb/inc/klbgui/klbui_css.h`, `klbui_css_ex.h`; 实现: `klb/src_c/klbgui/klbui_css*.c`

API [../api/klbui_css.md](../api/klbui_css.md). Lua 脚本 CSS 约定 [lua/klbcore/css/klbui_css.md](../../../lua/klbcore/css/klbui_css.md).

## 结论

klbgui CSS 参考 **CSS3 盒模型**（margin / border / padding / element）。**基础窗口 `klb_wnd_t` 不携带 CSS 字段**；带样式控件在子类或 wrapper 中持有 CSS map / 结构体。

`klbui_css.h` 结构体已**去除** `__KLB_GUI_CSS3__` 门控，字段固定为基础 CSS，并按 4 字节对齐（32 位）。`klbui_css_ex.h` 是 `klbuicssex_*` 操作函数。

## 盒模型

```
*---------------- margin ----------------*
|  *------------- border ----------------* |
|  |  *---------- padding --------------*  |
|  |  |  *-------- element ------------*  |  |
|  |  |  |      (窗口绘制区)          |  |  |
|  |  |  *-----------------------------*  |  |
|  |  *----------------------------------*  |
|  *----------------------------------------* |
*---------------------------------------------*
```

- `klb_wnd_t.pos.rect_in_parent` — 相对父窗口 **含 margin** 的实际区域
- `klb_wnd_t.pos.rect_in_canvas` — 相对画布的实际区域
- 控件绘制须覆盖 margin/border/padding/element；常设 `margin = 0`

## 主要结构体 (`klbui_css.h`)

| 结构体 | CSS 对应 | 现行 |
|--------|----------|------|
| `klbuicss_margin_t` | margin 四边 | 默认可用 |
| `klbuicss_padding_t` | padding 四边 | 默认可用 |
| `klbuicss_text_t` | `color`、`text-align`（left/center/right） | 默认可用 |
| `klbuicss_font_t` | `font-size` | 默认可用 |
| `klbuicss_background_t` | `background-color`、`background-image` | 默认可用 |
| `klbuicss_border_t` | `border-width`、`border-color` 四边 | 默认可用 |
| `klbuicss_box_t` | min/max-width/height | **仅声明未引用** |
| `klbuicss_layout_t` | `z-index` | **仅声明未引用** |
| `klbuicss_outline_t` | outline-* | **仅声明未引用** |
| `klbuicss_util_t` | `cursor` | **仅声明未引用** |

`klbuicssex_attributes_t`（`klbui_css_ex.h`）只嵌入 text/font/background/border。

属性名与 Lua `csser` 字符串键对齐（如 `"margin"`, `"background-color:focus"`）。

## 头文件分工 (`klbui_css.h` / `klbui_css_ex.h`)

| 头 | 职责 |
|----|------|
| `klbui_css.h` | 结构体与宏；CSS map / globalcss API；结构体 4 字节对齐 |
| `klbui_css_ex.h` | `klbuicssex_*` set/get 与参考绘制 |
| `klbui_css_std_function.c` | 控件通用命令/状态键（`move`、`resize`、`visibility`…） |

## 对齐与现行字段 (`klbui_css.h`)

已去除 `__KLB_GUI_CSS3__`。结构体用 `reserved` 补齐，32 位 sizeof 均为 4 的倍数、无隐式 pad。64 位下 `klbuicss_background_t` 因 `sds` 指针升为 8 字节对齐。

非 CSS 私有字段：`background.image_mode`（scale9）、`image_flags`（color_key）。

## 相对标准 CSS (`klbui_css.h`)

本模型贴近 CSS3 **属性名**，不做级联、选择器、`em`/`%`/`calc()`。`width`/`height`/`left`/`top` 走 wui 命令 `resize`/`move`，不在本头。

已有属性的常见缺值：`text-align` 无 justify；`visibility` 无 collapse。本头已不含 `font-style`/`font-weight`/`border-style`/`border-radius`。

嵌入式高相关、本头无：`overflow`、`opacity`、`font-family`、`text-decoration`、`vertical-align`、`box-sizing`、`display`、`position`、`background-size`、flex/`gap`、`box-shadow`、`:hover`。低相关不做：Grid、动画、`var()`、媒体查询。

补齐建议（未实施）：先接线死结构体；然后 `font-family`/`opacity`/`overflow`；布局类与现有 wnd 树重叠，不宜当补字段。

## 全局 CSS 与共享模板

| 机制 | 说明 |
|------|------|
| `klb_gui_css_map` / `new_css_map` | 按 type 取/建控件 CSS 属性表 |
| `klb_gui_globalcss_*` | 全局 CSS map（跨控件共享） |
| `klbui_shwnd` + `klb_gui_shwnd_css_*` | 按 path 共享 CSS 模板 |
| `klbuiex_default` | VS 深色系默认主题 |

Lua 解析：`klb/bin/klbcore/klbui/csser.lua`, `parser.lua`。

## 布局时机

部分控件须在 **全部 CSS 属性设置完成** 后才调整布局：

- C 流程：`KLBUI_onpredraw` 或控件 `layout` 事件
- Lua 流程：`KLBUI_onparsewindow` / `KLBUI_onparsedialog`（`klbui_event.h`）

## 工具

| 项 | 说明 |
|----|------|
| `klbui_util` / `klbuiex_util` | 去 margin/padding/border 算绘制区 |
| `klb_gui_css_map_append_std_function` | 向 map 注册标准解析函数（`klb_gui_in.h`） |

## 关联

| 主题 | 入口 |
|------|------|
| 窗口坐标 | [wnd.md](wnd.md) |
| 默认主题 | [lua/klbcore/css/klbui_default_css.md](../../../lua/klbcore/css/klbui_default_css.md) |
| 事件 onparse | [event.md](event.md) |
| Lua CSS 文档 ↔ C bind 台账 | ai 技能 **klbcore-klbui-css-design** / `reference.md` |
