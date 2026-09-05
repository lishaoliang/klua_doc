# 自动布局概念对照（映射）

> `klua_doc/klb/klbgui/design/map/layout-map.md` — **【映射对照】** 外部概念 ↔ klb BoxFlow  
> 状态: **桩文档**

自有设计 [../layout.md](../layout.md). 外部原理 [../../ref/layout-principles.md](../../ref/layout-principles.md).

## 结论

本表说明 klb **采纳 / 省略** 哪些外部布局概念；**klb 键名与行为**以 [layout.md](../layout.md) 为准。

## 1. 容器与模式

| 外部 | klb BoxFlow | 备注 |
|------|-------------|------|
| CSS `display:flex` | `layout:flow` | 仅 flow；无 `display` 键 |
| Qt `QBoxLayout` | `layout:flow` + `flex-direction` | |
| Android `LinearLayout` | `layout:flow` | |
| CSS `display:grid` | — | 首版不做 |
| Qt `QFormLayout` | — | P2 规划 `layout:form` |

## 2. 方向与间距

| 外部 | klb | 备注 |
|------|-----|------|
| `flex-direction` | `flex-direction` | row / column |
| CSS `gap` | `gap` | int px |
| Qt `setSpacing()` | `gap` | |
| Android `divider` | — | 用 `gap` + margin |

## 3. 尺寸与分剩余

| 外部 | klb | 备注 |
|------|-----|------|
| `flex-grow:1` | `flex:1` | 无 shrink/basis |
| Qt `Expanding` / stretch | `flex:1` | |
| Android `layout_weight` | `flex:1` | |
| Android `match_parent` | `fill` / parse `pos=-1` | |
| Android `wrap_content` | 显式 wh 或 type 默认 | **不用** suggestw |
| Qt `sizeHint()` | — | 将来或 `measure-w`/`measure-h` |
| klb `suggestw`/`suggesth` | — | **禁止**作布局输入 |

## 4. 对齐

| 外部 | klb | 备注 |
|------|-----|------|
| `align-items` | `align-items` | start/center/end/stretch |
| `justify-content` | `justify-content` | start/center/end；无 space-between（首版） |
| `align-self` | `layout-align-self` | 可选 |

## 5. 明确不采纳（首版）

| 外部 | 原因 |
|------|------|
| flex-wrap | ARM 单遍；复杂度 |
| `%` / `calc()` / em | 无单位解析 |
| CSS Grid | 二维；ROI 低 |
| ConstraintLayout | 约束求解过重 |
| 每帧 relayout | 仅 dirty 触发 |
| suggestw 驱动布局 | 不可信；见 layout.md §3 |
