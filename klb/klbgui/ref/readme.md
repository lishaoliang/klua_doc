# 外部 UI 参照索引

> `klua_doc/klb/klbgui/ref/` — **【外部参照】** 非 klbgui 实现说明

## 免责声明

本目录归纳 **Web / Qt / Android / 嵌入式 UI** 等通用概念与原理，**不是** klb 行为规格。  
klbgui **自有实现**以 [../design/](../design/) 为准；冲突时以源码与 `design/` 文档为准。

## 文档清单

| 文件 | 说明 | 状态 |
|------|------|------|
| [layout-principles.md](layout-principles.md) | 各平台自动布局原理（Measure/Layout、Flex/Qt/Android/LVGL 对照） | 参照 |
| [extension-principles.md](extension-principles.md) | 插件/扩展架构通论（register/生命周期/tick；Qt/引擎等对照） | 参照 |

## 与自有文档关系

```
ref/          外部怎么做（原理、术语）
  ↓ 对照
design/map/   外部概念 ↔ klb 键/API
  ↓ 实现
design/       klb 怎么做（BoxFlow 等）
```

## 关联

| 入口 | 说明 |
|------|------|
| [../design/layout.md](../design/layout.md) | 【自有】BoxFlow |
| [../readme.md](../readme.md) | klbgui 文档枢纽 |
