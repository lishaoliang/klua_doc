# kpfs.vfs (Lua 子模块)

> **require**: `kpfs.vfs` | 代码: `portfs/src_klua/pfs_klua_vfs.c`, `pfs_klua_file.c`, `pfs_klua_dir.c`, `pfs_klua_vfs_path.c`
> **文档格式**: Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档
> **枢纽**: [readme.md](readme.md) (总览、返回值) | **前置**: 须先 `kpfs.mount` 登记盘符

### 导出 API

**前置**: 须 `kpfs.mount`. path 级固定返回 **`rc, msg`**. `open`/`opendir` 成功返回 userdata, 失败 **`rc, msg`**.

#### 模块函数

| 函数 | 返回 | 说明 |
|------|------|------|
| `open(drive_path [, mode])` | file / `rc, msg` | 打开文件 |
| `opendir(drive_path)` | dir / `rc, msg` | 打开目录 |
| `access` / `mkdir` / `rmdir` / `unlink` / `remove` / `rename` / `copy` | `rc, msg` | path 级 VFS; `copy` 支持盘符路径与 `@` 宿主路径 |

#### file userdata (`kpfs.file*`)

`read`, `write`, `seek`, `tell`, `flush`, `fsync`, `truncate`, `close`, `error`, `eof`

#### dir userdata (`kpfs.dir*`)

`readdir`, `close`/`closedir`, `direrror`

### 伪代码

