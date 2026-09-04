# 第 1 章 § 1.4 简易控件 (basic)

> API: [klbui.md](../../klbcore/klbui.md), [kgui.md](../../klua/kgui.md) | 枢纽: [readme.md](readme.md)
> 源码: `klua_run/lua_test/`klbui/basic/ch1_s4_{z}.lua` → `lua_test.klbui.basic.ch1_s4_{z}`

`1.4.3` **已实现** (UI). 其余条仍为桩: `M.run` 返回 `nil`, **待实现**, 不登记.

---

## lua 对照

| doc_id   | 语义 id               | lua `@brief`     | 文件                    |
| -------- | ------------------- | ---------------- | --------------------- |
| `1.4.1`  | `klbui.kstatic`     | kstatic stub     | `basic/ch1_s4_1.lua`  |
| `1.4.2`  | `klbui.kbutton`     | kbutton stub     | `basic/ch1_s4_2.lua`  |
| `1.4.3`  | `klbui.kpicture`    | 1.4.3 UI kpicture | `basic/ch1_s4_3.lua`  |
| `1.4.4`  | `klbui.kdemo`       | kdemo stub       | `basic/ch1_s4_4.lua`  |
| `1.4.5`  | `klbui.kline`       | kline stub       | `basic/ch1_s4_5.lua`  |
| `1.4.6`  | `klbui.kcheck`      | kcheck stub      | `basic/ch1_s4_6.lua`  |
| `1.4.7`  | `klbui.kradio`      | kradio stub      | `basic/ch1_s4_7.lua`  |
| `1.4.8`  | `klbui.kgroup`      | kgroup stub      | `basic/ch1_s4_8.lua`  |
| `1.4.9`  | `klbui.kedit`       | kedit stub       | `basic/ch1_s4_9.lua`  |
| `1.4.10` | `klbui.kpassword`   | kpassword stub   | `basic/ch1_s4_10.lua` |
| `1.4.11` | `klbui.knum`        | knum stub        | `basic/ch1_s4_11.lua` |
| `1.4.12` | `klbui.kspin`       | kspin stub       | `basic/ch1_s4_12.lua` |
| `1.4.13` | `klbui.kip`         | kip stub         | `basic/ch1_s4_13.lua` |
| `1.4.14` | `klbui.kcombo`      | kcombo stub      | `basic/ch1_s4_14.lua` |
| `1.4.15` | `klbui.kprogress`   | kprogress stub   | `basic/ch1_s4_15.lua` |
| `1.4.16` | `klbui.kslider`     | kslider stub     | `basic/ch1_s4_16.lua` |
| `1.4.17` | `klbui.kvslider`    | kvslider stub    | `basic/ch1_s4_17.lua` |
| `1.4.18` | `klbui.khscrollbar` | khscrollbar stub | `basic/ch1_s4_18.lua` |
| `1.4.19` | `klbui.kvscrollbar` | kvscrollbar stub | `basic/ch1_s4_19.lua` |
| `1.4.20` | `klbui.kticker`     | kticker stub     | `basic/ch1_s4_20.lua` |
| `1.4.21` | `klbui.kanimation`  | kanimation stub  | `basic/ch1_s4_21.lua` |
| `1.4.22` | `klbui.kqrcode`     | kqrcode stub     | `basic/ch1_s4_22.lua` |

---

## 1.4 简易控件

每条: CLI = `doc_id` / 语义 id; 模块 = `lua_test.klbui.basic.ch1_s4_{z}`. 除 `1.4.3` 外状态 **待实现**; lua `M.run` 返回 `nil`.

### 1.4.1 kstatic

- `klbui.kstatic`

### 1.4.2 kbutton

- `klbui.kbutton`

### 1.4.3 kpicture

- `klbui.kpicture`

| 项 | 值 |
|----|-----|
| doc_id | `1.4.3` |
| CLI | `1.4.3` / `klbui.kpicture` |
| 模块 | `lua_test.klbui.basic.ch1_s4_3` |
| 状态 | **已实现** |
| registry | `ui=true`; `batch_ok=false` |

页面模块: 按 `page_tmpl`; 本页满窗 parse; 开窗由 `lua_test.klbui.ui.run`; 元素只写本页. 文案 `lang.str`; 词条 `demores/language/`; `apply_lang` 由 `ui.run` 调用

图 key: 无扩展名; 检索根 `demores/images/tmpimage`. 一个 `cmb_img` 合并扫 `tmpimage/png` 与 `tmpimage/bmp` (标题 `png/<type>/<stem>` / `bmp/<type>/<stem>`; key `/png/...` `/bmp/...`). 开窗默认优先 PNG stem `cover`, 左右 `pic`/`pic_png` 同 key. 左示意图: 图不大于框则 `default`(原图), 大于框则 `resize`(缩放). `plain` 不加载图.

步骤: 1. `klua test.lua 1.4.3` (无 `wsdl` 则转 `wlua`); 2. 右侧大图 `pic` 可操作; 左侧示意图 `pic_png` 与右图同一张; Get/Set/hide 及一个选图 `kcombo`; 3. 点 `Get` 读 `pic` 的 title/value/image/mode; 4. 点 `Set` 写 `pic` title/value; 5. 在 `cmb_img` 选一项, 左右同时切到该 key; 点 `Clear` 清空左右图; 6. 点 `default` / `resize` / `scale9` 分别切右侧 `pic` 的 `background-image-mode` (左图 mode 仍按框自适应); 7. 点 `Toggle hide` 显隐 `pic`; 8. 关窗

预期: 无图或 `plain` 时为色块; 非 `plain` 且对应格式文件已 load 时左右同图; 左图小则原图、大则缩放; 右侧 `default` 原图拷贝超出裁剪, `resize` 拉伸铺满, `scale9` 九宫格; `Get`/`Set` 后标签与控制台有对应字串; `cmb_img` 列出 png+bmp 格式文件, 选择后左右立刻切同一张; 模式三按钮只改右侧 `pic`; 切 hide 可见变化; 切语言后面板文案跟着变; 关窗结束

失败: 未开窗; `kpicture` 未出现或 parse 失败; combo 无条目或选择无反馈; 按钮无反馈

### 1.4.4 kdemo

- `klbui.kdemo`

### 1.4.5 kline

- `klbui.kline`

### 1.4.6 kcheck

- `klbui.kcheck`

### 1.4.7 kradio

- `klbui.kradio`

### 1.4.8 kgroup

- `klbui.kgroup`

### 1.4.9 kedit

- `klbui.kedit`

### 1.4.10 kpassword

- `klbui.kpassword`

### 1.4.11 knum

- `klbui.knum`

### 1.4.12 kspin

- `klbui.kspin`

### 1.4.13 kip

- `klbui.kip`

### 1.4.14 kcombo

- `klbui.kcombo`

### 1.4.15 kprogress

- `klbui.kprogress`

### 1.4.16 kslider

- `klbui.kslider`

### 1.4.17 kvslider

- `klbui.kvslider`

### 1.4.18 khscrollbar

- `klbui.khscrollbar`

### 1.4.19 kvscrollbar

- `klbui.kvscrollbar`

### 1.4.20 kticker

- `klbui.kticker`

### 1.4.21 kanimation

- `klbui.kanimation`

### 1.4.22 kqrcode

- `klbui.kqrcode`
