# klua 环境生命周期

> `klua_doc/klb/klua/design/lifecycle.md` — 代码: [klb/inc/klua/klua_env.h](https://gitee.com/klua/klb/blob/trunk/inc/klua/klua_env.h), [klb/src_c/klua/klua_env.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua_env.c)

分层总览 [layers.md](layers.md).

## 定位

`klua_env_t` 封装 `lua_State` 与主循环, 驱动协程 (`kco`), 定时器, GUI, 网络等 env 扩展. C 产品经 **klbapp** 持有; 独立进程用 `klua_main`.

## 标准调用序 (C)

> 代码: [klb/inc/klua/klua_env.h](https://gitee.com/klua/klb/blob/trunk/inc/klua/klua_env.h) @history

```
klua_env_create(cb_pre_load)     // 创建, 不初始化 Lua
  → [可选] klua_env_set_preload   // 须在 doinit 前
  → klua_env_doinit               // 初始化 Lua + 预加载链
  → klua_env_dofile / dolibrary   // 加载入口脚本
  → klua_env_loop_once (循环)     // 消息/协程/定时驱动
  → klua_env_exit                 // 置退出标志
  → klua_env_doend                // 清理
  → klua_env_destroy              // 销毁 (或 create 对称)
```

| 阶段 | API | 说明 |
|------|-----|------|
| 创建 | `klua_env_create` | `cb_pre_load` 写入预加载链; 见 [preload.md](preload.md) |
| 配置 | `klua_env_set_udata` / `set_name` / `set_args` / `set_loop_sleep` | 用户数据, 名称, 全局参数, loop 休眠上限 |
| 初始化 | `klua_env_doinit` | `luaL_openlibs` + 预加载回调 + 注册 env 扩展 |
| 加载 | `klua_env_dofile(path)` | 按文件路径执行入口 |
| 加载 | `klua_env_dolibrary(name)` | 按库名 `require` 并设全局 (类似 `lua -l`) |
| 驱动 | `klua_env_loop_once` | **须定期调用**; 处理协程唤醒, LPC 消息, 定时器等 |
| 退出 | `klua_env_exit` | Lua 侧 `ksys.exit()` / `kenv.exit()` 也会置位 |
| 清理 | `klua_env_doend` | 结束 dofile/dolibrary 上下文 |
| 销毁 | `klua_env_destroy` | 释放 env |

## doinit 内部 (要点)

> 代码: [klb/src_c/klua/klua_env.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua_env.c) (`klua_pmain`)

1. `luaL_openlibs(L)` — Lua 5.4 标准库
2. 依次执行 `p_preload_nlist` 中每条 `lua_CFunction` (产品 `klua_loadlib_all`, plugins 等)
3. `klua_register_extension_std` — 注册协程/GUI/netmulti/LPC 等 env 扩展; 详 [env-extension.md](env-extension.md)

## loop_once 职责

单次 `klua_env_loop_once` 大致处理:

- env 扩展 `cb_loop_once` (协程调度, GUI, netmulti, 定时器等)
- LPC 消息分发 (`klua_env_push_lpc_msg`)
- 协程 timeout / wakeup

详 [env-extension.md](env-extension.md) (扩展表, 两阶段注册, LPC 路径).

`klua_env_set_loop_sleep` 限制单次 loop 内 sleep 上限 (毫秒), 避免阻塞过久.

## 与 klbapp 的关系

| 场景 | 入口 | 主循环 |
|------|------|--------|
| 产品进程 | `klb_app_main` | `klb_app_loop_once` → 内含 `klua_env_loop_once` |
| 独立 `klua` | `klua_main` | 直接 `klua_env_loop_once` |
| worker 子线程 | `kthread.start` | 子 env 独立 loop; 预加载链与主线程相同 |

详 **klbapp** 启动: [startup.md](../../klbapp/design/startup.md).

## Lua 侧常用 API

| require | 用途 |
|---------|------|
| `kenv` | `base_path`, `tick_count`, `exit`, loop 休眠等 |
| `ksys` | `exit`, `pack_string`/`unpack`, `get_args` |
| `kco` | 协程 fork/timeout/co_sleep |

入口脚本配置 `package.path` 后 `require("klbcore.*")`; 见 [require-guide.md](require-guide.md).
