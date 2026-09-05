## CSS样式

* 本文: klbui **CSS 语法约定** (键名、值类型、伪类、分层)
* 默认参考属性: [klbui_default_css.md](klbui_default_css.md)
* 各 type 有效键与示例: 各 [klbui_k*.md](klbui_kview.md) (格式标准 = `klbui_kview.md`)
* 图片资源加载: **klbcore-klbui-page** § 图片资源

### 1. 样式格式

* 原则1: 主体参考 CSS3 定义
* 原则2: 键名以本节命名公式为准

**命名公式**：

```
[part.]property[:checked][:focus][:disabled][:input]
```

| 段 | 说明 |
|----|------|
| `part` | 可选，控件内部逻辑子区域（非独立子 wnd） |
| `property` | kebab-case，优先 CSS3 同名 |
| `:checked` 等 | 可选伪类; normal 省略; 顺序 `:checked` -> `:focus` -> `:disabled` -> `:input` |

* 控件通用键 (如 `move`、`wndpos-canvas`) 同时提供 `-` 与 `_` 两种写法; 盒模型键一般为 kebab-case
* 键名须与 C `func_map` **精确匹配**; 写错键无效 (无级联、无选择器)

* 示例

```lua
{
    ['color'] = {255,220,220,220},
    ['color:focus'] = 0xFFA0A0A0,
    ['color:disabled'] = '0xFF808080',
    ['color:checked'] = '0xFFA0A0A0',
    ['color:checked:focus'] = 0xFFB0B0B0,
    ['image.text-align'] = 'left',
}
```

### 2. 元素(窗口)状态

* 常规状态: normal（键无伪类后缀）
* 聚焦状态: `:focus`
* 不使能状态: `:disabled`
* 选中状态: `:checked`（toggle 控件；可与 `:focus` / `:disabled` 组合）
* 输入状态: `:input` (主要用于 `default_css` 参考; 多数控件不支持)

* 组合规则:
  * `:focus` / `:disabled` / `:input` 在同一绘制分支上通常**互斥** (由运行时 `state.status` 决定)
  * `:checked` 可与上述之一**叠加** (toggle 控件六态, 如 `color:checked:focus`)
  * 各 type 实际支持的伪类见对应 `klbui_k*.md` §2

```lua
{
    ['background-color'] = {255,220,220,220},
    ['background-color:focus'] = 0xFFA0A0A0,
    ['background-color:disabled'] = '0xFFA0A0A0',
    ['background-color:checked'] = '0xFFA0A0A0',
    ['background-color:checked:focus'] = 0xFFB0B0B0,
}
```

### 3. 私有子区域 (part)

* 对含有命名绘制区域的控件，在 property 前加 `part.`
* **仅一层**: `part.property[:pseudo…]`; 禁止 `a.b.c.color` 等多级 part
* 独立子 wnd 节点不用 part, 对子路径 `kgui.set` 或 csser 选择器设置

```lua
{
    ['button.background-color'] = {255,220,220,220},
    ['button.background-color:focus'] = 0xFFA0A0A0,
    ['button.background-color:disabled'] = '0xFFA0A0A0',
    ['button.background-color:checked'] = '0xFFA0A0A0',
}
```

* 批量：嵌套 `style` 表由 csser 展平为 `part.property`（详 wui `css-design.md` §3）

### 4. 边框模型
* 参考: https://www.w3school.com.cn/css/css_boxmodel.asp


* 模型图

```c
///  *---------------- margin(外边距) ------------------*
///  |  *------------- border(边框) -----------------*  |
///  |  |  *---------- padding(内边距) -----------*  |  |
///  |  |  |  *--------------------------------*  |  |  |
///  |  |  |  |                                |  |  |  |
///  |  |  |  |        element(元素/wnd)       |  |  |  |
///  |  |  |  |                                |  |  |  |
///  |  |  |  *--------------------------------*  |  |  |
///  |  |  *--------------------------------------*  |  |
///  |  *--------------------------------------------*  |
///  *--------------------------------------------------*
```

* 背景应用于由内容(元素/wnd)和内边距、边框组成的区域; margin 在其外, 不参与背景绘制
* `margin` / `padding` / `border-width` 通常**无伪类** (作用于控件框, 不随 focus 分支)
* 默认值见 [klbui_default_css.md](klbui_default_css.md) (如 margin 0、padding 1)

#### 4.1 外边距与内边距

