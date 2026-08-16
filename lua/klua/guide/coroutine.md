# 协程: 为何用 kco

> `klua_doc/lua/klua/guide/coroutine.md` — 类别: ④ klua 扩展 | 代码: `klb/src_c/klua/klua_base/klua_kcoro.c`, `klb/src_c/klua/extension/klua_ex_coroutine.c`

## 结论

**业务脚本禁止使用 Lua 内置 `coroutine` 库, 须用 `require("kco")`.**

## 原因

| 维度 | Lua `coroutine` | klb `kco` |
|------|-----------------|-----------|
| 调度 | 解释器自带, 与 env 无关 | 由 `klua_ex_coroutine` 在 `loop_once` 中统一唤醒 |
| 定时 | 无框架级 timeout | `kco.timeout`, `kco.co_sleep` 挂到 env 滴答 |
| LPC/IO | 与 `klpc:co_call` 等不集成 | `co_recv` / `co_call` 在协程内 yield |
| 冲突 | 与 klb 协程扩展 **冲突** | 替代方案 |

`klua_env_doinit` 仍调用 `luaL_openlibs`, 故 `coroutine` 表存在; 但 klb 框架 (GUI 事件, 网络, LPC) 假定协程经 **kco** 调度.

## kco API 概要

> 详 [kco.md](../kco.md)

| 函数 | 说明 |
|------|------|
| `kco.fork(func, ...)` | 立即 fork 新协程 |
| `kco.timeout(ms, func, ...)` | 延迟 ms 后启动协程 |
| `kco.co_sleep(ms)` | **仅协程内** 休眠; 内部 `lua_yield` |

## 与 klpc 配合

跨模块 RPC 须在协程中等待:

```lua
local kco = require('kco')
local klpc = require('klpc')

local mo = klpc.new_module('worker')
local lpc = klpc.new()

kco.fork(function ()
    local src, msg = mo:co_recv()
    mo:response(src, 'ok', msg)
end)

kco.fork(function ()
    local ret = lpc:co_call('worker', 'hello')
    print(ret)
end)
```

详 [klpc.md](../klpc.md).

## 与 ktime 的区别

| API | 阻塞对象 | 使用场景 |
|-----|----------|----------|
| `kco.co_sleep` | **协程** yield, 不阻塞 C 线程 | 协程内异步等待 |
| `ktime.sleep` | **C 线程** 直接 sleep | 非协程, 或极短阻塞 |
| `ktime.timer` / `ticker` | C 层定时回调 | **仅非协程** (help 注释) |

## 编码规范

- 新脚本: `kco.fork` / `kco.timeout` / `kco.co_sleep`, 不用 `coroutine.create` / `coroutine.resume`
- yield 路径经 `klua_coroutine_yield` (C 绑定内部)
- 规则 **coding-lua**: 协程一节

## 相关 env 扩展

| 注册名 | 源文件 | 作用 |
|--------|--------|------|
| `_KLUA_EX_COROUTINE_` | `klua_ex_coroutine.c` | kco 调度, wakeup, timeout |

非 Lua `require`; 由 `klua_register_extension_std` 在 create 时注册. 全表见 [klb/klua/design/env-extension.md](../../../klb/klua/design/env-extension.md).
