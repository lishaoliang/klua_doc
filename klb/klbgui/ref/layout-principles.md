# 外部自动布局原理（参照）

> `klua_doc/klb/klbgui/ref/layout-principles.md` — **【外部参照】**  
> 状态: **参照文档**（非 klb 行为规格）

索引 [readme.md](readme.md). klb 自有设计 [../design/layout.md](../design/layout.md). 对照 [../design/map/layout-map.md](../design/map/layout-map.md).

## 结论

各平台自动布局本质相同：给定父容器内容区尺寸 `(W, H)` 和 N 个子项，为每个可见子项算出相对父的 `(x, y, w, h)`。

差异在约束表达与求解复杂度。klb BoxFlow 仅取 **一维流式 + 分剩余** 交集；本文帮助理解选型，**不定义 klb API**（键名与行为以 [layout.md](../design/layout.md) 为准）。

## 1. 共同原理

### 1.1 核心概念

| 概念 | 说明 |
|------|------|
| 容器 | 负责排布直接子节点 |
| 主轴 / 交叉轴 | 排列方向 vs 垂直方向 |
| 内容区 | 扣除 padding 后可用于排布的区域 |
| 尺寸策略 | 固定 / 包裹内容 / 填满剩余 |
| 间距 | gap、margin 等叠加规则 |
| 对齐 | 主轴分布 + 交叉轴对齐 |

### 1.2 尺寸策略（三态）

| 策略 | 语义 | Web | Qt | Android |
|------|------|-----|-----|---------|
| 固定 | 明确像素或等价固定值 | `width:120px` | `QSizePolicy::Fixed` | 显式 `layout_width` |
| 包裹内容 | 由内容 / intrinsic 决定 | `width: auto` | `sizeHint()` | `wrap_content` |
| 分剩余 | 吃掉主轴剩余空间 | `flex-grow:1` | `Expanding` + stretch | `layout_weight` |

**分剩余**是 Box 类布局核心：先扣固定项与 gap，剩余按 grow / weight 比例分配。

### 1.3 间距叠加

大致顺序：

```
父 padding → [子 margin + border + padding + 内容] → gap → 下一子项 …
```

- **CSS**：子项 `margin` 参与流式占位；容器 `gap` 在 flex 项之间额外插入
- **Qt**：`setContentsMargins()` + `setSpacing()`
- **Android**：`layout_margin` + `divider` / padding
- **klb**（规划）：`rect_in_parent` 含 margin（见 [css.md](../design/css.md)）；容器 `gap` 独立于子 margin

### 1.4 Measure → Layout 两阶段

Android / Qt 典型流程：

| 阶段 | 方向 | 作用 |
|------|------|------|
| **Measure** | 父 → 子 | 父传约束 (min/max)；子报 measured size |
| **Layout** | 父 → 子 | 父已知最终尺寸；按算法分配每个子的 `(x,y,w,h)` |

CSS Flexbox 规范亦隐含类似步骤（base size → 分剩余 → 交叉轴对齐），浏览器实现多为优化单遍。

klb BoxFlow（规划）刻意简化：**单遍、整数 px、O(n)**；不做完整 shrink 迭代与 wrap（见 [layout.md](../design/layout.md)）。

## 2. Web CSS

### 2.1 Flexbox

| 属性 | 作用 |
|------|------|
| `display: flex` | 启用 flex 容器 |
| `flex-direction` | row / column / *-reverse → 定主轴 |
| `flex-wrap` | nowrap / wrap（klb 首版不做 wrap） |
| `gap` | 子项间距 |
| `justify-content` / `align-items` | 主轴 / 交叉轴对齐 |
| `flex-grow` / `flex-shrink` / `flex-basis` | 分剩余 / 收缩 / 基准尺寸 |

**子项 flex 三元组**（简写 `flex:1` ≈ `1 1 0%`）：

| 属性 | 默认 | 含义 |
|------|------|------|
| `flex-grow` | 0 | 主轴剩余分配比例 |
| `flex-shrink` | 1 | 空间不足时收缩比例 |
| `flex-basis` | auto | 参与分配前的基准尺寸 |

**算法概要（简化）**：

1. 确定主轴与容器主轴尺寸 `main_size`
2. 各子项算 flex base size（basis 或 content size）
3. `free = main_size - Σ(base + margin + gap)`
4. `free > 0`：按 `flex-grow` 分配；`free < 0`：按 `flex-shrink` 收缩
5. 交叉轴：`align-items` / `align-self`；`stretch` 拉满未显式指定的交叉轴尺寸

| 对齐属性 | 轴 | 常用值 |
|----------|-----|--------|
| `justify-content` | 主轴 | start / center / end / space-between / space-around |
| `align-items` | 交叉轴 | start / center / end / stretch |
| `align-self` | 单个子项 | 覆盖 `align-items` |

klb 采纳子集见 [layout-map.md](../design/map/layout-map.md)。**不采纳**: wrap、%、`calc()`、完整 shrink、`space-between`（首版）。

