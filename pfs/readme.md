# 简述
	在应用层，做跨平台文件系统

# 目标文件系统
1. FAT12
2. FAT16
3. FAT32
4. exFAT
5. NTFS

待做目标
UDF
ext4（兼挂 ext2/3）
isofs

容器格式
raw（`.img` / `.raw`）
VHD
VHDX

# 目录结构

> 文档真源: `klua_doc/pfs/` (Obsidian vault 子树). `portfs/docs/` 仅留跳转.

```
klua_doc/pfs/
  readme.md                 ← 本文件 (文档根总览)
  api/                      ← 对外 C API (手写 readme/overview + 生成 *.md + Doxygen HTML)
  pfs_basics.md             ← 跨 FS 基础概念
  pfs_test.md               ← 控制台自测用例说明
  pfs_tool.md               ← 用户工具 pfs CLI 说明
  pfs_fat.md / pfs_exfat.md / pfs_ntfs.md  ← 个人对标准的理解
  std_exfat/ / std_fat/ / std_ntfs/        ← 规范镜像
```

## 导航

| 类别 | 入口 |
|------|------|
| API 文档 | [api/readme.md](api/readme.md) |
| 基础概念 | [pfs_basics.md](pfs_basics.md) |
| 自测用例 | [pfs_test.md](pfs_test.md) |
| 用户工具 CLI | [pfs_tool.md](pfs_tool.md) |
| FAT 个人笔记 | [pfs_fat.md](pfs_fat.md) |
| exFAT 个人笔记 | [pfs_exfat.md](pfs_exfat.md) |
| NTFS 个人笔记 | [pfs_ntfs.md](pfs_ntfs.md) |
| exFAT 规范镜像 | [std_exfat/std-exfat-specification_cn.md](std_exfat/std-exfat-specification_cn.md) |
| FAT 规范镜像 | [std_fat/std-fat-specification_cn.md](std_fat/std-fat-specification_cn.md) |
| NTFS 公开说明镜像 | [std_ntfs/std-ntfs-specification_cn.md](std_ntfs/std-ntfs-specification_cn.md) |

