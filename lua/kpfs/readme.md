# kpfs 扩展

> `klua_doc/lua/kpfs/` — 类别: ⑤ kpfs | 代码: `portfs/src_klua/` | 设计技能 **pfs-klua-design**
> **Lua API 文档**: 四层 (导出 API 简略 → 伪代码详注 → 示例 → 注意) — [k-bindings.md](../../klb/klua/design/k-bindings.md) § Lua API 文档; 样板 [kpfs.md](kpfs.md)
> **与 CLI**: 制盘/探测参数须与 [pfs_tool.md](../../pfs/pfs_tool.md) (**pfs-tool-design**) **保持接近**; 对照见 §4

plugins `dlopen` 注入的 klua 预加载模块. 构建期 klua **不链接** libkpfs.

---

## 1 总览

**kpfs** 是 pfs 的 **klua 扩展**: 脚本中可 **制盘/探测** (对齐 `pfs` CLI) 与 **挂载盘符** 访问分区文件.

| 项 | 说明 |
|----|------|
| Lua 模块名 | `kpfs` (`require("kpfs")`) |
| 子模块 (预加载) | `kpfs.probe`, `kpfs.disk`, `kpfs.image`, `kpfs.mtd`, `kpfs.vfs` |
| 库产物 | `libkpfs.so` / `libkpfs.dll` (含 **libpfs 全部 `PFS_API` 导出** + `klbappex_*`) |
| 挂接 | klb **plugins** `dlopen`; klua **构建期不链接** libkpfs |
| env 状态 | 每 `klua_env_t` 单例 `pfs_klua_ex_t` (L2; 脚本不可见) |

**预加载表** (`portfs/src_klua/pfs_klua_plugin.c` → `klbappex_pre_open`):

| 预加载名 | `require` | 职责 |
|----------|-----------|------|
| `kpfs` | `require("kpfs")` | 盘符 `mount` / `remount` / `umount` |
| `kpfs.vfs` | `require("kpfs.vfs")` | 盘符路径文件/目录与 path 级 API |
| `kpfs.probe` | `require("kpfs.probe")` | 容器/分区/FS 探测 (只读) |
| `kpfs.disk` | `require("kpfs.disk")` | 块设备 `mkpt` / `mkfs` / `mkvol` |
| `kpfs.image` | `require("kpfs.image")` | 虚拟磁盘 `create` / `info` |
| `kpfs.mtd` | `require("kpfs.mtd")` | MTD 格式化与 `ubi` / `raw` 制盘 |

**盘符路径** (`mount` 后): `"D2:/aa/bb.txt"` — 盘名 + **1-based** 分区槽 + `/` 分隔; 详见 [kpfs_vfs.md](kpfs_vfs.md).

---

## 2 加载与部署

```text
klb/klua 进程 (仅链 libklb)
  → enable_plugins + plugins 扫描目录
  → dlopen libkpfs.so
  → klbappex_pre_open → package.loaded["kpfs", "kpfs.probe", …]
  → require("kpfs")
```

| 项 | 约定 |
|----|------|
| 插件符号 | `klbappex_pre_count`, `klbappex_pre_open` |
| 部署 | `libkpfs.so` 放入 app **plugins** 目录 |
| 编译 | `cd portfs && make kpfs` (或 `make all`) |
| 符号 | 动态导出与 `libpfs.so` 一致 (`pfs_*` / `pfsmtd_*`) + 插件入口 `klbappex_pre_*` |

`src_klua/Makefile` 通过 `--whole-archive libpfs.a` 编入 libpfs 全部对象; 插件 `.so` **不**使用 `--no-undefined` (lua 由 klua 宿主解析). 详见 **pfs-klua-design**.

详见 `klua_doc/klb/klbapp/design/plugins.md` § libkpfs.so.

---

## 3 返回值约定

| 类别 | 成功 | 失败 |
|------|------|------|
| **写操作** (`mkpt`, `mkfs`, `mount`, …) | `rc == 0`, `msg` 为 `pfs_strerror(0)` | `rc` 为负 `PFS_*`, `msg` 为 `pfs_strerror(rc)` |
| **探测 / 只读 info** (`kpfs.probe.*`, `kpfs.image.info`) | `info` 为载荷表, `rc == 0`, `msg` 为 `pfs_strerror(0)` | `info` 为 `{}`, `rc` 为负 `PFS_*`, `msg` 为 `pfs_strerror(rc)` |

