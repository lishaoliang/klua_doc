> **来源**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn)
> **本地镜像**: `portfs/doc/std_exfat/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；VolumeDirty / 写序对齐内核脏卷标志与常见实现实践
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 8 实现说明 (Implementation Notes)

### 8.1 推荐写序 (Recommended Write Ordering)

实现方宜尽可能使卷在电源故障及其他不可避免失败面前具有韧性。在创建新目录项或修改簇分配时，一般宜遵循下列写序：

1. 将 VolumeDirty 置为 1
2. 如有必要，更新活动 FAT
3. 更新活动 Allocation Bitmap
4. 如有必要，创建或更新目录项
5. 若第一步之前 VolumeDirty 为 0，则将其清为 0

在删除目录项或释放簇分配时，宜遵循下列写序：

1. 将 VolumeDirty 置为 1
2. 如有必要，删除或更新目录项
3. 如有必要，更新活动 FAT
4. 更新活动 Allocation Bitmap
5. 若第一步之前 VolumeDirty 为 0，则将其清为 0

### 8.2 无法识别的目录项的含义 (Implications of Unrecognized Directory Entries)

同一主版本号 1、次版本号高于 0 的未来 exFAT 规范，可能定义新的良性主项 (benign primary)、关键次项 (critical secondary) 与良性次项 (benign secondary)。仅更高主版本号的规范可定义新的关键主项 (critical primary)。本规范（exFAT Revision 1.00 File System Basic Specification）的实现宜能够挂载并访问主版本号为 1、任意次版本号的任何 exFAT 卷。因此实现可能遇到无法识别的目录项。下列场景的含义如下：

1. 根目录中出现无法识别的关键主项使卷无效。除 File 目录项外，任何非根目录中出现关键主项使该宿主目录无效。
2. 实现方 **shall not** 修改无法识别的良性主项或其关联簇分配。但在删除目录时（且仅在删除目录时），实现方 **shall** 删除无法识别的良性主项并释放全部关联簇分配（若有）。
3. 实现方 **shall not** 修改无法识别的关键次项或其关联簇分配。目录项集合中出现一个或多个无法识别的关键次项，使整个目录项集合被视为无法识别。删除含此类项的集合时，实现方 **shall** 释放与无法识别关键次项关联的全部簇分配（若有）。进一步，若该集合描述的是目录，实现方 **may**：

 - 进入该目录
 - 枚举其中的目录项
 - 删除其中的目录项
 - 将其中的目录项移动到另一目录

 但实现方 **shall not**：

 - 修改其中的目录项（删除除外，见上）
 - 在其中创建新目录项
 - 打开其中的目录项（遍历与枚举除外，见上）
4. 实现方 **shall not** 修改无法识别的良性次项或其关联簇分配。实现方宜忽略无法识别的良性次项。删除目录项集合时，实现方 **shall** 释放与无法识别良性次项关联的全部簇分配（若有）。