```lua
--[[
-- Copyright(c) 2026, LGPL All Rights Reserved
-- @file   kpfs_vfs.lua
-- @brief  C kpfs.vfs
--   require("kpfs.vfs")
--   C: portfs/src_klua/pfs_klua_vfs.c, pfs_klua_file.c, pfs_klua_dir.c, pfs_klua_vfs_path.c
-- @version 0.1
--]]

local vfs = {}


-- @brief 打开盘符路径下文件
-- @param [in]	drive_path[string]		盘符路径, eg. "D1:/aa/bb.txt"
-- @param [in]	mode[string|number(int)]	[可选] "r"/"rb", "r+", "w"/"w+", "a"/"a+" 或 PFS_O_* 整数; 默认 "r"
-- @return file[userdata]				成功; metatable `kpfs.file*`
-- @return rc[number(int)]				失败
-- @return msg[string]					失败时 pfs_strerror(rc)
-- @note eg. local f = vfs.open("D1:/aa/bb.txt", "r+")
--		或 local f, second = vfs.open(...); if type(f) ~= "userdata" then error(second) end
--		失败仅双值 rc, msg; 勿写 local f, rc, msg (失败时 rc/msg 变量错位)
--		须先 kpfs.mount; 路径语法见本文 §盘符路径语法
vfs.open = function (drive_path, mode)
	-- mode[string|number(int)]	[可选] 打开模式:
	--   "r"/"rb"     只读 (PFS_O_RD)
	--   "r+"         读写 (PFS_O_RDWR)
	--   "w"/"w+"     读写+创建 (PFS_O_RDWR|PFS_O_CREAT)
	--   "a"/"a+"     读写+创建+定位到末尾

	local rc = 0
	local msg = "OK"

	-- file userdata (metatable `kpfs.file*`; 带 __gc)
	local f = {}

	-- @brief 读取至多 n 字节
	-- @param [in]	n[number(int)]			[可选] 默认 4096; 上限 1MiB; 0 返回 ""
	-- @return data[string]					成功; 短读为实际长度
	-- @return rc[number(int)]				失败; 负值为 PFS_*
	-- @return msg[string]					失败时 pfs_strerror(rc)
	f.read = function (n)
		return ""
	end

	-- @brief 写入数据
	-- @param [in]	s[string]				待写字节串
	-- @return nbytes[number(int)]			成功; 实际写入字节数
	-- @return rc[number(int)]				失败; 负值为 PFS_*
	-- @return msg[string]					失败时 pfs_strerror(rc)
	f.write = function (s)
		return #s
	end

	-- @brief 定位文件偏移
	-- @param [in]	off[number(int)]		偏移量
	-- @param [in]	whence[string|number(int)]	[可选] "set"/"cur"/"end" 或 PFS_SEEK_*; 默认 "set"
	-- @return pos[number(int)]				成功; 新偏移
	-- @return rc[number(int)]				失败; 负值为 PFS_*
	-- @return msg[string]					失败时 pfs_strerror(rc)
	f.seek = function (off, whence)
		return 0
	end

	-- @brief 当前偏移
	-- @return pos[number(int)]				成功
	-- @return rc[number(int)]				失败; 负值为 PFS_*
	-- @return msg[string]					失败时 pfs_strerror(rc)
	f.tell = function ()
		return 0
	end

	-- @brief 刷新用户缓冲
	-- @return rc[number(int)]				0 成功; 负值为 PFS_*
	-- @return msg[string]					pfs_strerror(rc)
	f.flush = function ()
		return rc, msg
	end

	-- @brief 同步到介质
	-- @return rc[number(int)]				0 成功; 负值为 PFS_*
	-- @return msg[string]					pfs_strerror(rc)
	f.fsync = function ()
		return rc, msg
	end

	-- @brief 截断到指定长度
	-- @param [in]	size[number(int)]		新长度 (字节)
	-- @return rc[number(int)]				0 成功; 负值为 PFS_*
	-- @return msg[string]					pfs_strerror(rc)
	f.truncate = function (size)
		return rc, msg
	end

	-- @brief 关闭文件 (亦可依赖 __gc)
	-- @return 无
	f.close = function ()
	end

	-- @brief 查询流错误码
	-- @return rc[number(int)]				0 无错; 负值为 PFS_*
	-- @return msg[string]					pfs_strerror(rc)
	f.error = function ()
		return rc, msg
	end

	-- @brief 是否到达 EOF
	-- @return eof[boolean]				成功
	-- @return rc[number(int)]				失败 (如已 close); 负值为 PFS_*
	-- @return msg[string]					失败时 pfs_strerror(rc)
	f.eof = function ()
		return false
	end

	return f
end


-- @brief 打开目录
-- @param [in]	drive_path[string]		盘符路径, eg. "D1:/aa"
-- @return dir[userdata]					成功; metatable `kpfs.dir*`
-- @return rc[number(int)]				失败
-- @return msg[string]					失败时 pfs_strerror(rc)
-- @note eg. local d = vfs.opendir("D1:/aa")
--		或 local d, second = vfs.opendir(...); if type(d) ~= "userdata" then error(second) end
--		失败仅双值 rc, msg; 勿写 local d, rc, msg (失败时 rc/msg 变量错位)
vfs.opendir = function (drive_path)
	local rc = 0
	local msg = "OK"

	-- dir userdata (metatable `kpfs.dir*`; 带 __gc)
	local d = {}

	-- @brief 读取下一目录项
	-- @return ent[table]					成功; 见内联 ent 字段
	-- @return 无							无更多条目 (非错误)
	-- @return rc[number(int)]				失败; 负值为 PFS_*
	-- @return msg[string]					失败时 pfs_strerror(rc)
	d.readdir = function ()
		return {
			name = "example.txt",			-- [必须] string    条目名
			type = "file",					-- [必须] string    "dir"/"file"/"?"
		}
	end

	-- @brief 关闭目录 (亦可依赖 __gc)
	-- @return 无
	d.close = function ()
	end

	-- @brief 同 close
	d.closedir = function ()
	end

	-- @brief 查询目录流错误码
	-- @return rc[number(int)]				0 无错; 负值为 PFS_*
	-- @return msg[string]					pfs_strerror(rc)
	d.direrror = function ()
		return rc, msg
	end

	return d
end


-- @brief 测试访问权限 (pfs_access)
-- @param [in]	drive_path[string]		盘符路径
-- @param [in]	mode[string|number(int)]	[可选] "f"/"r"/"w"/"x" 组合或 PFS_* 整数; 默认 "f"
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = vfs.access("D1:/aa", "f")
vfs.access = function (drive_path, mode)
	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 创建目录 (pfs_mkdir)
-- @param [in]	drive_path[string]		盘符路径
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
vfs.mkdir = function (drive_path)
	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 删除空目录 (pfs_rmdir)
-- @param [in]	drive_path[string]		盘符路径
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
vfs.rmdir = function (drive_path)
	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 删除文件 (pfs_unlink)
-- @param [in]	drive_path[string]		盘符路径
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
vfs.unlink = function (drive_path)
	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 删除文件或目录 (pfs_remove)
-- @param [in]	drive_path[string]		盘符路径
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
vfs.remove = function (drive_path)
	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 重命名; 同分区 pfs_rename; 跨分区 pfs_rename_cross (普通文件 copy+unlink 源)
-- @param [in]	old[string]				源盘符路径 (不支持 `@` 宿主路径)
-- @param [in]	new[string]				目标盘符路径 (不支持 `@` 宿主路径)
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = vfs.rename("D1:/old.txt", "D1:/new.txt")
--		跨分区 eg. vfs.rename("D1:/a.txt", "D2:/b.txt"); 目录跨分区 PFS_EISDIR
--		C: pfs_rename_cross (portfs/src/vfs/pfs_path.c)
vfs.rename = function (old, new)
	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 拷贝单文件; 保留源; 对齐 pfs cp
-- @param [in]	src[string]				源路径: 盘符路径 eg. "D1:/a.txt"; 宿主路径须 `@` 前缀 eg. "@./local.bin"
-- @param [in]	dst[string]				目标路径: 同上; 一侧 `@` 即可宿主↔卷互拷; 两侧均为 `@` 非法
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = vfs.copy("D1:/a.txt", "D1:/b.txt")
--		宿主→卷 eg. vfs.copy("@./payload.bin", "D1:/data/payload.bin")
--		卷→宿主 eg. vfs.copy("D1:/readme.txt", "@./readme.txt")
--		跨分区卷内 eg. vfs.copy("D1:/a.txt", "D2:/b.txt")
--		仅单文件; 无递归目录 (整树灌入用 kpfs.disk.mkvol); 与 rename 跨分区内部 copy 不同 (copy 不删源)
--		写卷侧须 RDWR mount; C: pfs_klua_vfs_path.c
vfs.copy = function (src, dst)
	local rc = 0
	local msg = "OK"

	return rc, msg
end


return vfs
```

