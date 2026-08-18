## GUI 绑定

> **require**: `kgui` | 代码: `klb/src_c/klua/klua_base/klua_kgui.c` | C 侧 **klbgui** 见 `klb/inc/klbgui/`
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

### 导出 API

多数写操作返回 `rc` (`0` 成功, 非 `0` 为错误码). `set_*` / `get_*` / `set` / `get` 通过 `klua_seri` 传键值对 (`...`).

#### 初始化 / CSS

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `using_cpp()` | — | — | 启用 C++ `CGui` 路径; env `_KLUA_EX_GUI_` / `_CKLUA_EX_GUI_` |
| `set_default_css(...)` | CSS 键值 | `rc` | 默认 CSS; **须在控件创建前** |
| `get_default_css(...)` | 查询键 | 多返回值 | 读取默认 CSS |
| `set_global_css(t, ...)` | `t` 控件类型; CSS 键值 | `rc` | 某类型全局 CSS |
| `get_global_css(t, ...)` | `t`; 查询键 | 多返回值 | 读取全局 CSS |
| `has_global_css(t)` | 控件类型 | boolean | 是否已有全局 CSS |
| `set_shwnd_css(path, ...)` | 共享窗口路径; CSS 键值 | `rc` | 共享窗口 CSS |
| `get_shwnd_css(path, ...)` | 路径; 查询键 | 多返回值 | 读取共享窗口 CSS |

#### 画布 / 资源 / 消息

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `is_multi_canvas_layer()` | — | boolean | 是否多画布图层模式 |
| `load_image(key, path)` | 关键字; 图片路径 | `rc` | 加载图片资源 |
| `clear_msg()` | — | — | 清空消息事件队列 |
| `get_kwnd(path)` | 窗口路径 | `kwnd` userdata / `nil` | 获取窗口操作接口 |

#### 窗口树

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `append(t, path, x, y, w, h [, style])` | 类型; 路径; 坐标与尺寸; 可选样式 | `rc` | 创建并挂载控件 |
| `remove(path)` | 路径 | `rc` | 移除窗口 |
| `clear()` | — | `rc` | 清理所有窗口 (异步) |
| `bind_command(path, func)` | 路径; Lua 回调 | `rc` | 绑定消息/事件处理 |
| `call_control_and_command(path, msg [, x1, y1, x2, y2 [, lparam [, wparam]]])` | 路径; 事件; 可选坐标与参数 | `rc` | 触发控件事件 |
| `set(path, ...)` | 路径; 键值 | `rc` | 设置样式/显示/状态等 |
| `get(path, ...)` | 路径; 查询键 | 多返回值 | 读取控件数据 |

#### modal / popup / messagebox

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `modal(path)` | 路径 | `rc` | modal 显示对话框 |
| `modal_end([all [, path]])` | 可选全部结束; 可选路径 | `rc` | 结束 modal |
| `modal_num()` | — | integer | 当前 modal 数量 |
| `popup(path)` | 路径 | `rc` | popup 显示菜单 |
| `popup_end([all])` | 可选全部结束 | `rc` | 结束 popup |
| `popup_num()` | — | integer | 当前 popup 数量 |
| `messagebox(path)` | 路径 | `rc` | 显示 messagebox |
| `messagebox_end()` | — | `rc` | 关闭 messagebox |
| `messagebox_num()` | — | integer | 当前 messagebox 数量 |

#### 布局 / 刷新

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `show(path, show)` | 路径; boolean | `rc` | 显示/隐藏 |
| `move(path, x, y)` | 路径; 相对父窗口坐标 | `rc` | 移动 |
| `resize(path, w, h)` | 路径; 宽高 | `rc` | 改大小 |
| `wndpos(path [, is_in_canvas])` | 路径; 默认 `true` 相对画布 | table `{x,y,w,h}` | 获取位置; 失败为空表 |
| `suggestw(path)` | 路径 | integer | 建议宽度 |
| `suggesth(path)` | 路径 | integer | 建议高度 |
| `focusdelay(tc)` | 毫秒 | — | 聚焦延时消息时间 |
| `refresh()` | — | — | 标记刷新 (由 UI 框架决定时机) |
| `update_tip([tip])` | 可选文本 | — | 更新全局 tip |
| `wh()` | — | `w, h` | 主画布(屏幕)宽高 |

