# API 总览

> 头文件真源: [portfs/src/*.h](https://gitee.com/klua/portfs/blob/trunk/src/*.h) 与 [portfs/src_mtd/pfsmtd.h](https://gitee.com/klua/portfs/blob/trunk/src_mtd/pfsmtd.h) **根级**对外头文件 (子目录内头文件不在 `docs/api/` 范围)

## include 策略

应用默认:

```c
#include "pfs.h"
```

- `pfs.h` 聚合 `pfs_ops.h` / `pfs_blkio.h` / `pfs_part.h`, 并声明 vfs 段 (ctx / mkfs / file / dir / path)
- 分册头可单独 include; 实现分别位于 `vfs/`, `base/` (blkio facade), `partitions/`
- MTD 扩展: `#include "pfsmtd.h"` ([portfs/src_mtd/](https://gitee.com/klua/portfs/tree/trunk/src_mtd/)); 依赖 `-I portfs/src` + `-I portfs/src_mtd`
- `pfs_compiler.h` / `pfs_endian.h` 为库内/扩展常用辅助; 一般经其它头间接 include

## 错误码

`src` 根对外 API 若返回错误码, 一律为 `pfs_ret_e` (`PFS_OK` / `PFS_*`); 见 `pfs_strerror`.

| 码 | 含义 |
|----|------|
| `PFS_OK` (0) | 成功 |
| `PFS_EPERM` … `PFS_EOPNOTSUPP` | 负 errno 对齐 linux-7.1 `asm-generic/errno*.h` |

blkio facade 另有 `PFS_BLKIO_RET_*` 局部约定; 经 vfs 封装后对外仍映射为 `pfs_ret_e`.

## 典型流程

1. **块设备**: `pfs_blkio_init` → `pfs_blkio_open` (或 `open_ex` / `open_ops`)
2. **分区** (可选): `pfs_part_probe` → 取 `part_offset`
3. **挂载**: `pfs_ctx_create` → `pfs_ctx_mount` (或先 `pfs_ctx_register_fs` 注册扩展)
4. **访问**: `pfs_fopen` / `pfs_opendir` / `pfs_access` 等
5. **卸载**: `pfs_ctx_umount` → `pfs_ctx_destroy`; `pfs_blkio_close` → `pfs_blkio_deinit`

不经 ctx 的卷制作: `pfs_mkfs` / `pfs_mkvol` (须已 open 的 `pfs_blkio_t`, 且勿对已挂载卷调用).

## 分册导航 (根级头文件)

- 总入口 → [pfs.md](pfs.md)
- blkio → [pfs_blkio.md](pfs_blkio.md)
- 分区表 → [pfs_part.md](pfs_part.md)
- 扩展契约 → [pfs_ops.md](pfs_ops.md)
- 编译器宏 → [pfs_compiler.md](pfs_compiler.md)
- 端序辅助 → [pfs_endian.md](pfs_endian.md)
- MTD 扩展 → [pfsmtd.md](pfsmtd.md)
