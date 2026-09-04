# 第 1 章 klbui (Lua 手测)

> `klua_doc/lua/lua_test/klbui/` | API: [klbui](../../klbcore/klbui.md), [kgui](../../klua/kgui.md) | 源码规划: `klua_run/lua_test/`klbui/`
> 约定 **klua-test-design**; 控件 CSS [klbcore/css](../../klbcore/css/)

记名 **klbui** = 脚本 UI `require("klbcore.klbui")` (P3); 经 **`kgui`** 落到 **klbgui** + **klbwui** 已注册 type. **非** `klua_run/sample/` 演示, **非** `klbcore/help/k_test`.

章号 **1.x** 与第 2/3 章、`pfs_test` **编号空间独立**. **冻结**: 1=klbui / 2=kpfs / 3=klb; 章内只追加节, 禁止再插入.

**组织原则**: 按 **手测流程** 分节 (custom UI → 脚本层 → 壳/层叠 → 简易控件 → 复合控件), **非** 照搬 API 子模块目录.

**源码目录** (`klua_run/lua_test/`klbui/`): 节子目录 `custom` `script` `shell` `basic` `composite`; 用例 `ch1_s{M}_{z}.lua`（`1.M.z`）; 章共享 `common.lua` (CLI) + `ui.lua` (UI 开窗) + `pref.lua` (`kenv.pref_path("klua","lua_test")` / `klbui.json`) + `theme.lua` (外观 `dark`/`win11-dark`/`ios-dark`/`material-dark`/`github-dark`/`light`/`fluent`); 1.1 模板 `custom/page_tmpl.lua`. 对齐第 2 章形态. **1.1 custom** 与 **1.2 script** 已登记; **1.3.1–1.3.8** / **1.4.3** 已登记; 其余 1.4–1.5 条文 **待实现** (有桩), 不登记.

---

## 运行

```bash
cd bin
./klua test.lua 1.1.1
./klua test.lua klbui.custom.chrome
./klua test.lua 1.1.x
./klua test.lua 1.2.1
./klua test.lua klbui.require
```

Windows: `klua.exe test.lua 1.1.1`. 双入口: doc_id 或语义 id (`klbui.*`).

默认宿主 **`klua`**: env 已注册 `_KLUA_EX_GUI_` 与 `klbwui_register_sim`, **无 SDL 窗口** 也可 `parse` / `set` / `get` (画布可为 NULL, `kgui.wh()` 可为 `0,0`).

**UI 条** (`registry` `ui=true`): 单条在 `klua` 下会转 **`wlua`** 开 SDL 窗 (框架 `lua_test.klbui.ui`); 也可直接 `wlua.exe test.lua 1.x.x`. 未部署 `wlua`/`wsdl` 则 skip. 批量 worker 无窗口, UI 条 skip.

## 批量 (第 1 章)

| 命令 | 说明 |
|------|------|
| `1.x` | 第 1 章全部已登记条 |
| `1.1.x` / `1.1` | 节 1.1 (UI custom) |
| `1.2.x` / `1.2` | 节 1.2 (脚本层) |
| `1.3.x` / `1.3` | 节 1.3 (壳; 现行 `1.3.1`–`1.3.8`) |
| `1.4.x` / `1.4` | 节 1.4 (简易; 现行 `1.4.3`) |
| `1.5.x` / `1.5` | 节 1.5 (复合; 待实现未登记) |
| `a` / `all` | 含本章已登记条 (现行 1.1.1 .. 1.2.9, `1.3.1`–`1.3.8`, `1.4.3`) |

无窗口依赖的 parse/set/get 可 `batch_ok=true`. 须 **wlua** 开窗/绘制的条标 **SINGLE_ONLY**.

## 流程与 API 对照

| 本节 | 手测文档 | 阶段 | 主要 API |
|------|----------|------|----------|
| 1.1 | [klbui_custom.md](klbui_custom.md) | 自定义内容 (UI) | `lua_test.klbui.ui` / [`klbui.parse`](../../klbcore/klbui.md) |
| 1.2 | [klbui_script.md](klbui_script.md) | 脚本层 (parse / select / css) | [`klbui.parse`](../../klbcore/klbui.md) / [`select`](../../klbcore/klbui.md) / [`has_global_css`](../../klbcore/klbui.md) |
| 1.3 | [klbui_shell.md](klbui_shell.md) | 壳 / 层叠 | `kdialog` `kview` `ktab` `kmenu` `kdiv`; `modal` / `popup` / `messagebox` |
| 1.4 | [klbui_basic.md](klbui_basic.md) | 简易控件 | `kstatic` `kbutton` `kedit` `kcombo` … |
| 1.5 | [klbui_composite.md](klbui_composite.md) | 复合控件 | `klist` `klistex` `kdate` … |

节号 1.3–1.5 已占; 只追加条, 不提前占 1.6+.

## 公共约定

| 项 | 约定 |
|----|------|
| 模块 | [`require("kgui")`](../../klua/kgui.md) 或 [`require("klbcore.klbui")`](../../klbcore/klbui.md) 失败 → skip (如 `no-gui` / `no-wui`) |
| 顶层 type | 脚本层 / 页面壳根用 **kview**; `kdialog` 在 1.3 测, 不作壳根 |
| `pos` | 写死非负 `{x,y,w,h}`; 避免 `AutoRect` 依赖 `kgui.wh()` |
| 工作目录 | `paths.case_dir(doc_id)` (本章无镜像 IO 也可不用) |
| 非本主题 | **禁止** 开 SDL 窗、加载字库/图片、测绘制像素 (主题即 wlua 除外) |
| 清理 | 条末可 `klbui.clear` / `kgui.clear` (异步); 单条进程退出即可 |

