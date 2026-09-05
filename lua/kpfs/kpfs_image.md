# kpfs.image (Lua 子模块)

> **require**: `kpfs.image` | 代码: [portfs/src_klua/pfs_klua_image.c](https://gitee.com/klua/portfs/blob/trunk/src_klua/pfs_klua_image.c)
> **文档格式**: Lua API 四层 (导出 API → 伪代码 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档
> **枢纽**: [readme.md](readme.md) (返回值、CLI 对照) | **CLI 真源**: [pfs_tool.md](../../pfs/pfs_tool.md) §3.4

### 导出 API

`create` 为破坏性写操作, 固定返回 **`rc, msg`**. `info` 为只读, 固定返回 **`info, rc, msg`** (`info` 恒为表; `rc==0` 时为载荷, `rc~=0` 时为 `{}`). 参数形态见 [readme.md](readme.md) §3.1.


| 函数 | 返回 | 说明 |
|------|------|------|
| `create(path [, opts])` | `rc, msg` | 新建虚拟磁盘 (脚本侧即破坏性; 无 `force` opts) |
| `info(path [, opts])` | `info, rc, msg` | 容器元数据 (只读) |


### 伪代码

```lua
--[[
-- Copyright(c) 2026, LGPL All Rights Reserved
-- @file   kpfs_image.lua
-- @brief  C kpfs.image
--   require("kpfs.image")
--   C: portfs/src_klua/pfs_klua_image.c
-- @version 0.1
--]]

local image = {}


-- @brief 新建虚拟磁盘 (破坏性; 对齐 pfs image create)
-- @param [in]	path[string|table]		路径字符串, 或仅 opts 表 (含 path)
-- @param [in]	opts[table]				[可选] 见内联 opts 注释; path+opts 双参时为本参
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. rc, msg = image.create("disk.vhdx")
--		rc, msg = image.create("disk.qcow2", { size = "32G" })
--  破坏性写; 无 force opts (CLI `-w`/`--force` 仅防误输入)
--  type 解析: 1) opts.type 已知名优先; 2) 未填/填错则 path 后缀; 3) 仍无则 "vhdx"
--  vmdk/vdi 返回 PFS_EOPNOTSUPP; 参数形态见 readme §3.1
image.create = function (path, opts)
	-- opts = {
	--   path[string]				[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	--   type[string]				[可选] vhd/vhdx/qcow2; 未填/填错时从 path 后缀推测; 仍无则 "vhdx"
	--   size[string|number(int)]	[可选] 默认 "8G"; 须 512 对齐
	--   layout[string]			[可选] "fixed"/"dynamic"(默认); qcow2 仅 dynamic
	-- }

	local rc = 0
	local msg = "OK"

	return rc, msg
end


-- @brief 读取容器元数据 (只读)
-- @param [in]	path[string|table]		同 create
-- @param [in]	opts[table]				[可选] 见内联 opts 注释
-- @return info[table]					固定表; rc==0 时载荷; rc~=0 时为 {}
-- @return rc[number(int)]				0 成功; 负值为 PFS_*
-- @return msg[string]					pfs_strerror(rc)
-- @note eg. local info = image.info("disk.vhdx")
--		或 local info, rc, msg = image.info("disk.vhdx"); if rc == 0 then ... end
--		失败: return {}, rc, msg
--  只读; 固定三返回值 (禁止多义返回个数)
image.info = function (path, opts)
	-- opts = {
	--   path[string]				[可选] 设备/镜像路径; 仅 opts 表单参时**必填**
	-- }

	local rc = 0
	local msg = "OK"

	return {
		path = path,					-- [必须] string    输入路径
		container = "vhdx",				-- [必须] string    blkio 后端名 (如 raw/vhdx)
		type = 0,						-- [必须] integer   pfs_blkio_type_e
		sector_size = 512,				-- [必须] integer   逻辑扇区大小
		size = 0,						-- [必须] integer   容器字节大小
	}, rc, msg
end


return image
```

### 示例

```lua
local image = require("kpfs.image")

local rc, msg = image.create("disk.vhdx")
assert(rc == 0, msg)

rc, msg = image.create("big.qcow2", {
    size = "32G",
    layout = "dynamic",
})
assert(rc == 0, msg)

-- path 后缀推测 type (可省略 type)
rc, msg = image.create("disk.vhd", { size = "4M" })
assert(rc == 0, msg)

-- 只关心 info 时可简写
local info = image.info("disk.vhdx")
print(info.container, info.size)

local info2, rc2, msg2 = image.info({ path = "disk.vhdx" })
if rc2 ~= 0 then
    print(msg2)
end
```

### 注意

- `create` 支持 `vhd` / `vhdx` / `qcow2`; `vmdk`/`vdi` 返回 `PFS_EOPNOTSUPP`
- `create` **无** `force` opts: 脚本调用即破坏性; CLI `image create` 覆盖已有文件仍须 `-w`/`--force` (防误输入)
- 参数形态与 [readme.md](readme.md) §3.1 一致 (路径字符串 / 路径+opts / 仅 opts 表)
- `info` 固定返回 **`info, rc, msg`** 三值 (`info` 恒为表; `rc~=0` 时为 `{}`); 禁止多义返回个数
- 写操作 `rc, msg` 约定见 [readme.md](readme.md) §3
- 制盘后续 (分区/格式化) 见 [kpfs.disk](kpfs_disk.md)

---

## info 表字段

> 字段注释以伪代码内联为准; 下表为速查.

**公共** (容器级):


| 字段 | 类型 | 说明 |
|------|------|------|
| `path` | string | 输入路径 |
| `container` | string | blkio 后端名 (如 `raw`, `vhdx`) |
| `type` | integer | `pfs_blkio_type_e` 枚举值 |
| `sector_size` | integer | 逻辑扇区; 缺省视为 512 |
| `size` | integer | 容器字节大小 |

---

## CLI 对照


| pfs CLI | kpfs Lua |
|---------|----------|
| `image create <path>` | `require("kpfs.image").create(path [, opts])` 或 `.create({ path = … })` |
| `image info <path>` | `kpfs.image.info(path)` 或 `.info({ path = … })` |


| 项 | pfs CLI | kpfs Lua |
|----|---------|----------|
| `image create` 类型 | `vhdx` (`-t` 或默认) | `type` opts 或 path 后缀; 仍无则 `vhdx` |
| `image create` 容量 | `8G` | `size = "8G"` |
| `image create` 布局 | `dynamic` | `layout = "dynamic"` |
| `image create` 覆盖 | `-w` / `--force` | Lua **无** `force` opts (脚本即破坏性) |
