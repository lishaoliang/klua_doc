## 操作系统接口 (os)

> **require**: `os` | 代码: `klb/src_c/klua/lua-5.4.6/src/loslib.c` (`luaopen_os`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.9](https://www.lua.org/manual/5.4/manual.html#6.9)

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `clock()` | number | CPU 时间 (秒, 依平台) |
| `time([table])` | integer / table fields | 当前时间戳; 表构造时间 |
| `date([format [, time]])` | string / table | 格式化或分解时间 |
| `difftime(t2, t1)` | number | 秒差 |
| `execute([command])` | exitstat / `true`/`nil` | 执行 shell 命令 |
| `exit([code])` | 无 | 终止进程 (**非** klua env 退出) |
| `getenv(name)` | string / `nil` | 环境变量 |
| `remove(filename)` | ok / `nil, msg` | 删除文件 |
| `rename(old, new)` | ok / `nil, msg` | 重命名 |
| `setlocale(category [, locale])` | string | 区域设置 |
| `tmpname()` | string | 临时文件名 |

### 伪代码

真源: `loslib.c` — `syslib[]`

```lua
--[[
-- @file   os.lua (伪代码)
-- @brief  Lua 5.4 os 库
--]]

local os = {}


-- @brief CPU 使用时间 (秒)
-- @return [number]
os.clock = function ()
	return 0.0
end


-- @brief Unix 时间戳; 传表 {year, month, day, hour?, min?, sec?, isdst?} 构造
-- @return [number(int)]
os.time = function (t)
	return 0
end


-- @brief 时间格式化; format 为 "*t" 返回表, 默认 "*t" 或 "!" 前缀 UTC
-- @param [in] time[number(int)]	[可选] 默认 os.time()
-- @return [string|table]
os.date = function (format, time)
	return ''
end


-- @brief t2 - t1 (秒)
os.difftime = function (t2, t1)
	return 0.0
end


-- @brief 执行系统命令 (嵌入式慎用)
-- @return [boolean|number] Windows: 退出码相关; Unix: true/nil + 可选 "exit"/"signal"
os.execute = function (command)
	return true
end


-- @brief 终止整个 OS 进程 (勿与 kenv.exit / ksys.exit 混淆)
os.exit = function (code)
end


-- @brief 读环境变量
os.getenv = function (name)
	return ''
end


-- @brief 删除文件
os.remove = function (filename)
	return true
end


-- @brief 重命名
os.rename = function (old, new)
	return true
end


-- @brief 区域设置 ("collate"|"ctype"|"monetary"|"numeric"|"time"|"all")
os.setlocale = function (category, locale)
	return 'C'
end


-- @brief 生成临时文件名 (每次调用不同)
os.tmpname = function ()
	return '/tmp/lua_XXXXXX'
end

return os
```

### 示例

```lua
print(os.date("%Y-%m-%d %H:%M:%S"))

local t = os.date("*t")
print(t.year, t.month, t.day)

local home = os.getenv("HOME") or os.getenv("USERPROFILE")
```

### 注意

- **`os.execute` / `os.exit` / `os.remove`**: 嵌入式或产品沙箱中慎用; 退出 klua 环境用 **`kenv.exit()`** / **`ksys.exit()`** → [kenv.md](../klua/kenv.md)
- 高精度计时 / 平台路径: 优先 **`ktime`** / **`kos`** → [ktime.md](../klua/ktime.md), [kos.md](../klua/kos.md)
- `os.tmpname` 只生成名, 不创建文件; 存在竞态, 生产环境慎用
