## 声明式 UI

> **require**: `klbcore.klbui` | 代码: `bin/klbcore/klbui/` | C 绑定 **`kgui`** 见 [klua/kgui.md](../klua/kgui.md)
> **文档样板**: Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

Lua table 描述界面 → `parse` → **`kgui`** → **klbgui**. 控件/CSS 约定见 [css/](css/); 架构 **klbcore-design** § klbui.

### 导出 API

#### 解析 / 选择器

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `parse(dialog [, commands [, css]])` | dialog table; 可选 commands/css | — | 解析并创建窗口树、绑定事件 |
| `update_css(dialog [, css])` | dialog; 可选 css | — | 更新已解析 dialog 的 CSS |
| `select(dialog [, multi])` | dialog; 可选多选 | function(s) | 返回 jQuery 式选择器函数 |

#### CSS（须在控件创建前设置 default/global）

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `default_css(...)` | 键值或 table | 多返回值 / `klbui` | 默认 CSS; 见 [css/klbui_css.md](css/klbui_css.md) |
| `global_css(t, ...)` | 控件类型; 键值或 table | 多返回值 / 未识别属性 / `klbui` | 某类型全局 CSS |
| `has_global_css(t)` | 控件类型 | boolean | 是否支持全局 CSS |
| `shwnd_css(path, ...)` | 共享窗口路径; 键值或 table | 多返回值 / `klbui` | 共享窗口 CSS |

#### 窗口 / modal / popup / messagebox

多数写操作返回 `rc` (`0` 成功). 与 [kgui](../klua/kgui.md) 同名 API 行为一致.

| 函数 | 说明 |
|------|------|
| `modal(path)` / `modal_end([all [, path]])` / `modal_num()` | 模态对话框 |
| `popup(path)` / `popup_end([all])` / `popup_num()` | 弹出菜单 |
| `messagebox(path)` / `messagebox_end()` / `messagebox_num()` | 消息框 |
| `show(path, show)` / `move(path, x, y)` / `resize(path, w, h)` | 显隐与布局 |
| `wndpos(path [, is_in_canvas])` | `{x,y,w,h}` 或 `{}` |
| `suggestw(path)` / `suggesth(path)` | 建议宽高 |
| `get_wnd(path)` | **wnder** 封装对象 (见下) |
| `clear([func])` | 异步清理全部 UI 与图片资源 |

#### 资源 / 画布 / 消息

| 函数 | 说明 |
|------|------|
| `using_cpp()` | 启用 C++ 扩展控件 |
| `is_multi_canvas_layer()` | modal/popup/msgbox 是否分画布 |
| `load_image(key, path)` | 加载图片资源 |
| `clear_msg()` | 清空消息队列 |
| `refresh()` / `update_tip([tip])` | 刷新 / 全局 tip |

#### 时间 / 图层

| 函数 | 说明 |
|------|------|
| `focusdelay(tc)` | 聚焦延时 (ms); 影响 tip |
| `tick_count()` | GUI 伪滴答 (ms) |
| `ticker_interval(interval)` | 内部定时器间隔 (ms) |
| `bind_udatalayer([path])` / `move_udatalayer` / `show_udatalayer` | 用户图层 |
| `bind_waitlayer([path])` / `move_waitlayer` / `wait([on])` | 等待图层 |
| `redraw_full_event([is_full])` | 完整绘制事件流程 |

#### 屏幕 / 协程

| 函数 | 说明 |
|------|------|
| `width()` / `height()` | 主画布宽高 (缓存) |
| `co_sync(func)` | 包装为 GUI 阻塞式协程回调 |

#### wnder (`get_wnd` 返回)

| 方法 | 说明 |
|------|------|
| `style(...)` / `show` / `hide` | 样式与显隐 |
| `tip` / `tip_dynamic` / `tip_update` | tip |
| `move` / `resize` / `refresh` | 布局与刷新 |
| `set(...)` / `get(...)` | 键值读写 |
| `tick_count()` | 窗口滴答 |

### 伪代码

源码: `bin/klbcore/klbui/init.lua`; wnder: `bin/klbcore/klbui/wnder.lua`

