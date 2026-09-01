## 默认全局CSS

* 无注册类型

### 1. 概述

```c
// C实现文件: ./klb/src_c/klbgui/extensions/klbuiex_default.c
```

* 控件创建前通过 `klbui.default_css` / `kgui.set_default_css` 设置; 控件 `css_init` 从 `klb_gui_get_std_default` 复制参考属性
* 键名公式: `property[:checked][:focus][:disabled][:input]` (见 [klbui_css.md](klbui_css.md))

### 2. 支持状态

| C 字段 | CSS 伪类 | 说明 |
|--------|----------|------|
| normal | (无后缀) | 常规 |
| focus | `:focus` | 聚焦 |
| disable | `:disabled` | 不使能 |
| check | `:checked` | 选中参考 (toggle 控件) |
| input | `:input` | 输入态 |

### 3. 默认配置

| 项 | 默认值 |
|----|--------|
| margin | 0,0,0,0 |
| padding | 1,1,1,1 |
| font-size | 24 |
| text-align | left |
| 配色 | Visual Studio 深色系 |

| 状态 | 文本色 | 边框色 | 背景色 |
|------|--------|--------|--------|
| normal | #DCDCDC | #505050 | #1F1F1F |
| focus | #DCDC0A | #DC5050 | #1F1F1F |
| disabled | #B4B4B4 | #505050 | #1F1F1F |
| checked | #0AD2D2 | #B450B4 | #1F1F1F |
| input | #B4B4B4 | #505050 | #1F1F1F |

### 4. 参考使用

* 设置
```lua
local klbui = require("klbcore.klbui")

-- 设置多个全局CSS属性
klbui.default_css({
    ['color'] = {255,80,80,80},
    ['color:focus'] = {255,220,220,220},
    ['color:disabled'] = {255,20,20,20},
})
```

* 获取
```lua
local klbui = require("klbcore.klbui")

-- 获取单个全局CSS属性
local color = klbui.default_css('color')
print('color:', color)
```

### 5. CSS属性

#### 5.1 外边距

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

#### 5.2 内边距

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

#### 5.3 文本

##### 5.3.1 文本颜色

* 属性: color
* 支持状态: normal, focus, disabled, checked, input

* 设置
```lua
{
    ['color'] = {255,220,220,220},          -- normal 常规状态
    ['color:focus'] = 0xFFA0A0A0,           -- focus 聚焦状态
    ['color:disabled'] = '0xFFA0A0A0',      -- disabled 不使能状态
    ['color:checked'] = {255,220,220,220},  -- checked 选中参考
    ['color:input'] = {255,220,220,220},    -- input 输入状态
}
```

* 获取
```lua
    0xFFA0A0A0
```

##### 5.3.2 文本对齐

* 属性: text-align
* 支持状态: normal, focus, disabled, checked, input

* 设置
```lua
{
    ['text-align'] = 'center',
    ['text-align:focus'] = 'left',
    ['text-align:disabled'] = 'right',
    ['text-align:checked'] = 'left',
    ['text-align:input'] = 'left'
}
```

* 获取
```lua
    'center'        -- 中心对齐
    'left'          -- 左对齐
    'right'         -- 右对齐
```

#### 5.4 字体

##### 5.4.1 字体大小

* 属性: font-size
* 支持状态: normal, focus, disabled, checked, input

* 设置
```lua
{
    ['font-size'] = 24,
    ['font-size:focus'] = 28,
    ['font-size:disabled'] = 20,
    ['font-size:checked'] = 20,
    ['font-size:input'] = 20,
}
```

#### 5.5 背景

##### 5.5.1 背景色

* 属性: background-color
* 支持状态: normal, focus, disabled, checked, input

* 设置
```lua
{
    ['background-color'] = {255,220,80,20},
    ['background-color:focus'] = 0xFFA0A0A0,
    ['background-color:disabled'] = '0xFFA0A0A0',
    ['background-color:checked'] = {255,220,80,20},
    ['background-color:input'] = {255,220,80,20},
}
```

* 获取
```lua
    0xFFA0A0A0
```

##### 5.5.2 背景图片

* 属性: background-image
* 支持状态: normal, focus, disabled, checked, input

* 设置
```lua
{
    ['background-image'] = '',
    ['background-image:focus'] = 'bbb.bmp',
    ['background-image:disabled'] = 'ccc.bmp',
    ['background-image:checked'] = 'eee.bmp',
    ['background-image:input'] = 'fff.bmp',
}
```

* 获取
```lua
    'bbb.bmp'
```

#### 5.6 边框

##### 5.6.1 边框宽度

* 属性: border-width
* 支持状态: normal, focus, disabled, checked, input

* 设置
```lua
{
    ['border-width'] = 1,
    ['border-width'] = {1, 1, 1, 1},

    ['border-width:focus'] = 1,
    ['border-width:disabled'] = 1,
    ['border-width:checked'] = 1,
    ['border-width:input'] = 1,
}
```

* 获取
```lua
    1
```

##### 5.6.2 边框颜色

* 属性: border-color
* 支持状态: normal, focus, disabled, checked, input

* 设置
```lua
{
    ['border-color'] = {255,220,30,30},
    ['border-color'] = 0xFFA0A0A0,
    ['border-color'] = '0xFFA0A0A0',

    ['border-color:focus'] = {255,220,30,30},
    ['border-color:disabled'] = {255,220,30,30},
    ['border-color:checked'] = {255,220,30,30},
    ['border-color:input'] = {255,220,30,30},
}
```

* 获取
```lua
    0xFFA0A0A0
```
