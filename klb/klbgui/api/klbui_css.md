# CSS C API 导读

> `klua_doc/klb/klbgui/api/klbui_css.md` — 头文件: `klb/inc/klbgui/klbui_css.h`, `klbui_css_ex.h`

架构 [../design/css.md](../design/css.md). Lua CSS 属性表见 [lua/klbcore/css/klbui_css.md](../../../lua/klbcore/css/klbui_css.md).

## 头文件

| 头文件 | 说明 |
|--------|------|
| `klbui_css.h` | 盒模型结构体、visibility、CSS map API；结构体 4 字节对齐 |
| `klbui_css_ex.h` | `klbuicssex_*` 单属性操作与参考绘制（非「高级 CSS3 头」） |

实现：`klb/src_c/klbgui/klbui_css.c`, `klbui_css_ex.c`, `klbui_css_std_function.c`.

## 可见性

| 宏 | 说明 |
|----|------|
| `KLBUICSS_visibility_visible` | 可见（默认） |
| `KLBUICSS_visibility_hidden` | 隐藏；对应 `klb_wnd_t` `KLB_WND_STATUS_HIDE` |

## 盒模型结构体（节选）

| 类型 | 字段要点 |
|------|----------|
| `klbuicss_margin_t` | top/right/bottom/left |
| `klbuicss_padding_t` | 四边内边距 |
| `klbuicss_box_t` | max/min width/height（仅声明未引用） |
| `klbuicss_text_t` | color, text-align |
| `klbuicss_font_t` | font-size |
| `klbuicss_background_t` | color, image；私有 image_mode / image_flags |
| `klbuicss_border_t` | width/color 四边 |
| `klbuicss_outline_t` | outline style/width/color（仅声明未引用） |

Lua 合并写法示例：`["margin"] = {25, 50, 75, 100}`（见头文件注释）。

## GUI CSS map API

| API | 说明 |
|-----|------|
| `klb_gui_check_color(p_gui, p_map, start, p_out_color)` | 从 map 解析颜色 |
| `klb_gui_css_map(p_gui, type)` | 获取 type 对应 CSS 属性表 |
| `klb_gui_new_css_map(p_gui, type)` | 新建属性表 |
| `klb_gui_globalcss_map` / `new_globalcss_map` | 全局 CSS 表 |
| `klb_gui_globalcss_set_ptr` / `get_ptr` | 挂接自定义 CSS 对象 + 销毁回调 |
| `klb_gui_globalcss_set` / `get` | map 形式全局 CSS |

控件子类通常持有 `klb_map_t*` 或通过 wndhash 工厂注入 CSS 解析逻辑。

## 相关头文件（非本文件 API）

| 头文件 | 说明 |
|--------|------|
| `klbui_shwnd.h` | 共享窗口 CSS 模板 |
| `klbui_default.h` | 默认主题入口 |

## 查阅顺序

1. [../design/css.md](../design/css.md)
2. `klb/inc/klbgui/klbui_css.h`
3. `klb/bin/klbcore/klbui/csser.lua`
