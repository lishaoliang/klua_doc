## GUI 绑定

> 代码: `klb/src_c/klua/klua_base/klua_kgui.c`, C 侧 **klbgui** 见 `klb/inc/klbgui/`

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


-- @brief 加载图片
-- @param [in] key[string]			关键字
-- @param [in] path[string]			图片路径
-- @return [number(int)] 	0.成功; 非0.失败
kgui.load_image = function (key, path)
	return 0
end

-- @brief 添加窗口
-- @param [in] t[string]			窗口/控件类型: eg. "kbutton"
-- @param [in] path[string]			窗口路径名: eg. "/home/btn1"
-- @param [in] x[number(int)]		相对父窗口x坐标
-- @param [in] y[number(int)]		相对父窗口y坐标
-- @param [in] w[number(int)]		宽
-- @param [in] h[number(int)]		高
-- @return [number(int)] 	0.成功; 非0.失败
kgui.append = function (t, path, x, y, w, h)
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


return kgui
```

完整桩代码 (含 `@note` 长注释): `klb/bin/klbcore/help/k/kgui.lua`.

### 注意

- **脚本 UI** 推荐 `require("klbcore.klbui")` → `klbui.parse` 再调 **kgui**; 见 [klbui 文档](../../klbui/readme.md)
- 直接调 `kgui.append` / `set` / `bind_command` 为底层 C 绑定; 控件类型须已在 klbgui 注册
- `using_cpp()` 启用 C++ `CGui` 路径; env 扩展 `_KLUA_EX_GUI_` / `_CKLUA_EX_GUI_`
- 事件常量与 [klbui event](../../klbui/design/extension.md) / `klbui_event.h` 对应; `to_event` / `b1_event` 等解析复合 msg
- CSS 键约定: [css/css.md](../../klbui/css/css.md)
