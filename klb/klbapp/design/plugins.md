# plugins 动态库

> `klua_doc/klb/klbapp/design/plugins.md` — 代码: [klb/src_c/klbapp/klbappex_plugins.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbapp/klbappex_plugins.c), [klb/inc/klbapp/klb_app_extension.h](https://gitee.com/klua/klb/blob/trunk/inc/klbapp/klb_app_extension.h)

## 概述

**plugins** 指 app 层 **dll/so 目录扫描** 机制 (内置扩展名 `KLBAPPEX-plugins`). 用于将闭源或跨子项目 C 绑定注入 klua 预加载链, 或注册额外 app 扩展.

| 项 | 说明 |
|----|------|
| 默认 | **关闭** (`klbappex_plugins_create` 设 `enable=false`) |
| 启用 | `klb_app_enable_plugins(true)` — **须在** `klb_app_main` **之前** |
| 路径 | `klb_app_push_plugins_path(dir)` — 追加目录, 可多次; 按 push 顺序扫描 |
| 扫描深度 | 配置目录 **一层**, **不递归** 子目录 |
| 文件后缀 | Win: `.dll`; Linux: `.so` |
| 安全 | 当前完全信任 dll/so, 未鉴权 (`klb_app.h` @todo) |

## 配置 API

> 代码: [klb/inc/klbapp/klb_app.h](https://gitee.com/klua/klb/blob/trunk/inc/klbapp/klb_app.h)

```c
klb_app_enable_plugins(true);
klb_app_push_plugins_path("/path/to/plugins");
// 可 push 多个目录
klb_app_main(argc, argv, cb_pre_load);
```

## 默认路径 (klua 入口)

> 代码: [klb/proj/klua/klua.c](https://gitee.com/klua/klb/blob/trunk/proj/klua/klua.c) (`klua_setup_plugins_paths`)

| 平台 | 默认 plugins 目录 |
|------|-------------------|
| Win32 | `.`, `./lib`, `./libs` (相对当前工作目录) |
| Linux | `getenv("LD_LIBRARY_PATH")` 按 `:` 拆段, 非空段逐个 push |

arm 部署惯例: rootfs 设 `LD_LIBRARY_PATH=/tmp/app:/tmp/app/lib`, `klua` 自动扫 app 分区下 `.so` (如 `libkpfs.so`).

## 加载时序

相对 `klb_app_main` step3 (`klbappex_plugins_preload`):

```
step2  klbappex_klua_push_preload(cb_pre_load)     // 产品 openlibs 先入链
step3  on_preload_klb_app                          // klb_app_push_preload 链
       klbappex_plugins_preload                    // 扫目录 dlopen
step4  klbappex_klua_do_preinit → klua_env_doinit  // 合并预加载链执行
```

`plugins_preload` 在 **`klua_env_doinit` 之前**, 插件 `klbappex_pre_open` 与产品 `cb_pre_load` 于 doinit 一并执行.

### 预加载链顺序

> 代码: [klb/src_c/klbapp/klbappex_klua.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbapp/klbappex_klua.c)

1. 产品 `cb_pre_load` (如 `klua_loadlib_all`)
2. 各成功插件经 `klbappex_pre_open` 注入的 `lua_CFunction` (按 dl 加载顺序)

子线程 (`kthread.start`) 新建 env 时复用 **同一预加载链**.

## 扫描与 dlopen 规则

> 代码: [klb/src_c/klbapp/klbappex_plugins.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbapp/klbappex_plugins.c)

| 步 | 行为 |
|----|------|
| 1 | 从 `p_path_nlist` 头依次取目录 |
| 2 | Win: `_findfirst(dir/*.dll)`; Linux: `opendir` + 后缀 `.so` |
| 3 | 每文件 `klb_dlopen(filepath)` |
| 4 | `dlsym` 取 `klbappex_*` 导出符号 |
| 5 | 至少注册一个 app 扩展 **或** 注入一个 klua 预加载 → 保留 dl; 否则 `klb_dlclose` |

注册 app 扩展成功后, 若导出 `klbappex_init` 则调用; 销毁时对每个已加载 dl 调 `klbappex_quit`.

## 动态库导出契约

> 代码: [klb/inc/klbapp/klb_app_extension.h](https://gitee.com/klua/klb/blob/trunk/inc/klbapp/klb_app_extension.h)

| 符号 | 类型 | 必填 | 作用 |
|------|------|------|------|
| `klbappex_init` | `int()` | 否 | dl 加载成功后调用 |
| `klbappex_quit` | `void()` | 否 | app 退出销毁 dl 时调用 |
| `klbappex_ex_count` | `int()` | 与 open 成对 | 本 dl 提供的 app 扩展个数 |
| `klbappex_ex_open` | `int(idx, klb_app_extension_t*, name, name_max)` | 与 count 成对 | 填充 vtable + 扩展名 (≤64 字符) |
| `klbappex_pre_count` | `int()` | 与 open 成对 | klua 预加载回调个数 |
| `klbappex_pre_open` | `int(idx, lua_CFunction* out)` | 与 count 成对 | 输出预加载 `lua_CFunction` |

**两种能力可并存** 于同一 dll/so; 也可只实现其中一种.

| 能力 | 效果 |
|------|------|
| App 扩展 | `klb_app_register_extension` — 与静态注册相同, 可有 `cb_loop_once` |
| Klua 预加载 | 回调内 `klua_loadlib(L, open, "name")` 写 `registry._PRELOAD` |

扩展名缓冲区: 调用方至少 `name_max+1` 字节; `strlen(name) <= KLBAPPEX_DL_name_max` (64).

## 与 Lua package.loadlib 区分

| 路径 | 机制 |
|------|------|
| **app plugins** (本节) | `klbappex_plugins` 扫目录 + `klbappex_pre_*` |
| **Lua 自带** | `require` / `package.loadlib` — **不经** klb plugins |

业务闭源扩展 (如 `libkpfs.so`) 走 **app plugins**; 勿与 Lua 运行时自行 `dlopen` 混用同一契约.

## 范例: libkpfs.so

| 项 | 说明 |
|----|------|
| 产物 | [portfs/lib/libkpfs.so](https://gitee.com/klua/portfs/blob/trunk/lib/libkpfs.so) (不链接进 `klua`; 运行时 dlopen) |
| 插件源 | [portfs/src_klua/pfs_klua_plugin.c](https://gitee.com/klua/portfs/blob/trunk/src_klua/pfs_klua_plugin.c) |
| 导出 | `klbappex_pre_count` / `klbappex_pre_open` |
| Lua | `require "kpfs"` |

## 端到端数据流

```
klb_app_enable_plugins + push_plugins_path
klb_app_main
  plugins_preload → dlopen → klbappex_pre_open → klbappex_klua_push_preload
  klua_env_doinit → klua_pmain → cb_pre_load(L) → klua_loadlib → _PRELOAD
  dofile(argv[1])
  require "短名" → 从 _PRELOAD 懒加载
```

平台层 `klb_dlopen` / `klb_dlsym`: [klb/inc/klbplatform/klb_dynamic_link.h](https://gitee.com/klua/klb/blob/trunk/inc/klbplatform/klb_dynamic_link.h).
