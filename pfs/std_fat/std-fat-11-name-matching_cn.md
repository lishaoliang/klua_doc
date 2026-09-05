> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## 短名与长名匹配 (Name Matching In Short & Long Names)

所有短目录项中的名称构成 **短名空间 (short name space)**. 所有长目录项中的名称构成 **长名空间 (long name space)**. 二者合为 **统一命名空间**, 同一目录内 **不得重名** — 无论短名或长名, 每名仅可出现一次.

长名 **保留大小写**, 但 **不得** 仅大小写不同而并存: 若已有短项 `FOOBAR` 或长项 `FooBar`, 则不能再创建 `foobar`.

文件系统内各类查找 (find、open、create、delete、rename) 均 **大小写不敏感**. `open("FOOBAR")` 可打开已存在的 `FooBar` 或 `foobar`. 以 `FOOBAR` 为模式的 find 同理. **带重音的扩展字符** 适用相同规则.

**短名查找** 仅匹配短目录项名称. **长名查找** 同时检查长项与短项. 遍历目录时, 文件系统缓存长目录项中的 LFN 子组件; 遇到与缓存长名关联的短目录项时, **先** 比缓存长名, **再** 比短名.

若介质上某字符 (OEM 或 UNICODE) 无法翻译为当前 OEM/ANSI 代码页对应字符, 返回给用户时 **一律** 显示为 `_` (下划线) — **磁盘上内容不改**. 该字符在所有 OEM 代码页与 ANSI 中相同.