* 属性: `margin` / `padding` (及 `-top` / `-right` / `-bottom` / `-left`)
* 值: 整数像素; 可写单值或四元表 `{上, 右, 下, 左}`

* 设置

```lua
{
    ['margin'] = {0, 0, 0, 0},
    ['padding'] = 1,
    ['padding-top'] = 2,
}
```

* 获取

```lua
    0       -- 单值键
    1       -- 四元表键亦可能返回单值 (见控件实现)
```

#### 4.2 边框

* 属性: `border-width` / `border-color`
* `border-width`: 整数或四元表 `{上, 右, 下, 左}`
* `border-color`: 同 §5 颜色写法; 可带 §2 伪类 (视控件而定)
* 有 `background-image` 时不画边框 (见 §6.1)

* 设置

```lua
{
    ['border-width'] = 1,
    ['border-width:focus'] = 2,
    ['border-color'] = {255, 80, 80, 80},
}
```

* 获取

```lua
    1
    0xFF505050
```

### 5. 颜色模型

* 内部格式: ARGB8888 (`KLB_ARGB8888`)
* 通道顺序: **A, R, G, B**; 每通道 0～255
* 打包: `0xAARRGGBB` (A 在最高字节)
* 不支持 CSS3 的 `#RRGGBB` / `rgb()` / `rgba()` / 颜色名

```
  31        24 23        16 15         8 7          0
 +----------+-----------+-----------+-----------+
 |    A     |     R     |     G     |     B     |
 +----------+-----------+-----------+-----------+
```

| 通道 | 含义 | 例 (`0xFFDCDCDC`) |
|------|------|-------------------|
| A | 透明度; 255 不透明, 0 全透明 | `0xFF` |
| R | 红 | `0xDC` |
| G | 绿 | `0xDC` |
| B | 蓝 | `0xDC` |

#### 5.1 编写方式

下列三种写法等价 (均为不透明浅灰 `{255,220,220,220}` == `0xFFDCDCDC`):

| 写法 | 类型 | 示例 |
|------|------|------|
| 表 | `{A, R, G, B}` | `{255, 220, 220, 220}` |
| 整数 | `0xAARRGGBB` | `0xFFDCDCDC` |
| 字符串 | 十六进制文本 | `'0xFFDCDCDC'` 或 `'FFDCDCDC'` |

* 获取: 返回整数 `0xAARRGGBB`

```lua
{
    ['color'] = {255,220,220,220},   -- 表 {A, R, G, B}
    ['color:focus'] = 0xFFA0A0A0,    -- 整数
    ['color:disabled'] = '0xFF808080', -- 字符串
}
```

### 6. 图片使用

* CSS 值均为 **图片资源 key** (字符串); 不写磁盘绝对路径
* key 相对当前 **image_dirs 检索根**, **以 `/` 开头**, **不含扩展名**; 子目录保留
* 空字符串 `''` 表示清除图片
* 设置时若带末尾 `.bmp` / `.png`, C 侧会自动剥除扩展名 (仍推荐直接写无扩展名 key)
* 图片须在 **parse 前** 通过 `klbui.load_image` / `uires.load_images` 登记; 未登记的 key 控件无图 (加载流程见 **klbcore-klbui-page** § 图片资源)

| 项 | 约定 |
|----|------|
| 格式 | `/dialog_close_normal`, `/icon/ok` |
| 类型 | 字符串 |
| 获取 | 返回登记后的 key (无扩展名) |
| 状态 | 可叠加 §2 伪类, 如 `background-image:focus` |

* 示例

```lua
{
    ['background-image'] = '/panel',
    ['background-image:focus'] = '/panel_f',
    ['background-image:disabled'] = '',
    ['background-image-mode'] = 'scale9',
    ['image.foreground-image:checked'] = '/check_on',
}
```

#### 6.1 背景图

* 属性: `background-image`
* 绘制于控件 **背景层** (内容 + padding + border 区域, 见 §4)
* 有背景图时按图绘制, **不再画** `border-*`; 无图时填充 `background-color` 并画边框
* `kpicture` 的 `image` 与 `background-image` 为同一 handler (别名)

* 设置

```lua
{
    ['background-image'] = '/panel',
    ['background-image:focus'] = '/panel_f',
}
```

* 获取

```lua
    '/panel'
```

#### 6.2 背景图画法

* 属性: `background-image-mode`
* 与 `background-image` **独立**, 可组合; 是否支持状态后缀视控件而定 (见各 `klbui_k*.md`)

