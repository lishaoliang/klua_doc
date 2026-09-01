# 第 1 章 § 1.1 自定义内容 (custom)

> API: [klbui.md](../../klbcore/klbui.md), [kgui.md](../../klua/kgui.md) | 枢纽: [readme.md](readme.md)
> 源码: `klua_run/lua_test/`klbui/custom/ch1_s1_{z}.lua` → `lua_test.klbui.custom.ch1_s1_{z}`

约定见 [readme.md](readme.md). 本节为 **可运行 UI 自定义内容**: 开窗看界面; **不** 做 CLI 断言. 解析/选择器/CSS 语义见 1.2 / 1.3 / 1.4.

页面写法: 复制 `klua_run/lua_test/klbui/custom/page_tmpl.lua`, 只改 `css` / `dialog` / `commands`.

---

## 1.1 自定义内容

### 1.1.1 分辨率与语言

- `klbui.custom.chrome`

| 项 | 值 |
|----|-----|
| doc_id | `1.1.1` |
| CLI | `1.1.1` / `klbui.custom.chrome` |
| 模块 | `lua_test.klbui.custom.ch1_s1_1` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

页面模块: 按 `page_tmpl`; 本页满窗 parse; 开窗由 `lua_test.klbui.ui.run`; 元素只写本页

配置目录: `kenv.pref_path("klua", "lua_test")` (org=`klua`, app=`lua_test`). 文件 `klbui.json`: `resolution` (`720p` / `1366x768` / `900p` / `1080p` / `1440p` / `4k`; `1366x768` 即 WXGA, `900p` 即 HD+, `1440p` 即 2k), `language` (`en` / `zh`). 缺省 `1080p` + `en`. 章共享 `lua_test.klbui.pref`.

步骤: 1. `klua test.lua 1.1.1` (无 `wsdl` 则转 `wlua`); 2. 看本页内容区; 3. 用下拉选分辨率, 选语言 (English / 中文); 4. 关窗

预期: SDL 窗按 `klbui.json` 分辨率打开; 语言切换后内容区文案立即更新; 分辨率写入 json, **下次开窗**生效; 关窗结束

失败: 未开窗; 下拉无效或不写 `klbui.json`

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
