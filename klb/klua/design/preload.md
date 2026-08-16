# 预加载与 require

> `klua_doc/klb/klua/design/preload.md` — 代码: `klb/src_c/klua/klua.c`, `klb/src_c/klua/klua_env.c`
>
> **Lua 脚本 API**: [lua/readme.md](../../../lua/readme.md); 清单 [require-guide.md](require-guide.md).

## 两条加载路径

| 路径 | 机制 | 典型 |
|------|------|------|
| **C 预加载** | `registry._PRELOAD` | `require("kco")`, `require("cjson")` |
| **Lua 脚本** | `package.path` + `require` | `require("klbcore.klbui")` |

klb 闭源 C 扩展 (如 `kpfs`) 走 app **plugins** 注入预加载链, 不走 Lua `package.loadlib` 扫目录.

## klua_loadlib

> 代码: `klb/src_c/klua/klua.c`

```c
klua_loadlib(L, openlib, "短名");
```

- 写入 `LUA_REGISTRYINDEX` 下 `_PRELOAD` 表
- `require("短名")` 时懒调用 `openlib(L)` 得到模块表
- 绑定源文件: `klua_<名>.c`; 导出函数: `klua_open_<名>`

## doinit 时的加载顺序

```
luaL_openlibs(L)              // §0 Lua 5.4 标准库
  → cb_pre_load(L)            // 产品 + plugins 预加载链
       每条回调内 klua_loadlib(...)
```

**`p_preload_nlist` 入链顺序** (均在 `klua_env_doinit` 之前 push):

| 顺序 | 来源 | 典型内容 |
|------|------|----------|
| 1 | `klb_app_main` step2: `klbappex_klua_push_preload` | 产品 `klua_loadlib_all` |
| 2 | step3: `klbappex_plugins_preload` | 插件 `klbappex_pre_open` (如 `kpfs`) |

独立 `klua` 可执行: `klua_setup_plugins_paths` 后同样走 plugins 链. 详 [startup.md](../../klbapp/design/startup.md), [plugins.md](../../klbapp/design/plugins.md).

## klua_loadlib_all

> 代码: `klb/src_c/klua/klua.c` (`klua_loadlib_all`)

默认注册 bundled 第三方 + 全部 `k*` + (未裁剪时) `kpa_*`. 产品可在自有 `cb_pre_load` 中:

- 直接调用 `klua_loadlib_all(L)`
- 或按需 `klua_loadlib` 子集 (裁剪体积)

裁剪宏示例: `__KLB_NO_LPEG__`, `__KLB_NO_ZLIB__`, `__KLB_NO_SQLITE__`, `__KLB_NO_PACKAGES__` (见 **klb-makefile** `clip.mk`).

## 与 Lua package.loadlib 的区别

| 项 | klb `_PRELOAD` 链 | Lua 5.4 `package.loadlib` |
|----|-------------------|---------------------------|
| 注册 | `klua_loadlib` / 预加载回调 | `package.preload` 或 CModule |
| plugins | `klbappex_pre_open` 注入 | **不经过** klb plugins |
| worker 子线程 | 复用同一 `p_preload_nlist` | 同左 |

业务 `require("kco")` 等短名 **只** 走 klb 预加载链.

## klbcore 脚本 (非预加载)

`klb/bin/klbcore/` 下纯 Lua **不在** `_PRELOAD`. 入口脚本须扩展 `package.path`:

```lua
local base = kenv.base_path() .. '/klbcore/'
package.path = package.path .. ';' .. base .. '?.lua;' .. base .. '?/init.lua;'
local klbui = require('klbcore.klbui')
```

示例: `bin/sample/sample_ui/main_ui.lua`. 脚本文档: [lua/klbcore/readme.md](../../../lua/klbcore/readme.md).

## worker 子线程

> 代码: `klb/src_c/klbapp/klbappex_klua.c`

`klbappex_klua_do_preinit` 为 worker 设 `klua_thread_set_preload(on_preload_klualib_klbappex_klua)`. `kthread.start` 新建 env 时 `klua_env_doinit` 复用 **同一** 预加载链 (含 plugins 已注入项).

## 新增 C 模块 (选型)

| 需求 | 做法 |
|------|------|
| 新 k* 绑定 | `klua_open_kxxx` + 加入 `klua_loadlib_all` 或产品 `openlibs` |
| 可选重组件 | `src_packages/kpa_xxx` + `klua_loadlib` |
| 纯业务 | klbcore 或产品 `bin/*lua/`, 配 `package.path` |
| 闭源 C | app `enable_plugins` + `klbappex_pre_open` |
| env 级 C 框架件 | `klua_env_register_extension`; 见 [env-extension.md](env-extension.md) |

完整 require 清单: [require-guide.md](require-guide.md).
