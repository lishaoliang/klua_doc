# 启动流程

> `klua_doc/klb/klbapp/design/startup.md` — 代码: `klb/src_c/klbapp/klb_app.c`, `klb/src_c/klbbase/klb_base.c`

## 定位

> 代码: `klb/inc/klbapp/klb_app.h`

klbapp 面向嵌入式产品 `main()`: 聚合模块, 统一扩展感知, 主循环驱动 Lua 业务.

| 对比项 | klbapp | `klua_main` 独立入口 |
|--------|--------|----------------------|
| 场景 | 产品进程 | 调试/工具进程 |
| 扩展 | `klb_app_extension_t` + plugins dll/so | 仅 `klua_env_extension` |
| 主循环 | `klb_app_loop_once` 聚合多扩展 | 直接 `klua_env_loop_once` |

## 标准调用序

> 代码: `klb/inc/klbapp/klb_app.h` (`klb_app_main` 注释)

```
klb_base_init(NULL)
  → klb_socket_init / klua_multithread_init / klb_app_init
klb_app_push_preload(product_preload)       // 可选, 须在 main 之前
klb_app_enable_plugins(true)                // 可选, 默认关闭
klb_app_push_plugins_path(dir)              // 可选, 可多次 push
klb_app_main(argc, argv, cb_pre_load)
klb_base_quit()
```

**须在 `klb_app_main` 之前完成的配置**: `klb_app_push_preload`, `klb_app_enable_plugins`, `klb_app_push_plugins_path`.

## klb_app_main 五步

> 代码: `klb/src_c/klbapp/klb_app.c`

| 步 | 动作 |
|----|------|
| 1 | 取 `klb_app` 单例, 取得内置 `klua` / `plugins` 扩展 |
| 2 | `klbappex_klua_push_preload(cb_pre_load)` — 产品 Lua 库预加载回调入链 |
| 3 | `on_preload_klb_app` — 执行 `klb_app_push_preload` 链; `klbappex_plugins_preload` — 扫插件目录 |
| 4 | `klbappex_klua_do_preinit` — `klua_env_doinit`, `dofile(argv[1])` |
| 5 | while 未退出: `klb_app_loop_once` + `klb_sleep` |

退出路径: `klbappex_klua_do_prequit` → `klua_env_exit` / `doend`.

## 命令行与入口脚本

> 代码: `klb/src_c/klbapp/klbappex_klua.c`

| 条件 | 行为 |
|------|------|
| `argc >= 2` 且 `argv[1]` 非空 | 作为 Lua 入口脚本 `dofile` |
| 否则 | preinit 失败, `is_load_entry` 为 false, 主循环不跑 Lua 业务 |

产品范例: openipc 以 `argv[1]` 指向入口脚本 (如 `ipcmain.lua`).

## klua 可执行默认行为

> 代码: `klb/klua/klua.cpp`

`klua` 工具入口在 `klb_base_init` 后自动:

1. `klb_app_enable_plugins(true)`
2. 按平台 push plugins 目录 (见 [plugins.md](plugins.md))
3. `klb_app_main(argc, argv, klua_openlibs)` — `klua_openlibs` 内调 `klua_loadlib_all`

产品自建 `main()` 若需 plugins, 须自行 `enable` + `push_path`; 不经过 `klua.cpp` 则 plugins **默认仍关闭**.

## 主循环与 sleep

> 代码: `klb/src_c/klbapp/klb_app.c` (`klb_app_loop_once`)

1. 取主 `klua_env` 的 `klua_env_get_loop_sleep` 为 `sleep_max`
2. 遍历已注册 loop 扩展, 累加各 `cb_loop_once` 返回值
3. 返回 `sleep_max - sleep` 作为建议休眠毫秒

`klb_app_main`: 返回值 `> 0` 则 `klb_sleep(sleep)`, 否则 `klb_sleep(1)`.

## 产品接入要点

| 项 | 说明 |
|----|------|
| preload | 在 preload 回调里 `klb_app_register_extension` 注册产品 C 扩展 |
| Lua 库 | `cb_pre_load` 内 `klua_loadlib` / `klua_loadlib_all` |
| 多 worker | 入口脚本用 `kthread.start`; 子 env 复用同一预加载链 |
| 范例 | `klb/klua/klua.cpp`, openipc 产品入口 |
