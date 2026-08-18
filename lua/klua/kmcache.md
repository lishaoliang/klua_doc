## 线程间共享缓存

> **require**: `kmcache` | 代码: `klb/src_c/klua/klua_multithread/klua_kmcache.c`
> **文档样板**: k* Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `set(key, ...)` | — | 写入/覆盖键值 |
| `get(key)` | `...` | 按键取值 (多返回值) |
| `size()` | number | 当前条目数 |
| `clear()` | — | 清空全部条目 |

### 伪代码

桩: `klb/bin/klbcore/help/k/kmcache.lua`

```lua
--[[
-- Copyright(c) 2022, GNU LESSER GENERAL PUBLIC LICENSE Version 3, 29 June 2007
-- @file   kmcache.lua
-- @author 随风(https://gitee.com/klua/klb)
-- @brief  C kmcache, memory cache
--   \n require("kmcache")
--   \n C导出文件: ./klb/src_c/klua/klua_multithread/klua_kmcache.c
--   \n 进程内部所有线程共享数据
--   \n 注意: 勿扩大使用范围, 最适用于版本等固定内容但又需要全局共享的信息
-- @version 0.1
--]]


local kmcache = {}


-- @brief 设置 K/V
-- @param [in] key[string]	key 值
-- @param [in] [...] 		任意类型 (见注意: 可序列化类型)
-- @return 无
-- @note eg. kmcache.set('ver', 1, 'abc', { a = 1, b = 2 })
kmcache.set = function (key, ...)
	return
end


-- @brief 获取 K/V
-- @param [in] key[string]	key 值
-- @return [...] 与 set 时第 2 个参数起对应的多个返回值; 键不存在时无返回值
-- @note eg. local n, s, t = kmcache.get('ver')
kmcache.get = function (key)
	return ...
end


-- @brief 获取条目总数
-- @return [number(int)] 总数
kmcache.size = function ()
	return 0
end


-- @brief 清除所有条目
-- @return 无
kmcache.clear = function ()
	return
end


return kmcache
```

### 示例

```lua
local kmcache = require("kmcache")

-- 写入: key 后每个参数独立序列化, get 时按序还原
kmcache.set("1", 1, "abc", { a = 1, b = 2 })
kmcache.set("2", 2, "efg", true, false, 3.14)

-- 同 key 覆盖
kmcache.set("1", 1, "egeg", 123456, { a = 1, b = 2 })

print("size", kmcache.size())  -- 2

local n, s, t = kmcache.get("1")
print(n, s, t)  -- 1  egeg  table:{a=1,b=2}

kmcache.clear()
print("size", kmcache.size())  -- 0
```

跨线程 (主线程写, worker 读):

```lua
local kthread = require("kthread")
local kmcache = require("kmcache")

kmcache.set("build", "20250817", { rev = 42 })

local name = kthread.start("worker.check_cache", true)
-- worker 内: local ver, info = kmcache.get("build")
```

### 注意

- **进程级单例**: `klua_multithread_init` 创建, 主线程与各 [kthread](kthread.md) worker 共享同一实例; 内部 `klb_mutex` 保护
- **值序列化**: `set` 从第 2 个参数起经 `luaseri_map_binary` 打包; `get` 解包为多返回值. 支持 `nil` / `boolean` / `number` / `string` / `table`; `function` / `userdata` / `thread` 等写入时变为 `nil`
- **键不存在**: `get(key)` 返回 0 个值 (非 `nil` 单值)
- **无单键删除**: 仅 `clear()` 清空全部; 覆盖请再次 `set` 同 key
- **适用场景**: 版本号、配置快照等**读多写少、体量小**的跨线程共享; 重量级对象单次交接用 `klist` (待文档); 消息/回调用 [klpc](klpc.md)
- **勿滥用**: 非通用数据库或大对象缓存; 生命周期随进程, `klua_multithread_quit` 时释放