破坏性写操作: `kpfs.disk.*` / `kpfs.mtd.*` (除 `mtd` 部分 API) 与 **`image.create`** 脚本侧**无** `force` opts (脚本即破坏性); CLI 仍须 `-w` / `--force` 防误输入. `mtd.ubi.create` / `mtd.raw.create` opts 须 **`force = true`**.

### 3.1 参数形态

多数「路径 + 可选 opts」API 支持两种调用:

| 形态 | 示例 |
|------|------|
| 路径字符串 | `probe.all("disk.vhdx")` |
| 路径 + opts 表 | `probe.all("disk.vhdx", { offset = 0 })` |
| 仅 opts 表 (`path` 字段) | `probe.all({ path = "disk.vhdx", offset = 0 })` |

`mkpt` / `mkfs` / `mkvol` / `image.create` 同 probe 三种形态. `mtd.*` 写操作仅 **opts 表** (`path` 必填).

**Lua 示例**: `require("kpfs")` / `require("kpfs.probe")` 见 **coding-lua** § **require（强制）**.

---

## 4 与 pfs CLI 参数对照

**原则**: kpfs opts 字段语义、默认值、分层边界与 **`pfs <command>`** 一致; 差异须在本表注明.

### 4.1 公共 opts 字段

| Lua opts | pfs CLI | 说明 |
|----------|---------|------|
| `path` | `-d <path>` 或位置参数 | 设备/镜像路径 (UTF-8) |
| `offset` | `-o <offset>` | 字节偏移; 默认 `0` |
| `force` | `-w` / `--force` | `mtd.ubi.create` / `mtd.raw.create` **必填** `true`; **`kpfs.disk.*` / `image.create` 无此字段** |
| `type` / `fs` | `-t <type>` | FS 名; `fs` 与 `type` 同义 (优先 `fs`) |
| `size` | `-s <size>` | `8G` / `32GiB` 或整数字节; mkfs/mtd 省略则自动推断 |
| `label` | `--label` | `mkvol` 卷标; mtd 可用 `volume_id` |
| `src` | `mkvol -s <dir>` | 宿主目录树 |
| `scheme` | `-p mbr\|gpt` 或 `mkpt gpt` | 默认 `mbr` |
| `layout` | `--fixed` / `--dynamic` | 仅 `image.create`: `fixed` / `dynamic` |

**未映射到 Lua** (CLI 独有): `-q` / `-v` (脚本侧自行处理输出).

### 4.2 命令对照

| pfs CLI               | kpfs Lua                                                 | 状态                                           |
| --------------------- | -------------------------------------------------------- | -------------------------------------------- |
| `probe <path>`        | `require("kpfs.probe").all(path)` 或 `.all({ path = … })` | 已实现                                          |
| `probe part -d …`     | `kpfs.probe.part(…)`                                     | 已实现                                          |
| `probe fs -d …`       | `kpfs.probe.fs(…)` (仅首分区)                                | 已实现                                          |
| `image create <path>` | `kpfs.image.create(path, opts)`                          | 已实现                                          |
| `image info <path>`   | `kpfs.image.info(path)`                                  | 已实现                                          |
| `mkpt -d …`           | `kpfs.disk.mkpt(path [, opts])` 或 `.mkpt({ path = … })` | 已实现                                          |
| `mkfs -d … -t …`      | `kpfs.disk.mkfs(…)`                                       | 已实现                                          |
| `mkvol -d … -s …`     | `kpfs.disk.mkvol(…)`                                      | 已实现                                          |
| `mtd mkfs`            | `kpfs.mtd.mkfs({ … })`                                   | 已实现                                          |
| `mtd mkvol`           | `kpfs.mtd.mkvol({ … })`                                  | 已实现                                          |
| `mtd ubi create`      | `kpfs.mtd.ubi.create({ … })`                             | 已实现                                          |
| `mtd raw create`      | `kpfs.mtd.raw.create({ … })`                             | 已实现                                          |
| `ls` / `cat` / …      | `kpfs.vfs`                                               | **已实现** (`open`/`read`/`opendir`/`access` 等) |
| `mount` (盘符)          | `kpfs.mount(name, path, mode)`                           | 已实现; **CLI 无盘符**                             |

