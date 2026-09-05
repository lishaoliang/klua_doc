# 自动布局 BoxFlow（自有设计）

> `klua_doc/klb/klbgui/design/layout.md` — **【自有设计】** klbgui C 自动布局（ARM 优先）  
> 代码（规划）: [klb/inc/klbgui/klbui_layout.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_layout.h), [klb/src_c/klbgui/klbui_layout.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/klbui_layout.c)  
> 状态: **桩文档**；实现未落地

分层 [layers.md](layers.md). 窗口坐标 [wnd.md](wnd.md). 盒模型 [css.md](css.md).  
外部原理 [../ref/layout-principles.md](../ref/layout-principles.md). 概念对照 [map/layout-map.md](map/layout-map.md).  
C API（桩） [../api/klbui_layout.md](../api/klbui_layout.md).

## 结论

klbgui 自动布局首版为 **BoxFlow**：容器（`kview`/`kdialog`）对**直接子节点**单轴流式排布；**C 为真源**；Lua `parser`/`csser` 仅声明与薄封装。主交付 **ARM embed**；单遍整数 px；**禁止**用 `suggestw`/`suggesth` 作布局输入。

## 1. 定位与硬约束

| 项 | 要求 |
|----|------|
| 主交付 | ARM / embed；PC/sim 同代码路径 |
| 算法 | 单遍、O(n)、整数 px；dirty 触发；禁止每帧 relayout |
| 范围 | 仅直接子节点；控件内部 relayout（list/calendar 等）不变 |
| 落地 | 结果写入 `move`/`resize` → `rect_in_parent` |

（细则待实现后补全。）

## 2. BoxFlow 模型（规划）

### 2.1 容器模式

| `layout` | 行为 |
|----------|------|
| `none` | 默认；保持 parse/`move` 几何 |
| `flow` | 单轴流式排直接子节点 |

### 2.2 容器键（父）

| 键 | 值 | 默认 |
|----|-----|------|
| `layout` | `none` / `flow` | `none` |
| `flex-direction` | `row` / `column` | `column` |
| `gap` | int px | `0` |
| `align-items` | `start` / `center` / `end` / `stretch` | `stretch` |
| `justify-content` | `start` / `center` / `end` | `start` |

### 2.3 子项键

| 键 | 值 | 说明 |
|----|-----|------|
| `width` / `height` | int px | 固定尺寸 |
| `flex` | `0` / `1` | `1` = 占主轴剩余 |
| `layout-align-self` | 同 `align-items` | 可选 |

首版不做: Grid、wrap、`%`、`calc()`、绝对定位布局。

## 3. 尺寸策略

| 优先 | 来源 |
|------|------|
| 1 | 子项显式 `width`/`height` |
| 2 | `flex:1` |
| 3 | type 默认尺寸表 |
| 4 | `min-width`/`max-width` 等（`klbuicss_box_t`） |
| 不用 | `suggestw`/`suggesth` |

与 parser: `pos` 宽/高 **-1** ≡ `fill` / `flex:1`（见 [klb/bin/klbcore/klbui/parser.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/klbui/parser.lua) `AutoRect`）。

## 4. C 架构（规划）

```
klbui_layout.c          BoxFlow 引擎
kview/kdialog           KLBUI_layout 入口
klbui_layout_css.c      layout/gap/flex bind（或容器 func_map）
```

| 时机 | 动作 |
|------|------|
| parse 完成 | 根→叶 relayout |
| 容器 resize | mark dirty |
| `layout` 命令 | 递归 `KLBUI_layout` + flow |

## 5. C / Lua 接口（规划）

| 层 | 接口 |
|----|------|
| C | `klbui_layout_flow(container)`（待定） |
| kgui | `kgui.layout(path)`（待定） |
| klbui | `klbui.layout(path)`；parser 容器 `layout={...}`（待定） |

## 6. 相关文档

| 文档 | 内容 |
|------|------|
| [ref/layout-principles.md](../ref/layout-principles.md) | 【外部】Web/Qt 等原理 |
| [map/layout-map.md](map/layout-map.md) | 外部概念 ↔ BoxFlow |
| [lua/klbcore/css/klbui_css.md](../../../lua/klbcore/css/klbui_css.md) | Lua CSS 语法（layout 键待补） |
| [lua/klbcore/css/klbui_kview.md](../../../lua/klbcore/css/klbui_kview.md) | 容器 type CSS（待补） |

## 7. 实施状态

| 阶段 | 内容 | 状态 |
|------|------|------|
| P0 | C BoxFlow + kview/kdialog + CSS bind | 未开始 |
| P1 | parser/csser + `klbui.layout()` | 未开始 |
| P2 | `layout:form` | 未开始 |
