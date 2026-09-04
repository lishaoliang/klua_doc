# klbgui C API 导读

> `klua_doc/klb/klbgui/api/klb_gui.md` — 头文件: `klb/inc/klbgui/`

面向 **C/C++ 产品** 与高级绑定开发.

> **Lua 脚本 API**: `require("kgui")` → [lua/klua/kgui.md](../../../lua/klua/kgui.md); `klbcore.klbui` → [lua/klbcore/readme.md](../../../lua/klbcore/readme.md). 分层 [layers.md](../design/layers.md).

## klb_gui.h — GUI 根对象

> 实现: `klb/src_c/klbgui/klb_gui.c`

### 生命周期

| API | 说明 |
|-----|------|
| `klb_gui_create(p_canvas)` | 创建; 注册 11 个 klbuiex; 可 `p_canvas=NULL` |
| `klb_gui_destroy` | quit 扩展反序 → destroy 激活表 |
| `klb_gui_loop_once(p_gui, tc)` | **须周期调用**; 消息/扩展/render |

### 附着

| API | 说明 |
|-----|------|
| `klb_gui_attach_canvas` / `get_canvas` | 主画布 |
| `klb_gui_attach_klua_env` / `get_klua_env` | 对应 `klua_env_t` |
| `klb_gui_attach_cppgui` / `get_cppgui` | C++ `CGui` |
| `klb_gui_is_multi_canvas_layer` | modal/popup/msgbox 独立画布模式 |

### 消息

| API | 说明 |
|-----|------|
| `klb_gui_push_msg` | 投递键鼠等消息 |
| `klb_gui_clear_msg` | 清空队列 |
| `klb_gui_canvas_ioctrl_opt8` | 画布直通 ioctrl (最多 8 参数) |

### 窗口树

| API | 说明 |
|-----|------|
| `klb_gui_register` / `get_creater` | 旧式 per-gui type 注册 |
| `klb_gui_create_wnd` | 按 type 创建 (未入树) |
| `klb_gui_append` | 按 path 加入树 (`/home/btn1`) |
| `klb_gui_find_wnd` / `remove` | 查找/移除 |
| `klb_gui_clear` / `clear_async` | 清 UI 数据; **勿在事件内同步 clear** |
| `klb_gui_bind_command` | path 绑定 `on_command` |
| `klb_gui_set` / `get` / `show` / `move` / `resize` | 控件数据与布局 |

### modal / popup / messagebox

| API | 说明 |
|-----|------|
| `klb_gui_modal` / `modal_wnd` / `modal_end` | 模态栈 (最多 16) |
| `klb_gui_popup` / `popup_wnd` / `popup_end` | 弹出栈 (最多 4) |
| `klb_gui_messagebox` / `messagebox_wnd` / `messagebox_end` | 消息框 |

### 聚焦与刷新

| API | 说明 |
|-----|------|
| `klb_gui_get_focus` / `get_focus_top` | 当前聚焦 |
| `klb_gui_set_focusdelay` | 聚焦延时 (默认 600ms) |
| `klb_gui_update` | 标记全树需刷新 |
| `klb_gui_update_tip` | 更新 tip 文本 |
| `klb_gui_set_redraw_full_event` | 是否绘制完整鼠标移动过程 |

### 资源与时间

| API | 说明 |
|-----|------|
| `klb_gui_load_image` / `image_size` | 图片资源 |
| `klb_gui_get_tick_count` | GUI 滴答 |
| `klb_gui_get/set_ticker_interval` | 控件定时器间隔 (默认 500ms) |

### 扩展

| API | 说明 |
|-----|------|
| `klb_gui_register_extension` | 见 [extension.md](../design/extension.md) |
| `klb_gui_get_extension` | 懒激活 |

## klb_wnd.h — 窗口基类

> 实现: `klb/src_c/klbgui/klb_wnd.c`

### 核心概念

| 项 | 说明 |
|----|------|
| `klb_wnd_pos_t` | `rect_in_parent`, `rect_in_canvas` |
| `klb_wnd_style_e` | TOP, PEEK_EVENT, TICKER, LAYER_POPUP/MSGBOX/TIP… |
| `klb_wnd_status_e` | HIDE, FOCUS, INPUT, TOPMOST… |
| `klb_wnd_vtable_t` | destroy, draw, control, command… |

### 扩展开发常用

| API | 说明 |
|-----|------|
| `klb_wnd_push_child` | 自建子窗口树 |
| `klb_wnd_on_paint_cb` | 替换绘制 |
| `klb_wnd_draw_opt*` | 扩展基础绘图 |
| `klb_wnd_bind_command` | 控件事件回调 |

基础 `klb_wnd` **不携带 CSS 字段**; 带样式控件在子类/wrapper.

## 其它对外头

| 头文件 | 说明 |
|--------|------|
| `klbui_extension.h` | GUI 扩展 vtable |
| `klb_msg.h` | 底层键鼠消息 |
| `klbui_event.h` | `KLBUI_click`, `onload`, `onpaint`… |
| `klbui_css.h` / `klbui_css_ex.h` | CSS3 盒模型 |
| `klbui_layer.h` | 多图层画布 |
| `klbui_shwnd.h` | 共享 CSS 模板窗口 |
| `klbui_default.h` | 默认主题常量 |

## 内部头 (模块内)

| 路径 | 说明 |
|------|------|
| `klb_gui_in.h` | `klb_gui_t` 完整定义 |
| `klb_wnd_in.h` | 窗口内部 |
| `extensions/klbuiex_*.h` | 各扩展模块 |

## klua 衔接 (非 klbgui/inc)

| 项 | 路径 |
|----|------|
| env GUI 扩展 | `klua/extension/klua_ex_gui.c` |
| kgui 绑定 | `klua/klua_base/klua_kgui.c` |

## 查阅顺序

1. [layers.md](../design/layers.md), 本页
2. `klb/inc/klbgui/`
3. `klb/src_c/klbgui/`
4. 设计技能 **klb-gui-design**