```lua
--[[
-- Copyright (c) 2022, GNU LESSER GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file   init.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  klbui init.lua
--   \n require("klbcore.klbui")
--   \n 源码: bin/klbcore/klbui/init.lua
-- @note 参考 html5 / CSS3 / jQuery 隐喻; C 绑定 kgui
-- @version 0.1
--]]

local klbui = {}


-- @brief 启用 C++ 扩展控件
-- @return 无
klbui.using_cpp = function ()
	return
end


-- @brief 解析对话框/命令, 并完成 gui 窗口树初始创建
-- @param [in] dialog[table]		对话框描述 (type/pos/name/child…)
-- @param [in] commands[table]		[可选] 命令响应集合
-- @param [in] css[table]			[可选] CSS 描述 table
-- @return 无
klbui.parse = function (dialog, commands, css)
	return
end


-- @brief 更新对话框 CSS
-- @param [in] dialog[table]		对话框描述 table
-- @param [in] css[table]			[可选] CSS 描述 table
-- @return 无
klbui.update_css = function (dialog, css)
	return
end


-- @brief 选择器 (jQuery 式)
-- @param [in] dialog[table]		与 parse 参数 1 一致
-- @param [in] multi[boolean]		[可选] true 多选; false 单选; 默认 false
-- @return [function] function(s) → table
-- @note s 规则: '*' 全部; 'name' 按 name; '#id' 按 id; '.class' 按 class; ':kbutton' 按 type
klbui.select = function (dialog, multi)
	return function (s)
		return {}
	end
end


-- @brief 设置/获取默认全局 CSS 参数
-- @param [in] [任意]...			键值或 table
-- @return [任意]... / [table] klbui (链式)
-- @note 设置须在控件创建之前
klbui.default_css = function (...)
	return klbui
end


-- @brief 设置/获取某控件类型全局 CSS
-- @param [in] t[string]			控件类型, eg. 'kbutton'
-- @param [in] [任意]...			键值或 table
-- @return [任意]... / [table] 未识别属性 / klbui
klbui.global_css = function (t, ...)
	return klbui
end


-- @brief 某控件类型是否支持全局 CSS
-- @param [in] t[string]			控件类型
-- @return [boolean] true 支持
klbui.has_global_css = function (t)
	return false
end


-- @brief 设置/获取共享窗口 CSS (share window css)
-- @param [in] path[string]			共享路径, eg. '/klbui/messagebox'
-- @param [in] [任意]...			键值或 table
-- @return [任意]... / [table] klbui
klbui.shwnd_css = function (path, ...)
	return klbui
end


-- @brief 是否为多图层画布模式
-- @return [boolean] true modal/popup/msgbox 分画布; false 共享主画布
klbui.is_multi_canvas_layer = function ()
	return false
end


-- @brief 加载资源图片
-- @param [in] key[string]			关键字
-- @param [in] path[string]			图片路径
-- @return [number(int)] 0 成功; 非 0 失败
klbui.load_image = function (key, path)
	return 0
end


-- @brief 清空消息事件
-- @return 无
klbui.clear_msg = function ()
	return
end


-- @brief 获取窗口操作接口 (wnder 封装)
-- @param [in] path[string]			窗口虚拟路径, eg. '/home'
-- @return [table] wnder 对象; 失败时方法 noop
klbui.get_wnd = function (path)
	return {}
end


-- @brief 模态显示窗口
-- @param [in] path[string]			窗口虚拟路径
-- @return [number(int)] 0 成功; 非 0 失败
klbui.modal = function (path)
	return 0
end


-- @brief 结束 modal 对话框
-- @param [in] all[boolean]			[可选] 是否关闭全部; 默认 true
-- @param [in] path[string]			[可选] 窗口路径
-- @return [number(int)] 0 成功; 非 0 失败
klbui.modal_end = function (all, path)
	return 0
end


-- @brief 获取 modal 窗口数
-- @return [number(int)] 数量
klbui.modal_num = function ()
	return 0
end


-- @brief 弹出窗口 (popup 菜单)
-- @param [in] path[string]			窗口虚拟路径
-- @return [number(int)] 0 成功; 非 0 失败
klbui.popup = function (path)
	return 0
end


-- @brief 结束 popup 菜单
-- @param [in] all[boolean]			[可选] 是否关闭全部; 默认 true
-- @return [number(int)] 0 成功; 非 0 失败
klbui.popup_end = function (all)
	return 0
end


-- @brief 获取 popup 窗口数
-- @return [number(int)] 数量
klbui.popup_num = function ()
	return 0
end


-- @brief 消息框
-- @param [in] path[string]			窗口虚拟路径
-- @return [number(int)] 0 成功; 非 0 失败
klbui.messagebox = function (path)
	return 0
end


-- @brief 关闭消息框
-- @return 无
klbui.messagebox_end = function ()
	return
end


-- @brief 获取 messagebox 窗口数
-- @return [number(int)] 数量
klbui.messagebox_num = function ()
	return 0
end


-- @brief 显示或隐藏窗口
-- @param [in] path[string]			窗口路径
-- @param [in] show[boolean]		true 显示; false 隐藏
-- @return [number(int)] 0 成功; 非 0 失败
klbui.show = function (path, show)
	return 0
end


-- @brief 移动窗口位置 (相对父窗口)
-- @param [in] path[string]			窗口路径
-- @param [in] x[number(int)]		x 坐标
-- @param [in] y[number(int)]		y 坐标
-- @return [number(int)] 0 成功; 非 0 失败
klbui.move = function (path, x, y)
	return 0
end


-- @brief 修改窗口大小
-- @param [in] path[string]			窗口路径
-- @param [in] w[number(int)]		宽
-- @param [in] h[number(int)]		高
-- @return [number(int)] 0 成功; 非 0 失败
klbui.resize = function (path, w, h)
	return 0
end


-- @brief 获取窗口位置
-- @param [in] path[string]			窗口路径
-- @param [in] is_in_canvas[boolean]	[可选] 画布坐标; 默认 true
-- @return [table] {x=0,y=0,w=0,h=0}; 失败 {}
klbui.wndpos = function (path, is_in_canvas)
	return {}
end


-- @brief 获取窗口建议宽度
-- @param [in] path[string]			窗口路径
-- @return [number(int)] 宽度
klbui.suggestw = function (path)
	return 0
end


-- @brief 获取窗口建议高度
-- @param [in] path[string]			窗口路径
-- @return [number(int)] 高度
klbui.suggesth = function (path)
	return 0
end


-- @brief 标记所有窗口需要刷新
-- @return 无
klbui.refresh = function ()
	return
end


-- @brief 更新全局 tip
-- @param [in] s[string]			[可选] tip 文本; '' 清空
-- @return 无
klbui.update_tip = function (s)
	return
end


-- @brief 设置聚焦延时消息时间 (ms, 默认 600)
-- @param [in] tc[number(int)]		毫秒
-- @return 无
klbui.focusdelay = function (tc)
	return
end


-- @brief 获取 GUI 伪系统滴答 (ms)
-- @return [number(int)] 滴答数
klbui.tick_count = function ()
	return 0
end


-- @brief 设置内部控件定时器间隔 (ms, 默认 500, 最小 10)
-- @param [in] interval[number(int)]	毫秒
-- @return 无
klbui.ticker_interval = function (interval)
	return
end


-- @brief 给用户图层绑定/解绑窗口
-- @param [in] path[string]			[可选] 路径; nil 解绑
-- @return [number(int)] 0 成功; 非 0 失败
-- @note 仅支持 onload/onunload/onticker
klbui.bind_udatalayer = function (path)
	return 0
end


-- @brief 移动用户图层
-- @param [in] x[number(int)]		X 坐标
-- @param [in] y[number(int)]		Y 坐标
-- @return 无
klbui.move_udatalayer = function (x, y)
	return
end


-- @brief 显示/隐藏用户图层
-- @param [in] show[boolean]		是否显示
-- @return 无
klbui.show_udatalayer = function (show)
	return
end


-- @brief 给等待图层绑定/解绑窗口
-- @param [in] path[string]			[可选] 路径; nil 解绑
-- @return [number(int)] 0 成功; 非 0 失败
klbui.bind_waitlayer = function (path)
	return 0
end


-- @brief 移动等待图层
-- @param [in] x[number(int)]		X 坐标
-- @param [in] y[number(int)]		Y 坐标
-- @return 无
klbui.move_waitlayer = function (x, y)
	return
end


-- @brief 开启/关闭 UI 等待 (丢弃外设事件, 定时器不受影响)
-- @param [in] is_wait[boolean]		是否等待
-- @return 无
klbui.wait = function (is_wait)
	return
end


-- @brief 设置/获取是否完整绘制事件流程
-- @param [in] is_full[boolean]		[可选] 设置; 无参则获取
-- @return [boolean] 是否完整绘制
klbui.redraw_full_event = function (is_full)
	return true
end


-- @brief 获取主显示宽度
-- @return [number(int)] 宽
klbui.width = function ()
	return 0
end


-- @brief 获取主显示高度
-- @return [number(int)] 高
klbui.height = function ()
	return 0
end


-- @brief 异步清理所有 UI 与图片资源
-- @param [in] func[function]			[可选] 清理完成回调
-- @return [number(int)] 0 成功; 非 0 失败
klbui.clear = function (func)
	return 0
end


-- @brief 协程同步: GUI 阻塞至 func 执行完毕
-- @param [in] func[function]			协程内执行的函数
-- @return [function] 事件回调包装函数
-- @note 回调签名 (x1, y1, x2, y2, lparam, wparam, b1, b2, b3)
klbui.co_sync = function (func)
	return function (...)
		return
	end
end


return klbui
```

