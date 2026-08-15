# 扩展机制

> `klua_doc/klb/klbapp/design/extension.md` — 代码: `klb/inc/klbapp/klb_app_extension.h`, `klb/src_c/klbapp/klb_app.c`

## 概述

app 层是 klb 扩展栈最外层容器. 各模块以 **具名扩展** 注册到 `klb_app_t`, 首次 `klb_app_get_extension` 时 **懒激活** (create 实例). 含 `cb_loop_once` 的扩展进入主循环链表.

App 与扩展均为 **单例** (2025-4).

## klb_app_extension_t

> 代码: `klb/inc/klbapp/klb_app_extension.h`

| 回调 | 必填 | 作用 |
|------|------|------|
| `cb_create` | 是 | 创建扩展实例, 返回 `void*` |
| `cb_destroy` | 是 | 销毁实例 |
| `cb_control` | 否 | 消息: `KLBAPPEX_MSG_init` / `KLBAPPEX_MSG_quit` |
| `cb_loop_once` | 否 | 每 tick; 非 NULL 则加入 `p_loop_nlist` |
| `cb_get_ioctrl` | 否 | 填充 `klbappex_ioctrl_t` (opt0~opt8) |

## 注册与获取

> 代码: `klb/inc/klbapp/klb_app_extension.h`

```c
// 注册模板 (preload 或模块 init 中)
klb_app_register_extension(p_app, "my-ext", &g_my_extension);

// 懒激活: 查激活表, 否则从注册表 create
void* p = klb_app_get_extension(p_app, "my-ext");
```

**跨模块访问**:

| API | 返回 |
|-----|------|
| `klb_app_get_extension(p_app, name)` | 扩展实例 `void*` |
| `klbappex_get_ioctrl(p_app, name)` | `klbappex_ioctrl_t*` (opt0~opt8) |

## ioctrl

> 代码: `klb/inc/klbapp/klb_app_extension.h`

`klbappex_ioctrl_t` 提供 `ioctrl_opt0` ~ `ioctrl_opt8`, 供扩展对外暴露次级接口. 调用方用 `klbappex_ioctrl_optN` 转发.

## 内置扩展

> 代码: `klb/src_c/klbapp/klbappex_klua.c`, `klbappex_plugins.c`

| 名称 | 头文件 | 作用 |
|------|--------|------|
| `KLBAPPEX-klua` | `klbappex_klua.h` | 持有主 `klua_env_t`, 合并预加载链, `cb_loop_once` → `klua_env_loop_once` |
| `KLBAPPEX-plugins` | `klbappex_plugins.h` | dll/so 扫描; 无 `cb_loop_once` |

### klua 内置扩展

| API | 说明 |
|-----|------|
| `klbappex_get_klua(p_app)` | 取 klua appex |
| `klbappex_klua_push_preload(p_appex, cb)` | 追加 Lua 预加载回调 |
| `klbappex_klua_get_klua_env(p_appex)` | 取主 `klua_env_t*` |

标志 `is_load_entry`: 入口脚本加载成功后, klua 扩展才参与 loop.

### plugins 内置扩展

专名 **plugins** = app 层 dll/so 动态库扫描 (与 klua 预加载/kpa/klbcore 口语「插件」区分).

详 [plugins.md](plugins.md).

## 静态扩展 vs plugins dll

| 维度 | 静态 `klb_app_register_extension` | plugins dll/so |
|------|-------------------------------------|----------------|
| 注册时机 | `klb_app_push_preload` 或模块 init | `plugins_preload` 扫目录 |
| 编译 | 链入主程序 / libklb | 独立 `.dll` / `.so` |
| 典型用途 | 产品同进程 C 模块 | 闭源扩展, 跨子项目 klua 绑定 |

静态扩展源文件命名: `klbappex_*.c` (`klb/src_c/klbapp/` 或产品目录).

## 与 klua 的关系

```
klbapp (壳)
  └── KLBAPPEX-klua → klua_env_t → kco / netmulti / GUI 绑定
  └── KLBAPPEX-plugins → 动态库扩展
```

klua env 生命周期, 预加载, 协程: 见 [../../klua/readme.md](../../klua/readme.md).
