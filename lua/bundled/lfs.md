## 文件系统 (LuaFileSystem)

> **require**: `lfs` | 代码: `klb/src_c/klua/luafilesystem-2.0/src/lfs.c`
> **文档样板**: bundled Lua API 四层 — 同 [ksys.md](../klua/ksys.md)
> **上游**: [LuaFileSystem](https://lunarmodules.github.io/luafilesystem/) 1.6.3 (Kepler / MIT)

### 导出 API

| 函数 | 返回 | 说明 |
|------|------|------|
| `attributes(path [, name\|table])` | table / value | 文件/目录属性 |
| `chdir(path)` | boolean | 切换当前目录 |
| `currentdir()` | string | 当前工作目录 |
| `dir(path)` | iterator | 目录项迭代器 |
| `mkdir(path)` | boolean | 创建目录 |
| `rmdir(path)` | boolean | 删除空目录 |
| `touch(path [, atime [, mtime]])` | boolean | 修改访问/修改时间 |
| `link(old, new [, symlink])` | boolean | 硬链接或符号链接 |
| `symlinkattributes(path [, name\|table])` | table / value | 符号链接目标属性 |
| `lock(file [, mode])` | lock_obj | 文件锁 |
| `unlock(lock_obj)` | boolean | 释放文件锁 |
| `lock_dir(path [, mode])` | lock_obj | 目录锁 |
| `setmode(file, mode)` | boolean | 文本/二进制模式 (Windows) |

**`attributes` / `symlinkattributes` 常用字段**: `mode` (`"file"` / `"directory"` / `"link"` …), `size`, `access`, `modification`, `change`, `dev`, `ino`, `nlink`, `uid`, `gid`, `rdev`, `blocks`, `blksize`.

**`dir` 迭代器**: 每次 `(name, inode)`; 结束 `nil`.

### 伪代码

```lua
--[[
-- @file   lfs.lua
-- @brief  require("lfs") — LuaFileSystem 1.6.3
--   \n C: ./klb/src_c/klua/luafilesystem-2.0/src/lfs.c
--]]

local lfs = {}


-- @brief 获取路径属性
-- @param [in] path[string]
-- @param [in] name[string|table]	[可选] 单字段名或预填 table
-- @return [table] attrs 或 [any] 单字段值; 失败 nil, err, errno
-- @note eg. local m = lfs.attributes("/tmp", "mode")
lfs.attributes = function (path, name)
	return { mode = 'directory', size = 0 }
end


-- @brief 切换进程当前目录
-- @param [in] path[string]
-- @return [boolean] ok; 失败 nil, err, errno
lfs.chdir = function (path)
	return true
end


-- @brief 当前工作目录
-- @return [string] path
lfs.currentdir = function ()
	return '.'
end


-- @brief 目录项迭代器
-- @param [in] path[string]
-- @return [function] iterator(name, inode)
-- @note eg. for name in lfs.dir(path) do ... end
lfs.dir = function (path)
	return function () return nil end
end


-- @brief 创建目录
-- @param [in] path[string]
-- @return [boolean] ok; 失败 nil, err, errno
lfs.mkdir = function (path)
	return true
end


-- @brief 删除空目录
-- @param [in] path[string]
-- @return [boolean] ok; 失败 nil, err, errno
lfs.rmdir = function (path)
	return true
end


-- @brief 修改时间戳
-- @param [in] path[string]
-- @param [in] atime[number(int)]	[可选] 访问时间 (os.time 基准)
-- @param [in] mtime[number(int)]	[可选] 修改时间
-- @return [boolean] ok
lfs.touch = function (path, atime, mtime)
	return true
end


-- @brief 创建链接
-- @param [in] old[string]	源路径
-- @param [in] new[string]	新链接路径
-- @param [in] symlink[boolean]	[可选] true 为符号链接
-- @return [boolean] ok
lfs.link = function (old, new, symlink)
	return true
end


-- @brief 符号链接目标属性 (参数同 attributes)
lfs.symlinkattributes = function (path, name)
	return lfs.attributes(path, name)
end


-- @brief 文件锁 (见 lock_obj:close / lfs.unlock)
lfs.lock = function (file, mode)
	return {}
end


-- @brief 释放 lock 返回的对象
lfs.unlock = function (lock_obj)
	return true
end


-- @brief 目录锁
lfs.lock_dir = function (path, mode)
	return {}
end


-- @brief 设置 stdio 文本/二进制模式 (Windows)
-- @param [in] file[userdata]	FILE*
-- @param [in] mode[string]	"binary" | "text"
lfs.setmode = function (file, mode)
	return true
end


return lfs
```

### 示例

```lua
local lfs = require("lfs")

-- 遍历目录
for name in lfs.dir(lfs.currentdir()) do
	if name ~= "." and name ~= ".." then
		local mode = lfs.attributes(name, "mode")
		print(name, mode)
	end
end

-- 创建并检查
lfs.mkdir("tmp_demo")
local ok, err = lfs.rmdir("tmp_demo")
if not ok then
	print(err)
end
```

### 注意

- 默认编入; 无裁剪宏
- `chdir` 影响进程全局 CWD; 脚本库路径解析常用 [kenv](../klua/kenv.md) `base_path`, 勿随意 `chdir`
- Windows 上 `setmode` / 链接语义与 Unix 有差异; 属性 `uid`/`gid` 在 Windows 常为 0
- 协议: `klua_doc/klb/licenses/LICENSE-luafilesystem-2.0`
