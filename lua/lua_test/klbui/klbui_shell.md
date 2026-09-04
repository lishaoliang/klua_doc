# 第 1 章 § 1.3 壳 / 层叠 (shell)

> API: [klbui.md](../../klbcore/klbui.md), [kgui.md](../../klua/kgui.md) | 枢纽: [readme.md](readme.md)
> 源码: `klua_run/lua_test/`klbui/shell/ch1_s3_{z}.lua` → `lua_test.klbui.shell.ch1_s3_{z}`

**1.3.1–1.3.8 已实现** (UI).

一 type 一条. 条序: `kdialog` `kview` `ktab` `kmenu` `kdiv`, 再栈 API. `ktab` 从 1.5 并入. 无 `ktable` / `kmessagebox` type. 消息框走 `klbui.messagebox` 栈 (parse 顶窗). 宿主弹出菜单见 1.4 / 1.5. 页壳根仍为 `kview`.

---

## lua 对照

| doc_id  | 语义 id            | lua `@brief`        | 文件                   |
| ------- | ---------------- | ------------------- | -------------------- |
| `1.3.1` | `klbui.kdialog`    | 1.3.1 UI kdialog    | `shell/ch1_s3_1.lua` |
| `1.3.2` | `klbui.kview`      | 1.3.2 UI kview      | `shell/ch1_s3_2.lua` |
| `1.3.3` | `klbui.ktab`       | 1.3.3 UI ktab       | `shell/ch1_s3_3.lua` |
| `1.3.4` | `klbui.kmenu`      | 1.3.4 UI kmenu      | `shell/ch1_s3_4.lua` |
| `1.3.5` | `klbui.kdiv`       | 1.3.5 UI kdiv       | `shell/ch1_s3_5.lua` |
| `1.3.6` | `klbui.modal`      | 1.3.6 UI modal      | `shell/ch1_s3_6.lua` |
| `1.3.7` | `klbui.popup`      | 1.3.7 UI popup      | `shell/ch1_s3_7.lua` |
| `1.3.8` | `klbui.messagebox` | 1.3.8 UI messagebox | `shell/ch1_s3_8.lua` |

---

## 1.3 壳 / 层叠

每条: 页面按 `page_tmpl`; 本页满窗 parse; 开窗由 `lua_test.klbui.ui.run`; 元素只写本页. 文案 `lang.str`; `apply_lang` 由 `ui.run` 调用. registry `ui=true`; `batch_ok=false`.

### 1.3.1 kdialog

- `klbui.kdialog`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.1` |
| CLI | `1.3.1` / `klbui.kdialog` |
| 模块 | `lua_test.klbui.shell.ch1_s3_1` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

页壳根 `kview`; 本条测子控件 `kdialog` (标题栏 + `title`/`value`). 体内一块 `kstatic`.

步骤: 1. `klua test.lua 1.3.1` (无 `wsdl` 则转 `wlua`); 2. 见带标题栏的 `kdialog` 与左侧 Get/Set/hide 按钮 (文案随 pref 语言); 3. 点 `Get` 读 `dlg` 的 title/value; 4. 点 `Set` 写 `dlg` title/value, 标题栏字立刻变; 5. 点 `Toggle hide` 显隐 `dlg`; 6. 关窗

预期: `kdialog` 有标题栏与边框; 体内 `kstatic` 可见; `Get`/`Set` 后标签与控制台有对应字串; `Set` 后标题栏文本立刻换成第二标题; 切 hide 可见变化; 切语言后面板与标题栏文案跟着变; 关窗结束

失败: 未开窗; `kdialog` 未出现或 parse 失败; 按钮无反馈; `Set` 后标题栏不变

---

### 1.3.2 kview

- `klbui.kview`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.2` |
| CLI | `1.3.2` / `klbui.kview` |
| 模块 | `lua_test.klbui.shell.ch1_s3_2` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

页壳根 `kview` (`page`); 本条测子 `kview` (`box`): 背景色 / `title` / hide. 体内一块 `kstatic`. `kview` 不画标题栏.

步骤: 1. `klua test.lua 1.3.2`; 2. 见色块容器与 Get/Set/hide; 3. 点 `Get` 读 `box` 的 title; 4. 点 `Set` 写 title 并换背景色; 5. 点 `Toggle hide` 显隐 `box`; 6. 关窗

预期: 子 `kview` 有底色与边框; 体内 `kstatic` 可见; `Get`/`Set` 后标签与控制台有字串; `Set` 后底色立刻变; 切 hide 可见变化; 切语言后面板文案跟着变; 关窗结束

失败: 未开窗; 子 `kview` 未出现; 按钮无反馈; `Set` 后底色不变

---

### 1.3.3 ktab

- `klbui.ktab`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.3` |
| CLI | `1.3.3` / `klbui.ktab` |
| 模块 | `lua_test.klbui.shell.ch1_s3_3` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

