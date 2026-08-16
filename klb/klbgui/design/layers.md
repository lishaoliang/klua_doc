# klbgui 分层总览

> `klua_doc/klb/klbgui/design/layers.md` — 代码: `klb/inc/klbgui/`, `klb/src_c/klbgui/`

## 结论

klbgui 自下而上 **四层 C + 脚本**: 核心对象/窗口基类 → **11 个 klbuiex_*** 框架扩展 → 控件 type 工厂 → klbcore 脚本 UI. 2025-8 核心 **不内置** kbutton/kedit 等具体控件.

与 klua 关系: 每 `klua_env_t` 经 `_KLUA_EX_GUI_` 持有一个 `klb_gui_t`; 见 [klua/design/layers.md](../../klua/design/layers.md).

## 层次图

```
L4  脚本 UI (klbcore.klbui)     klb/bin/klbcore/klbui/
L3  C→Lua 绑定 (kgui)           klua_kgui.c
L2′ klua env GUI 扩展           klua_ex_gui.c (每 env 一个 p_gui)
L2  GUI 框架扩展 (klbuiex_*)    extensions/ (每 p_gui 实例)
L1  C 核心                      klb_gui.c, klb_wnd.c, klbui_css*
L0  画布 (klbutil)              klb_canvas_t
```

## 分层表

| 层 | 名称 | 谁用 | 典型路径 | 对外形态 |
|----|------|------|----------|----------|
| L0 | 画布 | 核心/扩展 | `klbutil/klb_canvas.h` | `klb_gui_attach_canvas` |
| L1 | C 核心 | C/C++ 产品 | `klb_gui.c`, `klb_wnd.c` | `klb_gui.h` API |
| L2 | klbuiex 扩展 | 框架内部 | `extensions/klbuiex_*.c` | `klbui_extension.h` |
| L2′ | klua env GUI | klua 运行时 | `klua_ex_gui.c` | env 扩展, 非 require |
| L3 | kgui 绑定 | Lua 脚本 | `klua_kgui.c` | `require("kgui")` |
| L4 | klbcore.klbui | Lua 业务 | `klb/bin/klbcore/klbui/` | `require("klbcore.klbui")` |

## 六部件 (L1 核心视角)

| # | 部件 | 主要文件 | 职责 |
|---|------|----------|------|
| ① | 核心对象 | `klb_gui_t`, `klb_canvas_t`, `klb_msg` | GUI 根, 画布, 消息队列 |
| ② | 窗口基类 | `klb_wnd_t`, vtable | 窗口树; **无具体 k* 控件体** |
| ③ | 框架扩展 | 11× `klbuiex_*` | render/redraw/wndhash/tip… |
| ④ | CSS | `klbui_css*.c` | 盒模型; 基础 wnd 无 CSS 字段 |
| ⑤ | 消息/事件 | `klb_msg.h`, `klbui_event.h` | 键鼠 + `KLBUI_*` |
| ⑥ | 脚本集成 | `kgui`, `klbcore.klbui` | Lua 驱动 |

详 **klb-gui-design** 技能.

## 两类 API

### A. C/C++ 开发者

| 类别 | 头文件 | 文档 |
|------|--------|------|
| GUI 主 API | `klb_gui.h` | [../api/klb_gui.md](../api/klb_gui.md) |
| 窗口基类 | `klb_wnd.h` | 同上 |
| GUI 扩展 | `klbui_extension.h` | [extension.md](extension.md) |
| CSS / 事件 | `klbui_css.h`, `klbui_event.h` | [lua/klbcore/css/klbui_css.md](../../../lua/klbcore/css/klbui_css.md) |

### B. Lua 脚本

| 加载 | 示例 | 文档 |
|------|------|------|
| `kgui` | `require("kgui")` | [lua/klua/kgui.md](../../../lua/klua/kgui.md) |
| klbcore | `require("klbcore.klbui")` | **klbcore-design** |

脚本 **不应** 直接 `klb_gui_register_extension`.

## 扩展机制 (L2) 要点

- 契约: `klb_gui_extension_t` — create/destroy/control/loop_once
- 双表: 注册表 + 激活表 (`klb_gui_get_extension` 懒激活)
- `klb_gui_create` → `KLBUIEX_register_extensions_std` (11 个)
- 消息: `KLBUI_EX_MSG_quit`, `KLBUI_EX_MSG_clear`

详 [extension.md](extension.md).

## 控件 type (L2 内 wndhash, 非独立层)

| 项 | 说明 |
|----|------|
| 工厂 | `klbuiex_wndhash` — path 树 + type→create |
| 注册 | `klbuiex_wndhash_register`; 宏 `klbui_register_k*` (`klbui_widgets.h`) |
| 现状 | **`KLB_GUI_REGISTER_STD` 已注释**; 控件实现归档 `klb/backup/wnd/` (**klb-backup-design**) |
| Lua | `dialog.type` 须已注册; `kgui.append(type, path, …)` |

## 我该改哪一层

| 需求 | 层 | 做法 | 文档 |
|------|-----|------|------|
| 框架能力 (渲染/图层) | L2 | 新 `klbuiex_*` + register | extension.md |
| 新 Lua 控件 API | L3 | `klua_open_kgui` 增补 + `kgui` | [lua/klua/kgui.md](../../../lua/klua/kgui.md) |
| 纯 Lua UI 描述 | L4 | klbcore.klbui | klbcore-design |
| 新具体控件 type | wndhash | `wndhash_register` 或扩展包 | [lua/klbcore/readme.md](../../../lua/klbcore/readme.md) § klbui 控件 |
| C 产品持 GUI | L1/L2′ | `klb_gui_*` / klbapp | [klb_gui.md](../api/klb_gui.md) |
| env 与 GUI 生命周期 | L2′ | `klua_ex_gui` | klua/env-extension.md |

## 主循环

```
klb_app_loop_once
  → klua_env_loop_once
      → klua_ex_gui_loop_once
          → klb_gui_loop_once
```

## 相关文档

| 文档 | 内容 |
|------|------|
| [extension.md](extension.md) | klbuiex 11 表, loop/clear |
| [klb_gui.md](../api/klb_gui.md) | `klb_gui.h` 导读 |
| [wnd.md](wnd.md) | 窗口树 |
| [layer.md](layer.md) | 多图层画布 |
| [klua/design/layers.md](../../klua/design/layers.md) | klua 八层 |
