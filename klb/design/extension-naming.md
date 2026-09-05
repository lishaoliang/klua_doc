# 扩展注册名规范

> `klua_doc/klb/design/extension-naming.md` — 技能真源: **klb-extension-design** § 扩展注册名规范

## 适用范围

注册名用于 C 层 `klb_app_register_extension` / `klb_gui_register_extension` / `klua_env_register_extension` 及对应 `*_get_extension` 查找键。

**不是**：

- Lua `require("kco")` 等预加载短名
- wndhash 控件 type（如 `"kbutton"`）
- plugins dll 导出符号（`klbappex_init` 等）
- LPC `dst_name`（最长 15 字符，独立命名）

## 层内格式

| 层 | 格式 | 示例 |
|----|------|------|
| App | `KLB-APPEX-<小写>` | `KLB-APPEX-klua`、`KLB-APPEX-plugins` |
| GUI 框架 | `KLB-GUIEX-<小写>` | `KLB-GUIEX-render`、`KLB-GUIEX-wndhash` |
| klua env | `_KLUAEX-<小写>_` | `_KLUAEX-gui_`、`_KLUAEX-coroutine_` |
| src_packages env | `_KLUAEX-<包短名>-<扩展短名>_` | `_KLUAEX-wui-extension_` |

| 规则 | 说明 |
|------|------|
| 段分隔 | 连字符 `-` |
| 段名 | 小写 |
| 源文件 | 仍 `klbappex_*` / `klbuiex_*` / `klua_ex_*` |
| C 宏 | 层前缀 + `_` + 小写短名 | 见下表 |

## C 宏命名

| 层 | 宏格式 | 示例 |
|----|--------|------|
| App | `KLB_APPEX_<小写>` | `KLB_APPEX_klua` |
| GUI 框架 | `KLB_GUIEX_<小写>` | `KLB_GUIEX_render` |
| klua env | `KLUAEX_<小写>` | `KLUAEX_gui` |
| src_packages env | `KLUAEX_<包短名>_<扩展短名>` | `KLUAEX_wui_extension` |

## klb 内置注册名清单

### App

| 注册名 | C 宏 | 源文件 |
|--------|------|--------|
| `KLB-APPEX-klua` | `KLB_APPEX_klua` | `klbappex_klua.c` |
| `KLB-APPEX-plugins` | `KLB_APPEX_plugins` | `klbappex_plugins.c` |

### GUI 框架（10）

| 注册名 | C 宏 | 源文件 |
|--------|------|--------|
| `KLB-GUIEX-wndhash` | `KLB_GUIEX_wndhash` | `klbuiex_wndhash.c` |
| `KLB-GUIEX-render` | `KLB_GUIEX_render` | `klbuiex_render.c` |
| `KLB-GUIEX-redraw` | `KLB_GUIEX_redraw` | `klbuiex_redraw.c` |
| `KLB-GUIEX-wndticker` | `KLB_GUIEX_wndticker` | `klbuiex_wndticker.c` |
| `KLB-GUIEX-default` | `KLB_GUIEX_default` | `klbuiex_default.c` |
| `KLB-GUIEX-shwnd` | `KLB_GUIEX_shwnd` | `klbuiex_shwnd.c` |
| `KLB-GUIEX-udatalayer` | `KLB_GUIEX_udatalayer` | `klbuiex_udatalayer.c` |
| `KLB-GUIEX-waitlayer` | `KLB_GUIEX_waitlayer` | `klbuiex_waitlayer.c` |
| `KLB-GUIEX-tip` | `KLB_GUIEX_tip` | `klbuiex_tip.c` |
| `KLB-GUIEX-util` | `KLB_GUIEX_util` | `klbuiex_util.c` |

### klua env 标准扩展

| 注册名 | C 宏 | 源文件 |
|--------|------|--------|
| `_KLUAEX-object_` | `KLUAEX_object` | `klua_ex_object.c` |
| `_KLUAEX-bufagent_` | `KLUAEX_bufagent` | `klua_ex_bufagent.c` |
| `_KLUAEX-coroutine_` | `KLUAEX_coroutine` | `klua_ex_coroutine.c` |
| `_KLUAEX-time_` | `KLUAEX_time` | `klua_ex_time.c` |
| `_KLUAEX-netmulti_` | `KLUAEX_netmulti` | `klua_ex_netmulti.c` |
| `_KLUAEX-lpc_` | `KLUAEX_lpc` | `klua_ex_lpc.c` |
| `_KLUAEX-gui_` | `KLUAEX_gui` | `klua_ex_gui.c` |

### src_packages

| 注册名 | C 宏 | 源文件 |
|--------|------|--------|
| `_KLUAEX-wui-extension_` | `KLUAEX_wui_extension` | `klbwui/core/klbwui_extension.c` |

### 子项目 env 扩展

| 注册名 | C 宏 | 源文件 |
|--------|------|--------|
| `_KLUAEX-kpfs-env_` | `KLUAEX_kpfs_env` | `portfs/src_klua/pfs_klua_ex.c` |

## 新增扩展

| 场景 | 注册名 |
|------|--------|
| 产品 app 模块 | `KLB-APPEX-<小写>` |
| GUI 子系统 | `KLB-GUIEX-<小写>` |
| klua env 能力 | `_KLUAEX-<小写>_` |
| src_packages `<pkg>` | `_KLUAEX-<包短名>-<扩展短名>_` |

## 相关文档

| 文档 | 内容 |
|------|------|
| [../klua/design/env-extension.md](../klua/design/env-extension.md) | klua env 扩展机制 |
| [../klbgui/design/extension.md](../klbgui/design/extension.md) | GUI klbuiex 扩展 |
| [../klbapp/design/plugins.md](../klbapp/design/plugins.md) | app plugins dll |