| 值 | 含义 |
|----|------|
| `'default'` | 原图拷贝, 超出控件区域裁剪 |
| `'resize'` | 缩放铺满控件区域 |
| `'scale9'` | 九宫格拉伸 |

* 设置

```lua
{
    ['background-image'] = '/panel',
    ['background-image-mode'] = 'scale9',
}
```

* 获取

```lua
    'scale9'
```

#### 6.3 关键色透明

* 属性: `background-image-color-key`
* 布尔; `true` 时按 BMP **关键色** 透明绘制; 可与 `scale9` 同时使用
* PNG 使用文件自身 alpha, 一般不须开启此项

* 设置

```lua
{
    ['background-image'] = '/panel',
    ['background-image-mode'] = 'scale9',
    ['background-image-color-key'] = true,
}
```

* 获取

```lua
    false
```

#### 6.4 前景图与子区域

* 属性: `foreground-image` (及 `part.foreground-image`, 如 `button-symbol.foreground-image`)
* 绘制于背景之上; 多用于图标、滑块 (`thumb.image`) 等 **part** 区域 (见 §3)
* 部分控件另有 `foreground-image-width` / `foreground-image-height` 限定图标尺寸
* 各控件支持的 part 名、前景/背景键及有效状态见对应 `klbui_k*.md`

#### 6.5 其它图片键

| 键 | 说明 |
|----|------|
| `scale9-image` | 部分控件 (kview/kbutton/kstatic) 九宫格路径; 与 `background-image-mode` 分工不同 |
| `image-stretch` | kpicture 布尔; 是否缩放铺满 (非 scale9) |

### 7. 文本

* 文本色 `color` 见 §5; 本节为对齐与字号
* 是否支持伪类因控件而异 (kview 无文本绑定; kbutton 等有六态)

#### 7.1 文本对齐

* 属性: `text-align`
* 取值: `'left'` / `'center'` / `'right'`

* 设置

```lua
{
    ['text-align'] = 'center',
    ['text-align:focus'] = 'left',
}
```

* 获取

```lua
    'center'
```

#### 7.2 字体大小

* 属性: `font-size`
* 值: 整数像素 (无单位后缀)

* 设置

```lua
{
    ['font-size'] = 24,
    ['font-size:disabled'] = 20,
}
```

* 获取

```lua
    24
```

### 8. 不支持项

* 贴近 CSS3 **属性名与伪类语义**; 下列 Web CSS 能力**不支持**:

| 不支持 | 说明 |
|--------|------|
| 级联 / 继承 | 无父→子自动继承; 须显式 set 或 `default_css` / `global_css` |
| 选择器 | 无 `.class` / `#id` 语法; 选择器在 parse/csser 层, 非键名一部分 |
| 单位 | 边距/字号等为整数像素; 无 `px`/`em`/`%` 解析 |
| 颜色语法 | 无 `#RRGGBB` / `rgb()` / 颜色名 (见 §5) |
| 复合 shorthand | 无 `border:` / `background:` 等一条多属性写法 |

* 布尔键 (如 `visibility`、`background-image-color-key`) 可接受 `true`/`false` 或控件文档列明的字符串

### 9. 样式分层

| 层 | API | 说明 | 文档 |
|----|-----|------|------|
| 默认参考 | `klbui.default_css` | 控件创建前设置; `css_init` 复制参考 | [klbui_default_css.md](klbui_default_css.md) |
| 按 type 全局 | `klbui.global_css(type, …)` | 某 type 全部实例的默认样式 | 各 `klbui_k*.md` §7 |
| 实例 | parse / `jq().css` | 页面 css、运行时改值 | 页面 css、各控件 §7 |

* 生效: 实例 css 覆盖同键的 global / default; 具体以 parse 与 `kgui.set` 为准

#### 9.1 命令键与视觉属性

| 类型 | 示例 | 说明 |
|------|------|------|
| 视觉属性 | `color`、`margin`、`background-image` | 绘制分支; 见 §1 公式 |
| 窗口命令 | `move`、`resize`、`layout`、`refresh` | 仅设置; 触发布局或重绘 |
| 显隐 / 状态 | `show`、`hide`、`visibility`、`check`、`disable` | 运行时状态, 非绘制属性 |
| 业务字段 | `title`、`value`、`index` | 控件文档 §5 自定义属性 |

* 控件通用命令键完整清单: 各 `klbui_k*.md` §3 (真源 `klbui_css_std_function.c`)

