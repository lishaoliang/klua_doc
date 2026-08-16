# kpfs.mtd (Lua 子模块)

> **require**: `kpfs.mtd` | 代码: `portfs/src_klua/pfs_klua_mtd.c`
> **文档格式**: Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档
> **枢纽**: [readme.md](readme.md) (返回值、CLI 对照) | **CLI 真源**: [pfs_tool.md](../../pfs/pfs_tool.md) §3.9 | 设计 **pfsmtd-design**

### 导出 API

写操作返回 `rc, msg` (`rc == 0` 成功). `mkfs`/`mkvol` 同 [kpfs.disk](kpfs_disk.md) 三种形态、**无** `force` (CLI `-w` 仅防误输入). `ubi.create`/`raw.create` 仅 **opts 表** 且须 **`force = true`**. MTD `fs` **禁止** 走 `kpfs.disk.mkfs`.

| 函数 | 返回 | 说明 |
|------|------|------|
| `mkfs(path [, opts])` | `rc, msg` | MTD 格式化; `opts` 为单分区表或多分区数组 |
| `mkvol(path [, opts])` | `rc, msg` | 从目录树制 MTD 卷; `opts` 同上; 分区项须 `src` |
| `ubi.create(opts)` | `rc, msg` | 创建 UBI 镜像 |
| `raw.create(opts)` | `rc, msg` | 创建 raw MTD 镜像 |

### 伪代码