### 示例

```lua
local kpfs = require("kpfs")
local vfs = require("kpfs.vfs")

local rc, msg = kpfs.mount("D", "d.vhdx")
assert(rc == 0, msg)

-- path 级
rc, msg = vfs.access("D1:/aa", "f")
rc, msg = vfs.mkdir("D1:/newdir")
rc, msg = vfs.remove("D1:/old.txt")
rc, msg = vfs.copy("@./payload.bin", "D1:/data/payload.bin")

-- 文件
local f, second = vfs.open("D1:/aa/bb.txt", "r+")
if type(f) ~= "userdata" then
    error(second)
end
local data = f:read(4096)
f:write("hello")
f:seek(0, "set")
f:close()

-- 目录
local d, second = vfs.opendir("D1:/aa")
if type(d) ~= "userdata" then
    error(second)
end
while true do
    local ent = d:readdir()
    if nil == ent then
        break
    end
    print(ent.name, ent.type)
end
d:close()
```

### 注意

- **前置**: 须 `kpfs.mount` 登记盘符; 盘符路径语法见下节
- `open`/`opendir` 成功返回 userdata, 失败 `rc, msg`; file/dir 带 `__gc`
- file 流 `read`/`write`/`seek`/`tell`/`eof` 成功与失败返回个数不同; 失败首值为 `rc` (number), 如 `read` 成功首值为 `string`, 可用 `type(v1)` 区分
- path 级 API 固定返回 **`rc, msg`** 双值
- `rename`: 同 mount 分区 `pfs_rename`; 跨分区 `pfs_rename_cross` (普通文件 copy+unlink 源); 目录跨分区 `PFS_EISDIR`; 目标已存在 `PFS_EEXIST`; **仅盘符路径**, 不支持 `@` 宿主路径 (宿主↔卷请用 `copy`)
- **`copy`**: 对齐 `pfs cp`; 单文件、保留源; 盘符路径或 `@` 宿主路径; 两侧均为 `@` 返回 `PFS_EINVAL`; 无递归目录; 写卷侧须 RDWR mount
- Lua API 分区 **`index` / `part_index` / 盘符路径槽号** 均为 **1-based**; `umount` 的 `part_index` 可为序号或序号表; C `pfs_disk_part_info` 入参为 **1-based**
- CLI 无盘符; VFS 为 Lua 独有能力

