## 列表

* 注册类型: klist

### 1. 概述

```c
// C实现文件: ./klb/src_packages/klbwui/embed_widgets/klbui_list.c
// 窗口实现: ./klb/src_packages/klbwui/embed_wnd/klbwnd_list.c
// 控件通用 CSS 方法: ./klb/src_c/klbgui/klbui_css_std_function.c
```

* 所有已注册控件均通过 `klb_gui_css_map_append_std_function` 挂载 **控件通用属性/方法**（见下文「3. 控件通用属性/方法」）
* 样式键支持 **normal** / **focus** / **disabled**; 无 `:checked`
* 默认窗口样式: `style-peek-event` + `style-focus-without-redraw`
* 题头/行/滚动条子样式见 `title.*` / `row.*` / `vscrollbar.*` / `vscrollbar-btn.*`
* 数据维护走 `append_column` / `update_column` / `append` / `clear` / `clear_data`; 选中走 `value`

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

#### 4.8 题头栏

* 属性: title.line-color / title.background-color
* 无状态
* 题头分隔线与背景色

* 设置
```lua
{
    ['title.line-color'] = 0xFF808080,
    ['title.background-color'] = 0xFF404040,
}
```

#### 4.9 行样式

* 属性: row.color / row.text-align / row.font-size + 伪类
* 行文本样式

* 属性: row1.background-color / row2.background-color + 伪类
* 奇偶行底色

* 属性: row-check.background-color
* 无状态
* 选中行背景色

* 设置
```lua
{
    ['row.color'] = 0xFFDCDCDC,
    ['row1.background-color'] = 0xFF303030,
    ['row2.background-color'] = 0xFF282828,
    ['row-check.background-color'] = 0xFF0060C0,
}
```

#### 4.10 垂直滚动条

* 属性: vscrollbar.background-color / vscrollbar.border-width / vscrollbar.border-color + 伪类
* 内置 `kvscrollbar` 子控件样式

* 属性: vscrollbar-btn.color + 伪类
* 滚动条内 `kbtnex` 按钮文本色

### 5. 自定义属性

#### 5.1 追加列

* 方法: append_column
* 无状态
* 仅设置; 追加题头列

* 设置
```lua
{
    ['append_column'] = {
        { ['title'] = '名称', ['width'] = 120 },
        { ['title'] = '值', ['width'] = 80 },
    },
}
```

#### 5.2 更新列

* 方法: update_column
* 无状态
* 仅设置; 更新题头列

#### 5.3 追加行

* 方法: append
* 无状态
* 仅设置; 追加数据行

* 设置
```lua
{
    ['append'] = {
        { ['col1'] = '项1', ['col2'] = '100' },
        { ['col1'] = '项2', ['col2'] = '200' },
    },
}
```

#### 5.4 清空

* 方法: clear
* 无状态
* 清空题头与数据

* 方法: clear_data
* 无状态
* 仅清空数据行

#### 5.5 选中

* 属性: value
* 无状态
* get 返回选中行索引 (Lua 从 1 起) 与行 map; set 可选

* 获取
```lua
    1, { col1 = '项1', col2 = '100' }
```

### 6. CSS总表

* 先公共后私有; ① 见 `klbui_css_std_function.c`; ②③ 见 `klbui_list.c` `init_func_map`
* 主体与 `row.*` / `vscrollbar.*` 支持三态; `title.*` / `row-check.*` / `row1|2.*` 无伪类或见 bind

| 分组    | 键                                 | 读写      | 说明                          |
| ----- | --------------------------------- | ------- | --------------------------- |
| 控件通用  | `style-peek-event`                | get/set | 冒泡读消息; klist 默认 `true`    |
| 控件通用  | `style-focus-without-redraw`      | get/set | 聚焦不重绘; klist 默认 `true`    |
| CSS公共 | `margin` / `padding` / `color` / `text-align` / `font-size` + 伪类 | get/set | 主体样式 |
| CSS公共 | `background-color` / `background-image` / `border-*` + 伪类 | get/set | 主体边框与背景 |
| CSS私有 | `title.line-color` / `title.background-color` | get/set | 题头栏 |
| CSS私有 | `row.*` + 伪类                      | get/set | 行文本                        |
| CSS私有 | `row1.background-color` / `row2.background-color` + 伪类 | get/set | 奇偶行 |
| CSS私有 | `row-check.background-color`      | get/set | 选中行                        |
| CSS私有 | `vscrollbar.*` + 伪类               | get/set | 内置滚动条                     |
| CSS私有 | `vscrollbar-btn.color` + 伪类       | get/set | 滚动条按钮                     |
| 命令    | `append_column` / `update_column` | set     | 题头列                        |
| 命令    | `append` / `clear` / `clear_data` | set     | 数据行                        |
| 自定义   | `value`                           | get/set | 选中行索引与 map                 |

#### 6.1 参考
```lua
{
    ['type'] = 'klist',
    ['pos'] = {10, 40, 280, 180},
    ['name'] = 'list1',
    ['append_column'] = {
        { ['title'] = '名称', ['width'] = 120 },
        { ['title'] = '值', ['width'] = 80 },
    },
    ['row1.background-color'] = 0xFF303030,
    ['row2.background-color'] = 0xFF282828,
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
            ['type'] = 'klist',
            ['pos'] = {10, 40, 280, 180},
            ['name'] = 'list1',
            ['append_column'] = {
                { ['title'] = '名称', ['width'] = 120 },
                { ['title'] = '值', ['width'] = 80 },
            },
            ['append'] = {
                { ['col1'] = '项1', ['col2'] = '100' },
            },
        },
    },
}

klbui.parse(dialog, {}, {})
```