#### 事件解析

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `to_event(e)` | 复合事件值 | integer | 去除特殊标记位 |
| `b1_event(e)` / `b2_event(e)` / `b3_event(e)` | 事件值 | boolean | 是否含对应标记位 |

#### 定时 / 图层

| 函数 | 参数 | 返回 | 说明 |
|------|------|------|------|
| `tick_count()` | — | integer | GUI 伪系统滴答 (ms); **勿与真实系统滴答混用** |
| `ticker_interval(interval)` | 毫秒 | — | 内部控件定时器间隔 |
| `bind_udatalayer([path])` | 路径绑定; 无参解绑 | `rc` | 用户图层绑定窗口 |
| `move_udatalayer([x [, y]])` | 坐标 | — | 移动用户图层 |
| `show_udatalayer([show])` | 默认 `false` | — | 显示/隐藏用户图层 |
| `bind_waitlayer([path])` | 路径绑定; 无参解绑 | `rc` | 等待图层绑定窗口 |
| `move_waitlayer([x [, y]])` | 坐标 | — | 移动等待图层 |
| `wait([on])` | 默认 `false` | — | 开启/关闭等待 |
| `redraw_full_event([is_full])` | 无参获取; boolean 设置 | boolean | 是否完整绘制 |

#### `kwnd` userdata (`get_kwnd` 成功时)

| 方法 | 说明 |
|------|------|
| `style(...)` | 设置/获取样式 |
| `show()` / `hide()` | 设置/获取显隐 |
| `tip(...)` / `tip_dynamic(...)` / `tip_update()` | 静态/动态 tip |
| `move(x, y)` / `resize(w, h)` | 相对父窗口移动/改大小 |
| `refresh()` | 刷新本窗口 |
| `set(...)` / `get(...)` | 设置/获取控件数据 |
| `tick_count()` | 本窗口 GUI 滴答 (ms) |

### 伪代码

