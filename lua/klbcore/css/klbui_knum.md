## 数字框

* 注册类型: knum

### 1. 概述

```c
// C实现文件: ./klb/src_packages/klbwui/embed_widgets/klbui_num.c
// 窗口实现: ./klb/src_packages/klbwui/embed_wnd/klbwnd_num.c
// 控件通用 CSS 方法: ./klb/src_c/klbgui/klbui_css_std_function.c
```

* 所有已注册控件均通过 `klb_gui_css_map_append_std_function` 挂载 **控件通用属性/方法**（见下文「3. 控件通用属性/方法」）
* 样式键支持 **normal** / **focus** / **disabled**; 无 `:checked`
* 默认窗口样式: 可聚焦 (无特殊 style 位)
* 文本走 `color` / `text-align` / `font-size`; 背景图走 `background-image` + `background-image-mode` + `background-image-color-key`
* 绑 `/klbui/decimal-menu`; 数值走 `value` / `min` / `max` / `ranges`

### 2. 支持状态

* 普通: normal
* 聚焦: focus
* 不使能: disabled（键后缀 `:disabled`）


### 3. 控件通用属性/方法

* 来源: `klb_gui_css_map_append_std_function`（`klbui_css_std_function.c`）
* 所有已注册控件均支持; 键名同时提供 `-` 与 `_` 两种写法（如 `wndpos-canvas` / `wndpos_canvas`）

#### 3.1 窗口坐标

##### 3.1.1 画布坐标

* 属性: wndpos-canvas
* 无状态
* 仅获取

* 获取
```lua
    { x = 10, y = 20, w = 120, h = 32 }     -- 相对画布
```

##### 3.1.2 父窗口坐标

* 属性: wndpos-parent
* 无状态
* 仅获取

* 获取
```lua
    { x = 10, y = 20, w = 120, h = 32 }     -- 相对父窗口
```

#### 3.2 窗口布局

##### 3.2.1 移动

* 方法: move
* 无状态
* 仅设置

* 设置
```lua
{
    ['move'] = { x = 10, y = 20 },
}
```

##### 3.2.2 重设大小

* 方法: resize
* 无状态
* 仅设置

* 设置
```lua
{
    ['resize'] = { w = 120, h = 32 },
}
```

##### 3.2.3 建议宽度

* 属性: suggestw
* 无状态
* 仅获取

* 获取
```lua
    120
```

##### 3.2.4 建议高度

* 属性: suggesth
* 无状态
* 仅获取

* 获取
```lua
    32
```

##### 3.2.5 重新布局

* 方法: layout
* 无状态
* 仅设置; 触发自身及子窗口 `KLBUI_layout`

* 设置
```lua
{
    ['layout'] = true,
}
```

##### 3.2.6 刷新

* 方法: refresh
* 无状态
* 仅设置; 触发窗口重绘

* 设置
```lua
{
    ['refresh'] = true,
}
```

#### 3.3 窗口样式

* 属性均为 bool; 无状态

##### 3.3.1 冒泡读取消息

* 属性: style-peek-event

* 设置
```lua
{
    ['style-peek-event'] = true,
}
```

* 获取
```lua
    true
```

##### 3.3.2 无聚焦

* 属性: style-nofocus

* 设置
```lua
{
    ['style-nofocus'] = true,
}
```

* 获取
```lua
    false
```

##### 3.3.3 无 on_command

* 属性: style-nocommand

* 设置
```lua
{
    ['style-nocommand'] = true,
}
```

* 获取
```lua
    false
```

##### 3.3.4 聚焦不重绘

* 属性: style-focus-without-redraw

* 设置
```lua
{
    ['style-focus-without-redraw'] = true,
}
```

* 获取
```lua
    false
```

##### 3.3.5 继续寻找焦点

* 属性: style-focus-continue

* 设置
```lua
{
    ['style-focus-continue'] = true,
}
```

* 获取
```lua
    false
```

##### 3.3.6 聚焦延时消息

* 属性: style-focus-delay

* 设置
```lua
{
    ['style-focus-delay'] = true,
}
```

* 获取
```lua
    false
```

#### 3.4 窗口状态

* 属性均为 bool; 无状态

##### 3.4.1 显示

* 属性: show

* 设置
```lua
{
    ['show'] = true,
}
```

* 获取
```lua
    true
```

##### 3.4.2 隐藏

* 属性: hide

* 设置
```lua
{
    ['hide'] = true,
}
```

* 获取
```lua
    false
```

##### 3.4.3 输入

* 属性: input

* 设置
```lua
{
    ['input'] = true,
}
```

* 获取
```lua
    false
```

##### 3.4.4 选中

* 属性: check

* 设置
```lua
{
    ['check'] = true,
}
```

* 获取
```lua
    false
```

##### 3.4.5 不使能

* 属性: disable

* 设置
```lua
{
    ['disable'] = true,
}
```

