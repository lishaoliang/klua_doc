# klbapp 文档

> `klua_doc/klb/klbapp/` — 代码: `klb/inc/klbapp/`, `klb/src_c/klbapp/`
> **C/C++ 模块文档格式样板** → **doc-writing** § klb C/C++ 模块文档

**klbapp** 是 klb 的进程级应用壳: 单例聚合模块, 用「注册 → 懒激活 → loop」驱动 Lua 主程序. 产品 C 入口经 `klb_base_init` → 配置 preload/plugins → `klb_app_main` 启动.

## 子目录

| 路径 | 内容 |
|------|------|
| [design/](design/) | 启动流程, 扩展机制, plugins |
| [api/](api/) | C API 说明 |

## 设计

| 文件 | 说明 |
|------|------|
| [design/startup.md](design/startup.md) | 进程启动与 `klb_app_main` 五步 |
| [design/extension.md](design/extension.md) | `klb_app_extension_t` 与 ioctrl |
| [design/plugins.md](design/plugins.md) | dll/so 扫描与导出契约 |

## API

| 文件 | 说明 |
|------|------|
| [api/klb_app.md](api/klb_app.md) | `klb_app.h` / `klb_app_extension.h` 等 |

## 相关文档

| 主题 | 入口 |
|------|------|
| klua 运行时, 预加载, env | [../klua/readme.md](../klua/readme.md) |
| 产品 Lua, worker 划分 | `klb/bin/klbcore/` (脚本侧) |
| 闭源 klua 模块范例 (kpfs) | portfs `libkpfs.so` |
| C API | `klb/inc/` 各模块头文件; app → [api/klb_app.md](api/klb_app.md) |
