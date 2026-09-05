# 外部插件/扩展架构原理（参照）

> `klua_doc/klb/klbgui/ref/extension-principles.md` — **【外部参照】**  
> 状态: **参照文档**（非 klb 行为规格）

索引 [readme.md](readme.md). klb 自有设计 [../design/extension.md](../design/extension.md). 分层 [../design/layers.md](../design/layers.md).

## 结论

各平台 GUI / 应用宿主普遍采用 **薄核心 + 可注册模块 + 统一生命周期 + 主循环 tick** 组织子系统.  
klb `klbuiex_*` 属于此族; 本文帮助理解 **为何** 拆扩展、**业界怎么说**, **不定义** klb API 与时序 (以 [extension.md](../design/extension.md) 与源码为准).

## 1. 共同模式

### 1.1 四阶段流水线

多数可插拔 UI / 应用框架可归纳为:

```
register（注册表 / 工厂）
  → create / 懒激活（实例化）
  → control / msg（quit、clear、init…）
  → loop_once / update（每 tick，可选）
```

| 阶段 | 典型职责 | 常见命名 |
|------|----------|----------|
| 注册 | 登记接口/vtable, 尚未分配实例 | register, load, install |
| 激活 | 首次使用时 create | get, lazy init, ensure |
| 控制 | 全局事件: 退出、清数据、配置变更 | onQuit, clear, shutdown hook |
| 驱动 | 每帧/每 tick 推进定时器、动画、脏区 | update, tick, on_frame |

klb GUI 对应: [klb/inc/klbgui/klbui_extension.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_extension.h) 的 `cb_create` / `cb_destroy` / `cb_control` / `cb_loop_once`.

### 1.2 注册表 vs 激活表

两表分离是常见优化:

| 表 | 内容 | 典型时机 |
|----|------|----------|
| 注册表 | 模块描述符 (vtable / 工厂) | 进程或容器 create 时 |
| 激活表 | 已实例化的模块对象 | 首次 get 或 eager 预激活 |

**收益**: 登记成本低; 未用模块不占内存; 「换页 clear」可只清业务数据而 **保留** 注册与已激活基础设施 (klb `klb_gui_clear` 即此语义).

### 1.3 生命周期消息

| 消息类 | 常见语义 | klb GUI (自有) |
|--------|----------|----------------|
| quit / shutdown | 容器即将销毁, 模块释放资源 | `KLBUI_EX_MSG_quit` |
| clear / reset | 清 UI/业务状态, 框架可保留 | `KLBUI_EX_MSG_clear` |
| init / configure | 参数变更后重建 (部分框架) | klb 无独立 init msg; 靠 create/get |

**反序通知**: 后注册者往往依赖先注册者; destroy/clear 时 **后入先出 (LIFO)** 可减少悬空引用. klb `cb_control(quit|clear)` 采用反序.

### 1.4 主循环中的扩展 tick

典型 GUI loop:

1. 采集/分发输入
2. 逻辑与定时器 update
3. 布局/脏区
4. 绘制合成

扩展 tick 常放在 **输入之后、呈现之前**, 以便定时器与重绘与帧对齐.  
klb: [klb/src_c/klbgui/klb_gui.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klb_gui.c) 在消息处理后调用各扩展 `cb_loop_once`, 最后 `render` 刷新; **`is_wait` 不冻结扩展 tick** (等待遮罩仍允许 ticker/time).

## 2. 业界对照

**说明**: 下列为 **模式类比**, 非 klb 实现映射; 链接供延伸阅读.

| 类比对象 | 参考 | 相似点 | 与 klbuiex 主要差异 |
|----------|------|--------|---------------------|
| **Qt 插件** | [QPluginLoader](https://doc.qt.io/qt-6/qpluginloader.html) | 接口对象、静/动态 load、懒加载 | 通用桌面; 无 klua env 嵌套; 多进程插件常见 |
| **游戏/工具引擎** | [Editor plugin architecture 通论](https://gamedev.stackexchange.com/questions/48649/game-editor-plugin-architecture) | 宿主定义接口、扩展点、load 注入 | 编辑器语境; 少 GUI clear 保留注册表语义 |
| **OBS Lua 插件** | [OBS Lua 插件](https://dev.to/hectorleiva/start-to-write-plugins-for-obs-with-lua-1172) | C 核心 + 脚本扩展 + load/unload 生命周期 | 音视频场景; 非窗口树 + CSS |
| **DOME 引擎** | [Plugins / Lifecycle hooks](https://domeengine.com/plugins/) | 原生插件 + load/update/draw/unload | 脚本为 Wren; 单层插件 |
| **REFramework** | [Native Plugin](https://deepwiki.com/praydog/REFramework/5.2-native-plugin-system) | dll + 共享脚本运行时 + 帧回调 | 注入已发行程序, 非自研 GUI 栈 |

**通式表述**: 宿主定义契约 → 模块 register → 生命周期 hook → 主循环各阶段回调 → 可选动态库与脚本层并存.

## 3. 与 klbgui klbuiex 的对照（只读指针）

| 外部概念 | klb 落点 (自有) |
|----------|-----------------|
| 插件接口 struct | `klb_gui_extension_t` |
| 注册 API | `klb_gui_register_extension` |
| 懒激活 | `klb_gui_get_extension` |
| 标准模块集 | 10× `klbuiex_*` (`KLBUIEX_register_extensions_std`) |
| 控件工厂 (非 extension 插件) | `klbuiex_wndhash` 内 type→create |
| env 级 GUI 壳 | `_KLUA_EX_GUI_` (每 env 一个 `p_gui`) |

完整契约、loop 步骤、clear/destroy 表: [../design/extension.md](../design/extension.md).

## 4. 选型提示（参照层, 非 klb 规范）

| 需求 | 外部常见做法 | klb 倾向 (见 design/) |
|------|--------------|------------------------|
| 闭源/热插 C 模块 | dll/so + 宿主扫描 | app **plugins** 层 (非 GUI 扩展) |
| GUI 子系统 (渲染/定时器) | 引擎子模块 register | `klbuiex_*` 或自定义 `klb_gui_register_extension` |
| 业务控件类型 | 工厂/类注册 | wndhash `klbuiex_wndhash_register` |
| 纯 Lua UI | 脚本 package | `klbcore.klbui`, 非 C extension |

## 关联

| 文档 | 说明 |
|------|------|
| [../design/extension.md](../design/extension.md) | **【自有】** klbuiex 契约与 10 扩展 |
| [../design/layers.md](../design/layers.md) | L2 扩展在分层中的位置 |
| [../../klua/design/env-extension.md](../../klua/design/env-extension.md) | klua env 扩展 (含 `_KLUA_EX_GUI_`) |