* 获取
```lua
    false
```

##### 3.4.6 使能

* 属性: enable

* 设置
```lua
{
    ['enable'] = true,
}
```

* 获取
```lua
    true
```

##### 3.4.7 最顶层

* 属性: topmost
* 仅获取; 是否为所有最顶层窗口中视觉最上层的那个

* 获取
```lua
    false
```


### 4. CSS属性

#### 4.1 显隐

* 属性: visibility
* 无状态

* 显示
```lua
{
    ['visibility'] = 'visible',
    ['visibility'] = true,
}
```

* 隐藏
```lua
{
    ['visibility'] = 'hidden',
    ['visibility'] = false,
}
```

* 获取
```lua
    'visible'   -- 显示
    'hidden'    -- 隐藏
```

#### 4.2 外边距

* 属性: margin
* 无状态

* 设置
```lua
{
    ['margin'] = {0, 0, 0, 0},
    ['margin-top'] = 0,
    ['margin-right'] = 0,
    ['margin-bottom'] = 0,
    ['margin-left'] = 0,
}
```

* 获取
```lua
    0
```

#### 4.3 内边距

* 属性: padding
* 无状态

* 设置
```lua
{
    ['padding'] = {1, 1, 1, 1},
    ['padding-top'] = 1,
    ['padding-right'] = 1,
    ['padding-bottom'] = 1,
    ['padding-left'] = 1,
}
```

* 获取
```lua
    0
```

#### 4.4 文本

##### 4.4.1 文本颜色

* 属性: color
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['color'] = {255,220,220,220},
    ['color:focus'] = 0xFFA0A0A0,
    ['color:disabled'] = '0xFFA0A0A0',
}
```

* 获取
```lua
    0xFFA0A0A0
```

##### 4.4.2 文本对齐

* 属性: text-align
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['text-align'] = 'left',
    ['text-align:focus'] = 'center',
    ['text-align:disabled'] = 'right',
}
```

* 获取
```lua
    'center'        -- 中心对齐
    'left'          -- 左对齐
    'right'         -- 右对齐
```

#### 4.5 字体

##### 4.5.1 字体大小

* 属性: font-size
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['font-size'] = 24,
    ['font-size:focus'] = 28,
    ['font-size:disabled'] = 20,
}
```

* 获取
```lua
    24
```

#### 4.6 背景

* 有背景图时按图绘制, 不再画 `border-*`; 无图时填充 `background-color` 并画边框

##### 4.6.1 背景色

* 属性: background-color
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['background-color'] = {255,220,80,20},
    ['background-color:focus'] = 0xFFA0A0A0,
    ['background-color:disabled'] = '0xFFA0A0A0',
}
```

* 获取
```lua
    0xFFA0A0A0
```

##### 4.6.2 背景图片

* 属性: background-image
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['background-image'] = 'combo.bmp',
    ['background-image:focus'] = 'combo_f.bmp',
    ['background-image:disabled'] = 'combo_d.bmp',
}
```

* 获取
```lua
    'combo.bmp'
```

#### 4.7 边框

##### 4.7.1 边框宽度

* 属性: border-width
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['border-width'] = 1,
    ['border-width'] = {1, 1, 1, 1},
    ['border-width:focus'] = 1,
    ['border-width:disabled'] = 1,
}
```

* 获取
```lua
    1
```

##### 4.7.2 边框颜色

* 属性: border-color
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['border-color'] = {255,220,30,30},
    ['border-color'] = 0xFFA0A0A0,
    ['border-color:focus'] = {255,220,30,30},
    ['border-color:disabled'] = {255,220,30,30},
}
```

* 获取
```lua
    0xFFA0A0A0
```

#### 4.8 背景图画法

* 属性: background-image-mode
* 支持状态: normal, focus, disabled
* 取值: `'default'` (原图拷贝, 超出裁剪), `'resize'` (缩放铺满), `'scale9'` (九宫格)
* 与 `background-image` 独立; 可组合

* 设置
```lua
{
    ['background-image'] = 'combo.bmp',
    ['background-image-mode'] = 'scale9',
    ['background-image-mode:focus'] = 'scale9',
    ['background-image-mode:disabled'] = 'default',
}
```

* 获取
```lua
    'default'
    'resize'
    'scale9'
```

#### 4.9 背景图关键色

* 属性: background-image-color-key
* 支持状态: normal, focus, disabled
* 布尔; `true` 时按关键色透明绘制, 可与 `scale9` 同时使用

* 设置
```lua
{
    ['background-image'] = 'combo.bmp',
    ['background-image-mode'] = 'scale9',
    ['background-image-color-key'] = true,
    ['background-image-color-key:focus'] = true,
    ['background-image-color-key:disabled'] = false,
}
```

* 获取
```lua
    false
```

### 5. 自定义属性

#### 5.1 值

* 属性: value
* 无状态
* 当前整数

* 设置
```lua
{
    ['value'] = 50,
}
```

* 获取
```lua
    50