```lua
--[[
-- Copyright(c) 2020, LGPL All Rights Reserved
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  GUI接口
--   \n require("kgui")
--   \n C导出文件: ./klb/src_c/klua/klua_base/klua_kgui.c
-- @version 0.1
--]]

local kgui = {}


-- @brief 启用 C++ 扩展控件: 即 支持使用 CPP 相关的GUI接口
kgui.using_cpp = function ()
	return
end



-- @brief 设置默认CSS参数
-- @param [in] [任意]...			设置参数
-- @return [number(int)] 	0.成功; 非0.失败
-- @note 注意必须在控件创建之前, 才能在初始化控件时生效;
kgui.set_default_css = function (...)
	return 0
end


-- @brief 获取默认CSS参数
-- @param [in] [任意]...			获取参数
-- @return [任意]...				值
kgui.get_default_css = function (...)
	return ...
end


-- @brief 设置控件全局CSS参数
-- @param [in] t[string]			窗口/控件类型: eg. "kbutton"
-- @param [in] [任意]...			设置参数
-- @return [number(int)] 	0.成功; 非0.失败
kgui.set_global_css = function (t, ...)
	return 0
end


-- @brief 获取控件全局CSS参数
-- @param [in] t[string]			窗口/控件类型: eg. "kbutton"
-- @param [in] [任意]...			获取参数
-- @return [任意]...				值
kgui.get_global_css = function (t, ...)
	return ...
end


-- @brief 获取控件全局CSS参数
-- @param [in] t[string]			窗口/控件类型: eg. "kbutton"
-- @return [boolean] 	true.有全局CSS; false.无全局CSS
kgui.has_global_css = function (t)
	return true
end


-- @brief 设置共享窗口CSS
-- @param [in] path[string]			共享窗口路径: eg. "/klbui/combomenu"
-- @param [in] [任意]...			设置参数
-- @return [number(int)] 	0.成功; 非0.失败
kgui.set_shwnd_css = function (path, ...)
	return 0
end


-- @brief 获取共享窗口CSS
-- @param [in] path[string]			共享窗口路径: eg. "/klbui/combomenu"
-- @param [in] [任意]...			获取参数
-- @return [任意]...				值
kgui.get_shwnd_css = function (path, ...)
	return ...
end


-- @brief 是否多画布图层模式 (modal/popup/messagebox 分画布)
-- @return [boolean] true 多画布; false 单画布
kgui.is_multi_canvas_layer = function ()
	return false
end


-- @brief 加载图片
-- @param [in] key[string]			关键字
-- @param [in] path[string]			图片路径
-- @return [number(int)] 	0.成功; 非0.失败
kgui.load_image = function (key, path)
	return 0
end


-- @brief 清空消息事件队列
-- @return 无
kgui.clear_msg = function ()
	return
end


-- @brief 获取窗口 kwnd 操作接口
-- @param [in] path[string]			窗口路径: eg. "/home/btn1"
-- @return [userdata] kwnd; 失败 nil
-- @note 成功时返回 kwnd userdata, 方法见下节 **kwnd 对象**
kgui.get_kwnd = function (path)
	return nil
end


-- @brief 添加窗口
-- @param [in] t[string]			窗口/控件类型: eg. "kbutton"
-- @param [in] path[string]			窗口路径名: eg. "/home/btn1"
-- @param [in] x[number(int)]		相对父窗口x坐标
-- @param [in] y[number(int)]		相对父窗口y坐标
-- @param [in] w[number(int)]		宽
-- @param [in] h[number(int)]		高
-- @param [in] style[number(int)]	[可选] 样式, 默认 0
-- @return [number(int)] 	0.成功; 非0.失败
kgui.append = function (t, path, x, y, w, h, style)
	return 0
end


-- @brief 移除窗口
-- @return 无
kgui.remove = function (path)
	return
end


-- @brief 清理所有窗口
-- @return int 0.成功; 非0.失败(错误码)
kgui.clear = function ()
	return 0
end

-- @brief 绑定消息(事件)处理函数
-- @param [in] path[string]			窗口路径名: eg. "/home/btn1"
-- @param [in] func[function]		lua函数
-- @return [number(int)] 	0.成功; 非0.失败
kgui.bind_command = function (path, func)
	return 0
end


-- @brief 调用(触发)控件(窗口)某个事件
kgui.call_control_and_command = function (path, msg, x1, x2, y1, y2, lparam, wparam)
	return 0
end


-- @brief 向窗口(控件)设置数据: 样式\显示\状态等等
kgui.set = function (path, ...)
	return 0
end


-- @brief 向窗口(控件)获取数据: 样式\显示\状态等等
kgui.get = function (path, ...)
	return ...
end


-- @brief 以modal方式的显示一个对话框
kgui.modal = function (path)
	return 0
end


-- @brief 结束一个modal方式的对话框
kgui.modal_end = function (all, path)
	return 0
end

-- @brief 以popup方式显示一个菜单
kgui.popup = function (path)
	return 0
end


-- @brief 结束一个popup方式的菜单
kgui.popup_end = function (all)
	return 0
end


-- @brief 以messagebox方式弹出一个消息提示框
kgui.messagebox = function (path)
	return 0
end


-- @brief 关闭messagebox消息提示框
kgui.messagebox_end = function ()
	return 0
end


-- @brief 当前 modal 窗口数量
-- @return [number(int)] 数量
kgui.modal_num = function ()
	return 0
end


-- @brief 当前 popup 窗口数量
-- @return [number(int)] 数量
kgui.popup_num = function ()
	return 0
end


-- @brief 当前 messagebox 窗口数量
-- @return [number(int)] 数量
kgui.messagebox_num = function ()
	return 0
end


-- @brief 显示或隐藏窗口
kgui.show = function (path, show)
	return 0
end


-- @brief 移动窗口位置
kgui.move = function (path, x, y)
	return 0
end


-- @brief 修改窗口大小
kgui.resize = function (path, w, h)
	return 0
end


-- @brief 获取窗口位置
kgui.wndpos = function (path, is_in_canvas)
	return { x = 0, y = 0, w = 32, h = 32 }
end


-- @brief 获取窗口(控件)建议宽度
kgui.suggestw = function (path)
	return 100
end


-- @brief 获取窗口(控件)建议高度
kgui.suggesth = function (path)
	return 100
end


-- @brief 窗口刷新(仅标记, UI框架决定刷新时机)
kgui.refresh = function ()
	return
end


-- @brief 更新全局 tip
-- @param [in] tip[string]			[可选] tip 文本; 省略则刷新当前 tip
-- @return 无
kgui.update_tip = function (tip)
	return
end


-- @brief 获取主窗口(画布)的宽高
kgui.wh = function ()
	return 1280, 720
end


-- @brief 获取不含特殊标记的事件类型
kgui.to_event = function (e)
	return 0x406
end


-- @brief 获取是否含有 b1/b2/b3 比特位标记
kgui.b1_event = function (e)
	return false
end

kgui.b2_event = function (e)
	return false
end

kgui.b3_event = function (e)
	return false
end


-- @brief 设置 聚焦延时消息 的时间(单位毫秒ms)
kgui.focusdelay = function (tc)
	return
end


-- @brief 获取系统当前 系统滴答数
kgui.tick_count = function ()
	return 1000
end


-- @brief 设置 内部控件定时器运行间隔 (单位毫秒ms)
kgui.ticker_interval = function (interval)
	return
end


-- @brief 绑定/解绑用户图层对应窗口
-- @param [in] path[string]			[可选] 窗口路径; 省略则解绑
-- @return [number(int)] 0 成功; 非 0 失败
kgui.bind_udatalayer = function (path)
	return 0
end


-- @brief 移动用户图层
-- @param [in] x[number(int)]		[可选] x 坐标, 默认 0
-- @param [in] y[number(int)]		[可选] y 坐标, 默认 0
-- @return 无
kgui.move_udatalayer = function (x, y)
	return
end


-- @brief 显示/隐藏用户图层
-- @param [in] show[boolean]		[可选] 默认 false
-- @return 无
kgui.show_udatalayer = function (show)
	return
end


-- @brief 绑定/解绑等待图层对应窗口
-- @param [in] path[string]			[可选] 窗口路径; 省略则解绑
-- @return [number(int)] 0 成功; 非 0 失败
kgui.bind_waitlayer = function (path)
	return 0
end


-- @brief 移动等待图层
-- @param [in] x[number(int)]		[可选] x 坐标, 默认 0
-- @param [in] y[number(int)]		[可选] y 坐标, 默认 0
-- @return 无
kgui.move_waitlayer = function (x, y)
	return
end


-- @brief 开启/关闭等待图层
-- @param [in] on[boolean]			[可选] 默认 false
-- @return 无
kgui.wait = function (on)
	return
end


-- @brief 设置/获取是否完整绘制事件流程
-- @param [in] is_full[boolean]		[可选] 设置时传入; 省略则只读当前值
-- @return [boolean] 是否完整绘制
kgui.redraw_full_event = function (is_full)
	return false
end


return kgui
```