**wnder** (`get_wnd` 返回):

```lua
--[[
-- @file   wnder.lua
-- @brief  单窗口 OOP 封装 (kgui.get_kwnd 之上)
--]]

local wnder = {}


-- @brief 设置/获取样式
-- @param [in] [任意]...			键值
-- @return [任意]... / [number(int)] 0
function wnder:style(...)
	return 0
end


-- @brief 设置/获取显示状态
-- @param [in] is_show[boolean]		[可选]
-- @return [boolean] 是否显示
function wnder:show(is_show)
	return false
end


-- @brief 设置/获取隐藏状态
function wnder:hide(is_hide)
	return
end


-- @brief 设置/获取静态 tip
-- @return [string] tip 文本
function wnder:tip(s)
	return ''
end


-- @brief 设置/获取动态 tip
function wnder:tip_dynamic(s)
	return ''
end


-- @brief 更新 tip
function wnder:tip_update()
	return
end


-- @brief 相对父窗口移动
function wnder:move(x, y)
	return
end


-- @brief 重新设置大小
function wnder:resize(w, h)
	return
end


-- @brief 刷新本窗口
function wnder:refresh()
	return
end


-- @brief 设置控件数据
-- @return [number(int)] 0 成功
function wnder:set(...)
	return 0
end


-- @brief 获取控件数据
-- @return [任意]...
function wnder:get(...)
	return {}
end


-- @brief 本窗口 GUI 滴答 (ms)
-- @return [number(int)]
function wnder:tick_count()
	return 0
end


-- @brief 由 kgui.get_kwnd 结果新建 wnder
wnder.new = function (kwnd)
	return {}
end


return wnder
```

