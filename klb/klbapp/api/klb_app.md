# klbapp C API

> `klua_doc/klb/klbapp/api/klb_app.md` — 代码: [klb/inc/klbapp/](https://gitee.com/klua/klb/tree/trunk/inc/klbapp/)

## 头文件

| 头文件 | 内容 |
|--------|------|
| `klb_app.h` | 主流程, preload, plugins 配置 |
| `klb_app_extension.h` | 扩展 vtable, ioctrl, dll 导出符号宏 |
| `klbappex_klua.h` | 内置 klua 扩展 |
| `klbappex_plugins.h` | 内置 plugins 扩展 |

## klb_app.h

> 代码: [klb/inc/klbapp/klb_app.h](https://gitee.com/klua/klb/blob/trunk/inc/klbapp/klb_app.h)

### klb_app_main

```c
int klb_app_main(int argc, char** argv, lua_CFunction cb_pre_load);
```

app 主流程. `cb_pre_load` 为所有 Lua 环境的预加载库回调.

调用次序: `klb_base_init()` → `klb_app_main()` → `klb_base_quit()`.

### klb_app_instance

```c
klb_app_t* klb_app_instance();
```

获取 app 单例.

### 须在 main 之前配置

| API | 说明 |
|-----|------|
| `void klb_app_push_preload(klb_app_preload_cb cb)` | 追加 preload 回调; 回调内可 `klb_app_register_extension` |
| `void klb_app_enable_plugins(bool enable)` | 是否启用 plugins; 默认未启用 |
| `void klb_app_push_plugins_path(const char* p_path)` | plugins 扫描目录 (非单文件) |

`klb_app_preload_cb` 类型: `int (*)(klb_app_t* p_app)`.

## klb_app_extension.h

> 代码: [klb/inc/klbapp/klb_app_extension.h](https://gitee.com/klua/klb/blob/trunk/inc/klbapp/klb_app_extension.h)

### 注册与获取

```c
int  klb_app_register_extension(klb_app_t* p_app, const char* p_name,
                                const klb_app_extension_t* p_extension);
void* klb_app_get_extension(klb_app_t* p_app, const char* p_name);
```

### ioctrl

```c
klbappex_ioctrl_t* klbappex_get_ioctrl(klb_app_t* p_app, const char* p_name);

int klbappex_ioctrl_opt0(klbappex_ioctrl_t* p_ioctrl, int opt);
// ... opt1 ~ opt8
```

### klb_app_extension_t

| 成员 | 说明 |
|------|------|
| `cb_create` | 创建扩展实例 (**必填**) |
| `cb_destroy` | 销毁 (**必填**) |
| `cb_control` | 消息 `KLBAPPEX_MSG_init` / `KLBAPPEX_MSG_quit` |
| `cb_loop_once` | 每 tick; 非 NULL 加入主循环 |
| `cb_get_ioctrl` | 填充 `klbappex_ioctrl_t` |

### 动态库导出符号宏

| 宏 | 符号名 |
|----|--------|
| `KLBAPPEX_DL_init` | `klbappex_init` |
| `KLBAPPEX_DL_quit` | `klbappex_quit` |
| `KLBAPPEX_DL_ex_count` | `klbappex_ex_count` |
| `KLBAPPEX_DL_ex_open` | `klbappex_ex_open` |
| `KLBAPPEX_DL_pre_count` | `klbappex_pre_count` |
| `KLBAPPEX_DL_pre_open` | `klbappex_pre_open` |

`KLBAPPEX_DL_name_max` = 64 (扩展名最大字符数, 不含 `'\0'`).

回调类型见头文件 `klbappex_init_cb`, `klbappex_ex_open_cb`, `klbappex_pre_open_cb` 等.

## klbappex_klua.h

> 代码: [klb/inc/klbapp/klbappex_klua.h](https://gitee.com/klua/klb/blob/trunk/inc/klbapp/klbappex_klua.h)

| API | 说明 |
|-----|------|
| `klbappex_klua_t* klbappex_get_klua(klb_app_t* p_app)` | 取 klua appex |
| `klbappex_klua_t* klbappex_get_klua2()` | 取 klua appex (无 app 参数) |
| `int klbappex_klua_push_preload(klbappex_klua_t*, lua_CFunction cb)` | 追加预加载回调 |
| `klua_env_t* klbappex_klua_get_klua_env(klbappex_klua_t*)` | 取主 Lua 环境 |

## klbappex_plugins.h

> 代码: [klb/inc/klbapp/klbappex_plugins.h](https://gitee.com/klua/klb/blob/trunk/inc/klbapp/klbappex_plugins.h)

| API | 说明 |
|-----|------|
| `klbappex_plugins_t* klbappex_get_plugins(klb_app_t* p_app)` | 取 plugins appex |
| `klbappex_plugins_t* klbappex_get_plugins2()` | 取 plugins appex (无 app 参数) |

扫描与加载逻辑在 `klbappex_plugins.c`; 配置见 `klb_app_enable_plugins` / `klb_app_push_plugins_path`.

## 设计文档

| 主题 | 文档 |
|------|------|
| 启动五步 | [../design/startup.md](../design/startup.md) |
| 扩展机制 | [../design/extension.md](../design/extension.md) |
| plugins | [../design/plugins.md](../design/plugins.md) |