**kwnd 对象** (`get_kwnd` 成功时, userdata):

```lua
-- @brief 设置/获取窗口样式
-- @param [in] style[number(int)]	[可选] 设置时传入; 省略则读取
-- @return [number(int)] 样式值 (读取时)
function kwnd:style(style)
	return 0
end


-- @brief 设置/获取显示状态
-- @param [in] show[boolean]		[可选] 设置时传入; 省略则读取
-- @return [boolean] 是否显示 (读取时)
function kwnd:show(show)
	return false
end


-- @brief 设置/获取隐藏状态
-- @param [in] hide[boolean]		[可选] 设置时传入; 省略则读取
-- @return [boolean] 是否隐藏 (读取时)
function kwnd:hide(hide)
	return false
end


-- @brief 设置/获取静态 tip
-- @param [in] tip[string]			[可选] 设置时传入; 省略则读取
-- @return [string] tip 文本 (读取时; 无则为 "")
function kwnd:tip(tip)
	return ""
end


-- @brief 设置/获取动态 tip
-- @param [in] tip[string]			[可选] 设置时传入; 省略则读取
-- @return [string] tip 文本 (读取时; 无则为 "")
function kwnd:tip_dynamic(tip)
	return ""
end


-- @brief 刷新 tip 显示
-- @return 无
function kwnd:tip_update()
	return
end


-- @brief 相对父窗口移动
-- @param [in] x[number(int)]		[可选] x, 默认 0
-- @param [in] y[number(int)]		[可选] y, 默认 0
-- @return 无
function kwnd:move(x, y)
	return
end


-- @brief 改大小
-- @param [in] w[number(int)]		宽
-- @param [in] h[number(int)]		高
-- @return 无
function kwnd:resize(w, h)
	return
end


-- @brief 刷新本窗口 (标记, 由 UI 框架决定时机)
-- @return 无
function kwnd:refresh()
	return
end


-- @brief 设置控件数据 (样式/显示/状态等)
-- @param [in] [任意]...			键值对
-- @return [number(int)] 0 成功; 非 0 失败
function kwnd:set(...)
	return 0
end


-- @brief 获取控件数据
-- @param [in] [任意]...			查询键
-- @return [任意]...				多返回值
function kwnd:get(...)
	return ...
end


-- @brief 本窗口 GUI 伪系统滴答 (ms)
-- @return [number(int)]
function kwnd:tick_count()
	return 0
end
```

