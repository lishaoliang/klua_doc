# kpfs (根模块)

> **require**: `kpfs` | 代码: `portfs/src_klua/pfs_klua_kpfs.c`
> **文档格式**: Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档
> **枢纽**: [readme.md](readme.md) (总览、加载、返回值、CLI 对照、完整流程)

盘符运行时: 登记 env 扩展并 mount/remount/umount. 制盘 (mkpt/mkfs/mkvol) 见 [kpfs.disk](kpfs_disk.md); 容器壳见 [kpfs.image](kpfs_image.md).

### 导出 API

写操作返回 `rc, msg` (`rc == 0` 成功).


| 函数 | 返回 | 说明 |
|------|------|------|
| `version()` | string | 模块版本 `"0.1"` |
| `mount(name, image_path [, mode [, opts]])` | `rc, msg` | 登记盘符并 mount; `mode` 为字符串或分区模式数组; `opts` 含 `offset`/`sector_size` |
| `remount(name [, mode])` | `rc, msg` | 改挂载模式; `mode` 为字符串或 remount 项表 |
| `umount(name [, part_index])` | `rc, msg` | 卸载; `part_index` 为序号或序号表 |


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
	--		`"r"`/`"rw"`/`"w"`; 填错或非许可值默认 `"r"`
	--		`"r"`  只读
	--		`"rw"` 先读写, 失败回退只读
	--		`"w"`  读写, 不回退只读
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
	--		`"r"`/`"rw"`/`"w"`; 填错或非许可值默认 `"r"`
	--		`"r"`  只读
	--		`"rw"` 先读写, 失败回退只读
	--		`"w"`  读写, 不回退只读
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
```

制盘示例见 [kpfs.disk](kpfs_disk.md); 跨子模块完整流程见 [readme.md](readme.md) §完整流程示例.

### 注意

- 返回值与参数形态见 [readme.md](readme.md) §返回值约定
- Lua API 分区 **`index` / `part_index` / 盘符路径槽号** 均为 **1-based** (C 绑定内部转换)
- **`mount`**: 盘名仅字母、同 env 不可重复; 省略 `mode` 全盘 `"r"`; `mode` 为字符串 (`"r"`/`"rw"`/`"w"`) 或分区模式数组; 可选 `opts` (`offset` 裸盘偏移、`sector_size` 512/1K/2K/4K); 或 `mount(name, path, opts)` 仅 opts; `"rw"` 失败可回退 `"r"`; `"w"` 不回退; 单槽 mount 失败不致使整盘 `rc` 失败
- **`remount`**: 省略 `mode` 全盘 `"r"`; `mode` 为字符串 (全盘) 或单项/数组表 (`index` **1-based** + `mode`); 至少一个分区成功则 `rc=0`
- **`umount`**: 省略 `part_index` 整盘卸载; `part_index` 为序号 (单分区) 或序号表 `{ 1, 2, … }` (批量); 序号 **1-based**
- **`mount` 与 CLI**: `pfs` CLI **无盘符**; 盘符 mount 为 kpfs Lua 独有能力; 路径语法见 [kpfs_vfs.md](kpfs_vfs.md)
- 制盘 (mkpt/mkfs/mkvol) 见 [kpfs.disk](kpfs_disk.md); 盘符访问见 [kpfs.vfs](kpfs_vfs.md); 须先 `mount`
