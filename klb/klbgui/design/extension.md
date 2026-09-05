# GUI 框架扩展 (klbuiex)

> `klua_doc/klb/klbgui/design/extension.md` — 头文件: [klb/inc/klbgui/klbui_extension.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_extension.h); 实现: [klb/src_c/klbgui/extensions/](https://gitee.com/klua/klb/tree/trunk/src_c/klbgui/extensions)

分层总览 [layers.md](layers.md).

## 概述

1. 将 GUI 内部模块拆为可插拔扩展 (`klbuiex_*`)
2. 允许产品注册自定义 GUI 扩展参与 loop/clear/quit

**不是** Lua `require`; **不是** klua `klua_env_extension` (那是每 env 一个 `_KLUAEX-gui_`).

## 设计收益

klbuiex 把 [klb/src_c/klbgui/klb_gui.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_gui.c) 压成 **薄核心 + 可插拔子系统**; 收益如下.

| 维度 | 做法 | 收益 |
|------|------|------|
| 职责 | L1 只管 loop/消息/clear 调度; L2 各 `klbuiex_*` 独立 `.c` | 核心不膨胀; 渲染/定时器/tip 等可单独演进 |
| 实例化 | 注册表登记 vtable; `get_extension` 懒 `cb_create` | 按需激活; 启动开销可控 |
| 换页 | `klb_gui_clear` 保留扩展与 wndhash type 注册 | 换场景不重搭 render/wndhash/ticker |
| 生命周期 | `quit` / `clear` 分消息; destroy 反序 control → destroy | teardown 顺序可预期; 扩展协作清理 |
| 主 tick | 扩展 `cb_loop_once` 不受 `is_wait` 影响 | 等待遮罩时 ticker/time 仍走 |
| 产品 | `klb_gui_register_extension` + 可选 control/loop | 不改核心即可挂自定义子系统 |
| 演进 | 2025-8 核心不内置 k*; type 走 wndhash | 控件包 (klbwui) 与框架解耦 |

与 klb 全栈 **register → 懒激活 → control → loop_once** 模式一致 (app / klua env / GUI 各层同族). 外部业界对照见 [ref/extension-principles.md](../ref/extension-principles.md) (**非** klb 行为规格).

### 三类「扩展」边界

| 机制 | 挂载 | 用途 |
|------|------|------|
| `klbuiex_*` | 每 `klb_gui_t` | 框架内部能力 (render, wndhash, tip…) |
| `_KLUAEX-gui_` | 每 `klua_env_t` | env 持有一个 `p_gui` |
| wndhash type 工厂 | `KLB-GUIEX-wndhash` 内 | `"kbutton"` → create; **非** extension 插件 |

误把 Lua `require`、env 扩展、框架扩展混用是常见集成错误; 上表为选型真源.

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
  → KLBUIEX_register_extensions_std   // 10 个 register
  → klbuiex_get_wndhash/render/...    // 部分立即 get → 激活

klb_gui_get_extension(name)
  → 未激活则 cb_create → 入激活表
```

`klb_gui_clear` **不**注销扩展表项; **不**注销 wndhash 的 type 注册.

## 标准 10 扩展

宏 `KLBUIEX_register_extensions_std` ([klb/src_c/klbgui/extensions/klbuiex_extensions.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_extensions.h)); 注册名规范见 [../../design/extension-naming.md](../../design/extension-naming.md):

| 注册名 | 源文件 | 职责 | create 缓存到 `p_gui` |
|--------|--------|------|----------------------|
| `KLB-GUIEX-wndhash` | [klb/src_c/klbgui/extensions/klbuiex_wndhash.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_wndhash.c) | path 树, type→create 工厂 | `p_wndhash` |
| `KLB-GUIEX-render` | [klb/src_c/klbgui/extensions/klbuiex_render.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_render.c) | 图形渲染 (2025-6) | `p_render` |
| `KLB-GUIEX-redraw` | [klb/src_c/klbgui/extensions/klbuiex_redraw.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_redraw.c) | 脏区/重绘 | `p_redraw` |
| `KLB-GUIEX-wndticker` | [klb/src_c/klbgui/extensions/klbuiex_wndticker.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_wndticker.c) | 控件定时器 | `p_wndticker` |
| `KLB-GUIEX-default` | [klb/src_c/klbgui/extensions/klbuiex_default.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_default.c) | 默认 CSS/主题 | — |
| `KLB-GUIEX-shwnd` | [klb/src_c/klbgui/extensions/klbuiex_shwnd.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_shwnd.c) | 共享 CSS 模板 | — |
| `KLB-GUIEX-udatalayer` | [klb/src_c/klbgui/extensions/klbuiex_udatalayer.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_udatalayer.c) | 用户自定义图层 | `p_udatalayer` |
| `KLB-GUIEX-waitlayer` | [klb/src_c/klbgui/extensions/klbuiex_waitlayer.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_waitlayer.c) | 等待遮罩 | `p_waitlayer` |
| `KLB-GUIEX-tip` | [klb/src_c/klbgui/extensions/klbuiex_tip.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_tip.c) | 焦点 tip | `p_tip` |
| `KLB-GUIEX-util` | [klb/src_c/klbgui/extensions/klbuiex_util.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_util.c) | 鼠标等状态 | `p_util` |

`default` / `shwnd` 在首次 `klbuiex_get_*` 时激活.

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
| 标准集 | 10× `klbuiex_*` | coroutine/gui/LPC… |

## 自定义扩展

1. 实现 `cb_create` / `cb_destroy` (+ 可选 control/loop_once)
2. `klb_gui_register_extension(p_gui, name, &ex)`
3. `klb_gui_get_extension` 获取实例

C 产品直接调用上述 C API; 旧 C++ 封装 `klbui::CGui`（含 `RegisterExtension` / `GetExtension`）已迁工作区 `backup/inc_hpp/klbgui/`、`backup/src_cpp/klbgui/`, **不编入**现行 klb.

## 控件 type (wndhash, 非 extension 插件)

`klbuiex_wndhash` 是标准扩展之一, 内含 **type 工厂**:

- `klbuiex_wndhash_register(wndhash, "kbutton", cb_create)`
- `kgui.append` → `klb_gui_append` → wndhash 查 creater

核心 **`KLB_GUI_REGISTER_STD` 已注释**; k* 实现见 `backup/wnd/`.

## 设计权衡

| 权衡 | 说明 |
|------|------|
| 间接调用 | `get_extension` + 函数指针, 较内联略慢; 嵌入式 GUI 通常可接受 |
| 字符串命名 | 注册名如 `KLB-GUIEX-render` 须规范, 避免重复 register |
| clear 语义 | 「清 UI 内容、保留扩展」与直觉「全清」不同; 须用 `clear_async`, 禁止事件内同步 clear |
| 文档分散 | 11 扩展 + 专题 md; 本文总览, 细节见下表与 ref |

## 专题深文

| 扩展 / 主题 | 文档 |
|-------------|------|
| wndhash | [wndhash.md](wndhash.md) |
| render + redraw | [render.md](render.md) |
| shwnd | [shwnd.md](shwnd.md) |
| tip | [tip.md](tip.md) |
| default | [default.md](default.md) |
| wndticker | [ticker.md](ticker.md) |

## 相关

| 文档 | 内容 |
|------|------|
| [layers.md](layers.md) | 分层与选型 |
| [wnd.md](wnd.md) | 窗口与 modal 栈 |
| [draw.md](draw.md) | 绘制 API |
| [klb_gui.md](../api/klb_gui.md) | `klb_gui.h` API |
| [klua/design/env-extension.md](../../klua/design/env-extension.md) | `_KLUAEX-gui_` |
| [ref/extension-principles.md](../ref/extension-principles.md) | 外部插件/扩展架构参照 |
