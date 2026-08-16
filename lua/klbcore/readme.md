# klbcore 纯 Lua 脚本库

> `klua_doc/lua/klbcore/` — 类别: ③ 纯 Lua | 代码: `klb/bin/klbcore/` | klua 分层 **L6**
> **文档布局**: 根目录 [readme.md](readme.md); klbui 控件/CSS 见 [css/](css/)

**非** C 预加载; 入口脚本须配置 `package.path`. 与 C 同名模块 (klbnet, klbgui) **分文档阅读**.

## package.path

```lua
local base = kenv.base_path() .. '/klbcore/'
package.path = package.path .. ';' .. base .. '?.lua;' .. base .. '?/init.lua'
```

详 [klb/klua/design/require-guide.md](../../klb/klua/design/require-guide.md) §4, 技能 **klbcore-design**.

## 模块 (require 清单)

| require 前缀 | 源码目录 | 设计文档 |
|--------------|----------|----------|
| `klbcore.klbui` | `klbui/` | **klbcore-design** § klbui; 控件见下表 |
| `klbcore.klbrtsp` | `klbrtsp/` | **klbcore-net-design** |
| `klbcore.net.http_mime` | `net/http_mime.lua` | **klbcore-net-design** |
| `klbcore.klbsmp` | `klbsmp/` | **klb-mnp-smp-design** § klbcore 脚本层 |
| `klbcore.util.*` | `util/` | **klbcore-design** § 通用模块 |
| `klbcore.base.*` | `base/` | **klbcore-design** |
| `klbcore.help.*` | `help/` | 示例/桩; 对照 `klb/bin/klbcore/help/` |

架构与 C/k* 对照: 技能 **klbcore-design**; klua L6 见 [klb/klua/design/layers.md](../../klb/klua/design/layers.md).

## klbui 控件与 CSS

见 [css/](css/).

| 主题 | 文档 |
|------|------|
| CSS 约定 | [css/klbui_css.md](css/klbui_css.md) |
| `kbutton` | [css/klbui_kbutton.md](css/klbui_kbutton.md) |
| `kcombo` | [css/klbui_kcombo.md](css/klbui_kcombo.md) |
| `kmessagebox` | [css/klbui_kmessagebox.md](css/klbui_kmessagebox.md) |
| `kstatic` | [css/klbui_kstatic.md](css/klbui_kstatic.md) |
| 默认 CSS | [css/klbui_default_css.md](css/klbui_default_css.md) |

## 相关

| 文档 | 内容 |
|------|------|
| 技能 **klbcore-design** | klbcore 与 C/k* 对照 (架构) |
| [klb/klua/design/layers.md](../../klb/klua/design/layers.md) | klua L6 klbcore |
| [index-by-require.md](../index-by-require.md) | §③ 索引 |
| [lua/readme.md](../readme.md) | Lua 脚本文档总入口 |
