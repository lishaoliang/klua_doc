> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## 目录内容校验 (Validating The Contents of a Directory)

下列准则供磁盘维护工具在保持与未来目录结构扩展兼容的前提下, 校验单条目录项是否「正确」.

1. **DO NOT** 查看标记为 **reserved** 的字段内容, 并因非零而认定其「坏」.
2. **DO NOT** 在 reserved 字段非零时将其 **清零** (假定其「坏」). 这些字段为 **reserved**, 而非 **must-be-zero**. 应用 **should ignore** 它们, 以便工具可在未来操作系统版本上继续运行.
3. **DO** 判定 LFN 与短项时 **优先** 使用 `ATTR_LONG` (`ATTR_LONG_NAME`) 属性. 正确算法:

```
if (((LDIR_Attr & ATTR_LONG_NAME_MASK) == ATTR_LONG_NAME) && (LDIR_Ord != 0xE5))
    /* 找到活动的 LFN 子组件. */
```

4. **DO** 判定短目录项类型时 **同时** 使用短项属性 **位 4 与位 3** (`ATTR_DIRECTORY` 与 `ATTR_VOLUME_ID`). 正确算法:

```
if (((DIR_Attr & ATTR_LONG_NAME_MASK) != ATTR_LONG_NAME) && (DIR_Name[0] != 0xE5))
    if ((DIR_Attr & (ATTR_DIRECTORY | ATTR_VOLUME_ID)) == 0x00)
        /* 找到文件. */
    else if ((DIR_Attr & (ATTR_DIRECTORY | ATTR_VOLUME_ID)) == ATTR_DIRECTORY)
        /* 找到目录. */
    else if ((DIR_Attr & (ATTR_DIRECTORY | ATTR_VOLUME_ID)) == ATTR_VOLUME_ID)
        /* 找到卷标. */
    else
        /* 无效目录项. */
```

5. **DO NOT** 假定 LFN 项 **type** 字段非零即表示坏项; **DO NOT** 强制将该字段清零.
6. 使用 **checksum** 字段校验目录项. **first cluster** 字段当前置 0, 将来可能变更.
