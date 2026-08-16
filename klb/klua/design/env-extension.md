# env 框架扩展

> `klua_doc/klb/klua/design/env-extension.md` — 代码: `klb/inc/klua/klua_env_extension.h`, `klb/src_c/klua/klua_env.c`, `klb/src_c/klua/extension/`

分层总览 [layers.md](layers.md).

## 定位

**env 框架扩展** 挂到每个 `klua_env_t` 上, 由 C 主循环驱动, **不是** Lua `require` 模块.

| 对比 | C 预加载 (`require`) | env 扩展 |
|------|----------------------|----------|
| 挂载点 | `registry._PRELOAD` | `klua_env_t` 扩展表 |
| Lua 可见 | `require("kco")` 等 | 否 (C 层 + k* 绑定间接使用) |
| 驱动 | `require` 懒加载 | `klua_env_loop_once` / LPC 消息 |
| 文档 | [preload.md](preload.md) | 本文 |

plugins dll (如 `libkpfs`) 走预加载链; **不在** `klua_env_create` 时注册 env 扩展, 但可在 **openlib 首次 `require`** 时调用 `klua_env_register_extension` 懒登记 (见下节 kpfs).

## 与五线扩展的关系

klua 扩展分五条线 (详 **klb-klua-design**); 本文对应 **D 线**:

```
A. C 预加载 (require)     → preload.md
B. kpa_* 包               → require-guide.md
C. klbcore 脚本           → require-guide.md
D. env 框架扩展 (本文)    → klua_ex_*, loop_once
E. plugins 注入           → klbapp/plugins.md
```

## 契约 (C API)

> 头文件: `klb/inc/klua/klua_env_extension.h`

```c
typedef struct klua_env_extension_t_ {
    void* (*cb_create)(klua_env_t* p_env);
    void  (*cb_destroy)(void* ptr);
    int   (*cb_ctrl)(void* ptr, klua_env_t* p_env, int opt,
                     uint8_t* p_param_in_out, int param_size);
    int   (*cb_msg)(void* ptr, klua_env_t* p_env, int64_t now, klua_msg_t* p_msg);
    int   (*cb_loop_once)(void* ptr, klua_env_t* p_env,
                          int64_t last_tc, int64_t now);
} klua_env_extension_t;
```

| API | 说明 |
|-----|------|
| `klua_env_register_extension` | 注册 vtable 到 env **注册表** |
| `klua_env_get_extension` | **懒激活** 并返回 `cb_create` 实例指针 |

| 回调 | 时机 | 必填 |
|------|------|------|
| `cb_create` | 首次 `get_extension` | 是 |
| `cb_destroy` | `klua_env_destroy` | 是 |
| `cb_ctrl` | 退出等控制消息 | 否 |
| `cb_msg` | LPC 消息 | 否 |
| `cb_loop_once` | 每轮 `loop_once` | 否 |

控制枚举 `klua_env_extension_opt_e` 当前仅 `KLUA_ENV_EX_quit`.

## 两阶段: 注册表 + 激活表

> 实现: `klb/src_c/klua/klua_env.c`

```
klua_env_create
  → klua_register_extension_std / _cpp   // 写入 p_extension_hlist (vtable 副本)

首次 klua_env_get_extension(name)
  → cb_create(p_env)                     // 写入 p_extension_activate_hlist (实例)

klua_env_loop_once
  → 仅遍历 p_extension_activate_hlist 的 cb_loop_once
```

要点:

- **懒激活**: 未 `get_extension` 的扩展不参与 `loop_once`
- **loop 顺序**: 按 **激活顺序**, 非注册顺序
- **销毁**: 先对激活表 `cb_destroy`, 再释放注册表 vtable; 未激活者无实例

## loop_once 流程

> 代码: `klua_env_loop_once` (`klua_env.c`)

1. 原子锁搬运 LPC 消息 `p_lpc_msg_list` → `p_msg_list`
2. **更新** `p_env->tc` (协程 `co_sleep` 等依赖滴答)
3. `klua_env_loop_msg` — LPC 四类消息 → `_KLUA_EX_LPC_`
4. 遍历激活扩展 `cb_loop_once`, 累加建议已消耗 sleep
5. 间隔约 30s 触发 `lua_gc`
6. 返回 `loop_sleep - sleep` (默认 `loop_sleep` 10ms)

生命周期总览见 [lifecycle.md](lifecycle.md).

## 退出路径

`klua_env_doend` → `klua_pquit`:

1. 调 Lua 全局 `kexit`
2. 对已激活扩展 `cb_ctrl(KLUA_ENV_EX_quit)`
3. `klua_ex_coroutine_exit` 排空协程队列
4. unref `G` / `kexit`, GC

## LPC 消息

> `klua_msg_t` 定义: `klb/inc/klua/klua_env.h`

