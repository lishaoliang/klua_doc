# klbcore 纯 Lua 脚本库

> `klua_doc/lua/klbcore/` — 类别: ③ 纯 Lua | 代码: `bin/klbcore/` (开发源 `klb/bin/klbcore/`) | klua 分层 **L6**
> **文档布局**: 根目录 [readme.md](readme.md); klbui 控件/CSS 见 [css/](css/)

**非** C 预加载; 入口脚本须配置 `package.path`. 与 C 同名模块 (klbnet, klbgui) **分文档阅读**.

## 查阅顺序

1. 本目录模块 `*.md` (Lua 调用面)
2. 技能 **klbcore-design** (架构) / **klbcore-net-design** (net/rtsp)
3. `bin/klbcore/` 源码

## package.path

```lua
local base = kenv.base_path() .. '/klbcore/'
package.path = package.path .. ';' .. base .. '?.lua;' .. base .. '?/init.lua'
```

详 [klb/klua/design/require-guide.md](../../klb/klua/design/require-guide.md) §4, 技能 **klbcore-design**.

## 模块 (require 清单)

### UI

| require | 文档 | 源码 | 设计 |
|---------|------|------|------|
| `klbcore.klbui` | [klbui.md](klbui.md) | `klbui/` | **klbcore-design** § klbui |
| klbui 控件/CSS | [css/](css/) | — | 同上 |

### 网络

| require | 文档 | 源码 | 设计 |
|---------|------|------|------|
| `klbcore.klbrtsp` | [klbrtsp.md](klbrtsp.md) | `klbrtsp/` | **klbcore-net-design** |
| `klbcore.klbsmp` | [klbsmp.md](klbsmp.md) | `klbsmp/` | **klb-mnp-smp-design** § klbcore 脚本层 |
| `klbcore.net.http_mime` | [net/http_mime.md](net/http_mime.md) | `net/http_mime.lua` | **klbcore-net-design** |
| `klbcore.net.httpc` | — | (待定) | **klbcore-net-design** |

### 通用

| require | 文档 | 源码 | 设计 |
|---------|------|------|------|
| `klbcore.util.stringex` 等 | [util.md](util.md) | `util/` | **klbcore-design** § 通用模块 |
| `klbcore.base.pname` | [base/pname.md](base/pname.md) | `base/pname.lua` | 同上 |
| `klbcore.base.klpcex` | [base/klpcex.md](base/klpcex.md) | `base/klpcex.lua` | 同上 |
| `klbcore.base.krpcex` | — | (待定) | 同上 |

### 示例 / 桩

| require | 文档 | 源码 | 说明 |
|---------|------|------|------|
| `klbcore.help.*` | — | `help/` | k* API 桩; 对照 [klua/](../klua/) |
| `klbcore.help.k_test.*` | — | `help/k_test/` | 可跑示例 |

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
| 技能 **klbcore-net-design** | 脚本 net/rtsp |
| [klua/readme.md](../klua/readme.md) | k* C 预加载 |
| [klb/klua/design/layers.md](../../klb/klua/design/layers.md) | klua L6 klbcore |
| [index-by-require.md](../index-by-require.md) | §③ 索引 |
| [lua/readme.md](../readme.md) | Lua 脚本文档总入口 |
