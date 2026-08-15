# klbui 文档

> `klua_doc/klb/klbui/` — 代码: `klb/inc/klbgui/`, `klb/src_c/klbgui/`

## 子目录

| 路径 | 内容 |
|------|------|
| [design/](design/) | 分层, 扩展, C API, 窗口/图层 |
| [widgets/](widgets/) | 控件说明 (type 须 wndhash 已注册) |
| [css/](css/) | CSS 约定 |

## 设计

| 文件 | 说明 |
|------|------|
| [design/layers.md](design/layers.md) | **分层总览**与选型 |
| [design/extension.md](design/extension.md) | klbuiex 11 扩展, loop/clear |
| [design/c-gui-api.md](design/c-gui-api.md) | `klb_gui.h` / `klb_wnd.h` C 导读 |
| [design/wnd.md](design/wnd.md) | 窗口 |
| [design/layer.md](design/layer.md) | 图层 |

## 控件

| 文件 | 说明 |
|------|------|
| [widgets/kbutton.md](widgets/kbutton.md) | 按钮 |
| [widgets/kcombo.md](widgets/kcombo.md) | 组合框 |
| [widgets/kmessagebox.md](widgets/kmessagebox.md) | 消息框 |
| [widgets/kstatic.md](widgets/kstatic.md) | 静态文本 |
| [widgets/default_css.md](widgets/default_css.md) | 默认 CSS |

核心 **2025-8** 已剥离内置 k* 控件; `dialog.type` 须产品/扩展包注册 type. 见 [design/layers.md](design/layers.md).

## CSS

| 文件 | 说明 |
|------|------|
| [css/css.md](css/css.md) | CSS 属性约定 |

## 关联

| 入口 | 说明 |
|------|------|
| [klua/k/kgui.lua.md](../klua/k/kgui.lua.md) | `require("kgui")` |
| [klbcore/readme.md](../klbcore/readme.md) | 脚本库; UI 见 [klbui](../klbui/readme.md) |
| [klua/design/env-extension.md](../klua/design/env-extension.md) | `_KLUA_EX_GUI_` |

## 查阅顺序 (ai)

1. [design/layers.md](design/layers.md)
2. 本目录
3. `klb/inc/klbgui/`
4. `klb/src_c/klbgui/`
5. **klb-gui-design**, **klb-klua-env-design**