```lua
--[[
-- Copyright(c) 2026, LGPL All Rights Reserved
-- @file   kpfs_mtd.lua
-- @brief  C kpfs.mtd
--   require("kpfs.mtd")
--   C: portfs/src_klua/pfs_klua_mtd.c
-- @version 0.1
--]]

local mtd = {}


-- @brief MTD 格式化 (破坏性; 对齐 pfs mtd mkfs)
-- @param [in]	path[string|table]		路径字符串, 或仅 opts 表 (含 path)
-- @param [in]	opts[table]				[可选] 单分区表或多分区数组; 省略则默认单分区; 见内联 opts 注释
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = mtd.mkfs("flash.img")                              -- 默认单分区
--		rc, msg = mtd.mkfs("flash.img", { fs = "squashfs" })
--		rc, msg = mtd.mkfs("flash.img", {
--			{ fs = "squashfs" },
--			{ fs = "jffs2", size = "8M" },
--		})
--		固定双返回值 (禁止多义返回个数)
--  破坏性写; 无 force opts (CLI `-w`/`--force` 仅防误输入)
--  opts[table] 单项: 格式化 **单分区**; 含全局字段 + 分区项字段 (见下)
--  opts[table] 数组: `{ { … }, { … } }` 格式化 **多分区**; `opts[N]`→第 N 分区 (Lua 1-based)
--  全局字段 (单分区表):
--		path[string]				[可选] 仅 opts 表单参时**必填**
--  分区项字段 (单分区表与多分区数组每项):
--		fs[string]					[可选] MTD FS 名; 同义 type; 默认 `squashfs`
--		type[string]				[可选] 同 fs
--		index[number(int)]			[可选] 分区序号 **1-based**; 省略则按数组位置或默认第 1 分区
--		offset[number(int)]			[可选] 字节偏移 (分区起点); 省略则按 index 或 probe 首分区
--		size[string|number(int)]	[可选] 卷大小 `8G`/字节; 省略则按分区容量自动推断
--  不支持 opts.label (与 CLI 一致); 带目录树用 mkvol
--  fs: squashfs, ubifs, jffs2, yaffs/yaffs2, erofs, cramfs, romfs (大小写不敏感)
--  MTD FS **不可** 用 kpfs.disk.mkfs; 须走本函数
mtd.mkfs = function (path, opts)
	-- 单分区 opts 示例:
	-- opts = {
	--   path[string]				[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	--   fs[string]					[可选] MTD FS 名; 同义 type; 默认 `squashfs`
	--   type[string]				[可选] 同 fs
	--   index[number(int)]			[可选] 分区序号 1-based; 默认 1
	--   offset[number(int)]			[可选] 字节偏移; 默认按 index/probe 解析
	--   size[string|number(int)]	[可选] 卷大小; 省略则自动推断
	-- }
	-- 多分区 opts 示例 (opts[1] 为第 1 分区项):
	-- opts = {
	--   {
	--     fs[string]					[可选] 第 1 分区 MTD FS 名
	--     type[string]				[可选] 同 fs
	--     index[number(int)]			[可选] 分区序号 1-based; 数组项默认等于数组下标
	--     offset[number(int)]			[可选] 第 1 分区字节偏移
	--     size[string|number(int)]	[可选] 第 1 分区卷大小
	--   },
	--   { fs = "jffs2", size = "8M" },	-- 第 2 分区项 (同字段)
	--   -- {}, …						第 N 分区项
	-- }

	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 从宿主目录树制 MTD 卷 (破坏性; 对齐 pfs mtd mkvol)
-- @param [in]	path[string|table]		路径字符串, 或仅 opts 表 (含 path)
-- @param [in]	opts[table]				[可选] 单分区表或多分区数组; 省略则默认单分区; 见内联 opts 注释
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = mtd.mkvol("flash.img", { fs = "romfs", src = "/tree", label = "VOL" })
--		rc, msg = mtd.mkvol("flash.img", {
--			{ fs = "squashfs", src = "/tree1", label = "VOL1" },
--			{ fs = "jffs2", src = "/tree2", label = "VOL2", jffs2_erase_size = 65536 },
--		})
--		固定双返回值 (禁止多义返回个数)
--  破坏性写; 无 force opts (CLI `-w`/`--force` 仅防误输入)
--  opts[table] 单项: 制 **单分区** 卷; 含全局字段 + 分区项字段 (见下)
--  opts[table] 数组: `{ { … }, { … } }` 制 **多分区** 卷; `opts[N]`→第 N 分区 (Lua 1-based)
--  全局字段 (单分区表):
--		path[string]				[可选] 仅 opts 表单参时**必填**
--  分区项字段 (单分区表与多分区数组每项):
--		src[string]					**必填** 宿主目录树路径
--		fs[string]					[可选] MTD FS 名; 同义 type; 默认 `squashfs`
--		type[string]				[可选] 同 fs
--		label[string]				[可选] 卷标; 同义 volume_id (优先 label)
--		volume_id[string]			[可选] 同 label; 仅 label 省略时采用
--		index[number(int)]			[可选] 分区序号 **1-based**; 省略则按数组位置或默认第 1 分区
--		offset[number(int)]			[可选] 字节偏移 (分区起点); 省略则按 index 或 probe 首分区
--		size[string|number(int)]	[可选] 卷大小; 省略则按分区容量自动推断
--		jffs2_erase_size[number(int)]	[可选] jffs2 擦除块大小; 默认 `0` (由后端决定)
--  fs: squashfs, ubifs, jffs2, yaffs/yaffs2, erofs, cramfs, romfs (大小写不敏感)
mtd.mkvol = function (path, opts)
	-- 单分区 opts 示例:
	-- opts = {
	--   path[string]				[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	--   src[string]					**必填** 宿主目录树路径
	--   fs[string]					[可选] MTD FS 名; 同义 type; 默认 `squashfs`
	--   type[string]				[可选] 同 fs
	--   label[string]				[可选] 卷标; 同义 volume_id (优先 label)
	--   volume_id[string]			[可选] 同 label; 仅 label 省略时采用
	--   index[number(int)]			[可选] 分区序号 1-based; 默认 1
	--   offset[number(int)]			[可选] 字节偏移; 默认按 index/probe 解析
	--   size[string|number(int)]	[可选] 卷大小; 省略则自动推断
	--   jffs2_erase_size[number(int)]	[可选] jffs2 擦除块; 默认 `0`
	-- }
	-- 多分区 opts 示例 (opts[1] 为第 1 分区项):
	-- opts = {
	--   {
	--     src[string]					**必填** 第 1 分区宿主目录
	--     fs[string]					[可选] 第 1 分区 MTD FS 名
	--     type[string]				[可选] 同 fs
	--     label[string]				[可选] 第 1 分区卷标; 同义 volume_id (优先 label)
	--     volume_id[string]			[可选] 同 label; 仅 label 省略时采用
	--     index[number(int)]			[可选] 分区序号 1-based
	--     offset[number(int)]			[可选] 第 1 分区字节偏移
	--     size[string|number(int)]	[可选] 第 1 分区卷大小
	--     jffs2_erase_size[number(int)]	[可选] 第 1 分区 jffs2 擦除块
	--   },
	--   { fs = "jffs2", src = "/tree2", label = "VOL2" },	-- 第 2 分区项
	--   -- {}, …						第 N 分区项
	-- }

	local rc = 0
	local msg = "OK"

	return rc, msg
end


mtd.ubi = {}


-- @brief 创建 UBI 镜像 (破坏性; 对齐 pfs mtd ubi create)
-- @param [in]	opts[table]				见内联 opts 注释; path/force 必填
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = mtd.ubi.create({ path = "ubi.img", force = true })
--		leb_size 须 >=512 且 512 对齐; leb_count 须 >=1
mtd.ubi.create = function (opts)
	-- opts = {
	--   path[string]					**必填** 输出镜像路径 (UTF-8)
	--   force[boolean]				**必填** true; 破坏性写 (对齐 CLI `-w`)
	--   leb_size[number(int)]			[可选] LEB 字节大小, 默认 `131072` (128KiB); 须 >=512 且 512 对齐
	--   leb_count[number(int)]		[可选] LEB 个数, 默认 `64`; 须 >=1
	-- }

	local rc = 0
	local msg = "OK"

	return rc, msg
end


mtd.raw = {}


-- @brief 创建 raw MTD 镜像 (破坏性; 对齐 pfs mtd raw create)
-- @param [in]	opts[table]				见内联 opts 注释; path/size/force 必填
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = mtd.raw.create({ path = "raw.bin", size = "32M", force = true })
--		size 须 >0 且 512 字节对齐
mtd.raw.create = function (opts)
	-- opts = {
	--   path[string]					**必填** 输出镜像路径 (UTF-8)
	--   size[string|number(int)]		**必填** 镜像字节大小, 如 `32M`/整数字节; 须 >0 且 512 对齐
	--   force[boolean]				**必填** true; 破坏性写 (对齐 CLI `-w`)
	-- }

	local rc = 0
	local msg = "OK"

	return rc, msg
end


return mtd
```