## UI 用例 (wlua)

单条须看界面的条走本框架, **不是** 命令行断言. 逻辑条 (parse/set/get) 仍用 `common.pass` / `klua`.

| 项 | 约定 |
|----|------|
| 模块 | `lua_test.klbui.ui` (`ui.run` / `ui.ensure_host`); 或 `common.run_ui` |
| registry | `ui=true`; list 标 `[UI]`; 建议 `batch_ok=false` (`[SINGLE_ONLY]`) |
| 单条 | 无 `wsdl` 时 `os.execute` 同目录 `wlua.exe` (或 `wlua`) 再跑 `test.lua <id>` |
| 批量 | worker 无 SDL; `ensure_host` → skip `need wlua (batch)` |
| 页面 | 元素只写在页面文件 (`page_tmpl`); `ui.run` 只开窗 / relocation / parse; **禁止** 外部注入或改 dialog child |
| 配置 | `lua_test.klbui.pref`: `kenv.pref_path("klua","lua_test")` 下 `klbui.json` (`resolution`, `language`, `font_size`, `font_face`, `appearance`); `ui.run` 开窗前读取; 1.1.1 可改; 外观见 `theme.lua` |
| 资源 | `demores/font` `images` `language` (可用 `opts.load_*=false` 关) |
| type | 壳根为 `kview`; `kdialog` 在 1.3 测, 不作 1.1 壳根 |

用例 `run()` 主路径:

```lua
local common = require("lua_test.klbui.common")

function M.run(...)
	common.run_ui({
		title = "1.x.x example",
		page = {
			dialog = {
				["type"] = "kview",
				["pos"] = {0, 0, -1, -1},
				["child"] = {},
			},
			commands = {},
			css = {},
		},
	})
end
```

关窗结束.

## 节索引与用例

| 节 | 文档 | 条数 | 已实现 |
|----|------|------|--------|
| 1.1 | [klbui_custom.md](klbui_custom.md) | 3 | 3 |
| 1.2 | [klbui_script.md](klbui_script.md) | 9 | 9 |
| 1.3 | [klbui_shell.md](klbui_shell.md) | 8 | 8 |
| 1.4 | [klbui_basic.md](klbui_basic.md) | 22 | 1 (`1.4.3`) |
| 1.5 | [klbui_composite.md](klbui_composite.md) | 7 | — (lua 桩 + 文档桩) |

新增: 先写本节 `### x.y.z` 条文 → `klua_run/lua_test/`klbui/` → `registry_ch1.lua` → 更新本页与根 [readme.md](../readme.md) § 已实现.

### 已实现

| doc_id | 语义 id (主) | 文档 |
|--------|--------------|------|
| `1.1.1` | `klbui.custom.chrome` | [klbui_custom.md](klbui_custom.md) |
| `1.1.2` | `klbui.custom.widgets` | [klbui_custom.md](klbui_custom.md) |
| `1.1.3` | `klbui.custom.types` | [klbui_custom.md](klbui_custom.md) |
| `1.2.1` | `klbui.require` | [klbui_script.md](klbui_script.md) |
| `1.2.2` | `klbui.parse.kview` | [klbui_script.md](klbui_script.md) |
| `1.2.3` | `klbui.parse.child` | [klbui_script.md](klbui_script.md) |
| `1.2.4` | `klbui.select.name` | [klbui_script.md](klbui_script.md) |
| `1.2.5` | `klbui.select.type` | [klbui_script.md](klbui_script.md) |
| `1.2.6` | `klbui.select.id` | [klbui_script.md](klbui_script.md) |
| `1.2.7` | `klbui.css.style` | [klbui_script.md](klbui_script.md) |
| `1.2.8` | `klbui.css.type` | [klbui_script.md](klbui_script.md) |
| `1.2.9` | `klbui.has_global_css` | [klbui_script.md](klbui_script.md) |
| `1.3.1` | `klbui.kdialog` | [klbui_shell.md](klbui_shell.md) |
| `1.3.2` | `klbui.kview` | [klbui_shell.md](klbui_shell.md) |
| `1.3.3` | `klbui.ktab` | [klbui_shell.md](klbui_shell.md) |
| `1.3.4` | `klbui.kmenu` | [klbui_shell.md](klbui_shell.md) |
| `1.3.5` | `klbui.kdiv` | [klbui_shell.md](klbui_shell.md) |
| `1.3.6` | `klbui.modal` | [klbui_shell.md](klbui_shell.md) |
| `1.3.7` | `klbui.popup` | [klbui_shell.md](klbui_shell.md) |
| `1.3.8` | `klbui.messagebox` | [klbui_shell.md](klbui_shell.md) |
| `1.4.3` | `klbui.kpicture` | [klbui_basic.md](klbui_basic.md) |

## 新增用例

1. 在对应节 md 增 `### 1.x.y` (只追加; 1.3–1.5 已占节).
2. `klua_run/lua_test/`klbui/<节>/ch1_s{M}_{z}.lua` 实现 `run(...)`.
3. `registry_ch1.lua` 登记 `doc_id` 与 `ids`.
4. 更新本页与根 [readme.md](../readme.md) § 已实现用例.