```

#### 5.2 最小值

* 属性: min
* 无状态

* 设置
```lua
{
    ['min'] = 0,
}
```

* 获取
```lua
    0
```

#### 5.3 最大值

* 属性: max
* 无状态

* 设置
```lua
{
    ['max'] = 100,
}
```

* 获取
```lua
    100
```

#### 5.4 范围

* 属性: ranges
* 无状态
* 读写 map: `min` / `max`

* 设置
```lua
{
    ['ranges'] = { ['min'] = 0, ['max'] = 100 },
}
```

* 获取
```lua
    { min = 0, max = 100 }
```

### 6. CSS总表

* 先公共后私有; ① 见 `klbui_css_std_function.c`; ②③ 见 `klbui_num.c` `init_func_map`
* 视觉键支持三态: 无后缀 / `:focus` / `:disabled`

| 分组    | 键                                 | 读写      | 说明                          |
| ----- | --------------------------------- | ------- | --------------------------- |
| 控件通用  | `wndpos-canvas` / `wndpos-parent` | get     | 画布 / 父窗口坐标 `{x,y,w,h}`      |
| 控件通用  | `move`                            | set     | `{x,y}`                     |
| 控件通用  | `resize`                          | set     | `{w,h}`                     |
| 控件通用  | `suggestw` / `suggesth`           | get     | 建议宽 / 高                     |
| 控件通用  | `layout`                          | set     | 触发 `KLBUI_layout`           |
| 控件通用  | `refresh`                         | set     | 重绘                          |
| 控件通用  | `style-peek-event`                | get/set | 冒泡读消息                       |
| 控件通用  | `style-nofocus`                   | get/set | 无聚焦                         |
| 控件通用  | `style-nocommand`                 | get/set | 无 `on_command`              |
| 控件通用  | `style-focus-without-redraw`      | get/set | 聚焦不重绘                       |
| 控件通用  | `style-focus-continue`            | get/set | 继续寻找焦点                      |
| 控件通用  | `style-focus-delay`               | get/set | 聚焦延时消息                      |
| 控件通用  | `show` / `hide`                   | get/set | 显示 / 隐藏                     |
| 控件通用  | `input` / `check`                 | get/set | 输入 / 选中                     |
| 控件通用  | `disable` / `enable`              | get/set | 不使能 / 使能                    |
| 控件通用  | `topmost`                         | get     | 是否视觉最顶层                     |
| 控件通用  | `tip`                             | get/set | 提示文本                        |
| 控件通用  | `visibility`                      | get/set | `visible` / `hidden` 或 bool |
| CSS公共 | `margin` / `margin-*`             | get/set | 四边及拆分; 无伪类                  |
| CSS公共 | `padding` / `padding-*`           | get/set | 四边及拆分; 无伪类                  |
| CSS公共 | `color` + 伪类                      | get/set | 文本色                         |
| CSS公共 | `text-align` + 伪类                 | get/set | `'left'` / `'center'` / `'right'` |
| CSS公共 | `font-size` + 伪类                  | get/set | 字号                          |
| CSS公共 | `background-color` + 伪类           | get/set | 无图时填充                       |
| CSS公共 | `background-image` + 伪类           | get/set | 背景图路径; 有图不画 border          |
| CSS公共 | `border-width` + 伪类               | get/set | 无图时生效                       |
| CSS公共 | `border-color` + 伪类               | get/set | 无图时生效                       |
| CSS私有 | `background-image-mode` + 伪类      | get/set | `'default'` / `'resize'` / `'scale9'` |
| CSS私有 | `background-image-color-key` + 伪类 | get/set | 关键色透明                       |
| 自定义   | `value`                           | get/set | 当前整数                        |
| 自定义   | `min` / `max`                     | get/set | 范围边界                        |
| 自定义   | `ranges`                          | get/set | `{min,max}`                 |

#### 6.1 参考
```lua
{
    ['type'] = 'knum',
    ['pos'] = {10, 120, 100, 32},
    ['name'] = 'num1',
    ['value'] = 50,
    ['min'] = 0,
    ['max'] = 100,
    ['background-image'] = 'edit.bmp',
    ['background-image-mode'] = 'scale9',
}
```

### 7. 使用示例

```lua
local klbui = require("klbcore.klbui")

local dialog = {
    ['type'] = 'kview',
    ['pos'] = {0, 0, 320, 240},
    ['name'] = 'view1',
    ['child'] = {
        {
            ['type'] = 'knum',
            ['pos'] = {10, 120, 100, 32},
            ['name'] = 'num1',
            ['ranges'] = { ['min'] = 0, ['max'] = 100 },
            ['value'] = 50,
            ['background-image'] = 'edit.bmp',
            ['background-image-mode'] = 'scale9',
        },
    },
}

klbui.parse(dialog, {}, {})
```