子控件 `ktab` 含两页 `kview` (title 作页签). 点页签切页; `Get` 读 tab title.

步骤: 1. `klua test.lua 1.3.3`; 2. 见两页签与体内文案; 3. 点第二页签切页; 4. 点 `Get`; 5. 关窗

预期: 两页签可见; 切页后对应体内 `kstatic` 可见; `Get` 后标签与控制台有字串; 关窗结束

失败: 未开窗; `ktab` 未出现或无页签; 切页无变化

---

### 1.3.4 kmenu

- `klbui.kmenu`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.4` |
| CLI | `1.3.4` / `klbui.kmenu` |
| 模块 | `lua_test.klbui.shell.ch1_s3_4` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

子控件 `kmenu`; `onload` `append` 三项. 点菜单项 `onchange` 写 lab. `Get` 读 value.

步骤: 1. `klua test.lua 1.3.4`; 2. 见三项菜单; 3. 点一项; 4. 点 `Get`; 5. 关窗

预期: 三项可见; 点击后标签与控制台有对应 value; `Get` 读到当前 value; 关窗结束

失败: 未开窗; 菜单空; 点击无 `onchange`

---

### 1.3.5 kdiv

- `klbui.kdiv`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.5` |
| CLI | `1.3.5` / `klbui.kdiv` |
| 模块 | `lua_test.klbui.shell.ch1_s3_5` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

`kdiv` 只排版不绘制. 体内两块 `kstatic` 看出布局. `Get`/`Set` title 与 index; hide 连子一起隐.

步骤: 1. `klua test.lua 1.3.5`; 2. 见两块体内文本; 3. 点 `Get` 读 title/index; 4. 点 `Set`; 5. 点 `Toggle hide`; 6. 关窗

预期: 体内两块可见; `Get`/`Set` 后标签有字串; hide 后两块一起消失; 关窗结束

失败: 未开窗; 体内不可见; 按钮无反馈

---

### 1.3.6 modal

- `klbui.modal`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.6` |
| CLI | `1.3.6` / `klbui.modal` |
| 模块 | `lua_test.klbui.shell.ch1_s3_6` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

页已由 `ui.run` `modal` 根窗. 本条再 `parse` 一层 `kdialog` (`/lua_test/s3_6`), 点 Open 叠 `klbui.modal`; 点 Close 调 `modal_end(false, path)`. 标签显示 `modal_num`.

步骤: 1. `klua test.lua 1.3.6`; 2. 点 `Open` 叠第二层对话框; 3. 点对话框 `Close` 只关第二层; 4. 关窗

预期: Open 后第二层 `kdialog` 在上, `modal_num` 增大; Close 后第二层消失, 本页仍在; 关窗结束

失败: 未开窗; Open 无第二层; Close 把本页一起关了

---

### 1.3.7 popup

- `klbui.popup`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.7` |
| CLI | `1.3.7` / `klbui.popup` |
| 模块 | `lua_test.klbui.shell.ch1_s3_7` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

`onload` `parse` 独立 `kmenu` (`/lua_test/s3_7`) 并 `append`. 点 Open 调 `klbui.popup`; 点菜单项 `onchange` 后 `popup_end`. 标签显示 `popup_num` 与 value.

步骤: 1. `klua test.lua 1.3.7`; 2. 点 `Open` 弹出菜单; 3. 点一项; 4. 关窗

预期: Open 后菜单弹出, `popup_num` 增大; 点项后菜单关, 标签有 value; 关窗结束

失败: 未开窗; Open 无菜单; 点项不关

---

### 1.3.8 messagebox

- `klbui.messagebox`

| 项 | 值 |
|----|-----|
| doc_id | `1.3.8` |
| CLI | `1.3.8` / `klbui.messagebox` |
| 模块 | `lua_test.klbui.shell.ch1_s3_8` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

无 `kmessagebox` type. Lua `messagebox(path)` 弹出 wndhash 顶窗. 本条 `parse` 一层 `kdialog` (`/lua_test/s3_8`); 点 Open 调 `klbui.messagebox`; 点确认/取消调 `messagebox_end`. 标签显示 `messagebox_num` 与 value (`ok`/`cancel`). 共享窗 `/klbui/messagebox` 仅 `shwnd_css` 写 title (C 弹出走 `messagebox_wnd`, Lua 未绑定).

步骤: 1. `klua test.lua 1.3.8`; 2. 点 `Open` 弹出对话框; 3. 点确认或取消; 4. 关窗

预期: Open 后对话框在 messagebox 层, `messagebox_num` 为 1; 点按钮后框关, 标签有 value; 关窗结束

失败: 未开窗; Open 无框; 点按钮不关
