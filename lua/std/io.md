## 文件 IO (io)

> **require**: `io` | 代码: [klb/src_c/klua/lua-5.4.6/src/liolib.c](https://gitee.com/klua/klb/blob/trunk/src_c/klua/lua-5.4.6/src/liolib.c) (`luaopen_io`)
> **文档样板**: 标准 Lua API 四层 — 同 [ksys.md](../klua/ksys.md); 权威参考 [Lua 5.4 手册 §6.8](https://www.lua.org/manual/5.4/manual.html#6.8)

### 导出 API

#### 模块函数

| 函数 | 返回 | 说明 |
|------|------|------|
| `open(filename [, mode])` | file / `nil, msg` | 打开文件 (`r`/`w`/`a`/`+`, 可选 `b`) |
| `close([file])` | ok / `nil, msg` | 关闭; 省略则关当前输出 |
| `read([fmt ...])` | `...` | 从当前输入读 |
| `write(...)` | file | 写入当前输出 |
| `flush()` | file | 刷新当前输出 |
| `input([file])` | file | 设/取当前输入 |
| `output([file])` | file | 设/取当前输出 |
| `lines([filename [, ...]])` | iterator | 按行迭代 |
| `type(obj)` | string / `nil` | `"file"` 或 nil |
| `tmpfile()` | file / `nil, msg` | 临时文件 |
| `popen(prog [, mode])` | file / `nil, msg` | 管道 (平台相关) |

#### 标准流 (字段)

| 字段 | 说明 |
|------|------|
| `io.stdin` | 标准输入 |
| `io.stdout` | 标准输出 |
| `io.stderr` | 标准错误 |

#### file userdata 方法

| 方法 | 说明 |
|------|------|
| `f:read([fmt ...])` | 同 `io.read` |
| `f:write(...)` | 写入 |
| `f:lines([...])` | 行迭代 |
| `f:flush()` | 刷新 |
| `f:seek([whence [, offset]])` | 定位 (`set`/`cur`/`end`) |
| `f:setvbuf(mode [, size])` | 缓冲 (`no`/`full`/`line`) |
| `f:close()` | 关闭 |

### 伪代码

真源: `liolib.c` — `iolib[]`, `meth[]`

```lua
--[[
-- @file   io.lua (伪代码)
-- @brief  Lua 5.4 io 库
--]]

local io = {}

io.stdin = {}		-- file userdata
io.stdout = {}
io.stderr = {}


-- @brief 打开文件
-- @param [in] filename[string]
-- @param [in] mode[string]			[可选] "r"|"w"|"a"|"+"|可选 "b"; 默认 "r"
-- @return [userdata|nil] file, [string|nil] err
io.open = function (filename, mode)
	return io.stdout
end


-- @brief 关闭文件; 省略 file 则关闭当前输出
-- @return [boolean] true; 失败 nil, err
io.close = function (file)
	return true
end


-- @brief 读当前输入 (fmt: "*l"|"*a"|number|…)
-- @return [...] 依格式
io.read = function (...)
	return ...
end


-- @brief 写当前输出
-- @return [userdata] file (链式)
io.write = function (...)
	return io.stdout
end


-- @brief 刷新当前输出
-- @return [userdata] file
io.flush = function ()
	return io.stdout
end


-- @brief 设/取当前输入 file
-- @param [in] file[userdata|string]	[可选] file 或文件名
-- @return [userdata] 当前输入
io.input = function (file)
	return io.stdin
end


-- @brief 设/取当前输出 file
io.output = function (file)
	return io.stdout
end


-- @brief 按行迭代; filename 省略则用当前输入
-- @return [function] iterator
io.lines = function (filename, ...)
	return function () end
end


-- @brief 是否 file userdata
-- @return [string|nil] "file" 或 nil
io.type = function (obj)
	return "file"
end


-- @brief 临时文件 (平台相关)
-- @return [userdata|nil] file, [string|nil] err
io.tmpfile = function ()
	return io.stdout
end


-- @brief 管道 (平台相关)
-- @param [in] prog[string]			命令
-- @param [in] mode[string]			[可选] "r"|"w"
-- @return [userdata|nil] file, [string|nil] err
io.popen = function (prog, mode)
	return io.stdout
end


-- file userdata 方法 (metatable)
local f = {}


-- @brief 同 io.read
f.read = function (...)
	return ...
end


-- @brief 写入
-- @return [userdata] self
f.write = function (...)
	return f
end


-- @brief 行迭代
f.lines = function (...)
	return function () end
end


-- @brief 刷新缓冲
f.flush = function ()
	return f
end


-- @brief 定位
-- @param [in] whence[string]		[可选] "set"|"cur"|"end"; 默认 "cur"
-- @param [in] offset[number(int)]	[可选] 默认 0
-- @return [number(int)] 新位置
f.seek = function (whence, offset)
	return 0
end


-- @brief 缓冲模式
-- @param [in] mode[string]		"no"|"full"|"line"
-- @param [in] size[number(int)]	[可选] full 时缓冲大小
f.setvbuf = function (mode, size)
	return
end


-- @brief 关闭
f.close = function ()
	return true
end


return io
```

### 示例

```lua
local f, err = io.open("data.txt", "r")
if not f then
	error(err)
end

for line in f:lines() do
	print(line)
end
f:close()

-- 或
for line in io.lines("data.txt") do
	print(line)
end
```

### 注意

- Win/WSL 路径与编码遵循宿主 OS; 盘符路径业务优先用 **kpfs** → [kpfs_vfs.md](../kpfs/kpfs_vfs.md)
- `io.popen` / `io.tmpfile` 在嵌入式或受限环境可能不可用
- 大文件/二进制场景可配合 `string.pack`/`string.unpack` 或 C 模块
