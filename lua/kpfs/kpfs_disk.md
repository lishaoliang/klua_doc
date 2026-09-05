# kpfs.disk (Lua 子模块)

> **require**: `kpfs.disk` | 代码: [portfs/src_klua/pfs_klua_disk.c](https://gitee.com/klua/portfs/blob/trunk/src_klua/pfs_klua_disk.c) (+ `pfs_klua_part.c`, `pfs_klua_mkfs.c`)
> **文档格式**: Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档
> **枢纽**: [readme.md](readme.md) (返回值、CLI 对照、完整流程) | **CLI 真源**: [pfs_tool.md](../../pfs/pfs_tool.md) §3.5–§3.7

块设备制盘写操作 (分区表 / 空卷格式化 / 目录树制卷). 与 [kpfs.image](kpfs_image.md) (容器壳) 、 [kpfs.probe](kpfs_probe.md) (只读探测) 配合使用; 挂盘符见 [kpfs](kpfs.md).

### 导出 API

写操作返回 `rc, msg` (`rc == 0` 成功). 同 probe/image 三种形态、**无** `force` (CLI `-w` 仅防误输入).


| 函数 | 返回 | 说明 |
|------|------|------|
| `mkpt(path [, opts])` | `rc, msg` | 写分区表; `opts` 为单分区表或多分区数组 |
| `mkfs(path [, opts])` | `rc, msg` | 空卷格式化; `opts` 为单分区表或多分区数组 |
| `mkvol(path [, opts])` | `rc, msg` | 从宿主目录制卷; `opts` 同上; 分区项须 `src` |


### 伪代码

```lua
--[[
-- Copyright(c) 2026, LGPL All Rights Reserved
-- @file   kpfs_disk.lua
-- @brief  C kpfs.disk
--   require("kpfs.disk")
--   C: portfs/src_klua/pfs_klua_disk.c, pfs_klua_part.c, pfs_klua_mkfs.c
-- @version 0.1
--]]

local disk = {}


-- @brief 写分区表 (破坏性; 对齐 pfs mkpt)
-- @param [in]	path[string|table]		路径字符串, 或仅 opts 表 (含 path)
-- @param [in]	opts[table]				[可选] 单分区表或多分区数组; 见内联 opts 注释
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = disk.mkpt("disk.vhdx")                              -- 默认单分区
--		rc, msg = disk.mkpt("disk.vhdx", { scheme = "gpt", fs = "exfat" })
--		rc, msg = disk.mkpt("disk.vhdx", {
--			scheme = "gpt",
--			{ fs = "exfat", size = "8G" },
--			{ fs = "ntfs", size = "16G" },
--		})
--		rc, msg = disk.mkpt("disk.vhdx", {
--			{ fs = "exfat", start_lba = 2048, size_lba = 16384000 },
--			{ fs = "ntfs" },
--		})
--		固定双返回值 (禁止多义返回个数)
--  破坏性写; 无 force opts (CLI `-w`/`--force` 仅防误输入)
--  opts[table] 单项: 制作 **单分区**; 含全局字段 + 分区项字段 (见下)
--  opts[table] 数组: `{ { … }, { … } }` 制作 **多分区**; 数组下标 Lua 1-based
--		每项为分区项表; 全局 `scheme` 可在 path+opts 时于外层同级指定 (混合表)
--  全局字段 (单分区表或混合表外层):
--		path[string]				[可选] 仅 opts 表单参时**必填**
--		scheme[string]				[可选] "mbr"(默认) / "gpt"
--  分区项字段 (单分区表与多分区数组每项; 对齐 C `pfs_mkpt_entry_t`):
--		fs[string]					[可选] FS 提示; 同义 type; 默认 `exfat`
--		type[string]				[可选] 同 fs
--		bootable[boolean]			[可选] 可引导; MBR 有效; 单分区现行实现默认 true
--		mbr_sys_ind[number(int)]	[可选] 仅 MBR; 0 或省略则按 fs 映射 sys_ind; GPT 忽略
--		start_lba[number(int)]		[可选] 起始 LBA (扇区, 相对盘首 0); 0 或省略由实现自算 (1MiB 对齐)
--		size_lba[number(int)]		[可选] 分区扇区数; 0 或省略由实现自算 (单分区占满剩余)
--		size[string|number(int)]	[可选] 分区大小 `8G`/字节; 绑定换算为 size_lba; 与 size_lba 互斥时优先 size_lba
--  fs: fat12/fat16/fat32, exfat, ntfs, isofs, udf, ext2/ext3/ext4 (大小写不敏感)
--  多分区 opts 数组已支持; 底层 `pfs_mbr_mkpt`/`pfs_gpt_mkpt` P0 暂仅 count==1 落盘, count>1 返回 PFS_EOPNOTSUPP
disk.mkpt = function (path, opts)
	-- 单分区 opts 示例:
	-- opts = {
	--   path[string]				[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	--   scheme[string]				[可选] "mbr"(默认) / "gpt"
	--   fs[string]					[可选] FS 提示; 同义 type; 默认 `exfat`
	--   type[string]				[可选] 同 fs
	--   bootable[boolean]			[可选] 可引导 (MBR)
	--   mbr_sys_ind[number(int)]	[可选] 仅 MBR; 0=按 fs 映射
	--   start_lba[number(int)]		[可选] 起始 LBA (扇区); 0=实现自算
	--   size_lba[number(int)]		[可选] 分区扇区数; 0=实现自算
	--   size[string|number(int)]	[可选] 分区大小; 换算为 size_lba
	-- }
	-- 多分区 opts 示例 (opts[1] 为第 1 分区项):
	-- opts = {
	--   scheme[string]				[可选] 混合表外层; "mbr"(默认) / "gpt"
	--   {
	--     fs[string]					[可选] 第 1 分区 FS 提示
	--     type[string]				[可选] 同 fs
	--     bootable[boolean]			[可选] 可引导 (MBR)
	--     mbr_sys_ind[number(int)]	[可选] 仅 MBR
	--     start_lba[number(int)]		[可选] 第 1 分区起始 LBA (扇区); 0=自算
	--     size_lba[number(int)]		[可选] 第 1 分区扇区数; 0=自算
	--     size[string|number(int)]	[可选] 第 1 分区大小 `8G`/字节
	--   },
	--   { fs = "ntfs", size = "16G" },	-- 第 2 分区项 (同字段)
	--   -- {}, …						第 N 分区项
	-- }

	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 空卷格式化 (破坏性; 对齐 pfs mkfs)
-- @param [in]	path[string|table]		路径字符串, 或仅 opts 表 (含 path)
-- @param [in]	opts[table]				[可选] 单分区表或多分区数组; 省略则默认单分区; 见内联 opts 注释
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = disk.mkfs("disk.vhdx")                              -- 默认单分区
--		rc, msg = disk.mkfs("disk.vhdx", { fs = "exfat" })
--		rc, msg = disk.mkfs("disk.vhdx", {
--			{ fs = "exfat" },
--			{ fs = "ntfs", size = "8G" },
--		})
--		固定双返回值 (禁止多义返回个数)
--  破坏性写; 无 force opts (CLI `-w`/`--force` 仅防误输入)
--  opts[table] 单项: 格式化 **单分区**; 含全局字段 + 分区项字段 (见下)
--  opts[table] 数组: `{ { … }, { … } }` 格式化 **多分区**; `opts[N]`→第 N 分区 (Lua 1-based)
--  全局字段 (单分区表):
--		path[string]				[可选] 仅 opts 表单参时**必填**
--  分区项字段 (单分区表与多分区数组每项):
--		fs[string]					[可选] FS 类型; 同义 type; 默认 `exfat`
--		type[string]				[可选] 同 fs
--		index[number(int)]			[可选] 分区序号 **1-based**; 省略则按数组位置或默认第 1 分区
--		offset[number(int)]			[可选] 字节偏移 (分区起点); 省略则按 index 或 probe 首分区
--		size[string|number(int)]	[可选] 卷大小 `8G`/字节; 省略则按分区容量自动推断
--  不支持 opts.label (与 CLI 一致); 带目录树用 mkvol
--  fs: fat12/fat16/fat32, exfat, ntfs, isofs, udf, ext2/ext3/ext4 (大小写不敏感)
disk.mkfs = function (path, opts)
	-- 单分区 opts 示例:
	-- opts = {
	--   path[string]				[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	--   fs[string]					[可选] FS 类型; 同义 type; 默认 `exfat`
	--   type[string]				[可选] 同 fs
	--   index[number(int)]			[可选] 分区序号 1-based; 默认 1
	--   offset[number(int)]			[可选] 字节偏移; 默认按 index/probe 解析
	--   size[string|number(int)]	[可选] 卷大小; 省略则自动推断
	-- }
	-- 多分区 opts 示例 (opts[1] 为第 1 分区项):
	-- opts = {
	--   {
	--     fs[string]					[可选] 第 1 分区 FS 类型
	--     type[string]				[可选] 同 fs
	--     index[number(int)]			[可选] 分区序号 1-based; 数组项默认等于数组下标
	--     offset[number(int)]			[可选] 第 1 分区字节偏移
	--     size[string|number(int)]	[可选] 第 1 分区卷大小
	--   },
	--   { fs = "ntfs", size = "8G" },	-- 第 2 分区项 (同字段)
	--   -- {}, …						第 N 分区项
	-- }

	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 从宿主目录树制卷 (破坏性; 对齐 pfs mkvol)
-- @param [in]	path[string|table]		路径字符串, 或仅 opts 表 (含 path)
-- @param [in]	opts[table]				[可选] 单分区表或多分区数组; 省略则默认单分区; 见内联 opts 注释
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = disk.mkvol("disk.vhdx", { fs = "exfat", src = "/data/tree", label = "VOL" })
--		rc, msg = disk.mkvol("disk.vhdx", {
--			{ fs = "exfat", src = "/tree1", label = "VOL1" },
--			{ fs = "ntfs", src = "/tree2", label = "VOL2" },
--		})
--		固定双返回值 (禁止多义返回个数)
--  破坏性写; 无 force opts (CLI `-w`/`--force` 仅防误输入)
--  opts[table] 单项: 制 **单分区** 卷; 含全局字段 + 分区项字段 (见下)
--  opts[table] 数组: `{ { … }, { … } }` 制 **多分区** 卷; `opts[N]`→第 N 分区 (Lua 1-based)
--  全局字段 (单分区表):
--		path[string]				[可选] 仅 opts 表单参时**必填**
--  分区项字段 (单分区表与多分区数组每项):
--		src[string]					**必填** 宿主目录树路径
--		fs[string]					[可选] FS 类型; 同义 type; 默认 `exfat`
--		type[string]				[可选] 同 fs
--		label[string]				[可选] 卷标
--		index[number(int)]			[可选] 分区序号 **1-based**; 省略则按数组位置或默认第 1 分区
--		offset[number(int)]			[可选] 字节偏移 (分区起点); 省略则按 index 或 probe 首分区
--		size[string|number(int)]	[可选] 卷大小; 省略则按分区容量自动推断
--  fs: fat12/fat16/fat32, exfat, ntfs, isofs, udf, ext2/ext3/ext4 (大小写不敏感)
disk.mkvol = function (path, opts)
	-- 单分区 opts 示例:
	-- opts = {
	--   path[string]				[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	--   src[string]					**必填** 宿主目录树路径
	--   fs[string]					[可选] FS 类型; 同义 type; 默认 `exfat`
	--   type[string]				[可选] 同 fs
	--   label[string]				[可选] 卷标
	--   index[number(int)]			[可选] 分区序号 1-based; 默认 1
	--   offset[number(int)]			[可选] 字节偏移; 默认按 index/probe 解析
	--   size[string|number(int)]	[可选] 卷大小; 省略则自动推断
	-- }
	-- 多分区 opts 示例 (opts[1] 为第 1 分区项):
	-- opts = {
	--   {
	--     src[string]					**必填** 第 1 分区宿主目录
	--     fs[string]					[可选] 第 1 分区 FS 类型
	--     type[string]				[可选] 同 fs
	--     label[string]				[可选] 第 1 分区卷标
	--     index[number(int)]			[可选] 分区序号 1-based
	--     offset[number(int)]			[可选] 第 1 分区字节偏移
	--     size[string|number(int)]	[可选] 第 1 分区卷大小
	--   },
	--   { fs = "ntfs", src = "/tree2", label = "VOL2" },	-- 第 2 分区项
	--   -- {}, …						第 N 分区项
	-- }

	local rc = 0
	local msg = "OK"

	return rc, msg
end


return disk
```

### 示例

```lua
local disk = require("kpfs.disk")
local image = require("kpfs.image")
local probe = require("kpfs.probe")

image.create("disk.vhdx", { size = "32G" })

local rc, msg = disk.mkpt("disk.vhdx", { scheme = "gpt", fs = "exfat" })
assert(rc == 0, msg)

rc, msg = disk.mkpt("disk.vhdx", {
    scheme = "gpt",
    { fs = "exfat", size = "8G" },
    { fs = "ntfs", size = "16G" },
})
assert(rc == 0, msg)

rc, msg = disk.mkfs("disk.vhdx")
assert(rc == 0, msg)

rc, msg = disk.mkfs("disk.vhdx", { fs = "exfat" })
assert(rc == 0, msg)

rc, msg = disk.mkfs("disk.vhdx", { { fs = "exfat" }, { fs = "ntfs", size = "8G" } })
assert(rc == 0, msg)

rc, msg = disk.mkvol("disk.vhdx", { fs = "exfat", src = "/data/tree", label = "VOL" })
assert(rc == 0, msg)

rc, msg = disk.mkvol("disk.vhdx", {
    { fs = "exfat", src = "/tree1", label = "VOL1" },
    { fs = "ntfs", src = "/tree2", label = "VOL2" },
})
assert(rc == 0, msg)

local info = probe.part("disk.vhdx")
```

跨子模块完整流程见 [readme.md](readme.md) §完整流程示例.

### 注意

- 返回值与参数形态见 [readme.md](readme.md) §返回值约定
- Lua API 分区 **`index`** 均为 **1-based** (C 绑定内部转换)
- `mkpt`/`mkfs`/`mkvol` 同 probe/image 三种形态、**无** `force`; opts 为 **单分区表** 或 **多分区数组** `{ { … }, … }`; 省略第 2 参默认 **单分区**; `opts[N]`→第 N 分区 (**1-based**); `mkpt` 分区项含 `start_lba`/`size_lba` 等; **底层 mkpt P0 暂仅单分区落盘** (`count>1` → `PFS_EOPNOTSUPP`); `mkfs` **不支持** `label`; `mkvol` 分区项 **须** `src`
- MTD FS 类型须走 [kpfs.mtd](kpfs_mtd.md), 不可用 `kpfs.disk.mkfs`
- 容器壳见 [kpfs.image](kpfs_image.md); 挂盘符见 [kpfs](kpfs.md)

---

## CLI 对照


| pfs CLI | kpfs Lua |
|---------|----------|
| `mkpt -d …` | `require("kpfs.disk").mkpt(path [, opts])` 或 `.mkpt({ path = … })` |
| `mkfs -d … -t …` | `kpfs.disk.mkfs(path [, opts])` 或 `.mkfs({ path = … })` |
| `mkvol -d … -s …` | `kpfs.disk.mkvol(path [, opts])` 或 `.mkvol({ path = … })` |


| 项 | pfs CLI | kpfs Lua |
|----|---------|----------|
| `mkpt` scheme | MBR | `scheme = "mbr"` |
| `mkpt` / `mkfs` FS 提示 | `exfat` | `fs = "exfat"` |
| 破坏性写覆盖 | `-w` / `--force` | Lua **无** `force` opts (脚本即破坏性) |