---

## 盘符路径语法

仿 Windows **盘符与分区号** (`D:`, `D1:`); 目录分隔符 **`/`** (非 `\`).

```text
盘名[分区槽]:/相对路径
```

| 成分 | 规则 | 示例 |
|------|------|------|
| 盘名 | 仅字母; `mount` 时指定 | `D`, `DEF` |
| 分区槽 | 可选; **1-based**; 省略视为第 1 分区 | `D1`, `D2` |
| 根 | `:` 后为分区根 | `D1:` |
| 分隔符 | **`/`** 固定 | `/aa/bb.txt` |

**`copy` 宿主路径** (与 `pfs cp` 一致): 路径以 **`@`** 开头表示宿主机文件系统; `@` 后为宿主相对或绝对路径. 卷内仍用盘符路径; 两侧均为 `@` 非法.

| 方向 | src | dst |
|------|-----|-----|
| 卷内 | `D1:/a.txt` | `D1:/b.txt` |
| 宿主→卷 | `@./local.bin` | `D1:/data/x.bin` |
| 卷→宿主 | `D1:/readme.txt` | `@./readme.txt` |
| 跨分区卷内 | `D1:/a.txt` | `D2:/b.txt` |

---

## file / dir 方法速查

> 字段与方法注释以伪代码内联为准; 下表为速查.

**`readdir` 返回 `ent` 表**:


| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 条目名 |
| `type` | string | `"dir"` / `"file"` / `"?"` |

**file 方法**:


| 方法 | 成功返回 | 失败返回 |
|------|----------|----------|
| `read([n])` | `string` (默认 n=4096) | `rc, msg` |
| `write(s)` | `nbytes` (integer) | `rc, msg` |
| `seek(off [, whence])` | `pos` (integer) | `rc, msg` |
| `tell()` | `pos` (integer) | `rc, msg` |
| `flush()` / `fsync()` / `truncate(size)` / `error()` | `rc, msg` | — |
| `eof()` | `boolean` | `rc, msg` (如已 close) |
| `close()` | 无 | — |

**dir 方法**:


| 方法 | 成功返回 | 失败返回 |
|------|----------|----------|
| `readdir()` | `ent` 表 | 无更多: 无返回值; 错误: `rc, msg` |
| `direrror()` | `rc, msg` | — |
| `close()` / `closedir()` | 无 | — |

---

## CLI 对照

| pfs CLI | kpfs Lua |
|---------|----------|
| `ls` | `opendir` + `readdir` |
| `cat` | `open` + `read` |
| `mkdir` / `rm` / `rmdir` | `mkdir` / `unlink` / `rmdir` |
| `mv` | `rename` (跨分区文件移动强于 CLI `mv`) |
| `cp` | `copy` |
| `access` | `access` (CLI 未实现) |
| `mount` (盘符) | `kpfs.mount(name, path, mode)` (**CLI 无盘符**) |

---

## 后续扩展 (可选)

| 项 | 说明 |
|----|------|
| 流式 API | 可按需补 `getc`/`putc` 等 |
