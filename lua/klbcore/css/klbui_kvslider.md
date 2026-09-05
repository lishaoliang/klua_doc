## 垂直滑块

* 注册类型: kvslider

### 1. 概述

```c
// C实现文件: ./klb/src_packages/klbwui/embed_widgets/klbui_vslider.c
// 窗口实现: ./klb/src_packages/klbwui/embed_wnd/klbwnd_vslider.c
// 控件通用 CSS 方法: ./klb/src_c/klbgui/klbui_css_std_function.c
```

* 所有已注册控件均通过 `klb_gui_css_map_append_std_function` 挂载 **控件通用属性/方法**（见下文「3. 控件通用属性/方法」）
* 样式键支持 **normal** / **focus** / **disabled**; 无 `:checked`
* 默认窗口样式: 可聚焦 (无特殊 style 位)
* 轨道走 `background-color` / `foreground-color` / `foreground-width`; 滑块走 `thumb.*`
* 数值走 `value` / `min` / `max` / `step`

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

#### 4.4 文本颜色

* 属性: color
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['color'] = 0xFFDCDCDC,
    ['color:focus'] = 0xFFFFFFFF,
    ['color:disabled'] = 0xFF808080,
}
```

* 获取
```lua
    0xFFDCDCDC
```

#### 4.5 背景色

* 属性: background-color
* 支持状态: normal, focus, disabled
* 轨道底色

* 设置
```lua
{
    ['background-color'] = 0xFF404040,
    ['background-color:focus'] = 0xFF505050,
}
```

* 获取
```lua
    0xFF404040
```

#### 4.6 前景色

* 属性: foreground-color
* 支持状态: normal, focus, disabled
* 已滑过轨道色

* 设置
```lua
{
    ['foreground-color'] = 0xFF0080FF,
    ['foreground-color:focus'] = 0xFF2090FF,
}
```

* 获取
```lua
    0xFF0080FF
```

#### 4.7 前景宽度

* 属性: foreground-width
* 无状态
* 轨道前景条宽度 (像素)

* 设置
```lua
{
    ['foreground-width'] = 4,
}
```

* 获取
```lua
    4
```

#### 4.8 滑块

##### 4.8.1 滑块颜色

* 属性: thumb.color
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['thumb.color'] = 0xFFFFFFFF,
    ['thumb.color:focus'] = 0xFF0080FF,
}
```

##### 4.8.2 滑块图片

* 属性: thumb.image
* 支持状态: normal, focus, disabled

* 设置
```lua
{
    ['thumb.image'] = 'thumb.bmp',
    ['thumb.image:focus'] = 'thumb_f.bmp',
}
```

##### 4.8.3 滑块尺寸

* 属性: thumb.width / thumb.height
* 无状态

* 设置
```lua
{
    ['thumb.width'] = 16,
    ['thumb.height'] = 16,
}
```

### 5. 自定义属性

#### 5.1 值

* 属性: value
* 无状态

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

#### 5.4 步进

* 属性: step
* 无状态

* 设置
```lua
{
    ['step'] = 1,
}
```

* 获取
```lua
    1
```

### 6. CSS总表

* 先公共后私有; ① 见 `klbui_css_std_function.c`; ②③ 见 `klbui_vslider.c` `init_func_map`
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
| CSS公共 | `background-color` + 伪类           | get/set | 轨道底色                        |
| CSS公共 | `foreground-color` + 伪类           | get/set | 已滑过轨道色                      |
| CSS私有 | `foreground-width`                | get/set | 前景条宽度                       |
| CSS私有 | `thumb.color` + 伪类                | get/set | 滑块颜色                        |
| CSS私有 | `thumb.image` + 伪类                | get/set | 滑块图片                        |
| CSS私有 | `thumb.width` / `thumb.height`    | get/set | 滑块尺寸                        |
| 自定义   | `value`                           | get/set | 当前值                         |
| 自定义   | `min` / `max`                     | get/set | 范围                          |
| 自定义   | `step`                            | get/set | 步进                          |

#### 6.1 参考
```lua
{
    ['type'] = 'kvslider',
    ['pos'] = {280, 40, 32, 160},
    ['name'] = 'vslider1',
    ['value'] = 50,
    ['min'] = 0,
    ['max'] = 100,
    ['step'] = 1,
    ['foreground-color'] = 0xFF0080FF,
    ['foreground-width'] = 4,
    ['thumb.image'] = 'thumb.bmp',
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
            ['type'] = 'kvslider',
            ['pos'] = {280, 40, 32, 160},
            ['name'] = 'vslider1',
            ['min'] = 0,
            ['max'] = 100,
            ['step'] = 1,
            ['value'] = 50,
            ['thumb.image'] = 'thumb.bmp',
        },
    },
}

klbui.parse(dialog, {}, {})
```