| type | 含义 |
|------|------|
| `KLUA_LPC_POST` | 无响应投递 |
| `KLUA_LPC_REQUEST` | 请求 |
| `KLUA_LPC_RESPONSE` | 响应 |
| `KLUA_LPC_NOTIFY` | 通知 |

`dst_name` / `src_name` 最长 15 字符.

路径:

```
klua_env_push_lpc_msg (跨线程)
  → loop_once 搬运
  → klua_ex_lpc_msg 按 dst_name 分发
  → klpc 模块唤醒 kco 或暂存
```

Lua 侧用法见 [lua/klua/klpc.md](../../../lua/klua/klpc.md). 找不到 `dst_name` 时 C 层 **assert** 失败.

## 标准 env 扩展

> 注册: `klb/src_c/klua/extension/klua_extension.c`

| 注册名 | 源文件 | 作用 | loop | quit |
|--------|--------|------|------|------|
| `_KLUA_EX_OBJECT_` | `klua_ex_object.c` | `klb_obj_t` | — | — |
| `_KLUAEX_BUFAGENT_` | `klua_ex_bufagent.c` | buf 池 | — | — |
| `_KLUA_EX_COROUTINE_` | `klua_ex_coroutine.c` | kco 调度 | 唤醒/超时 | yes |
| `_KLUA_EX_TIME_` | `klua_ex_time.c` | timer/ticker | 调 Lua 定时 | — |
| `_KLUAEX_NETMULTI_` | `klua_ex_netmulti.c` | 新 netmulti | sleep 建议 | — |
| `_KLUA_EX_LPC_` | `klua_ex_lpc.c` | 跨 env LPC | 消息驱动 | — |
| `_KLUA_EX_GUI_` | `klua_ex_gui.c` | `klb_gui_t` | GUI loop | yes |
| `_CKLUA_EX_GUI_` | `CKluaExGui.cpp` | C++ GUI | GUI loop | — |

**非** `require` 名; C 绑定通过 `klua_ex_get_*` 懒激活:

| require / 场景 | 触发的扩展 |
|----------------|------------|
| `kco` | coroutine |
| `ktime` | time |
| `krtsp` / `ksmp` 等 | netmulti + coroutine |
| `kgui` | gui |
| `klpc` | lpc + coroutine |

协程详 [lua/klua/guide/coroutine.md](../../../lua/klua/guide/coroutine.md); GUI 详 [lua/klua/kgui.md](../../../lua/klua/kgui.md).

## 新增 env 扩展 (C 开发者)

1. 新建 `klua_ex_xxx.c`, 实现 `cb_create` / `cb_destroy` 及可选 `cb_loop_once` / `cb_msg` / `cb_ctrl`
2. `klua_ex_register_xxx` 内 `klua_env_register_extension`
3. 在 `klua_register_extension_std` 或产品 `klua_env_create` 后注册
4. 对外提供 `klua_ex_get_xxx` → `klua_env_get_extension`

闭源动态库 **不能** 编入 `klua_register_extension_std`; 可在运行时 `klua_env_register_extension` (kpfs 范例), 或经 app `klbappex_ex_open` (app 级扩展).

### kpfs (`libkpfs.so`, pfs klua)

| 项 | 约定 |
|----|------|
| 注册名 | `_KPFS_EX_ENV_` (`PFS_KLUA_EX_NAME`) |
| 源文件 | `portfs/src_klua/pfs_klua_ex.{h,c}` |
| 触发 | `pfs_klua_ex_get` / `pfs_klua_ex_get_by_L` — 首次 get 注册 vtable 并创建实例; 绑定层按需调用 |
| 作用 | 每 env 单例; disk 登记表; `KLUA_ENV_EX_quit` 兜底 close |
| 设计 | **pfs-klua-design** § env 扩展 |

## 审查要点

| 项 | 说明 |
|----|------|
| 懒激活 | 扩展须被 `get_extension` 后才进 loop |
| `tc` 顺序 | loop 内须先更新 `tc` 再跑扩展 |
| quit | coroutine 与 gui 有 `cb_ctrl(quit)` 清理 |
| plugins | `kpfs` 预加载 + `pfs_klua_ex_get` 懒注册 env 扩展 (非 create 时 std 注册) |

## 相关文档

| 文档 | 内容 |
|------|------|
| [lifecycle.md](lifecycle.md) | `klua_env` 调用序与 loop 总览 |
| [preload.md](preload.md) | `require` 预加载 (A 线) |
| [coroutine.md](coroutine.md) | kco 与 coroutine 扩展 |
| [lua/klua/klpc.md](../../../lua/klua/klpc.md) | LPC Lua API |
| [klbapp/plugins.md](../../klbapp/design/plugins.md) | plugins 预加载 (E 线) |
