# GUI 框架扩展 (klbuiex)

> `klua_doc/klb/klbgui/design/extension.md` — 头文件: `klb/inc/klbgui/klbui_extension.h`; 实现: `klb/src_c/klbgui/extensions/`

分层总览 [layers.md](layers.md).

## 概述

1. 将 GUI 内部模块拆为可插拔扩展 (`klbuiex_*`)
2. 允许产品注册自定义 GUI 扩展参与 loop/clear/quit

**不是** Lua `require`; **不是** klua `klua_env_extension` (那是每 env 一个 `_KLUA_EX_GUI_`).

## 契约 (`klbui_extension.h`)

```c
typedef struct klb_gui_extension_t_ {
    void* (*cb_create)(klb_gui_t* p_gui);
    void  (*cb_destroy)(void* ptr, klb_gui_t* p_gui);
    int   (*cb_control)(void* ptr, klb_gui_t* p_gui, int msg,
                        uint8_t* p_param_in_out, int param_size);
    int   (*cb_loop_once)(void* ptr, klb_gui_t* p_gui, int64_t now);
} klb_gui_extension_t;
```

| API | 说明 |
|-----|------|
| `klb_gui_register_extension` | 登记 vtable 到注册表 |
| `klb_gui_get_extension` | 懒激活, 返回实例指针 |

| 回调 | 必填 |
|------|------|
| `cb_create`, `cb_destroy` | 是 |
| `cb_control`, `cb_loop_once` | 否 |

### 控制消息

| msg | 时机 |
|-----|------|
| `KLBUI_EX_MSG_quit` | `klb_gui_destroy` 前, **反序**通知 |
| `KLBUI_EX_MSG_clear` | `klb_gui_clear` 时, **反序**通知 |

## 两阶段: 注册表 + 激活表

```
klb_gui_create
  → KLBUIEX_register_extensions_std   // 11 个 register
  → klbuiex_get_wndhash/render/...    // 部分立即 get → 激活

klb_gui_get_extension(name)
  → 未激活则 cb_create → 入激活表
```

`klb_gui_clear` **不**注销扩展表项; **不**注销 wndhash 的 type 注册.

## 标准 11 扩展

宏 `KLBUIEX_register_extensions_std` (`klbuiex_extensions.h`):

| 注册名 | 源文件 | 职责 | create 缓存到 `p_gui` |
|--------|--------|------|----------------------|
| `KLBUIEX-wndhash` | `klbuiex_wndhash.c` | path 树, type→create 工厂 | `p_wndhash` |
| `KLBUIEX-render` | `klbuiex_render.c` | 图形渲染 (2025-6) | `p_render` |
| `KLBUIEX-redraw` | `klbuiex_redraw.c` | 脏区/重绘 | `p_redraw` |
| `KLBUIEX-wndticker` | `klbuiex_wndticker.c` | 控件定时器 | `p_wndticker` |
| `KLBUIEX-default` | `klbuiex_default.c` | 默认 CSS/主题 | — |
| `KLBUIEX-time` | `klbuiex_time.c` | GUI 时间基准 | — |
| `KLBUIEX-shwnd` | `klbuiex_shwnd.c` | 共享 CSS 模板 | — |
| `KLBUIEX-udatalayer` | `klbuiex_udatalayer.c` | 用户自定义图层 | `p_udatalayer` |
| `KLBUIEX-waitlayer` | `klbuiex_waitlayer.c` | 等待遮罩 | `p_waitlayer` |
| `KLBUIEX-tip` | `klbuiex_tip.c` | 焦点 tip | `p_tip` |
| `KLBUIEX-util` | `klbuiex_util.c` | 鼠标等状态 | `p_util` |

`default` / `time` / `shwnd` 在首次 `klbuiex_get_*` 时激活.

## `klb_gui_loop_once` 流程

1. 更新 `loop_tc`
2. 处理消息队列 (键鼠; `is_wait` 时不分发外设消息)
3. focusdelay → `KLBUI_focusdelay` + tip
4. **各扩展 `cb_loop_once`** (不受 `is_wait` 影响)
5. 若 `is_need_clear` → `klb_gui_clear` + 回调
6. `render` 重绘刷新

有消息处理返回 0.

## 销毁与清理

| 路径 | 行为 |
|------|------|
| `klb_gui_destroy` | `cb_control(quit)` 反序 → 激活表 `cb_destroy` |
| `klb_gui_clear` | `cb_control(clear)` 反序; 清窗口/图片; 保留扩展与 type 注册 |
| `klb_gui_clear_async` | 标记 `is_need_clear`; 在 loop step5 执行 clear |

**禁止**在 GUI 事件流程中同步 `klb_gui_clear` (Lua 用 `kgui.clear_async`).

## 与 klua env 扩展对比

| 项 | `klb_gui` | `klua_env` |
|----|-----------|------------|
| 挂载 | 每 `klb_gui_t` | 每 `klua_env_t` |
| 头文件 | `klbui_extension.h` | `klua_env_extension.h` |
| clear 消息 | `KLBUI_EX_MSG_clear` | 无 (仅 quit) |
| 标准集 | 11× `klbuiex_*` | coroutine/gui/LPC… |

## 自定义扩展

1. 实现 `cb_create` / `cb_destroy` (+ 可选 control/loop_once)
2. `klb_gui_register_extension(p_gui, name, &ex)`
3. `klb_gui_get_extension` 获取实例

C++: `CGui::RegisterExtension` 薄封装.

## 控件 type (wndhash, 非 extension 插件)

`klbuiex_wndhash` 是标准扩展之一, 内含 **type 工厂**:

- `klbuiex_wndhash_register(wndhash, "kbutton", cb_create)`
- `kgui.append` → `klb_gui_append` → wndhash 查 creater

核心 **`KLB_GUI_REGISTER_STD` 已注释**; k* 实现见 `klb/backup/wnd/`.

## 相关

| 文档 | 内容 |
|------|------|
| [layers.md](layers.md) | 分层与选型 |
| [klb_gui.md](../api/klb_gui.md) | `klb_gui.h` API |
| [klua/design/env-extension.md](../../klua/design/env-extension.md) | `_KLUA_EX_GUI_` |
