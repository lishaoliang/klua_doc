# klbgui 文档

> `klua_doc/klb/klbgui/` — 代码: `klb/inc/klbgui/`, `klb/src_c/klbgui/` — **C klbgui 框架**. Lua 脚本 UI 真源: [lua/klbcore/](../../lua/klbcore/readme.md).

## 子目录

| 路径 | 内容 |
|------|------|
| [design/](design/) | 分层, 扩展, 窗口/图层 |
| [api/](api/) | C API 说明 |

## 设计 (C)

| 文件 | 说明 |
|------|------|
| [design/layers.md](design/layers.md) | **分层总览**与选型 |
| [design/extension.md](design/extension.md) | klbuiex 11 扩展, loop/clear |
| [design/wnd.md](design/wnd.md) | 窗口 |
| [design/layer.md](design/layer.md) | 图层 |

## API

| 文件 | 说明 |
|------|------|
| [api/klb_gui.md](api/klb_gui.md) | `klb_gui.h` / `klb_wnd.h` C 导读 |

## Lua 脚本 UI (在 lua/klbcore/)

| 文档 | 说明 |
|------|------|
| [lua/klbcore/readme.md](../../lua/klbcore/readme.md) | `klbcore.klbui` 总览与索引 |
| [klbui_kbutton.md](../../lua/klbcore/css/klbui_kbutton.md) | 按钮 |
| [klbui_kcombo.md](../../lua/klbcore/css/klbui_kcombo.md) | 组合框 |
| [klbui_kmessagebox.md](../../lua/klbcore/css/klbui_kmessagebox.md) | 消息框 |
| [klbui_kstatic.md](../../lua/klbcore/css/klbui_kstatic.md) | 静态文本 |
| [klbui_default_css.md](../../lua/klbcore/css/klbui_default_css.md) | 默认 CSS |
| [klbui_css.md](../../lua/klbcore/css/klbui_css.md) | CSS 属性约定 |

核心 **2025-8** 已剥离内置 k* 控件; `dialog.type` 须产品/扩展包注册 type. 见 [design/layers.md](design/layers.md).

## 关联

| 入口 | 说明 |
|------|------|
| [lua/klua/kgui.md](../../lua/klua/kgui.md) | `require("kgui")` (L3 C→Lua) |
| [lua/klbcore/readme.md](../../lua/klbcore/readme.md) | `klbcore.klbui` (L6) |
| [klua/design/env-extension.md](../klua/design/env-extension.md) | `_KLUA_EX_GUI_` |

## 查阅顺序 (ai)

1. [design/layers.md](design/layers.md)
2. C API: [api/klb_gui.md](api/klb_gui.md)
3. Lua UI: [lua/klbcore/readme.md](../../lua/klbcore/readme.md)
4. `klb/inc/klbgui/`, `klb/src_c/klbgui/`
5. **klb-gui-design**, **klb-klua-env-design**