### 4.3 默认值对照

| 项 | pfs CLI | kpfs Lua |
|----|---------|----------|
| `image create` 类型 | `vhdx` | `type` opts 或 path 后缀; 仍无则 `vhdx` |
| `image create` 容量 | `8G` | `size = "8G"` |
| `image create` 布局 | `dynamic` | `layout = "dynamic"` |
| `mkpt` scheme | MBR | `scheme = "mbr"` |
| `mkpt` / `mkfs` FS 提示 | `exfat` | `fs = "exfat"` |
| `mtd mkfs` 默认 FS | 依子命令 | `fs = "squashfs"` |
| `mtd ubi create` LEB | 128KiB × 64 | `leb_size` / `leb_count` 可省略 |

### 4.4 CLI 示例 → Lua 示例

```bash
pfs image create disk.vhdx -w
pfs mkpt gpt -d disk.vhdx -t exfat -w
pfs mkfs -d disk.vhdx -t exfat -w
pfs probe disk.vhdx
```

```lua
local kpfs = require("kpfs")
local disk = require("kpfs.disk")
local probe = require("kpfs.probe")
local image = require("kpfs.image")

image.create("disk.vhdx")
disk.mkpt({ path = "disk.vhdx", scheme = "gpt", fs = "exfat" })
disk.mkfs({ path = "disk.vhdx", fs = "exfat" })
local info = probe.all("disk.vhdx")
```

---

## 5 完整流程示例

```lua
local kpfs = require("kpfs")
local disk = require("kpfs.disk")
local vfs = require("kpfs.vfs")
local image = require("kpfs.image")
local probe = require("kpfs.probe")

-- 制盘 (对齐 pfs CLI 主路径)
image.create("d.vhdx", { size = "32G" })
disk.mkpt({ path = "d.vhdx", scheme = "gpt", fs = "exfat" })
disk.mkfs({ path = "d.vhdx", fs = "exfat" })

local info = probe.all("d.vhdx")
for i, p in ipairs(info.parts or {}) do
    print(i, p.fs_type, p.mount_ok)
end

-- 挂盘符 + VFS (Lua 独有; CLI 无盘符)
local rc, msg = kpfs.mount("D", "d.vhdx")
assert(rc == 0, msg)

rc, msg = vfs.mkdir("D1:/demo")
assert(rc == 0, msg)

local f = vfs.open("D1:/demo/hello.txt", "w")
f:write("kpfs")
f:close()

rc, msg = kpfs.remount("D", "rw")
assert(rc == 0, msg)
rc, msg = kpfs.umount("D")
assert(rc == 0, msg)
```

---

## 6 文档索引

| 文件 | require | 说明 |
|------|---------|------|
| [kpfs.md](kpfs.md) | `kpfs` | 根模块: 盘符 mount / remount / umount |
| [kpfs_disk.md](kpfs_disk.md) | `kpfs.disk` | 块设备 mkpt / mkfs / mkvol |
| [kpfs_probe.md](kpfs_probe.md) | `kpfs.probe` | 容器/分区/FS 探测 |
| [kpfs_image.md](kpfs_image.md) | `kpfs.image` | 虚拟磁盘 create / info |
| [kpfs_mtd.md](kpfs_mtd.md) | `kpfs.mtd` | MTD 格式化与 ubi / raw |
| [kpfs_vfs.md](kpfs_vfs.md) | `kpfs.vfs` | 盘符路径文件/目录 |

---

## 7 相关文档

| 文档 | 内容 |
|------|------|
| [pfs_tool.md](../../pfs/pfs_tool.md) | `pfs` CLI 真源 |
| [pfs/api/readme.md](../../pfs/api/readme.md) | pfs C API |
| [index-by-require.md](../index-by-require.md) | §⑤ 索引 |
| `klua_doc/klb/klua/design/env-extension.md` | env 扩展; § kpfs |
| `klua_doc/klb/klbapp/design/plugins.md` | plugins 加载 libkpfs |
