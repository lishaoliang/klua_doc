# klua C API 导读 (L1/L2)

> `klua_doc/klb/klua/api/klua_env.md` — 头文件: `klb/inc/klua/`

面向 **C/C++ 产品代码** (klbapp, 独立 `klua`, 高级绑定).

> **Lua 脚本 API (L3 k\*)**: [lua/klua/readme.md](../../../lua/klua/readme.md) 及各 `kxxx.md`; 机制见 [k-bindings.md](../design/k-bindings.md); 清单 [require-guide.md](../design/require-guide.md). 分层 [layers.md](../design/layers.md).

## klua_env.h (L1 核心)

> 实现: `klb/src_c/klua/klua_env.c`

### 生命周期

| API | 说明 |
|-----|------|
| `klua_env_create(cb_pre_load)` | 创建 `lua_State`, **不**初始化标准库 |
| `klua_env_set_preload` | 须在 `doinit` 前 |
| `klua_env_doinit` | `luaL_openlibs` + 预加载链 |
| `klua_env_dofile` / `dolibrary` | 加载入口; 保存 `G`/`kexit` |
| `klua_env_loop_once` | **须周期调用**; 消息/扩展/GC |
| `klua_env_exit` / `doend` / `destroy` | 退出与释放 |

详 [lifecycle.md](../design/lifecycle.md).

### 配置与状态

| API | 说明 |
|-----|------|
| `klua_env_set_udata` / `get_udata` | 用户指针 |
| `klua_env_set_name` / `get_name` | env 名称 (sds) |
| `klua_env_set_args` / `get_args` | 启动参数 (`klb_buf_t`) |
| `klua_env_set_loop_sleep` / `get_loop_sleep` | loop 休眠上限 (ms) |
| `klua_env_get_tick_count` / `update_tick_count` | 滴答 |
| `klua_env_is_exit` | 退出标志 |
| `klua_env_get_L` / `get_by_L` | `lua_State` 与 env 互查 |
| `klua_env_report` / `report_by_L` | 错误报告 |

### LPC 消息

| API / 类型 | 说明 |
|------------|------|
| `klua_msg_t` | `POST`/`REQUEST`/`RESPONSE`/`NOTIFY`; name 最长 15 |
| `klua_env_push_lpc_msg` | 跨线程投递 |
| `klua_msg_free` | 释放消息 |

Lua 侧见 [lua/klua/klpc.md](../../../lua/klua/klpc.md); 路径详 [env-extension.md](../design/env-extension.md).

## klua_env_extension.h (L2)

| API / 类型 | 说明 |
|------------|------|
| `klua_env_extension_t` | vtable: create/destroy/ctrl/msg/loop_once |
| `klua_env_register_extension` | 注册到 env **注册表** |
| `klua_env_get_extension` | **懒激活** 并返回实例 |
| `KLUA_ENV_EX_quit` | 唯一控制枚举 (退出) |

标准 `klua_ex_*` 表见 [env-extension.md](../design/env-extension.md).

## klua_coroutine.h (L2, kco 底层)

| API | 说明 |
|-----|------|
| `klua_coroutine_get` / `get_by_L` | 取 coroutine 扩展 |
| `klua_coroutine_yield` | 替代 `lua_yield`; 退出时可强制唤醒 |
| `klua_coroutine_rawgeti` | 协程栈恢复 |
| `klua_coroutine_debug_check` | 调试校验 |

业务脚本用 `require("kco")`; Lua 文档见 [lua/klua/guide/coroutine.md](../../../lua/klua/guide/coroutine.md).

## klua.h (L1 工具 + L3 预加载声明)

### 进程入口与预加载

| API | 说明 |
|-----|------|
| `klua_main(argc, argv, cb)` | 独立可执行入口 |
| `klua_loadlib(L, openlib, name)` | 写 `registry._PRELOAD` |
| `klua_openlibs_cb` | 预加载回调类型 |

`klua_open_kco`, `klua_open_khttp` 等声明于此; 聚合 `klua_loadlib_all` 在 `klua.c`.

### 栈与类型辅助

| API | 说明 |
|-----|------|
| `KLUA_HELP_TOP_B/E` | 栈深度 assert |
| `klua_ref_registryindex` / `unref` | registry 引用 |
| `klua_check_option_*` | 可选参数 |
| `klua_is_*` / `klua_setfield_*` | 类型探测与设表字段 |
| `klua_is_coroutine` / `check_coroutine` | 协程检测 |

绑定实现须优先复用上述辅助, 见 **klb-coding-c** Lua 绑定行.

## klua_thread.h (L1/L5 worker)

| API | 说明 |
|-----|------|
| `klua_thread_set_preload` | worker 预加载链 (与主线程同链) |
| `klua_thread_register` / `unregister` | 注册外部 env |
| `klua_thread_register_lpc_module` | LPC 模块名注册 |
| `klua_thread_register_lpc` | 分配 LPC 实例名 |

配合 `require("kthread")`; 见 [lua/klua/kthread.md](../../../lua/klua/kthread.md).

## klua_seri.h (L1 LPC 载荷)

| API | 说明 |
|-----|------|
| `klua_seri_json_pack` / `unpack` | JSON 与栈 |
| `klua_seri_map_pack` / `unpack` | `klb_map_t` |
| `klua_seri_map_binary_pack` / `unpack` | 二进制 (klpc 常用) |

## klua_object.h (L2/L3)

| API | 说明 |
|-----|------|
| `klua_object_get` / `get_by_L` | object 扩展 |
| `klua_object_register_ops` | 注册 `klb_obj_t` ops |

## 其它对外头 (L2 桥)

| 头文件 | 说明 |
|--------|------|
| `klua_gui.h` | `klb_gui_t` 与 env 挂接 |
| `klua_netmulti.h` | 新 `klb_netmulti_t` |
| `klua_buffer.h` | env 缓冲辅助 |

实现多在 `extension/klua_ex_*.c`; Lua 侧通常经 `kgui`/`krtsp` 等 k* 访问.

**已迁 backup**: `klua_multiplex.h` / `klua_ex_multiplex.c` — `backup/inc/klua/`, `backup/klua/extension/`.

## 内部头 (非产品直接 include)

| 路径 | 说明 |
|------|------|
| `src_c/klua/extension/klua_ex_*.h` | 扩展模块内部 |
| `src_c/klua/klua_help.h` | 错误报告等 |
| `src_c/klua/extension/klua_extension.h` | `klua_register_extension_std` |

## app 集成 (L5, 非 inc/klua)

| 头文件 | 说明 |
|--------|------|
| `klb/inc/klbapp/klbappex_klua.h` | 主 env, 预加载 push |
| `klb/inc/klbapp/klb_app_extension.h` | plugins `klbappex_pre_*` |

详 [klbapp 启动](../../klbapp/design/startup.md), [plugins](../../klbapp/design/plugins.md).

## 查阅顺序

1. 本页 + [layers.md](../design/layers.md)
2. `klb/inc/klua/*.h`
3. `klb/src_c/klua/klua_env.c`, `extension/`
4. 设计技能 **klb-klua-env-design**
