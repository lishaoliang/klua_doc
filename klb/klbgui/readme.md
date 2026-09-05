# klbgui 文档

> `klua_doc/klb/klbgui/` — 代码: [klb/inc/klbgui/](https://gitee.com/klua/klb/tree/trunk/inc/klbgui), [klb/src_c/klbgui/](https://gitee.com/klua/klb/tree/trunk/src_c/klbgui), L0 [klb/inc/klbutil/klb_canvas.h](https://gitee.com/klua/klb/blob/trunk/inc/klbutil/klb_canvas.h) — **C klbgui 框架**. Lua 脚本 UI 真源: [lua/klbcore/](../../lua/klbcore/readme.md).

## 代码路径与 Gitee

klb 源码路径统一写 **[klb/inc/...](https://gitee.com/klua/klb/blob/trunk/inc/...)**、**[klb/src_c/...](https://gitee.com/klua/klb/blob/trunk/src_c/...)** 并链 Gitee **`trunk`**. 完整规则与 URL 表: [_meta/code-path-gitee.md](../../_meta/code-path-gitee.md).

## 子目录

| 路径 | 内容 |
|------|------|
| [design/](design/) | **自有** C 设计: 分层, 画布, 布局, 窗口/图层, 事件, CSS |
| [design/map/](design/map/) | 外部概念 ↔ klb 对照 |
| [ref/](ref/) | **外部** UI 原理参照（非 klb 实现） |
| [api/](api/) | C API 说明 |

P0a 桌面宿主: [wlua/readme.md](../../wlua/readme.md).

## 设计 (C) — 自有

| 文件 | 说明 |
|------|------|
| [design/layers.md](design/layers.md) | **分层总览**与选型 |
| [design/canvas.md](design/canvas.md) | **P0 平台画布** (`klb_canvas_t`) |
| [design/wnd.md](design/wnd.md) | **窗口树**、vtable、聚焦、modal/popup 栈 |
| [design/wndhash.md](design/wndhash.md) | path 二级索引与 type 工厂 |
| [design/render.md](design/render.md) | redraw + render 绘制刷新管线 |
| [design/draw.md](design/draw.md) | `klb_wnd_draw_*` / `klb_wnd_ex` |
| [design/layer.md](design/layer.md) | 图层栈与多画布 |
| [design/shwnd.md](design/shwnd.md) | 共享窗口与 `/klbui/tip` |
| [design/tip.md](design/tip.md) | 焦点 tip 与独立 tip 画布 |
| [design/default.md](design/default.md) | VS 深色系默认 CSS 参考 |
| [design/ticker.md](design/ticker.md) | 控件定时器 `KLBUI_onticker` |
| [design/extension.md](design/extension.md) | klbuiex 10 扩展, loop/clear |
| [design/event.md](design/event.md) | 消息队列与 `KLBUI_*` 事件 |
| [design/css.md](design/css.md) | C 侧 CSS 盒模型 |
| [design/layout.md](design/layout.md) | **自动布局 BoxFlow**（ARM；桩） |

## 对照

| 文件 | 说明 |
|------|------|
| [design/map/layout-map.md](design/map/layout-map.md) | 外部布局概念 ↔ BoxFlow（桩） |

## 参照 — 外部 UI

| 文件 | 说明 |
|------|------|
| [ref/readme.md](ref/readme.md) | 外部参照索引与免责声明 |
| [ref/layout-principles.md](ref/layout-principles.md) | Web/Qt/Android 等自动布局原理（桩） |
| [ref/extension-principles.md](ref/extension-principles.md) | 插件/扩展架构通论（Qt/引擎等对照） |

## API

| 文件 | 说明 |
|------|------|
| [api/klb_gui.md](api/klb_gui.md) | `klb_gui.h` / `klb_wnd.h` C 导读 |
| [api/klb_canvas.md](api/klb_canvas.md) | `klb_canvas.h` / `klb_wnd_ex.h` |
| [api/klb_msg.md](api/klb_msg.md) | `klb_msg.h` 底层消息 |
| [api/klbui_layer.md](api/klbui_layer.md) | `klbui_layer.h` udata/wait |
| [api/klbui_css.md](api/klbui_css.md) | `klbui_css.h` / `klbui_css_ex.h` |
| [api/klbui_layout.md](api/klbui_layout.md) | `klbui_layout.h` BoxFlow（桩） |

## Lua 脚本 UI (在 lua/klbcore/)

| 文档 | 说明 |
|------|------|
| [lua/klbcore/readme.md](../../lua/klbcore/readme.md) | `klbcore.klbui` 总览与索引 |
| [klbui_kbutton.md](../../lua/klbcore/css/klbui_kbutton.md) | 按钮 |
| [klbui_kstatic.md](../../lua/klbcore/css/klbui_kstatic.md) | 静态文本 |
| [klbui_default_css.md](../../lua/klbcore/css/klbui_default_css.md) | 默认 CSS |
| [klbui_css.md](../../lua/klbcore/css/klbui_css.md) | CSS 属性约定 |

核心 **2025-8** 已剥离内置 k* 控件; `dialog.type` 须产品/扩展包注册 type. 见 [design/layers.md](design/layers.md).

## 关联

| 入口 | 说明 |
|------|------|
| [wlua/readme.md](../../wlua/readme.md) | P0a 桌面宿主 `wlua.exe` |
| [lua/klua/kgui.md](../../lua/klua/kgui.md) | `require("kgui")` (L3 C→Lua) |
| [lua/klbcore/readme.md](../../lua/klbcore/readme.md) | `klbcore.klbui` (L6) |
| [klua/design/env-extension.md](../klua/design/env-extension.md) | `_KLUA_EX_GUI_` |

## 查阅顺序 (ai)

1. [design/layers.md](design/layers.md)
2. L0 画布: [design/canvas.md](design/canvas.md)
3. C API: [api/klb_gui.md](api/klb_gui.md), [api/klb_canvas.md](api/klb_canvas.md)
4. Lua UI: [lua/klbcore/readme.md](../../lua/klbcore/readme.md)
5. [klb/inc/klbgui/](https://gitee.com/klua/klb/tree/trunk/inc/klbgui), [klb/inc/klbutil/klb_canvas.h](https://gitee.com/klua/klb/blob/trunk/inc/klbutil/klb_canvas.h), [klb/src_c/klbgui/](https://gitee.com/klua/klb/tree/trunk/src_c/klbgui)
6. **klb-gui-design**, **klb-klua-env-design**, **wlua-design**
