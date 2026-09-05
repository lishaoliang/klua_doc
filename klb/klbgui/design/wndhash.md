# 窗口 path 与 type 工厂 (wndhash)

> `klua_doc/klb/klbgui/design/wndhash.md` — 头文件: [klb/src_c/klbgui/extensions/klbuiex_wndhash.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_wndhash.h); 实现: [klb/src_c/klbgui/extensions/klbuiex_wndhash.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_wndhash.c)

扩展总览 [extension.md](extension.md). 窗口 [wnd.md](wnd.md). API [../api/klb_gui.md](../api/klb_gui.md).

## 结论

`klbuiex_wndhash` 是标准扩展 **`KLBUIEX-wndhash`**：维护 **path → 窗口** 索引与 **type → create 回调** 工厂。`kgui.append(type, path, …)` 最终走 `klbuiex_wndhash_append` → `get_creater(type)` → `cb_create`。

核心 **`KLB_GUI_REGISTER_STD` 已注释**；k* 控件须产品或 **klbwui** 扩展包注册（**klb-wui-design**）。

## 二级 path 索引（2023-8）

```
path "/home/btn1"
  第一级 hash: 顶层 "/home"
  第二级 hash: 该顶层窗内的 "/btn1"
```

第一级以 **顶层窗口路径**（如 `/home`）为键；第二级在对应顶层窗子树内解析剩余 path。旧版一级索引已废弃。

## type 工厂

| API | 说明 |
|-----|------|
| `klbuiex_wndhash_register(wndhash, type, cb_create)` | 注册创建函数；type 命名 `"k*"` |
| `klbuiex_wndhash_get_creater(wndhash, type)` | 查找；NULL 表示未注册 |
| `klbui_register_k*` | 宏声明于 [klb/src_c/klbgui/klbui_widgets.h](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klbui_widgets.h)；实现多在 klbwui 或 `backup/wnd/` |

创建链：

```
klb_gui_append(type, path, x,y,w,h, style)
  → klbuiex_wndhash_get_creater(type)
  → cb_create(p_gui, x,y,w,h) → klb_wnd_t*
  → klbuiex_wndhash_append 入树 + 登记 path
```

## path 操作

| API | 说明 |
|-----|------|
| `klbuiex_wndhash_append` | 创建并入树 |
| `klbuiex_wndhash_find` | 按 path 查窗口 |
| `klbuiex_wndhash_remove` | 移除 |

`klb_gui_find_wnd` / `remove` / `append` 为对外薄封装。

## 与旧 API 关系

| 旧 | 新 |
|----|-----|
| `klb_gui_register` / `get_creater` | 仍存在于 `klb_gui.h`；标准路径用 wndhash |
| per-gui 注册表 | wndhash 扩展内统一管理 |

`klb_gui_clear` **不**注销 type 注册，仅清窗口实例。

## 现状

| 项 | 说明 |
|----|------|
| 核心内置 type | 无（2025-8 剥离 kbutton 等） |
| 仍启用 shwnd | tip → `klbui_shwnd_get_tip`（[shwnd.md](shwnd.md)） |
| klbwui | `klbwui_register_embed` / `sim` 批量 register |
| Lua | `dialog.type` 须已 register |

## 关联

| 主题 | 入口 |
|------|------|
| 控件包 | **klb-wui-design** |
| modal 查窗 | [wnd.md](wnd.md) — `wndhash_find` + `KLB_WND_STYLE_TOP` |
| 扩展 | [extension.md](extension.md) |
