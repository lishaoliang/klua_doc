## CSS样式

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
* 输入状态: `:input`

```lua
{
    ['background-color'] = {255,220,220,220},
    ['background-color:focus'] = 0xFFA0A0A0,
    ['background-color:disabled'] = '0xFFA0A0A0',
    ['background-color:checked'] = '0xFFA0A0A0',
    ['background-color:input'] = '0xFFA0A0A0',
}
```

### 3. 私有子区域 (part)

* 对含有命名绘制区域的控件，在 property 前加 `part.`

```lua
{
    ['button.background-color'] = {255,220,220,220},
    ['button.background-color:focus'] = 0xFFA0A0A0,
    ['button.background-color:disabled'] = '0xFFA0A0A0',
    ['button.background-color:checked'] = '0xFFA0A0A0',
    ['button.background-color:input'] = '0xFFA0A0A0',
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

* 背景应用于由内容(元素/wnd)和内边距、边框组成的区域

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