### 示例

```lua
local mtd = require("kpfs.mtd")

local rc, msg = mtd.mkfs("flash.img")
rc, msg = mtd.mkfs("flash.img", { fs = "squashfs" })
rc, msg = mtd.mkfs("flash.img", { { fs = "squashfs" }, { fs = "jffs2", size = "8M" } })

rc, msg = mtd.mkvol("flash.img", { fs = "romfs", src = "/tree", label = "VOL" })
rc, msg = mtd.mkvol("flash.img", {
    { fs = "squashfs", src = "/tree1", label = "VOL1" },
    { fs = "jffs2", src = "/tree2", label = "VOL2", jffs2_erase_size = 65536 },
})

rc, msg = mtd.ubi.create({ path = "ubi.img", force = true })
rc, msg = mtd.ubi.create({
    path = "ubi.img",
    leb_size = 131072,
    leb_count = 64,
    force = true,
})

rc, msg = mtd.raw.create({ path = "raw.bin", size = "32M", force = true })
assert(rc == 0, msg)
```

### 注意

- 返回值与参数形态见 [readme.md](readme.md) §返回值约定
- `mkfs`/`mkvol` 同根模块三种形态、**无** `force`; opts 为 **单分区表** 或 **多分区数组** `{ { … }, … }`; 省略第 2 参默认 **单分区**; `opts[N]`→第 N 分区 (**1-based**)
- `mkfs` **不支持** `label`; `mkvol` 分区项 **须** `src`; `label` 与 `volume_id` 同义 (**优先** `label`); jffs2 可设 `jffs2_erase_size`
- MTD FS 类型 **不可** 用 `kpfs.disk.mkfs`; 须走 `kpfs.mtd.mkfs`
- `ubi.create`/`raw.create` 仅 **opts 表** 且须 **`force = true`** (对齐 CLI `-w`)

---

## opts 字段速查

> 字段注释以伪代码内联为准; 下表为速查.

**mkfs / mkvol 公共** (单分区表与多分区数组每项):


| 字段 | 类型 | 说明 |
|------|------|------|
| `path` | string | 设备/镜像路径; 仅 opts 表单参时 **必填** |
| `fs` / `type` | string | MTD FS 名; 同义; 默认 `squashfs` |
| `index` | integer | 分区序号 **1-based**; 数组项默认等于数组下标 |
| `offset` | integer | 字节偏移; 省略则按 index/probe 解析 |
| `size` | string \| integer | 卷大小; 省略则自动推断 |

**mkvol 专有**:


| 字段 | 类型 | 说明 |
|------|------|------|
| `src` | string | 宿主目录树 (**必填**) |
| `label` | string | 卷标 (可选); 同义 `volume_id` (**优先** `label`) |
| `volume_id` | string | 同 `label`; 仅 `label` 省略时采用 |
| `jffs2_erase_size` | integer | jffs2 擦除块; 默认 `0` |

**ubi.create**:


| 字段 | 类型 | 说明 |
|------|------|------|
| `path` | string | 输出路径 (**必填**) |
| `force` | boolean | (**必填** `true`) |
| `leb_size` | integer | 默认 `131072`; 须 >=512 且 512 对齐 |
| `leb_count` | integer | 默认 `64`; 须 >=1 |

**raw.create**:


| 字段 | 类型 | 说明 |
|------|------|------|
| `path` | string | 输出路径 (**必填**) |
| `size` | string \| integer | 镜像大小 (**必填**); 须 >0 且 512 对齐 |
| `force` | boolean | (**必填** `true`) |

---

## CLI 对照

| pfs CLI | kpfs Lua |
|---------|----------|
| `mtd mkfs` | `kpfs.mtd.mkfs("…" [, { … }])` |
| `mtd mkvol` | `kpfs.mtd.mkvol("…" [, { … }])` |
| `mtd ubi create` | `kpfs.mtd.ubi.create({ … })` |
| `mtd raw create` | `kpfs.mtd.raw.create({ … })` |

| 项 | pfs CLI | kpfs Lua |
|----|---------|----------|
| `mtd mkfs` 默认 FS | 依子命令 | `fs = "squashfs"` |
| `mtd ubi create` LEB | 128KiB × 64 | `leb_size` / `leb_count` 可省略 |
