## 定时器

* 注册类型: kticker

### 1. 概述

```c
// C实现文件: ./klb/src_packages/klbwui/embed_widgets/klbui_ticker.c
// 窗口实现: ./klb/src_packages/klbwui/embed_wnd/klbwnd_ticker.c
// 控件通用 CSS 方法: ./klb/src_c/klbgui/klbui_css_std_function.c
```

* 所有已注册控件均通过 `klb_gui_css_map_append_std_function` 挂载 **控件通用属性/方法**（见下文「3. 控件通用属性/方法」）
* kticker 仅支持 **normal** 状态的样式属性; 无 `:focus` / `:disabled` / `:checked`
* 默认窗口样式: `style-nofocus` + `style-ticker`; 默认隐藏 (`visibility` = hidden)
* 无文本 / 背景 / 边框绑定; 仅 `margin` / `padding` 盒模型
* 定时器键: `ticker` / `ticker-default` / `ticker-interval`; 索引走 `index`

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
    'visible'
    'hidden'
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
    ['padding'] = {0, 0, 0, 0},
    ['padding-top'] = 0,
    ['padding-right'] = 0,
    ['padding-bottom'] = 0,
    ['padding-left'] = 0,
}
```

* 获取
```lua
    0
```

#### 4.4 定时器

* 属性: ticker
* 无状态
* 当前定时器开关

* 设置
```lua
{
    ['ticker'] = true,
}
```

* 获取
```lua
    true
```

* 属性: ticker-default
* 无状态
* 默认定时器开关 (创建时默认 `true`)

* 设置
```lua
{
    ['ticker-default'] = true,
}
```

* 属性: ticker-interval
* 无状态
* 定时间隔, 单位毫秒

* 设置
```lua
{
    ['ticker-interval'] = 100,
}
```

* 获取
```lua
    100
```

### 5. 自定义属性

#### 5.1 索引

* 属性: index
* 无状态

* 设置
```lua
{
    ['index'] = 0,
}
```

* 获取
```lua
    0
```

### 6. CSS总表

* 先公共后私有; ① 见 `klbui_css_std_function.c`; ②③ 见 `klbui_ticker.c` `init_func_map`
* 仅 **normal**; 无伪类后缀

| 分组    | 键                                 | 读写      | 说明                          |
| ----- | --------------------------------- | ------- | --------------------------- |
| 控件通用  | `visibility`                      | get/set | 默认 hidden                  |
| CSS公共 | `margin` / `margin-*`             | get/set | 外边距                        |
| CSS公共 | `padding` / `padding-*`           | get/set | 内边距                        |
| 命令    | `ticker`                          | get/set | 定时器开关                     |
| 命令    | `ticker-default`                  | get/set | 默认定时器开关                  |
| 命令    | `ticker-interval`                 | get/set | 间隔 (ms)                     |
| 自定义   | `index`                           | get/set | 索引                          |

#### 6.1 参考
```lua
{
    ['type'] = 'kticker',
    ['pos'] = {0, 0, 1, 1},
    ['name'] = 'ticker1',
    ['visibility'] = 'hidden',
    ['ticker'] = true,
    ['ticker-default'] = true,
    ['ticker-interval'] = 100,
    ['index'] = 0,
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
            ['type'] = 'kticker',
            ['pos'] = {0, 0, 1, 1},
            ['name'] = 'ticker1',
            ['visibility'] = 'hidden',
            ['ticker-interval'] = 100,
        },
    },
}

klbui.parse(dialog, {}, {})
```
