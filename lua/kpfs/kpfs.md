# kpfs (根模块)

> **require**: `kpfs` | 代码: [portfs/src_klua/pfs_klua_kpfs.c](https://gitee.com/klua/portfs/blob/trunk/src_klua/pfs_klua_kpfs.c)
> **文档格式**: Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档
> **枢纽**: [readme.md](readme.md) (总览、加载、返回值、CLI 对照、完整流程)

盘符运行时: 登记 env 扩展并 mount/remount/umount/mount_info. 制盘 (mkpt/mkfs/mkvol) 见 [kpfs.disk](kpfs_disk.md); 容器壳见 [kpfs.image](kpfs_image.md).

### 导出 API

写操作返回 `rc, msg` (`rc == 0` 成功). 只读查询 `mount_info` 固定返回 `info, rc, msg` (见 [readme.md](readme.md) §返回值约定).


| 函数 | 返回 | 说明 |
|------|------|------|
| `version()` | string | 模块版本 `"0.1"` |
| `mount(name, image_path [, mode [, opts]])` | `rc, msg` | 登记盘符并 mount; `mode` 为字符串或分区模式数组; `opts` 含 `offset`/`sector_size` |
| `remount(name [, mode])` | `rc, msg` | 改挂载模式; `mode` 为字符串或 remount 项表 |
| `umount(name [, part_index])` | `rc, msg` | 卸载; `part_index` 为序号或序号表 |
| `mount_info([name [, index]])` | `info, rc, msg` | 查询 env 已登记 mount 状态; 省略参数查全部; `index` 为 **1-based** 分区序号 |


### 伪代码

```lua
--[[
-- Copyright(c) 2026, LGPL All Rights Reserved
-- @file   kpfs.lua
-- @brief  C kpfs (根模块)
--   require("kpfs")
--   C: portfs/src_klua/pfs_klua_kpfs.c
-- @version 0.1
--]]

local kpfs = {}


-- @brief 模块版本
-- @return [string] 当前 "0.1"
kpfs.version = function ()
    return "0.1"
end


-- @brief 挂载镜像到盘名 (登记 env 扩展, probe 并 mount 各分区)
-- @param [in]	name[string]			盘名; 仅字母 A-Z/a-z, 不含分区号. eg. "D", "DEF"
-- @param [in]	image_path[string]		设备/镜像路径 (UTF-8)
-- @param [in]	mode[string|table]		[可选] 挂载模式; 省略则全盘 `"r"`; 见内联 mode 注释
-- @param [in]	opts[table]				[可选] `offset`/`sector_size`/`mode`; 见内联 opts 注释
-- @return	rc[number(int)], msg[string]	0 成功; 负值为 PFS_* 错误码
-- @note eg. rc, msg = kpfs.mount("D", "disk.vhdx")                          -- 全盘 `"r"`
--		rc, msg = kpfs.mount("E", "disk.vhdx", "rw")                    -- 全盘 `"rw"`
--		rc, msg = kpfs.mount("F", "disk.vhdx", { "r", "rw" })           -- 按分区独立模式
--		rc, msg = kpfs.mount("G", "bare.img", { offset = 0 })           -- 裸盘 offset
--		rc, msg = kpfs.mount("H", "disk.vhdx", "r", { sector_size = 4096 })
--  行为: 同 env 盘名不可重复 (PFS_EEXIST); 单槽 mount 失败记入 mount_err, 不致使整盘 rc 失败
--  盘符路径 (分区槽 Lua 1-based): `"D:/"`/`"D1:/"` 第1分区; `"D2:/…"` 第2分区; 详见 kpfs_vfs.md
kpfs.mount = function (name, image_path, mode, opts)
	-- mode[string]					[可选] 全盘挂载模式; 省略则 `"r"`
	--		`"r"`/`"rw"`/`"w"`; `"w"` 同 `"rw"`; 填错或非许可值默认 `"r"`
	--		`"r"`  只读
	--		`"rw"`/`"w"`  先读写, 失败回退只读
	-- mode[table] 数组: 分区模式数组 (Lua 1-based)
	--		modes[1]→第1分区, modes[2]→第2分区, …
	--		元素[string]			同 mode[string]; 缺项或填错默认 `"r"`
	-- opts[table]					[可选] 第 4 参, 或第 3 参 (无 mode 时)
	--		offset[number(int)]		[可选] 无分区表时裸卷字节偏移; 默认 `0`
	--		sector_size[number(int)]	[可选] 512/1024/2048/4096; 默认 `0` (blkio 探测)
	--		mode[string]			[可选] 仅 opts 作第 3 参时全盘模式; 同 mode[string]

	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 重新挂载已登记盘符的分区模式
-- @param [in]	name[string]			盘名 (mount 时指定)
-- @param [in]	mode[string|table]		[可选] remount 模式; 省略则全盘 `"r"`; 见内联 mode 注释
-- @return	rc[number(int)], msg[string]
-- @note eg. rc, msg = kpfs.remount("D")                                  -- 全盘 `"r"`
--		rc, msg = kpfs.remount("D", "rw")                           -- 全盘 `"rw"`
--		rc, msg = kpfs.remount("D", { index = 1, mode = "rw" })     -- 单分区
--		rc, msg = kpfs.remount("D", { { index = 1, mode = "r" }, { index = 2, mode = "rw" } })
--  至少一个分区 remount 成功则 rc=0; 全部失败返回末次错误码
kpfs.remount = function (name, mode)
	-- mode[string]					[可选] 全盘 remount 模式; 省略则 `"r"`
	--		`"r"`/`"rw"`/`"w"`; `"w"` 同 `"rw"`; 填错或非许可值默认 `"r"`
	--		`"r"`  只读
	--		`"rw"`/`"w"`  先读写, 失败回退只读
	-- mode[table] 单项: 单分区 remount 项
	--		index[number(int)]		分区序号 **1-based**
	--		mode[string]			挂载模式; 填错默认 `"r"`
	-- mode[table] 数组: 批量 remount 项 `{ { index, mode }, … }`
	--		数组下标 Lua 1-based; 每项字段同单项

	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 卸载盘符
-- @param [in]	name[string]			盘名
-- @param [in]	part_index[number(int)|table]	[可选] 分区序号; 省略则整盘卸载; 见内联 part_index 注释
-- @return	rc[number(int)], msg[string]
-- @note eg. rc, msg = kpfs.umount("D")                           -- 整盘卸载
--		rc, msg = kpfs.umount("D", 1)                        -- 单分区 (第 1 分区)
--		rc, msg = kpfs.umount("D", { 1, 2, 3 })            -- 批量卸载多分区
kpfs.umount = function (name, part_index)
	-- 省略 part_index: 整盘卸载并从 env 登记表移除
	-- part_index[number(int)]		[可选] 单分区序号 **1-based**
	-- part_index[table] 数组:		分区序号数组, 如 `{ 1, 2, 3 }`
	--		元素[number(int)]		分区序号 **1-based**; 数组下标 Lua 1-based

	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 查询 env 已登记盘符的 mount 状态 (只读; 非 kpfs.probe 路径探测)
-- @param [in]	name[string]			[可选] 盘名; 省略则查全部
-- @param [in]	index[number(int)]		[可选] 分区序号 **1-based**; 须与 name 同用
-- @return info[table]					固定表; rc==0 时载荷; rc~=0 时为 {}
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. local info = kpfs.mount_info()                               -- 全部已登记盘符
--		local info, rc, msg = kpfs.mount_info("D")                  -- 单盘
--		local info, rc, msg = kpfs.mount_info("D", 2)               -- 单分区 (第 2 分区)
--		失败: return {}, rc, msg
--  只读; 固定三返回值 (禁止多义返回个数)
--  省略 name: info.mounts 为盘项数组; 指定 name: info 为单盘表; name+index: info 为单分区项表
--  盘名不存在 PFS_ENOENT; index 越界 PFS_EINVAL
kpfs.mount_info = function (name, index)
	-- 省略 name: 查当前 env 全部已登记盘符
	-- name[string]					[可选] 盘名; 仅字母 A-Z/a-z
	-- index[number(int)]			[可选] 分区序号 **1-based**; 须与 name 同用

	local rc = 0
	local msg = "OK"

	-- 查全部: info.mounts
	return {
		mounts = {						-- [必须] array     仅省略 name 时存在
			{
				name = "D",				-- [必须] string    盘名
				path = "disk.vhdx",		-- [必须] string    mount 时 image_path
				container = "vhdx",		-- [必须] string    blkio 后端名 (如 raw/vhdx)
				type = 0,				-- [必须] integer   pfs_blkio_type_e
				sector_size = 512,		-- [必须] integer   逻辑扇区大小
				size = 0,				-- [必须] integer   容器字节大小
				scheme = "gpt",			-- [必须] string    "mbr"/"gpt"/"none"
				part_count = 2,			-- [必须] integer   分区条目数; 无分区表为 1 (裸盘单槽)
				parts = {				-- [必须] array     长度 part_count
					{
						index = 1,			-- [必须] integer   1-based 分区序号
						part_offset = 0,	-- [必须] integer   字节偏移
						start_lba = 0,		-- [必须] integer   起始 LBA; 裸盘为 0
						size_lba = 0,		-- [必须] integer   扇区数; 裸盘为 0
						fs_type = "exFAT",	-- [必须] string    已 mount 时 FS 名; 未 mount 为 ""
						mounted = true,		-- [必须] boolean   本槽是否已 mount
						mount_err = 0,		-- [必须] integer   末次 mount 失败码; 成功为 0
						mode = "r",			-- [必须] string    当前实际模式 `"r"`/`"rw"`
					},
					-- {}, ...				同结构分区项 (共 part_count 项)
				},
			},
			-- {}, ...					同结构盘项
		},
	}, rc, msg

	-- 查单盘 (指定 name, 省略 index): 同上盘项表 (无 mounts 包裹)
	-- return {
	--	name = "D",					-- [必须] string    盘名
	--	path = "disk.vhdx",			-- [必须] string    mount 时 image_path
	--	container = "vhdx",			-- [必须] string    blkio 后端名
	--	type = 0,					-- [必须] integer   pfs_blkio_type_e
	--	sector_size = 512,			-- [必须] integer   逻辑扇区大小
	--	size = 0,					-- [必须] integer   容器字节大小
	--	scheme = "gpt",				-- [必须] string    "mbr"/"gpt"/"none"
	--	part_count = 2,				-- [必须] integer   分区条目数
	--	parts = {					-- [必须] array     长度 part_count; 字段同下
	--		{ index, part_offset, start_lba, size_lba, fs_type, mounted, mount_err, mode },
	--	},
	-- }, rc, msg

	-- 查单分区 (name + index): 分区项表 (无 name/path/parts 包裹)
	-- return {
	--	index = 2,					-- [必须] integer   1-based 分区序号
	--	part_offset = 0,			-- [必须] integer   字节偏移
	--	start_lba = 0,				-- [必须] integer   起始 LBA
	--	size_lba = 0,				-- [必须] integer   扇区数
	--	fs_type = "exFAT",			-- [必须] string    已 mount 时 FS 名; 未 mount 为 ""
	--	mounted = true,				-- [必须] boolean   本槽是否已 mount
	--	mount_err = 0,				-- [必须] integer   末次 mount 失败码; 成功为 0
	--	mode = "rw",				-- [必须] string    当前实际模式 `"r"`/`"rw"`
	-- }, rc, msg
end


return kpfs
```

### 示例

```lua
local kpfs = require("kpfs")
local vfs = require("kpfs.vfs")

-- 只读挂载 (省略 mode, 全盘 "r")
local rc, msg = kpfs.mount("D", "disk.vhdx")
assert(rc == 0, msg)

-- 全盘读写 (先 rw, 失败回退 r)
rc, msg = kpfs.mount("E", "disk.vhdx", "rw")
assert(rc == 0, msg)

-- 按分区独立模式: 第1分区只读, 第2分区读写
rc, msg = kpfs.mount("F", "disk.vhdx", { "r", "rw" })
assert(rc == 0, msg)

-- 裸盘镜像 (无分区表) 指定 offset
rc, msg = kpfs.mount("G", "bare.img", { offset = 0 })
assert(rc == 0, msg)

-- mount 后 VFS 盘符路径
rc, msg = vfs.mkdir("D1:/demo")
assert(rc == 0, msg)

-- remount: 省略 mode 全盘 "r"
rc, msg = kpfs.remount("D")
assert(rc == 0, msg)

-- 单槽 remount (index 1-based)
rc, msg = kpfs.remount("D", { index = 1, mode = "rw" })
assert(rc == 0, msg)

-- 批量 remount
rc, msg = kpfs.remount("D", { { index = 1, mode = "r" }, { index = 2, mode = "rw" } })
assert(rc == 0, msg)

-- umount: 省略 part_index 整盘卸载
rc, msg = kpfs.umount("E")
assert(rc == 0, msg)

-- 单分区卸载
rc, msg = kpfs.umount("F", 1)
assert(rc == 0, msg)

-- 批量卸载多分区
rc, msg = kpfs.umount("F", { 1, 2 })
assert(rc == 0, msg)

-- mount_info: 查全部 / 单盘 / 单分区
local all_info = kpfs.mount_info()
for i, m in ipairs(all_info.mounts or {}) do
    print(i, m.name, m.path, m.part_count)
end

local disk_info, rc, msg = kpfs.mount_info("D")
assert(rc == 0, msg)

local part_info, rc2, msg2 = kpfs.mount_info("D", 1)
assert(rc2 == 0, msg2)
assert(part_info.mounted)
```

制盘示例见 [kpfs.disk](kpfs_disk.md); 跨子模块完整流程见 [readme.md](readme.md) §完整流程示例.

### 注意

- 返回值与参数形态见 [readme.md](readme.md) §返回值约定
- Lua API 分区 **`index` / `part_index` / 盘符路径槽号** 均为 **1-based** (C 绑定内部转换)
- **`mount`**: 盘名仅字母、同 env 不可重复; 省略 `mode` 全盘 `"r"`; `mode` 为字符串 (`"r"`/`"rw"`/`"w"`, `"w"` 同 `"rw"`) 或分区模式数组; 可选 `opts` (`offset` 裸盘偏移、`sector_size` 512/1K/2K/4K); 或 `mount(name, path, opts)` 仅 opts; `"rw"`/`"w"` 失败可回退 `"r"`; 单槽 mount 失败不致使整盘 `rc` 失败
- **`remount`**: 省略 `mode` 全盘 `"r"`; `mode` 为字符串 (全盘) 或单项/数组表 (`index` **1-based** + `mode`); 至少一个分区成功则 `rc=0`
- **`umount`**: 省略 `part_index` 整盘卸载; `part_index` 为序号 (单分区) 或序号表 `{ 1, 2, … }` (批量); 序号 **1-based**
- **`mount_info`**: 只读; 固定 `info, rc, msg`; 省略 `name` 返回 `{ mounts = … }`; 指定 `name` 返回单盘表; `name`+`index` 返回单分区项; `index` **1-based**; `mode` 为 remount 后当前实际 `"r"`/`"rw"`; **非** `kpfs.probe` 路径试探
- **`mount` 与 CLI**: `pfs` CLI **无盘符**; 盘符 mount 为 kpfs Lua 独有能力; 路径语法见 [kpfs_vfs.md](kpfs_vfs.md)
- 制盘 (mkpt/mkfs/mkvol) 见 [kpfs.disk](kpfs_disk.md); 盘符访问见 [kpfs.vfs](kpfs_vfs.md); 须先 `mount`
