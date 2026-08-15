# pfs API 文档

> **性质**: 对外 C API 中文说明; **真源**为 `portfs/src/*.h` 与 `portfs/src_mtd/*.h` **根级**头文件 Doxygen  
> **范围**: 本目录 `*.md` (除本文件与 [overview.md](overview.md)) **仅**对应上述根级对外头文件; **不含** `src/{fat,vfs,...}/` 等子目录内头文件  
> **冲突**: 头文件 / 已实现代码 > 生成物 > 手写概览  
> **查阅顺序**: 见 **doc-writing** § API / 模块查阅顺序 (pfs)

## 生成 (须用户明确触发)

**仅**在用户说 **同步 api 文档** / **更新 api 文档** (或同义: 生成 api 文档, 跑 api 文档) 时运行; **禁止**在改头文件, 编译验证或其它任务中附带执行.

脚本位于 `.tmp/pfs-api-docs/` (可再生, 勿手改):

```powershell
powershell -ExecutionPolicy Bypass -File .tmp/pfs-api-docs/gen-api-docs.ps1
```

或 VS Code 任务 **pfs-gen-api-docs** (手动).

| 输出 | 路径 | 说明 |
|------|------|------|
| **A HTML** | `.tmp/pfs-doxygen/html/index.html` | Doxygen 完整参考 (本地浏览) |
| **B Markdown** | `klua_doc/pfs/api/pfs*.md`, `pfsmtd.md` | 根级头文件提取 (**勿手改**) |
| 手写概览 | [overview.md](overview.md) | include 策略, 典型流程 |

日常改 `src/*.h` / `src_mtd/pfsmtd.h` 只改头文件注释; **不**自动刷新本目录生成物, 待显式同步后再更新.

## 目录 (`src/` / `src_mtd/` 根级头文件)

| 文档 | 头文件 | 说明 |
|------|--------|------|
| [overview.md](overview.md) | — | include 策略, 错误码, 典型调用流程 (手写) |
| [pfs.md](pfs.md) | `portfs/src/pfs.h` | 总入口: ctx / mount, mkfs / mkvol, file / dir / path |
| [pfs_blkio.md](pfs_blkio.md) | `portfs/src/pfs_blkio.h` | 块设备与镜像 I/O facade |
| [pfs_part.md](pfs_part.md) | `portfs/src/pfs_part.h` | MBR / GPT 探测与 mkpt |
| [pfs_ops.md](pfs_ops.md) | `portfs/src/pfs_ops.h` | blkio + FS 扩展契约类型 |
| [pfs_compiler.md](pfs_compiler.md) | `portfs/src/pfs_compiler.h` | 编译器提示与结构体探测宏 |
| [pfs_endian.md](pfs_endian.md) | `portfs/src/pfs_endian.h` | 端序转换与 buffer 读写 (锁定头) |
| [pfsmtd.md](pfsmtd.md) | `portfs/src_mtd/pfsmtd.h` | MTD 扩展库对外 API |
| [HTML 索引](../../../.tmp/pfs-doxygen/html/index.html) | — | 生成后本地打开 |

## 与其它文档分工

| 文档类 | 路径 | 用途 |
|--------|------|------|
| **本目录** (`api/`) | `klua_doc/pfs/api/` | 根级对外头文件 API (手写概览 + 生成 md/html) |
| 规范镜像 | `std_{fat,exfat,ntfs}/` | 官方/公开 on-disk 规范本地镜像 |
| 个人笔记 | `pfs_{fat,exfat,ntfs}.md` | 个人对标准的理解 |
| 自测用例 | `pfs_test.md` | 控制台 harness 用例说明 |
| 设计技能 | **pfs-*-design** | 实现架构, 扩展挂接, 非 API 条文 |
