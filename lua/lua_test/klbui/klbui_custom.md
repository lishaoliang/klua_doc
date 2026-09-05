# 第 1 章 § 1.1 自定义内容 (custom)

> API: [klbui.md](../../klbcore/klbui.md), [kgui.md](../../klua/kgui.md) | 枢纽: [readme.md](readme.md)
> 源码: `klua_run/lua_test/`klbui/custom/ch1_s1_{z}.lua` → `lua_test.klbui.custom.ch1_s1_{z}`

约定见 [readme.md](readme.md). 本节为 **可运行 UI 自定义内容**: 开窗看界面; **不** 做 CLI 断言. 解析/选择器/CSS 语义见 klbcore [klbui.md](../../klbcore/klbui.md).

页面写法: 复制 `klua_run/lua_test/klbui/custom/page_tmpl.lua`, 只改 `css` / `dialog` / `commands`.

## lua 对照

| doc_id | 语义 id | lua `@brief` | 文件 |
|--------|---------|--------------|------|
| `1.1.1` | `klbui.custom.chrome` | 1.1.1 UI 分辨率 / 语言 / 字体 / 外观 | `custom/ch1_s1_1.lua` |
| `1.1.2` | `klbui.custom.widgets` | 1.1.2 UI 自定义文本与按钮 | `custom/ch1_s1_2.lua` |
| `1.1.3` | `klbui.custom.types` | 1.1.3 UI 已注册 type 一览 | `custom/ch1_s1_3.lua` |

共享: `custom/page_tmpl.lua`; `common.lua` / `ui.lua` / `pref.lua` / `theme.lua`.

---

## 1.1 自定义内容

### 1.1.1 分辨率, 语言, 字体与外观

- `klbui.custom.chrome`

| 项 | 值 |
|----|-----|
| doc_id | `1.1.1` |
| CLI | `1.1.1` / `klbui.custom.chrome` |
| 模块 | `lua_test.klbui.custom.ch1_s1_1` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

页面模块: 按 `page_tmpl`; 本页满窗 parse; 开窗由 `lua_test.klbui.ui.run`; 元素只写本页

配置目录: `kenv.pref_path("klua", "lua_test")` (org=`klua`, app=`lua_test`). 文件 `klbui.json`: `resolution` 标准称谓 `HD` / `WXGA` / `WXGA+` / `HD+` / `WSXGA+` / `Full HD` / `WUXGA` / `QHD` / `WQXGA` / `QHD+` / `UW-FHD` / `UWQHD` / `4K UHD`; 下拉显示经 `pref.rs_title` 附像素 (如 `Full HD (1920x1080)`). `language` (`en` / `zh` / `zh_tw`), `font_size` (`16` / `20` / `24` / `28` / `32`; 旧档名 `small`/`medium`/`large`/`xlarge` 读入时映射到上列), `font_face` (`auto` 自动选 `demores/font` 首个字库 / 或指定 stem 如 `simsun`), `appearance` (`dark` 仿 VS Code 黑 / `win11-dark` 仿 Win11 深色 / `ios-dark` 仿 iOS 深色 / `material-dark` 仿 Material 深色 / `github-dark` 仿 GitHub 深色 / `light` 仿 Windows XP 蓝 / `fluent` 仿 Win11 浅色; 旧值 `plain` 读入时映射为 `dark`). 缺省 `Full HD` + `en` + `24` + `auto` + `dark`. 章共享 `lua_test.klbui.pref` / `theme`.

步骤: 1. `klua test.lua 1.1.1` (无 `wsdl` 则转 `wlua`); 2. 看本页内容区; 3. 用下拉选分辨率, 选语言 (English / 简体中文 / 繁體中文), 选字体大小, 选字体 (`auto` / 指定字库), 选外观; 4. 关窗

预期: SDL 窗按 `klbui.json` 分辨率打开; 分辨率 / 语言 / 字体大小 / 字体 / 外观在窗口内居中; 窗大于当前桌面时仍可点到下拉; 语言切换后内容区文案立即更新; 字体大小, 字库与外观配色切换后本页立即更新并写入 json; 图片检索根为 `demores/images/tmpimage`; 分辨率 **下次开窗**生效; 其它 UI 条若未写死配色则按已存 `appearance` + `font_size` + `font_face`; 页面有 `apply_theme` 时由 `ui.run` 调用; 关窗结束

失败: 未开窗; 下拉无效或不写 `klbui.json`; 改字体/外观后本页不变; 小屏大窗时无法点到分辨率下拉

---

### 1.1.2 文本与按钮

- `klbui.custom.widgets`

| 项 | 值 |
|----|-----|
| doc_id | `1.1.2` |
| CLI | `1.1.2` / `klbui.custom.widgets` |
| 模块 | `lua_test.klbui.custom.ch1_s1_2` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

页面模块: 按 `page_tmpl`; 开窗由 `lua_test.klbui.ui.run`

步骤: 1. `klua test.lua 1.1.2`; 2. 本页见 `kstatic` 与 `kbutton`; 3. 点 `Ping` 后标签应变; 4. 关窗

预期: 文本与按钮可见; 点 `Ping` 后标签为 `ping ok`

失败: 控件不可见; 点击无变化

---

### 1.1.3 已注册 type

- `klbui.custom.types`

| 项 | 值 |
|----|-----|
| doc_id | `1.1.3` |
| CLI | `1.1.3` / `klbui.custom.types` |
| 模块 | `lua_test.klbui.custom.ch1_s1_3` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

页面模块: 按 `page_tmpl`; 开窗由 `lua_test.klbui.ui.run`

步骤: 1. `klua test.lua 1.1.3`; 2. 本页见 `kview` / `kstatic` / `kbutton` / `kpicture` / `kdemo`; 3. 关窗

预期: 五种已注册 type 均可看见 (无图时 `kpicture` 为色块)

失败: 某 type 未出现或 parse 失败

注: 顶层用 **kview**; **不用** `kdialog`.
