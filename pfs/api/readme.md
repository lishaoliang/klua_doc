# pfs API 文档

> **性质**: 对外 C API 中文说明; **真源**为 [portfs/src/*.h](https://gitee.com/klua/portfs/blob/trunk/src/*.h) 与 [portfs/src_mtd/*.h](https://gitee.com/klua/portfs/blob/trunk/src_mtd/*.h) **根级**头文件 Doxygen  
> **范围**: 本目录 `*.md` (除本文件与 [overview.md](overview.md)) **仅**对应上述根级对外头文件; **不含** `src/{fat,vfs,...}/` 等子目录内头文件  
> **冲突**: 头文件 / 已实现代码 > 生成物 > 手写概览

## 生成

本目录 `pfs*.md` / `pfsmtd.md` 由根级头文件 Doxygen 注释生成, **勿手改**. 手写仅 [overview.md](overview.md) 与本文件.

日常改 `src/*.h` / `src_mtd/pfsmtd.h` 只改头文件注释; 生成物需另行刷新.

## 目录 (`src/` / `src_mtd/` 根级头文件)

| 文档 | 头文件 | 说明 |
|------|--------|------|
| [overview.md](overview.md) | — | include 策略, 错误码, 典型调用流程 (手写) |
| [pfs.md](pfs.md) | [portfs/src/pfs.h](https://gitee.com/klua/portfs/blob/trunk/src/pfs.h) | 总入口: ctx / mount, mkfs / mkvol, file / dir / path |
| [pfs_blkio.md](pfs_blkio.md) | [portfs/src/pfs_blkio.h](https://gitee.com/klua/portfs/blob/trunk/src/pfs_blkio.h) | 块设备与镜像 I/O facade |
| [pfs_part.md](pfs_part.md) | [portfs/src/pfs_part.h](https://gitee.com/klua/portfs/blob/trunk/src/pfs_part.h) | MBR / GPT 探测与 mkpt |
| [pfs_ops.md](pfs_ops.md) | [portfs/src/pfs_ops.h](https://gitee.com/klua/portfs/blob/trunk/src/pfs_ops.h) | blkio + FS 扩展契约类型 |
| [pfs_compiler.md](pfs_compiler.md) | [portfs/src/pfs_compiler.h](https://gitee.com/klua/portfs/blob/trunk/src/pfs_compiler.h) | 编译器提示与结构体探测宏 |
| [pfs_endian.md](pfs_endian.md) | [portfs/src/pfs_endian.h](https://gitee.com/klua/portfs/blob/trunk/src/pfs_endian.h) | 端序转换与 buffer 读写 (锁定头) |
| [pfsmtd.md](pfsmtd.md) | [portfs/src_mtd/pfsmtd.h](https://gitee.com/klua/portfs/blob/trunk/src_mtd/pfsmtd.h) | MTD 扩展库对外 API |

## 与其它文档分工

| 文档类 | 路径 | 用途 |
|--------|------|------|
| **本目录** (`api/`) | `klua_doc/pfs/api/` | 根级对外头文件 API (手写概览 + 生成 md) |
| 规范镜像 | `std_{fat,exfat,ntfs}/` | 官方/公开 on-disk 规范本地镜像 |
| 个人笔记 | `pfs_{fat,exfat,ntfs}.md` | 个人对标准的理解 |
| 自测用例 | `pfs_test.md` | 控制台 harness 用例说明 |