子模块 (按需 `require`):

| require | 文件 | 职责 |
|---------|------|------|
| `klbcore.klbui.parser` | `parser.lua` | table → 控件树 |
| `klbcore.klbui.csser` | `csser.lua` | CSS 解析 |
| `klbcore.klbui.selector` | `selector.lua` | 选择器实现 |
| `klbcore.klbui.event` | `event.lua` | 事件字符串 ↔ C 码 |
| `klbcore.klbui.wnder` | `wnder.lua` | 单窗 OOP 封装 |
| `klbcore.klbui.widgets.messagebox` | `widgets/messagebox.lua` | 预制 messagebox |

### 示例

```lua
local klbui = require("klbcore.klbui")

local dialog = {
	['type'] = 'kdialog',
	['pos'] = { 0, 0, 640, 480 },
	['title'] = 'demo',
	['name'] = 'home',
	['child'] = {
		{
			['type'] = 'kbutton',
			['pos'] = { 10, 10, 120, 32 },
			['title'] = 'OK',
			['name'] = 'btn_ok',
		},
	},
}

local commands = {
	['btn_ok'] = {
		['click'] = function (x1, y1, x2, y2, lparam, wparam, b1, b2, b3)
			print('clicked')
			return 0
		end,
	},
}

local css = {
	['type'] = {
		['kbutton'] = { ['color'] = { 255, 220, 220, 220 } },
	},
}

klbui.parse(dialog, commands, css)

-- 选择器
local jq = klbui.select(dialog)
local btns = jq(':kbutton')

-- modal
klbui.modal('/home')
```

### 注意

- **dialog 三件套**: `dialog` + `commands` + `css`; `path` 可省略 (内部 `krand` 生成唯一路径)
- **commands 优先级**: `_commands` > 外部 `commands` > dialog 内嵌 `commands`
- **事件名** 为字符串 (`click`、`load`…), 非 C 枚举; 回调返回 `-1` 终止传播
- **GUI 仅主 env**; worker 线程禁止直接调用 — 见 **klbcore-design** § kthread
- 协程用 **`kco`**, 勿用内置 `coroutine` — [klua/guide/coroutine.md](../klua/guide/coroutine.md)
- C 侧控件注册 / `kgui.append` 细节 → [klua/kgui.md](../klua/kgui.md)、**klb-gui-design**
