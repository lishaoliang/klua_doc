> **来源**: Microsoft fatgen103 (FAT32 File System Specification v1.03)
> **本地镜像**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/)；权威以官方 fatgen103.doc 为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/fat/` 与 `pfs_fat_*`
> **Fetched**: 2026-07-25

## 命名约定与长名 (Naming Conventions and Long Names)

API 允许调用方指定文件/目录的 **长名**, **不允许** 单独指定短名. 因短名与长名属 **统一命名空间**, 且 **不支持重名** — 某文件的长名 (忽略大小写) 不得与另一文件的短名相同. 该限制为避免用户与应用对「正确名称」混淆. 为透明实现该限制: 创建长名且不存在匹配长名时, **自动** 由长名生成不与现有短名冲突的短名.

自动生成短名的方法仿照 Windows NT. 自动短名由 **basis-name（基名）** 与可选 **numeric-tail（数字尾）** 组成.

### Basis-Name 生成算法

下列为示例算法, 说明如何从长名生成短名; 实现 **should** 遵循相同步骤顺序.

1. 传入文件系统的 UNICODE 名转 **大写**.
2. 大写 UNICODE 名转 **OEM**.
   - 若大写 UNICODE 字形在 OEM 代码页中不存在, **或** OEM 字形在 8.3 名中非法, 则替换为 OEM `_`, 并置 **lossy conversion（有损转换）** 标志.
3. 去掉长名中所有 **前导与中间** 空格.
4. 去掉长名中所有 **前导** 句点.
5. 当 (未到长名末尾) 且 (当前字符不是句点) 且 (已复制字符数 < 8) 时, 将字符复制到 basis-name **主部**.
6. 若长名最后一个句点之后有扩展名, 在 basis-name 主部末尾插入 `.`.
7. 扫描长名中 **最后一个嵌入句点**.
   - 若找到: 当 (未到末尾) 且 (扩展部已复制 < 3) 时, 复制字符到 basis-name **扩展部**.
8. 进入 numeric-tail 生成.

### Numeric-Tail 生成算法

- 若 (**未** 置有损转换标志) **且** (长名符合 8.3 约定) **且** (basis-name 不与任何现有短名冲突), 则短名 **仅** 为 basis-name, **无** 数字尾.
- 否则在主名末尾插入 numeric-tail `~n`, 选取 `n` 使全名不与现有短名冲突, 且主部不超过 8 字符.

`~n` 可从 `~1` 到 `~999999`. `n` 取为同 basis-name 序列中的 **下一编号**. 例如已有 `LETTER~1.DOC`、`LETTER~2.DOC`, 下一自动名为 `LETTER~3.DOC`. 若已有 `LETTER~1.DOC`、`LETTER~3.DOC`, 下一自动名 **通常** 为 `LETTER~2.DOC` — **但不可依赖此行为**. 目录中此类名称极多时, 选择算法为 **速度优化**, 可能根据以 `~n` 结尾且主部模式相近的短名特征选取其他 `n`.
