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

见 [css/](css/). 文档与 C 实现对应、更新流程见 ai 技能 **klbcore-klbui-css-design**（台账 `reference.md`）。

| 文档 | type | C 真源 (现行) | 状态 |
|------|------|---------------|------|
| [css/klbui_css.md](css/klbui_css.md) | — | 语法 / `csser.lua` | 稳定 |
| [css/klbui_default_css.md](css/klbui_default_css.md) | — | `klbuiex_default.c` | 稳定 |
| [css/klbui_kstatic.md](css/klbui_kstatic.md) | `kstatic` | `klbwui/embed_widgets/klbui_static.c` | 已对齐 |
| [css/klbui_kview.md](css/klbui_kview.md) | `kview` | `klbwui/embed_widgets/klbui_view.c` | 已对齐 |
| [css/klbui_kbutton.md](css/klbui_kbutton.md) | `kbutton` | `klbwui/embed_widgets/klbui_button.c` | 已对齐 |
| [css/klbui_kpicture.md](css/klbui_kpicture.md) | `kpicture` | `klbwui/embed_widgets/klbui_picture.c` | 已对齐 |
| [css/klbui_kdemo.md](css/klbui_kdemo.md) | `kdemo` | `klbwui/embed_widgets/klbui_demo.c` | 已对齐 |
| [css/klbui_kcombo.md](css/klbui_kcombo.md) | `kcombo` | `klbwui/embed_widgets/klbui_combo.c` | 已对齐 |
| [css/klbui_kip.md](css/klbui_kip.md) | `kip` | `klbwui/embed_widgets/klbui_ip.c` | 已对齐 |
| [css/klbui_kcalendar.md](css/klbui_kcalendar.md) | `kcalendar` | `klbwui/embed_widgets/klbui_calendar.c` | 已对齐 |
| [css/klbui_kbtnex.md](css/klbui_kbtnex.md) | `kbtnex` | `klbwui/embed_widgets/klbui_btnex.c` | 已对齐 |
| [css/klbui_kline.md](css/klbui_kline.md) | `kline` | `klbwui/embed_widgets/klbui_line.c` | 已对齐 |
| [css/klbui_kanimation.md](css/klbui_kanimation.md) | `kanimation` | `klbwui/embed_widgets/klbui_animation.c` | 已对齐 |
| [css/klbui_kticker.md](css/klbui_kticker.md) | `kticker` | `klbwui/embed_widgets/klbui_ticker.c` | 已对齐 |
| [css/klbui_kcheck.md](css/klbui_kcheck.md) | `kcheck` | `klbwui/embed_widgets/klbui_check.c` | 已对齐 |
| [css/klbui_kdiv.md](css/klbui_kdiv.md) | `kdiv` | `klbwui/embed_widgets/klbui_div.c` | 已对齐 |
| [css/klbui_kedit.md](css/klbui_kedit.md) | `kedit` | `klbwui/embed_widgets/klbui_edit.c` | 已对齐 |
| [css/klbui_kpassword.md](css/klbui_kpassword.md) | `kpassword` | `klbwui/embed_widgets/klbui_password.c` | 已对齐 |
| [css/klbui_kvscrollbar.md](css/klbui_kvscrollbar.md) | `kvscrollbar` | `klbwui/embed_widgets/klbui_vscrollbar.c` | 已对齐 |
| [css/klbui_khscrollbar.md](css/klbui_khscrollbar.md) | `khscrollbar` | `klbwui/embed_widgets/klbui_hscrollbar.c` | 已对齐 |
| [css/klbui_kprogress.md](css/klbui_kprogress.md) | `kprogress` | `klbwui/embed_widgets/klbui_progress.c` | 已对齐 |
| [css/klbui_kslider.md](css/klbui_kslider.md) | `kslider` | `klbwui/embed_widgets/klbui_slider.c` | 已对齐 |
| [css/klbui_kvslider.md](css/klbui_kvslider.md) | `kvslider` | `klbwui/embed_widgets/klbui_vslider.c` | 已对齐 |
| [css/klbui_kdate.md](css/klbui_kdate.md) | `kdate` | `klbwui/embed_widgets/klbui_date.c` | 已对齐 |
| [css/klbui_ktime.md](css/klbui_ktime.md) | `ktime` | `klbwui/embed_widgets/klbui_time.c` | 已对齐 |
| [css/klbui_knum.md](css/klbui_knum.md) | `knum` | `klbwui/embed_widgets/klbui_num.c` | 已对齐 |
| [css/klbui_kspin.md](css/klbui_kspin.md) | `kspin` | `klbwui/embed_widgets/klbui_spin.c` | 已对齐 |
| [css/klbui_kgroup.md](css/klbui_kgroup.md) | `kgroup` | `klbwui/embed_widgets/klbui_group.c` | 已对齐 |
| [css/klbui_kradio.md](css/klbui_kradio.md) | `kradio` | `klbwui/embed_widgets/klbui_radio.c` | 已对齐 |
| [css/klbui_klist.md](css/klbui_klist.md) | `klist` | `klbwui/embed_widgets/klbui_list.c` | 已对齐 |
| [css/klbui_klistex.md](css/klbui_klistex.md) | `klistex` | `klbwui/embed_widgets/klbui_listex.c` | 已对齐 |
| [css/klbui_ktab.md](css/klbui_ktab.md) | `ktab` | `klbwui/embed_widgets/klbui_tab.c` | 已对齐 |
| [css/klbui_kmenu.md](css/klbui_kmenu.md) | `kmenu` | `klbwui/embed_widgets/klbui_menu.c` | 已对齐 |
| [css/klbui_krichtext.md](css/klbui_krichtext.md) | `krichtext` | `klbwui/embed_widgets/klbui_richtext.c` | 已对齐 |
| [css/klbui_kqrcode.md](css/klbui_kqrcode.md) | `kqrcode` | `klbwui/embed_widgets/klbui_qrcode.c` | 已对齐 |
| [css/klbui_kdialog.md](css/klbui_kdialog.md) | `kdialog` | `klbwui/embed_widgets/klbui_dialog.c` | 已对齐 |

## 相关

| 文档 | 内容 |
|------|------|
| 技能 **klbcore-design** | klbcore 与 C/k* 对照 (架构) |
| 技能 **klbcore-net-design** | 脚本 net/rtsp |
| [klua/readme.md](../klua/readme.md) | k* C 预加载 |
| [klb/klua/design/layers.md](../../klb/klua/design/layers.md) | klua L6 klbcore |
| [index-by-require.md](../index-by-require.md) | §③ 索引 |
| [lua/readme.md](../readme.md) | Lua 脚本文档总入口 |