### 2.2 Grid

二维 **行 × 列** 分配：

| 概念 | 说明 |
|------|------|
| `grid-template-columns/rows` | 轨道（px / `fr` / auto） |
| `fr` | 分剩余列/行（类似 flex-grow 的二维版） |
| `grid-area` | 子项跨行跨列 |
| `gap` | 行列间距 |

klb 首版 **不做** Grid；P2 规划 `layout:form`（表单两列）仍非完整 Grid。

## 3. Qt QLayout

| 类 | 说明 |
|----|------|
| `QHBoxLayout` / `QVBoxLayout` | 一维 box |
| `QGridLayout` | 二维网格 |
| `QFormLayout` | 标签 + 字段两列（klb P2 `layout:form` 参照） |

| 概念 | 说明 |
|------|------|
| `QSizePolicy` | Fixed / Minimum / Expanding / Preferred |
| `sizeHint()` | 控件建议尺寸（klb `suggestw` **不对标**为布局输入） |
| `addStretch()` | 空白弹簧，等价 grow-only spacer |

**QSizePolicy 与 Flex 近似**：

| Qt | Flex |
|----|------|
| `setSpacing(n)` | `gap` |
| `addStretch()` | 空 flex item + `flex-grow:1` |
| `setAlignment()` | `align-items` / `justify-content` |
| `Expanding` | `flex-grow:1` |

Layout 先 query 各 widget `sizeHint()`，再分配 rect；与 Android measure/layout 同构。

## 4. Android ViewGroup

| 布局 | 说明 |
|------|------|
| `LinearLayout` | `orientation` + `layout_weight` 分剩余 |
| `ConstraintLayout` | 约束图联立求解（klb 不做） |

**LinearLayout 要点**：

| 属性 | 含义 |
|------|------|
| `orientation` | horizontal / vertical → 主轴 |
| `layout_width/height` | match_parent / wrap_content / 具体 dp |
| `layout_weight` | 主轴分剩余（常配合 `0dp` + weight） |
| `layout_gravity` | 子项在父内对齐 |

Measure：`0dp + weight` 子项在固定项 measure 后按 `remaining × (weight/Σweight)` 分配。

**ConstraintLayout**：边对边、比例、链式、bias 等约束，由 solver 求 `(x,y,w,h)`；表达力强，ARM 单遍场景通常排除。

**两阶段 API**：`onMeasure` → `onLayout`；与 Qt layout、Flex 隐含流程同构。

## 5. 嵌入式 LVGL

Flex 子集，面向资源受限设备：

| 特性 | 说明 |
|------|------|
| `flex_flow` | row/column（可选 wrap） |
| `flex_grow` | 分剩余 |
| `flex_align` | 主轴 / 交叉轴对齐 |
| 数值 | 整数 px，无 float |
| 算法 | 单遍为主，无完整 CSS shrink 迭代 |

与 klb BoxFlow 目标接近：**ARM 友好、整数、单遍、1D 为主**。

## 6. 平台对照表

| 维度 | Web Flex | Qt Box | Android Linear | LVGL Flex | klb BoxFlow |
|------|----------|--------|----------------|-----------|-------------|
| 维度 | 1D（可 wrap） | 1D | 1D | 1D（可 wrap） | 1D only |
| 分剩余 | flex-grow | stretch / Expanding | layout_weight | flex_grow | `flex:1` |
| 收缩 | flex-shrink | policy + min | 有限 | 简化 | 无 |
| 间距 | gap + margin | spacing + margin | margin + divider | gap | gap + margin |
| 对齐 | justify + align | alignment | gravity | flex_align | justify + align |
| 内容尺寸 | auto | sizeHint() | wrap_content | 内容驱动 | type 默认（不用 suggestw） |
| 触发 | reflow | updateGeometry | requestLayout | 属性变更 | dirty 触发 |

详键名对照 [../design/map/layout-map.md](../design/map/layout-map.md).

## 7. 与 klb 现状（边界说明）

| 层级 | 说明 |
|------|------|
| 本文 | 【外部】各平台原理；非 klb 规格 |
| [layout.md](../design/layout.md) | 【自有】BoxFlow 键名与硬约束 |
| C 实现 | `klbui_layout.c` 未落地；现有 `css_map` 的 `layout` 命令递归触发 `KLBUI_layout`（控件**自布局**，非 BoxFlow） |
| parser `AutoRect` | `pos` 中 `-1` = 单子项相对父 fill；**非**多子项流式排布 |

## 8. 权威出处（查阅用）

| 平台 | 参考 |
|------|------|
| CSS Flex | [MDN flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout) |
| CSS Grid | [MDN grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout) |
| Qt | [QLayout](https://doc.qt.io/qt-6/qlayout.html) |
| Android | [Layouts](https://developer.android.com/develop/ui/views/layout/declaring-layout) |
| LVGL | [Flex](https://docs.lvgl.io/master/layouts/flex.html) |
