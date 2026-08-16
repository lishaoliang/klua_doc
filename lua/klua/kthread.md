## 工作线程

> **require**: `kthread` | 代码: `klb/src_c/klua/klua_multithread/klua_kthread.c`
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `start(entry [, wait [, cpu, ...]])` | string | 启动 worker 线程 |
| `stop(name)` | boolean | 停止线程 |
| `wait(...)` | — | 批量等待线程就绪 |

### 伪代码

桩: `klb/bin/klbcore/help/k/kthread.lua`

```lua
--[[
-- Copyright(c) 2022, LGPL All Rights Reserved
-- @file   kthread.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  C kthread
--   \n require("kthread")
--   \n C导出文件: ./klb/src_c/klua/klua_multithread/klua_kthread.c
-- @version 0.1
--]]

local kthread = {}


-- @brief 开启一个线程
-- @param [in] entry[string]			加载入口脚本; eg.'aaa.bbb'
-- @param [in] wait[boolean]			[可选](默认true): true.等待线程启动完成; false.不等待线程启动完成
-- @param [in] cpu[number(int)]			[可选](默认-1): -1.自动选择CPU; 大于0. 选择对应的CPU
-- @param [in] [...]					[可选]线程启动附加全局参数: 在目标线程中使用 ksys.get_arg() 获取得到
-- @return [string] 返回线程名称
-- @note eg. local name = kthread.start('aaa.bbb', true, -1, '111', {a=1})
kthread.start = function (entry, wait, cpu, ...)
	return ''
end


-- @brief 停止一个线程: 会等待到线程停止完毕
-- @param [in] name[string]			线程名称: 由 kthread.start() 生成
-- @return [boolean] true.成功停止; false. 未停止
kthread.stop = function (name)
	return true
end


-- @brief 批量等待线程启动完成
-- @param [in] [...]			线程名称(string)
-- @return 无
-- @note eg. 1. kthread.wait('aaa', 'bbb', 'ccc')	按名称字符串参数排列
--		2. kthread.wait({'aaa', 'bbb', 'ccc'})		将名称组成table, 合为一个参数
kthread.wait = function (...)
	return 
end

return kthread
```

### 示例

```lua
local kthread = require("kthread")

-- 启动 worker, 传入全局参数
local name1 = kthread.start("worker.main", true, -1, "init", { port = 8080 })
local name2 = kthread.start("worker.io", false)

-- 等待未同步启动的线程就绪
kthread.wait(name2)

-- 或批量等待
kthread.wait(name1, name2)
-- kthread.wait({ name1, name2 })

-- 停止
local ok = kthread.stop(name1)
```

### 注意

- 每个 worker 为独立 `klua_env_t`, 自有 `loop_once`; 预加载链与主线程相同 (见 [preload 设计](../../klb/klua/design/preload.md))
- 第 4 个参数起可通过 [ksys](ksys.md) / [kenv](kenv.md) `get_args` 读取
- 跨线程通信用 [klpc](klpc.md); 业务划分见 **klbcore-design** § kthread
- `entry` 为 **库名** (`dolibrary`), 非文件路径; 须已配置 `package.path`
