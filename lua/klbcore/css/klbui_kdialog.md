## 对话框

* 注册类型: kdialog

### 1. 概述

```c
// C实现文件: ./klb/src_packages/klbwui/embed_widgets/klbui_dialog.c
// 窗口实现: ./klb/src_packages/klbwui/embed_wnd/klbwnd_dialog.c
// 控件通用 CSS 方法: ./klb/src_c/klbgui/klbui_css_std_function.c
```

* 所有已注册控件均通过 `klb_gui_css_map_append_std_function` 挂载 **控件通用属性/方法**（见下文「3. 控件通用属性/方法」）
* 容器与 `titlebar.*` 视觉键仅 **normal**; 无 `:focus` / `:disabled` / `:checked`
* 默认窗口样式: `style-top` + `style-peek-event`
* 标题文本走 `color` / `text-align` / `font-size`; 背景图走 `background-image` + `background-image-mode` + `background-image-color-key`
* 标题栏样式见 `titlebar.background-color` / `titlebar.background-image` 等
* `title` 为窗口标题; `value` 为业务字符串

### 2. 支持状态

* 普通: normal


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
* 支持状态: normal

* 设置
```lua
{
    ['color'] = 0xFFDCDCDC,
}
```

* 获取
```lua
    0xFFA0A0A0
```

##### 4.4.2 文本对齐

* 属性: text-align
* 支持状态: normal

* 设置
```lua
{
    ['text-align'] = 'left',
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
* 支持状态: normal

* 设置
```lua
{
    ['font-size'] = 24,
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
* 支持状态: normal

* 设置
```lua
{
    ['background-color'] = 0xFF303030,
}
```

* 获取
```lua
    0xFFA0A0A0
```

##### 4.6.2 背景图片

* 属性: background-image
* 支持状态: normal

* 设置
```lua
{
    ['background-image'] = 'dialog.bmp',
}
```

* 获取
```lua
    'dialog.bmp'
```

#### 4.7 边框

##### 4.7.1 边框宽度

* 属性: border-width
* 支持状态: normal

* 设置
```lua
{
    ['border-width'] = 1,
}
```

* 获取
```lua
    1
```

##### 4.7.2 边框颜色

* 属性: border-color
* 支持状态: normal

* 设置
```lua
{
    ['border-color'] = 0xFF505050,
}
```

* 获取
```lua
    0xFF505050
```

#### 4.8 背景图画法

* 属性: background-image-mode
* 支持状态: normal
* 取值: `'default'` / `'resize'` / `'scale9'`

* 设置
```lua
{
    ['background-image'] = 'dialog.bmp',
    ['background-image-mode'] = 'scale9',
}
```

#### 4.9 背景图关键色

* 属性: background-image-color-key
* 支持状态: normal

* 设置
```lua
{
    ['background-image-color-key'] = true,
}
```

#### 4.10 标题栏

* 属性: titlebar.background-color / titlebar.background-image / titlebar.background-image-mode / titlebar.background-image-color-key
* 支持状态: normal
* 标题栏区域样式 (与容器键独立)

* 设置
```lua
{
    ['titlebar.background-color'] = 0xFF404040,
    ['titlebar.background-image'] = 'dialog_title.bmp',
    ['titlebar.background-image-mode'] = 'scale9',
    ['titlebar.background-image-color-key'] = true,
}
```

### 5. 自定义属性

#### 5.1 标题

* 属性: title
* 无状态
* 窗口标题

* 设置
```lua
{
    ['title'] = '对话框1',
}
```

* 获取
```lua
    '对话框1'
```

#### 5.2 值

* 属性: value
* 无状态
* 业务字符串; 与 **title** 独立

* 设置
```lua
{
    ['value'] = 'dlg1',
}
```

* 获取
```lua
    'dlg1'
```

### 6. CSS总表

* 先公共后私有; ① 见 `klbui_css_std_function.c`; ②③ 见 `klbui_dialog.c` `init_func_map`
* 容器与 `titlebar.*` 仅 **normal**; 无伪类后缀

| 分组    | 键                                 | 读写      | 说明                          |
| ----- | --------------------------------- | ------- | --------------------------- |
| 控件通用  | `wndpos-canvas` / `wndpos-parent` | get     | 画布 / 父窗口坐标 `{x,y,w,h}`      |
| 控件通用  | `move`                            | set     | `{x,y}`                     |
| 控件通用  | `resize`                          | set     | `{w,h}`                     |
| 控件通用  | `suggestw` / `suggesth`           | get     | 建议宽 / 高                     |
| 控件通用  | `layout`                          | set     | 触发 `KLBUI_layout`           |
| 控件通用  | `refresh`                         | set     | 重绘                          |
| 控件通用  | `style-peek-event`                | get/set | 冒泡读消息; kdialog 默认 `true`  |
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
| CSS公共 | `margin` / `margin-*`             | get/set | 四边及拆分                       |
| CSS公共 | `padding` / `padding-*`           | get/set | 四边及拆分                       |
| CSS公共 | `color`                           | get/set | 标题文本色                       |
| CSS公共 | `text-align`                      | get/set | 标题对齐                        |
| CSS公共 | `font-size`                       | get/set | 标题字号                        |
| CSS公共 | `background-color`                | get/set | 客户区背景                       |
| CSS公共 | `background-image`                | get/set | 客户区背景图                      |
| CSS公共 | `border-width`                    | get/set | 客户区边框                       |
| CSS公共 | `border-color`                    | get/set | 客户区边框色                      |
| CSS私有 | `background-image-mode`           | get/set | 客户区图画法                      |
| CSS私有 | `background-image-color-key`      | get/set | 客户区关键色透明                   |
| CSS私有 | `titlebar.background-color`       | get/set | 标题栏背景色                      |
| CSS私有 | `titlebar.background-image`       | get/set | 标题栏背景图                      |
| CSS私有 | `titlebar.background-image-mode`  | get/set | 标题栏图画法                      |
| CSS私有 | `titlebar.background-image-color-key` | get/set | 标题栏关键色透明               |
| 自定义   | `title`                           | get/set | 窗口标题                        |
| 自定义   | `value`                           | get/set | 业务字符串; 与 title 独立           |

#### 6.1 参考
```lua
{
    ['type'] = 'kdialog',
    ['pos'] = {20, 20, 280, 200},
    ['name'] = 'dlg1',
    ['title'] = '对话框1',
    ['value'] = 'dlg1',
    ['color'] = 0xFFDCDCDC,
    ['background-color'] = 0xFF303030,
    ['titlebar.background-color'] = 0xFF404040,
    ['titlebar.background-image'] = 'dialog_title.bmp',
    ['titlebar.background-image-mode'] = 'scale9',
}
```

### 7. 使用示例

```lua
local klbui = require("klbcore.klbui")

local dialog = {
    ['type'] = 'kdialog',
    ['pos'] = {20, 20, 280, 200},
    ['name'] = 'dlg1',
    ['title'] = '对话框1',
    ['color'] = 0xFFDCDCDC,
    ['background-color'] = 0xFF303030,
    ['titlebar.background-color'] = 0xFF404040,
    ['child'] = {
        {
            ['type'] = 'kstatic',
            ['pos'] = {10, 40, 200, 32},
            ['name'] = 'sta1',
            ['title'] = '内容',
        },
    },
}

klbui.parse(dialog, {}, {})
```
