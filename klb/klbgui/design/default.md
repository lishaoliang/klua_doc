# 默认主题与 CSS 参考值

> `klua_doc/klb/klbgui/design/default.md` — 头文件: [klb/inc/klbgui/klbui_default.h](https://gitee.com/klua/klb/blob/trunk/inc/klbgui/klbui_default.h); 实现: [klb/src_c/klbgui/extensions/klbuiex_default.c](https://gitee.com/klua/klb/blob/trunk/src_c/klbgui/extensions/klbuiex_default.c)

盒模型 [css.md](css.md). Lua 镜像 [lua/klbcore/css/klbui_default_css.md](../../../lua/klbcore/css/klbui_default_css.md).

## 结论

`klbui_default_t` 提供控件 **css_init 复制源**：margin/padding + 五态参考属性（normal/focus/disable/check/input）。扩展 **`KLBUIEX-default`** 持有一份实例；默认配色 **Visual Studio 深色系**。

## 结构体 (`klbui_default.h`)

| 字段 | CSS 伪类 | 用途 |
|------|----------|------|
| `margin` / `padding` | （无） | 盒模型默认 |
| `normal` | 无后缀 | 常规 |
| `focus` | `:focus` | 聚焦 |
| `disable` | `:disabled` | 不使能 |
| `check` | `:checked` | 选中参考 |
| `input` | `:input` | 输入态 |

各态为 `klbuicssex_attributes_t`（text/font/background/border）。

## 默认值（C 初始化）

| 项 | 值 |
|----|-----|
| margin | 0,0,0,0 |
| padding | 1,1,1,1 |
| font-size | 24 |
| text-align | left |
| normal 文本 | `#DCDCDC`；边框 `#505050`；背景 `#1F1F1F` |
| focus 文本 | `#DCDC0A`；边框 `#DC5050` |
| disable 文本 | `#B4B4B4` |
| check 文本 | `#0AD2D2`；边框 `#B450B4` |
| input 文本 | `#B4B4B4` |

## API

| API | 说明 |
|-----|------|
| `klb_gui_get_std_default(p_gui)` | 只读默认结构体指针 |
| `klb_gui_default_css_set` / `get` | map 形式读写全局默认 CSS |

控件创建前可通过上述 API 或 Lua `klbui.default_css` 修改；各 k* 的 `css_init` 通常从 `get_std_default` 复制。

## 与 shwnd / globalcss

| 机制 | 粒度 |
|------|------|
| default | 全局主题参考 |
| globalcss | 跨控件 CSS map |
| shwnd | 按 path 整窗模板 → [shwnd.md](shwnd.md) |

## 关联

| 主题 | 入口 |
|------|------|
| CSS 结构体 | [css.md](css.md), [../api/klbui_css.md](../api/klbui_css.md) |
| 扩展 | [extension.md](extension.md) § default |
