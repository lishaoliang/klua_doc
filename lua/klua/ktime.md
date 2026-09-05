## 时间与定时器

> **require**: `ktime` | 代码: [klb/src_c/klua/klua_platform/klua_ktime.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/klua_platform/klua_ktime.c)
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `sleep(ms)` | — | 阻塞 C 线程 |
| `sleep_ns(ns)` | — | 暂未实现 |
| `tick_count()` | integer | 系统滴答 (ms) |
| `timer(interval, cb)` | boolean | 一次性定时器 (**非协程**) |
| `ticker(name, interval, cb)` | boolean | 周期定时器 (**非协程**) |
| `stop_ticker(name)` | boolean | 停止 ticker |

### 伪代码

桩: [klb/bin/klbcore/help/k/ktime.lua](https://gitee.com/klua/klb/blob/trunk/bin/klbcore/help/k/ktime.lua)

```lua
--[[
-- Copyright(c) 2021, LGPL v3 All Rights Reserved
--
-- @file    ktime.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  	C ktime
--   \n require("ktime")
--   \n C导出文件: ./klb/src_c/klua/klua_platform/klua_ktime.c
-- @version 0.1
--]]
local ktime = {}


-- @brief 休眠毫秒
-- @param [in]  	ms[number(int)]	休眠毫秒
-- @return 无
-- @note 直接休眠C线程
ktime.sleep = function (ms)
	return
end


-- @brief 休眠纳秒[暂未实现]
-- @param [in]  	ns[number(int)]	休眠纳秒
-- @return 无
-- @note 直接休眠C线程
ktime.sleep_ns = function (ns)
	return
end


-- @brief 获取系统滴答数(毫秒)
-- @return [number(int)] 系统滴答数
ktime.tick_count = function ()
	return 123456
end


-- @brief 一次调用定时器
-- @param [in]  	interval[number(int)]	等待间隔(毫秒)
-- @param [in]  	cb[function]			回调函数
-- @return [boolean] true, false
--  cb = function (tc)
--		-- tc[number(int)]	-- 当前系统滴答数(毫秒)
--		return				-- 无返回值
--  end
-- @note 仅非协程使用
ktime.timer = function (interval, cb)
	return true
end


-- @brief 长期调用定时器
-- @param [in]  	name[string]			名称
-- @param [in]  	interval[number(int)]	等待间隔(毫秒)
-- @param [in]  	cb[function]			回调函数
-- @return [boolean] true, false
--  cb = function (tc)
--		-- tc[number(int)]	-- 当前系统滴答数(毫秒)
--		return				-- 无返回值
--  end
-- @note 仅非协程使用
ktime.ticker = function (name, interval, cb)
	return true
end


-- @brief 关闭长期调用定时器
-- @param [in]  	name[string]			名称
-- @return [boolean] true, false
ktime.stop_ticker = function (name)
	return true
end

return ktime
```

### 示例

```lua
local ktime = require("ktime")

print("tick", ktime.tick_count())

-- 阻塞当前 C 线程
ktime.sleep(100)

-- 一次性定时器 (主线程/非协程)
ktime.timer(500, function (tc)
    print("timer", tc)
end)

-- 周期定时器
ktime.ticker("heartbeat", 1000, function (tc)
    print("beat", tc)
end)

-- 停止
ktime.stop_ticker("heartbeat")
```

### 注意

- `sleep` 阻塞 **C 线程**, 不同于 [kco](kco.md) `co_sleep` (协程 yield)
- `timer` / `ticker` 标注 **仅非协程**; 协程内延时用 `kco.timeout` / `kco.co_sleep`
- env 级滴答见 [kenv](kenv.md) `tick_count`