对照桩: `klb/bin/klbcore/help/k/kgui.lua` (模块级 API 与本文一致; `@note` 长注释见 help 桩).

### 示例

#### 直接创建控件 (底层)

```lua
local kgui = require("kgui")

-- 全局按钮样式 (须在 append 之前)
kgui.set_global_css("kbutton", "background-color", 0xFF404040)

local rc = kgui.append("kbutton", "/home/btn_ok", 20, 20, 120, 36)
if 0 ~= rc then
    error("append failed: " .. tostring(rc))
end

kgui.set("/home/btn_ok", "title", "确定")
kgui.show("/home/btn_ok", true)

kgui.bind_command("/home/btn_ok", function (obj, msg, x1, y1, x2, y2, lparam, wparam)
    local e = kgui.to_event(msg)
    print("event", e, kgui.b1_event(msg))
    -- 按 klbui_event.h / klbcore.klbui.event 处理具体 msg
end)

kgui.refresh()
```

#### `get_kwnd` 操作单窗口

```lua
local kgui = require("kgui")

local kwnd = kgui.get_kwnd("/home/btn_ok")
if nil ~= kwnd then
    kwnd:set("title", "取消")
    kwnd:move(40, 40)
    kwnd:refresh()
end
```

#### modal / messagebox

```lua
local kgui = require("kgui")

kgui.append("kdialog", "/dlg/settings", 100, 80, 400, 300)
kgui.modal("/dlg/settings")
-- ... 用户操作 ...
kgui.modal_end(true)          -- 结束全部 modal
-- kgui.modal_end(false, "/dlg/settings")  -- 结束指定路径

kgui.messagebox("/msg/tip")
kgui.messagebox_end()
```

#### 推荐: 经 klbui 声明式解析

```lua
local klbui = require("klbcore.klbui")

local dialog = {
    ['type'] = 'kdialog',
    ['path'] = '/main',
    ['title'] = '示例',
    {
        ['type'] = 'kbutton',
        ['path'] = 'btn_ok',
        ['title'] = '确定',
        ['x'] = 20, ['y'] = 20, ['w'] = 100, ['h'] = 32,
    },
}

local commands = {
    btn_ok = function (obj, msg)
        print("btn_ok clicked", msg)
    end,
}

klbui.parse(dialog, commands)   -- 内部调用 kgui.append / set / bind_command
```

详 [klbcore/readme.md](../klbcore/readme.md) § klbui 控件与 CSS.

### 注意

- **脚本 UI** 推荐 `require("klbcore.klbui")` → `klbui.parse` 再调 **kgui**; 见 [lua/klbcore/readme.md](../klbcore/readme.md)
- 直接调 `kgui.append` / `set` / `bind_command` 为底层 C 绑定; 控件类型须已在 klbgui 注册
- `using_cpp()` 启用 C++ `CGui` 路径; env 扩展 `_KLUA_EX_GUI_` / `_CKLUA_EX_GUI_`
- 事件常量与 [klbgui extension](../../klb/klbgui/design/extension.md) / `klbui_event.h` 对应; `to_event` / `b1_event` 等解析复合 msg
- CSS 键约定: [klbui_css.md](../klbcore/css/klbui_css.md)
