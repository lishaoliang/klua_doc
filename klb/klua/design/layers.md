# klua 分层总览

> `klua_doc/klb/klua/design/layers.md` — 代码: [klb/inc/klua/](https://gitee.com/klua/klb/tree/trunk/inc/klua/), [klb/src_c/klua/](https://gitee.com/klua/klb/tree/trunk/src_c/klua/)

## 结论

klua 自下而上 **8 层**: VM/bundled → C 运行时 → env 扩展 → C→Lua 绑定 → C++ 包装 → app 集成 → klbcore 脚本 → 产品脚本. **C 产品 API** 与 **Lua `require`** 不是同一层.

## 层次图

```
L7  产品业务脚本          bin/*lua/
L6  klbcore 纯 Lua        klb/bin/klbcore/     require klbcore.*
L5  app 集成              klbapp / plugins     klb_app_main, klbappex_*
L4  C++ 包装 (已迁 backup) backup/src_cpp/klua/
L3  C→Lua 绑定 k*         klua_k*.c            require("kco") 等
L2  env 框架扩展           extension/klua_ex_*  无 require
L1  C 运行时核心           klua_env.c           klua_env_*
L0  VM + bundled           lua-5.4.6/ 等        require("cjson") 等
```

## 分层表

| 层 | 名称 | 谁用 | 典型路径 | 对外形态 |
|----|------|------|----------|----------|
| L0 | VM / bundled | 全员间接 | `lua-5.4.6/`, `lua-cjson-*`… | `require("cjson")` 等 |
| L1 | C 运行时 | C 产品 / klbapp | `klua_env.c`, `klua_main.c` | `klua_env_*`, `klua_main` |
| L2 | env 扩展 | C 绑定内部 | `extension/klua_ex_*.c` | `klua_env_register_extension` |
| L3 | C→Lua k* | Lua 脚本 | `klua_base/`, `klua_net/`… | `require("kco")` 等 |
| L4 | C++ 包装 | — | `backup/src_cpp/klua/` | 已迁 backup, 不编入 |
| L5 | app 集成 | 产品进程 | `klbapp/klbappex_klua.*` | 预加载链, plugins |
| L6 | klbcore | Lua 业务 | [klb/bin/klbcore/](https://gitee.com/klua/klb/tree/trunk/bin/klbcore/) | `require("klbcore.*")` |
| L7 | 产品脚本 | 应用 | `bin/*lua/` | 入口 `main.lua` |

## 两类 C 接口

### A. C 开发者 (L1/L2 + 工具头)

现行 klb 以 **C API** 为主; 旧 C++ 包装 (`backup/src_cpp/klua/` 等) 已迁 backup, 不编入.

| 头文件 | 层 | 说明 |
|--------|-----|------|
| `klua_env.h` | L1 | 环境生命周期, LPC 消息, loop |
| `klua_env_extension.h` | L2 | 扩展 register/get |
| `klua_coroutine.h` | L2 | kco 底层 yield |
| `klua.h` | L1/L3 | `klua_loadlib`, 栈辅助, `klua_open_*` 声明 |
| `klua_thread.h` | L1/L5 | worker env, LPC 注册 |
| `klua_seri.h` | L1 | LPC 序列化 |
| `klua_object.h` | L2/L3 | `klb_obj_t` 桥 |
| `klua_gui.h` 等 | L2 | GUI / multiplex 桥 |

详 [../api/klua_env.md](../api/klua_env.md).

### B. Lua 脚本 (L3/L6/L7)

| 加载 | 示例 | 文档 |
|------|------|------|
| C 预加载 | `require("kco")` | [lua/klua/readme.md](../../../lua/klua/readme.md) |
| bundled | `require("cjson")` | [lua/bundled/readme.md](../../../lua/bundled/readme.md) |
| klbcore | `require("klbcore.klbui")` | [lua/klbcore/readme.md](../../../lua/klbcore/readme.md) |
| klbcore net/rtsp | `require("klbcore.net.*")`, `klbcore.klbrtsp` | **klbcore-net-design** |

Lua **不应**直接调用 `klua_env_register_extension`.

## 五线扩展对照

设计技能 **klb-klua-design** 五线与层次:

| 线 | 层次 | 文档 |
|----|------|------|
| A C 预加载 | L0 + L3 | [preload.md](preload.md) |
| B src_packages | env / 可选 | [require-guide.md](require-guide.md) |
| C klbcore | L6 | **klbcore-design**; [lua/klbcore/readme.md](../../../lua/klbcore/readme.md) |
| D env 扩展 | **L2** | [env-extension.md](env-extension.md) |
| E plugins | L5→A | [plugins.md](../../klbapp/design/plugins.md) |

plugins (如 `libkpfs` / `kpfs`) 走 **L5 注入 L3**, **不进 L2**.

## 我该改哪一层

| 需求 | 层 | 做法 | 文档 |
|------|-----|------|------|
| 新 Lua 可调 C API | L3 | `klua_open_kxxx` + `klua_loadlib` | 新建 `lua/klua/kxxx.md` |
| env 级调度/资源 | L2 | `klua_ex_*.c` + register | [env-extension.md](env-extension.md) |
| 纯业务逻辑 | L6/L7 | klbcore 或产品脚本 | klbcore / 产品 path |
| 闭源 C dll | L5 | `klbappex_pre_open` | [plugins.md](../../klbapp/design/plugins.md) |
| 升级 Lua | L0 | vendor 目录 | **klb-vendor-subagents** |
| C 产品持 env | L1/L5 | `klua_env_*` / klbapp | [lifecycle.md](lifecycle.md) |
| GUI C 框架 | — | `klb_gui_*`, klbuiex | [klbgui/design/layers.md](../../klbgui/design/layers.md) |
| 模块间异步消息 | L2+L3 | `klpc` + `kco` | [lua/klua/klpc.md](../../../lua/klua/klpc.md) |

## 横切能力

| 能力 | 头 / 实现 | 层 |
|------|-----------|-----|
| 栈/ref 辅助 | `klua.h`, `klua_help.*` | L1 |
| worker 子 env | `klua_thread.h`, `klua_kthread.c` | L1+L3+L5 |
| 序列化 | `klua_seri.h` | L1 (LPC) |

## 相关文档

| 文档 | 内容 |
|------|------|
| [lifecycle.md](lifecycle.md) | L1 调用序 |
| [env-extension.md](env-extension.md) | L2 扩展机制 |
| [preload.md](preload.md) | L0/L3 预加载 |
| [coroutine.md](coroutine.md) | kco vs `coroutine` |
| [k-bindings.md](k-bindings.md) | L3 k* 机制 |
| [lua/klua/readme.md](../../../lua/klua/readme.md) | k* Lua API 索引 |
| [klua_env.md](../api/klua_env.md) | L1/L2 C 头导读 |
| [klbgui/design/layers.md](../../klbgui/design/layers.md) | klbgui 分层 |
| [lua/klbcore/readme.md](../../../lua/klbcore/readme.md) | klbcore 域分配 (**klbcore-design**) |
