# kpfs.probe (Lua 子模块)

> **require**: `kpfs.probe` | 代码: [portfs/src_klua/pfs_klua_probe.c](https://gitee.com/klua/portfs/blob/trunk/src_klua/pfs_klua_probe.c)
> **文档格式**: Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档
> **枢纽**: [readme.md](readme.md) (返回值、CLI 对照) | **CLI 真源**: [pfs_tool.md](../../pfs/pfs_tool.md) §3.1–§3.3

### 导出 API

探测只读; 固定返回 **`info, rc, msg`** (`info` 恒为表; `rc==0` 时为载荷, `rc~=0` 时为 `{}`). 参数形态见 [readme.md](readme.md) §3.1.


| 函数                    | 返回              | 说明           |
| --------------------- | --------------- | ------------ |
| `all(path [, opts])`  | `info, rc, msg` | 全分区 + FS 试探  |
| `part(path [, opts])` | `info, rc, msg` | 仅分区表         |
| `fs(path [, opts])`   | `info, rc, msg` | 首分区或裸盘 FS 试探 |


### 伪代码

```lua
--[[
-- Copyright(c) 2026, LGPL All Rights Reserved
-- @file   kpfs_probe.lua
-- @brief  C kpfs.probe
--   require("kpfs.probe")
--   C: portfs/src_klua/pfs_klua_probe.c
-- @version 0.1
--]]

local probe = {}


-- @brief 全分区探测 + 每分区 FS 试探 mount (对齐 pfs probe)
-- @param [in]	path[string|table]		路径字符串, 或仅 opts 表 (含 path)
-- @param [in]	opts[table]				[可选] 见内联 opts 注释; path+opts 双参时为本参
-- @return info[table]					固定表; rc==0 时载荷; rc~=0 时为 {}
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. local info = probe.all("disk.vhdx")
--		或 local info, rc, msg = probe.all("disk.vhdx"); if rc == 0 then ... end
--		失败: return {}, rc, msg
--  只读; 不登记盘符; 固定三返回值 (禁止多义返回个数)
probe.all = function (path, opts)
	-- opts = {
	--   path[string]			[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	--   offset[number(int)]	[可选] 字节偏移, 默认 `0`; 仅无分区表时 bare_fs 试探起点
	-- }

	local rc = 0
	local msg = "OK"

	return {
		path = path,					-- [必须] string    输入路径
		container = "vhdx",				-- [必须] string    blkio 后端名 (如 raw/vhdx)
		type = 0,						-- [必须] integer   pfs_blkio_type_e
		sector_size = 512,				-- [必须] integer   逻辑扇区大小
		size = 0,						-- [必须] integer   容器字节大小
		scheme = "gpt",					-- [必须] string    part_count>0: "mbr"/"gpt"; 否则 "none"
		part_count = 2,					-- [必须] integer   分区条目数; 无分区表为 0
		parts = {						-- [必须] array     始终存在; part_count==0 时为 {}
			{
				index = 1,				-- [必须] integer   1-based 分区序号
				part_offset = 0,		-- [必须] integer   字节偏移
				start_lba = 0,			-- [必须] integer   起始 LBA
				size_lba = 0,			-- [必须] integer   扇区数
				mbr_sys_ind = 0,		-- [必须] integer   MBR 系统 ID
				bootable = false,		-- [必须] boolean   可引导标志
				fs_type = "exFAT",		-- [必须] string    试探 mount 成功时 FS 名; 失败为 ""
				mount_ok = true,		-- [必须] boolean   试探 mount 是否成功
				mount_err = 0,			-- [必须] integer   失败为负 PFS_*; 成功为 0
			},
			-- {}, ...					同结构分区项 (共 part_count 项)
		},
		bare_fs = {						-- [必须] table     始终存在; part_count>0 时为 {}
			index = 1,					-- [必须] integer   固定 1 (裸盘单槽)
			part_offset = 0,			-- [必须] integer   裸盘试探字节偏移
			fs_type = "",				-- [必须] string    试探 mount 成功时 FS 名; 失败为 ""
			mount_ok = false,			-- [必须] boolean   试探 mount 是否成功
			mount_err = 0,				-- [必须] integer   失败为负 PFS_*; 成功为 0
		},
	}, rc, msg
end


-- @brief 仅探测分区表 (不试探 FS)
-- @param [in]	path[string|table]		同 all
-- @param [in]	opts[table]				[可选] 见内联 opts 注释
-- @return info[table]					固定表; rc==0 时载荷; rc~=0 时为 {}
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. local info = probe.part("disk.vhdx")
--		固定三返回值; parts[] **无** fs_type/mount_ok/mount_err
probe.part = function (path, opts)
	-- opts = {
	--   path[string]			[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	-- }

	local rc = 0
	local msg = "OK"

	return {
		path = path,					-- [必须] string    输入路径
		container = "vhdx",				-- [必须] string    blkio 后端名
		type = 0,						-- [必须] integer   pfs_blkio_type_e
		sector_size = 512,				-- [必须] integer   逻辑扇区大小
		size = 0,						-- [必须] integer   容器字节大小
		scheme = "gpt",					-- [必须] string    "mbr"/"gpt"; 无分区时 "none"
		part_count = 2,					-- [必须] integer   分区条目数; 无分区表为 0
		parts = {						-- [必须] array     始终存在; part_count==0 时为 {}
			{
				index = 1,				-- [必须] integer   1-based 分区序号
				part_offset = 0,		-- [必须] integer   字节偏移
				start_lba = 0,			-- [必须] integer   起始 LBA
				size_lba = 0,			-- [必须] integer   扇区数
				mbr_sys_ind = 0,		-- [必须] integer   MBR 系统 ID
				bootable = false,		-- [必须] boolean   可引导标志
			},
			-- {}, ...					同结构分区项 (共 part_count 项)
		},
	}, rc, msg
end


-- @brief 试探 FS (有分区表时仅首分区; 无分区表等同 bare_fs)
-- @param [in]	path[string|table]		同 all
-- @param [in]	opts[table]				[可选] 见内联 opts 注释
-- @return info[table]					固定表; rc==0 时载荷; rc~=0 时为 {}
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. local info = probe.fs("disk.vhdx")
--		固定三返回值; 无分区表时含 bare_fs
probe.fs = function (path, opts)
	-- opts = {
	--   path[string]			[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	--   offset[number(int)]	[可选] 字节偏移, 默认 `0`; 仅无分区表时 bare_fs 试探起点
	-- }

	local rc = 0
	local msg = "OK"

	return {
		path = path,					-- [必须] string    输入路径
		container = "vhdx",				-- [必须] string    blkio 后端名
		type = 0,						-- [必须] integer   pfs_blkio_type_e
		sector_size = 512,				-- [必须] integer   逻辑扇区大小
		size = 0,						-- [必须] integer   容器字节大小
		scheme = "gpt",					-- [必须] string    part_count>0: "mbr"/"gpt"; 否则 "none"
		part_count = 1,					-- [必须] integer   有分区表时为 1 (仅首分区); 无分区表为 0
		parts = {						-- [必须] array     始终存在; part_count==0 时为 {}
			{
				index = 1,				-- [必须] integer   1-based 分区序号
				part_offset = 0,		-- [必须] integer   字节偏移
				start_lba = 0,			-- [必须] integer   起始 LBA
				size_lba = 0,			-- [必须] integer   扇区数
				mbr_sys_ind = 0,		-- [必须] integer   MBR 系统 ID
				bootable = false,		-- [必须] boolean   可引导标志
				fs_type = "exFAT",		-- [必须] string    试探 mount 成功时 FS 名; 失败为 ""
				mount_ok = true,		-- [必须] boolean
				mount_err = 0,			-- [必须] integer
			},
		},
		bare_fs = {						-- [必须] table     始终存在; part_count>0 时为 {}
			index = 1,					-- [必须] integer   固定 1 (裸盘单槽)
			part_offset = 0,			-- [必须] integer   裸盘试探字节偏移
			fs_type = "",				-- [必须] string    试探 mount 成功时 FS 名; 失败为 ""
			mount_ok = false,			-- [必须] boolean   试探 mount 是否成功
			mount_err = 0,				-- [必须] integer   失败为负 PFS_*; 成功为 0
		},
	}, rc, msg
end


return probe
```

### 示例

```lua
local probe = require("kpfs.probe")

-- 全量探测 (只关心 info 时可简写)
local info = probe.all("disk.vhdx")
for i, p in ipairs(info.parts) do
    print(i, p.fs_type, p.mount_ok)
end

local info2, rc2, msg2 = probe.all({ path = "disk.vhdx", offset = 0 })
if rc2 ~= 0 then
    print(msg2)
end

-- 仅分区表 (无 fs_type / mount_ok)
local part = probe.part("disk.vhdx")
print(part.scheme, part.part_count)

-- 首分区 FS 或裸盘
local fs = probe.fs("disk.vhdx")
print(fs.parts[1].mount_ok)
```

### 注意

- 探测为只读; 不调用 `kpfs.mount`
- 固定返回 **`info, rc, msg`** 三值 (`info` 恒为表; `rc~=0` 时为 `{}`); 禁止多义返回个数
- `info.parts` / `info.bare_fs` (**all** / **fs**) **须始终存在**; 无对应内容时为 **空表 `{}`**
- `probe.part` 的 `parts[]` **无** `fs_type` / `mount_ok` / `mount_err`
- `probe.fs` 对非 raw 容器仅试探 **首分区**
- 写操作 `rc, msg` 约定见 [readme.md](readme.md) §3

---

## info 表字段

> 字段注释以伪代码内联为准; 下表为速查.

**公共** (容器级):


| 字段            | 类型      | 说明                          |
| ------------- | ------- | --------------------------- |
| `path`        | string  | 输入路径                        |
| `container`   | string  | blkio 后端名 (如 `raw`, `vhdx`) |
| `type`        | integer | `pfs_blkio_type_e` 枚举值      |
| `sector_size` | integer | 逻辑扇区; 缺省视为 512              |
| `size`        | integer | 容器字节大小                      |


**分区与裸盘** (`all` / `part` / `fs` 公共):


| 字段           | 类型      | 说明                                                |
| ------------ | ------- | ------------------------------------------------- |
| `scheme`     | string  | `part_count>0`: `"mbr"` / `"gpt"`; 否则 `"none"`    |
| `part_count` | integer | 分区条目数; 无分区表为 `0`                                 |
| `parts`      | array   | **始终存在**; `part_count==0` 时为 `{}`; 见下表 `parts[i]` |


`parts[i]` 条目 (`all` / `fs` 含 FS 字段; `part` **无** `fs_type` / `mount_ok` / `mount_err`):


| 字段            | 类型      | 说明                                       |
| ------------- | ------- | ---------------------------------------- |
| `index`       | integer | **1-based** 分区序号                        |
| `part_offset` | integer | 字节偏移                                     |
| `start_lba`   | integer | 起始 LBA                                   |
| `size_lba`    | integer | 扇区数                                      |
| `mbr_sys_ind` | integer | MBR 系统 ID                                |
| `bootable`    | boolean | 可引导标志                                    |
| `fs_type`     | string  | 试探 mount 成功时 FS 名 (如 `exFAT`); 失败为 `""` |
| `mount_ok`    | boolean | 试探 mount 是否成功                            |
| `mount_err`   | integer | 失败时为负 `PFS_*`; 成功为 `0`                   |


**裸盘 FS** (`all` / `fs` 专有):


| 字段        | 类型    | 说明                                                                                      |
| --------- | ----- | --------------------------------------------------------------------------------------- |
| `bare_fs` | table | **始终存在**; `part_count>0` 时为 `{}`; `part_count==0` 时为 `{ index, part_offset, fs_type, mount_ok, mount_err }` |


---

## CLI 对照


| pfs CLI           | kpfs Lua                                                 |
| ----------------- | -------------------------------------------------------- |
| `probe <path>`    | `require("kpfs.probe").all(path)` 或 `.all({ path = … })` |
| `probe part -d …` | `kpfs.probe.part(…)`                                     |
| `probe fs -d …`   | `kpfs.probe.fs(…)` (仅首分区)                                |


