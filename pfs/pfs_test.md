# pfs 测试用例

约定见 **pfs-test-design**。批量 `a`/`all`：章 1 普通 IO（`1.3`～`1.8` / `1.10`）标 `SHARE_MOUNT`，连续用例可共用一次 FS mount；`1.1.x` 挂载主题与 `UMOUNT_FLOW` 仍各自 mount/umount。

---

## 一 功能测试（章 1：vfs / `pfs.h`）

### 1.1 上下文与挂载

**1.1.1 `pfs_ctx_create` / `pfs_ctx_destroy`**

步骤：1. `pfs_ctx_create`；2. 确认返回非 NULL；3. `pfs_ctx_destroy`；4. 对 NULL 再调 `destroy`（应可安全返回）

预期：创建成功；销毁无崩溃；`destroy(NULL)` 可调用

失败：create 失败；destroy 崩溃

**1.1.2 挂载合法卷（只读）**

步骤：1. `pfs_blkio` 打开已含合法 FAT/exFAT 的设备；2. `pfs_ctx_create`；3. `pfs_ctx_mount(..., PFS_MNT_RDONLY)`；4. `pfs_ctx_fs_type` 取类型；5. `umount` + `destroy`（不 close blkio 由调用方负责）

预期：`mount` 返回 `PFS_OK`；`fs_type` 为 `PFS_FAT12` / `16` / `32` / `PFS_EXFAT` 之一（与介质一致）

失败：误报 `PFS_ENOEXEC` / `PFS_EBADMSG`；类型不符

**1.1.3 挂载合法卷（读写）**

步骤：同 1.1.2，mode 改为 `PFS_MNT_RDWR`

预期：`PFS_OK`；后续写类用例可在此挂载上执行

失败：读写挂载失败

**1.1.4 重复挂载**

步骤：1. 已成功 mount；2. 再次对同一 ctx `mount`

预期：返回 `PFS_EBUSY`（或文档约定等价负码）；原挂载仍有效

失败：二次 mount 成功导致状态错乱；或静默覆盖

**1.1.5 非法参数 mount**

步骤：分别用 `p_ctx==NULL`、`p_blkio==NULL`、非法 mode、明显非法 `part_offset` 调用 `mount`

预期：`PFS_EINVAL`（或约定负码）；不崩溃

失败：崩溃；或误返回 `PFS_OK`

**1.1.6 非支持 / 损坏卷**

步骤：1. blkio 打开全零或随机数据镜像；2. `mount`

预期：`PFS_ENOEXEC` 或 `PFS_EBADMSG`；ctx 未处于已挂载态（`fs_type==PFS_NONE`）

失败：误挂载成功

说明：会写坏卷首，标 **MANUAL**（默认 SKIP）

**1.1.7 `remount` 只读↔读写**

步骤：1. `PFS_MNT_RDWR` 挂载；2. `remount(PFS_MNT_RDONLY)`；3. 尝试 `fopen` 写；4. 再 `remount(PFS_MNT_RDWR)`；5. 写应恢复可用

预期：`remount` 成功；只读阶段写失败；切回读写后写成功

失败：remount 无效；只读下仍可写

**1.1.8 `umount`**

说明：含显式 `umount` 流程，标 **UMOUNT_FLOW**（`a`/`all` 省略；仅精确 `1.1.8` 可跑）

步骤：1. 挂载后打开文件不关闭；2. 直接 `umount`；3. 另测：全部关闭后再 `umount`

预期：有未关闭句柄时 `PFS_EBUSY`（若实现如此）或文档约定行为；无打开句柄时 `PFS_OK`，之后 `fs_type==PFS_NONE`

失败：umount 后仍可经旧句柄写坏盘；或泄漏

---

### 1.2 格式化（`pfs_mkfs`）

前置：blkio 已 open；**未** mount 目标卷。

说明：本节文档 case 标 **MANUAL**（跑 `1.2.1` 等会 SKIP，避免单条/节 filter 误格式化）。需要真正 mkfs 时用数字 id 调试通道：`20`=FAT12、`21`=FAT16、`22`=FAT32、`23`=exFAT、`24`=NTFS。数字 id `1`=dump 扇区（默认 0，可跟扇区号）；`2`=探测分区表并打印；`3`=mount 打印 superblock；文档 `3.1.1`/`3.2.1`=创建 Fixed 16G VHD/VHDX；文档 `3.1.2`/`3.2.2` 与数字 id `40`/`41`=创建 Dynamic 32G VHD/VHDX（两通道并存）；`7`=单盘 MBR 单主分区（`pfs_mbr_mkpt` count=1，默认 `PFS_EXFAT`/`sys_ind=0x07`，毁写 LBA0）；`8`=单盘 GPT 单数据分区（`pfs_gpt_mkpt` count=1，Basic Data，毁写 LBA0 与 GPT）；`15`=列根目录（打印文件/目录项）；`16`=根目录随机文本写读校验（大小 `[128K,1M]`，行最长 1024，存在则更新，保留文件）；`17`=固定三级子目录 `/pfs_id14/sub2/sub3` + 同 id16 写文件（保留目录与文件）。

**1.2.1 制作 FAT12**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_FAT12)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_FAT12`

失败：mkfs 失败；挂载类型不符

**1.2.2 制作 FAT16**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_FAT16)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_FAT16`

失败：mkfs 失败；挂载类型不符

**1.2.3 制作 FAT32**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_FAT32)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_FAT32`

失败：mkfs 失败；挂载类型不符

**1.2.4 制作 exFAT**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_EXFAT)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_EXFAT`

失败：mkfs 失败；挂载类型不符

**1.2.5 制作 NTFS**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_NTFS)`；再 `mount` + `fs_type` 核对（boot 级挂载）

预期：`PFS_OK`；`fs_type == PFS_NTFS`

失败：mkfs 失败；挂载类型不符

注：文档 case 标 **MANUAL**；可执行摸底用数字 id `14`。NTFS 现行仅 boot/几何 mount，文件/目录 API 仍 `EOPNOTSUPP`。

**1.2.6 已有文件系统上再次 mkfs**

步骤：1. 已有 FAT/exFAT 卷；2. 再 `pfs_mkfs` 为另一类型或同类型；3. mount 校验

预期：格式化成功；旧数据不可再按原布局访问

失败：mkfs 失败；或挂载仍认旧类型

**1.2.7 `vol_bytes` 边界**

步骤：1. 合法扇区对齐容量 mkfs 成功；2. 非扇区整数倍应失败；3. 超过对应 `PFS_*_VOL_BYTES_MAX` 时按截断前缀制作后仍可 mount

预期：对齐成功；非对齐 `PFS_EINVAL`；超限截断后可挂载且可用容量不超过上限

失败：超限写穿设备；非对齐仍写成功

**1.2.8 已挂载时 mkfs**

步骤：ctx 已 mount 同一 blkio 分区；调用 `pfs_mkfs`

预期：失败（`PFS_EBUSY` / `PFS_EINVAL` 等）；原挂载数据不被静默毁掉（以实现为准，须有明确错误）

失败：mkfs「成功」且卷损坏无错误提示

---

### 1.3 写文件

约定：读写挂载；`fopen` 使用 `PFS_O_RDWR | PFS_O_CREAT`（新建）或 `PFS_O_RDWR`（已存在）。

**1.3.1 根目录写已存在文件**

步骤：1. 预置 `/pfs_t_w_exist.txt` 已知内容与长度 S；2. `fopen` RDWR；3. 自偏移 0 覆写长度 ≤ S；4. `fclose` 后读回

预期：覆写区与写入一致；长度行为符合实现（可保持 S 或按 truncate/写扩展规则，须可复现）

失败：写失败；读回错乱

**1.3.2 根目录写不存在文件（CREAT）**

步骤：1. 确保路径不存在；2. `fopen(..., RDWR|CREAT)`；3. `fwrite` 已知数据；4. `fclose`；5. 再读校验

预期：新建并写入成功；内容一致

失败：创建/写入失败

**1.3.3 至少 3 级目录写已存在文件**

步骤：预置 `/pfs_t_d1/d2/pfs_t_w.txt`；同 1.3.1

预期：同 1.3.1

失败：同 1.3.1

**1.3.4 至少 3 级目录写不存在文件**

步骤：1. 先 `mkdir` 建齐父目录（或分步创建）；2. `CREAT` 写文件；3. 读回

预期：父目录存在时可新建并写入

失败：父目录缺失时误成功；或有父目录仍失败

说明：`fopen` **不**自动递归建目录；缺父目录应失败（`PFS_ENOENT`）

**1.3.5 写小文件**

步骤：新建文件，写入长度范围 `[0, 8KiB)`

预期：成功；含 0 字节写（空写后 close，空文件语义见 1.6.4）

失败：小文件写失败

**1.3.6 写较长文件（分次写入）**

步骤：一次 `fopen`，多次 `fwrite`，累计 `[512KiB, 16MiB)`，一次 `fclose`

预期：内容完整一致；中间无短写除非满盘

失败：内容缺失/错位；未报 `PFS_ENOSPC` 却截断

**1.3.7 `fputc` / `fwrite` 混合写**

步骤：交替 `fputc` 与小块 `fwrite`，关闭后读回

预期：字节序列与写入顺序一致

失败：字节丢失或错序

**1.3.8 只读挂载下写**

步骤：`PFS_MNT_RDONLY` 挂载后 `fopen` RDWR 或 CREAT

预期：打开失败（NULL）或后续写返回负码

失败：只读卷被改写

**1.3.9 满盘再写**

步骤：持续写入直至 `PFS_ENOSPC`；再尝试新建/追加

预期：明确 `PFS_ENOSPC`；已成功关闭的文件仍可读

失败：无错误码却损坏元数据

---

### 1.4 读文件

**1.4.1 根目录读不存在路径**

步骤：`fopen(..., PFS_O_RD)` 不存在路径

预期：返回 NULL

失败：误打开成功

**1.4.2 根目录读存在文件**

步骤：预置已知数据；`PFS_O_RD` 顺序 `fread` 至 EOF

预期：内容一致；`feof` 在 EOF 后非 0；`ferror==0`

失败：内容不符；EOF/错误标志错乱

**1.4.3 三级目录读不存在 / 存在**

步骤：分别对缺失路径与已写路径 `fopen` 读

预期：缺失失败；存在内容一致

失败：误判

**1.4.4 `fgetc` 逐字节读**

步骤：对已知文件循环 `fgetc` 至 EOF

预期：字节序列一致；EOF 时返回 -1 且 `feof` 非 0

失败：提前 EOF；或出错未置 `ferror`

**1.4.5 短读 / 按块读**

步骤：用大于剩余长度的 `nmemb` 调用 `fread`

预期：返回已读个数 ≥0；不越界写缓冲；随后 `feof` 可读出

失败：返回值与实际不符；缓冲越界

**1.4.6 读打开后不关闭即退出用例**

说明：用例内 teardown `umount` 后再 mount，标 **UMOUNT_FLOW**（`a`/`all` 省略；仅精确 `1.4.6` 可跑）

步骤：`fopen` 读部分数据后不 `fclose`，直接结束 case（由 teardown umount/destroy）

预期：不崩溃；再次完整 mount 后文件仍可读

失败：崩溃或卷损坏

---

### 1.5 seek 读写

**1.5.1 合法范围 seek 写**

步骤：新建文件写入一段数据；`fseek`（`PFS_SEEK_SET`/`CUR`/`END`）在合法范围跳跃写；`ftell` 核对；读回各区

预期：各偏移内容正确；`ftell` 与预期偏移一致

失败：错位写；`ftell` 错误

**1.5.2 合法范围 seek 读**

步骤：已知文件上跳跃 `fread`/`fgetc`

预期：各偏移与源一致；建议覆盖跨簇边界（文件长度 ≥ 2 簇）

失败：跨簇读错

**1.5.3 非法 seek**

步骤：对已打开文件尝试负偏移、非法 whence、极端大偏移（按实现：失败或允许超 EOF，须稳定）；之后仍能 `SEEK_SET 0`

预期：负偏移 / 非法 whence 返回 `PFS_EINVAL` 等；极端偏移成功或 `EINVAL`/`ERANGE`；不破坏已有数据

失败：越界写；静默错误偏移；极端 seek 后无法回到合法偏移

**1.5.4 seek 到 EOF 再读**

步骤：`fseek(..., 0, PFS_END)` 后 `fread`

预期：读 0 个成员；`feof` 行为符合约定

失败：读出垃圾数据

---

### 1.6 缓冲、同步与截断

**1.6.1 `fflush`**

步骤：写部分数据 → `fflush` → 不 close，另开只读句柄（若支持）或 umount/remount 后读

预期：`fflush` 返回 `PFS_OK`；已 flush 数据对后续可见（以实现为准，至少 `fsync`/`fclose` 后可见）

失败：flush 失败却返回 OK

**1.6.2 `fsync`**

说明：写后 umount/remount 校验持久化，标 **UMOUNT_FLOW**（`a`/`all` 省略；仅精确 `1.6.2` 可跑）

步骤：写 → `fsync` → 强制断电模拟（进程退出 + 重新 open 设备 mount）后读

预期：`PFS_OK`；已 sync 数据可读

失败：sync 后数据丢失（在可测范围内）

**1.6.3 `ftruncate` 缩短 / 扩展**

步骤：1. 文件长度 S；2. truncate 到 S/2，读尾应 EOF；3. truncate 到 >S，读扩展区为零或未定义但不得崩溃；长度 dual 校验用读/再写

预期：`PFS_OK`；缩短后旧尾不可读；扩展后可继续写

失败：长度与目录项不一致；缩小时簇未释放导致后续 ENOSPC 异常（FAT/exFAT 语义）

**1.6.4 空文件（CREAT 后不写即 close）**

步骤：`CREAT` → 直接 `fclose` → `access` / 再打开读 / `unlink`

预期：行为稳定可文档化（允许 0 长度文件存在）；不泄漏无法回收的目录项

失败：留下损坏项；无法删除

---

### 1.7 目录遍历

**1.7.1 `opendir` / `readdir` 根目录**

步骤：mount 后 `opendir("/")`；遍历至 NULL；检查 `direrror==0`；应见到 `.` / `..`（若该 FS 暴露）；统计 DIR/REG

预期：句柄非 NULL；EOF 时 `pfs_direrror==0`；`d_type`/`d_name` 合理

失败：漏项；EOF 却 `direrror<0`；崩溃

**1.7.2 子目录枚举**

步骤：预置目录内 ≥3 文件 + ≥3 子目录；`opendir` 该路径枚举齐全；`access` 交叉

预期：名称齐全；`d_type` 正确

失败：遗漏；类型错

**1.7.3 负例**

步骤：对不存在路径、普通文件路径 `opendir`；`closedir(NULL)`

预期：失败返回 NULL；`closedir(NULL)` 可安全调用

失败：误成功；崩溃

**1.7.4 `readdir` 后条目寿命**

步骤：保存上次 `readdir` 返回指针，再 `readdir`/`closedir` 后解引用（负例，仅文档警示）

预期：实现约定「仅下次 readdir/closedir 前有效」；测试勿依赖悬空指针

---

### 1.8 路径操作

**1.8.1 `pfs_access`**

步骤：对存在文件/目录、不存在路径分别 `PFS_F_OK`；对存在文件测 `PFS_R_OK`；读写挂载下对可写文件测 `PFS_W_OK`；只读挂载下 `W_OK` 应失败（若实现检查挂载模式）

预期：存在性与权限判定正确；`PFS_X_OK` 在 FAT/exFAT 上可忽略（常成功或与 R 相同，须稳定）

失败：存在误判

**1.8.2 `pfs_mkdir`**

步骤：1. 单级 mkdir；2. 重复 mkdir 同路径（期望 `PFS_EEXIST`）；3. 缺父目录的多级路径（期望 `PFS_ENOENT`，**无** `-p` 语义除非另行实现）；4. 目录下可 CREAT 文件

预期：首次成功；重复/缺父失败码明确；目录可用

失败：静默成功破坏已有文件；无空槽时未返回 `PFS_ENOSPC`

**1.8.3 `pfs_rmdir`**

步骤：1. 空目录 rmdir 成功；2. 非空目录期望 `PFS_ENOTEMPTY`；3. 对文件路径 rmdir 失败

预期：同上

失败：非空被删导致子项孤儿

**1.8.4 `pfs_rename`**

步骤：1. 同目录改名；2. 改到已存在目标（期望 `PFS_EEXIST` 或覆盖策略须文档化）；3. 跨目录改名（父目录均存在）；4. 源不存在失败

预期：成功后旧路径 `ENOENT`、新路径可读且内容一致

失败：双路径并存错乱；内容丢

**1.8.5 `pfs_unlink` / `pfs_remove`**

步骤：1. 删除已存在文件，再 `fopen` 应失败；2. 删除不存在路径失败；3. 对目录调用 `unlink`（期望失败，应用 `rmdir`）；4. `remove` 与 unlink/rmdir 语义一致（文件或空目录）

预期：删除成功后不可读；错误码明确

失败：误删其他项；删后目录项残留可打开

**1.8.6 三级路径综合**

步骤：在 `/pfs_t_a/b/c/` 下组合 mkdir → 写文件 → access → rename → unlink → rmdir 逐级清理

预期：全程成功；最终路径树清理干净

失败：任一步残留

---

### 1.9 错误码与 `pfs_strerror`

**1.9.1 常见负码描述**

步骤：对 `PFS_OK`、`PFS_ENOENT`、`PFS_EINVAL`、`PFS_ENOSPC`、`PFS_ENOTEMPTY`、`PFS_EBUSY`、未知码调用 `pfs_strerror`

预期：返回非 NULL 只读串；带 `[pfs]`  前缀；未知码为 `Unknown error` 类文案

失败：崩溃；空指针

---

### 1.10 名字与路径

**1.10.1 UTF-8 非 ASCII 名**

步骤：创建 `/pfs_t_中文.txt`（或日文等）；pfs 读回校验

预期：pfs 读写成功（`PFS_NAME_MAX` 内）

失败：pfs 找不到；目录项损坏

**1.10.2 文件名长度边界**

步骤：构造长度 1、`PFS_NAME_MAX`、`PFS_NAME_MAX+1` 的文件名组件

预期：≤`PFS_NAME_MAX` 成功；超出 `PFS_ENAMETOOLONG` 或 `EINVAL`

失败：超长截断误覆盖

**1.10.3 非法文件名字符**

步骤：名中含 `"*/:<>?\|` 及控制字符等（按 FAT/exFAT / Windows 非法集；VFS 创建侧统一拒绝）

预期：创建失败；不留下坏项

失败：创建成功但无法再次打开

**1.10.4 大小写**

步骤：创建 `Abc.txt` 后访问 `abc.txt` / `ABC.TXT`

预期：行为符合该 FS（通常不敏感）；记录实际结果

失败：同一目录出现仅大小写不同的双项（exFAT 一般不允许）


---

## 二 章 2：blkio

对应：`portfs/src/pfs_blkio.h`、`portfs/src/base/pfs_blkio.c`

### 2.1 句柄打开关闭

**2.1.1 `pfs_blkio_init` / `pfs_blkio_deinit`**

步骤：1. 未 open 的 struct 调 `init`；2. 核对 `status` 含 `PFS_BLKIO_NOT_OPEN`、`is_open==0`、`align_size==0`；3. `deinit`；4. `deinit(NULL)`（应可安全返回）

预期：`init` 返回 `PFS_BLKIO_RET_OK`；deinit 无崩溃；`deinit(NULL)` 可调用

失败：init 失败；deinit 崩溃；泄漏 fd/HANDLE

**2.1.2 缓冲模式 `open` / `close`**

步骤：1. `init`；2. `open(..., PFS_BLKIO_O_RDWR, S)`（S 为逻辑扇区 512/1024/2048/4096；探测失败可用 512）；3. `is_open` 非 0、`status==0`、`align_size==0`；4. `close`；5. `is_open==0`、`status` 含 `CLOSED`；6. 再 `open` 同一路径同 S

预期：open/close 成功；close 后可再次 open；Win 缓冲 open 内部分配 head/tail 扇区缓存（每块至少 4K）

失败：open 失败；close 后状态错乱；二次 open 失败

**2.1.3 重复 `open`**

步骤：已 open 成功后再对同一句柄 `open` / `open_direct`

预期：`PFS_BLKIO_RET_STATE`；原句柄仍可用

失败：二次 open「成功」泄漏旧 fd；或静默覆盖

**2.1.4 未 open 时 `close` / I/O**

步骤：`init` 后不 open，分别 `close`、`pread`、`pwrite`、`fsync`

预期：close 可按实现返回 OK 或 STATE（须稳定）；I/O / fsync 为 `PFS_BLKIO_RET_STATE`；不崩溃

失败：崩溃；或误返回传输字节数

**2.1.5 `open` 非法参数 / 非法路径**

步骤：分别 `p_blkio==NULL`、`p_devpath==NULL`/空串、非法 flags、非法 `sector_size`（0/256/非 512~4096 集合）、明显不存在路径

预期：PARAM 或 IO（路径类）；不崩溃；`last_error` 在 IO 失败时非 0

失败：崩溃；误返回 OK

**2.1.6 close 后不 `init` 再 `open`**

步骤：open → close → 不 init → 再 open（仍传合法 S）

预期：允许（头文件约定）；二次 open 成功

失败：要求强制 init 才能 reopen（与约定不符）

---

### 2.2 扇区探测

**2.2.1 `pfs_blkio_sector_size`**

步骤：对合法块设备路径调用；对非法路径再测

预期：合法返回 512/1024/2048/4096；非法 `PFS_BLKIO_RET_PROBE` 或 `PARAM`

失败：返回非常规值却当对齐用；崩溃

**2.2.2 `pfs_blkio_sector_size_physical`**

步骤：同路径探测物理扇区

预期：返回合法集合之一；**不**强制等于逻辑扇区；文档约定勿作 `open_direct` 默认 align

失败：崩溃；非法路径误成功

**2.2.3 探测结果指导 `open_direct`**

步骤：若批量共享 env 已持有卷锁则先 `env_close`；以逻辑 `sector_size` 作为 `align_size`，`open_direct(..., RDONLY, S)`；做一次对齐 `pread`；关闭

预期：成功；**不** `pwrite`、**不** `pfs_mkfs`、**不** 格式化盘

失败：逻辑扇区可用却 direct open/读失败；卷仍 LOCK 时二次探测得 `PROBE`

---

### 2.3 读写扇区对齐数据

前置：缓冲 `open(..., S)`（env 缓存 `sector_size`，首次经 `pfs_test_resolve_open_sector_size`；探测失败默认 **512**）。用例内 S 优先 `env.sector_size` / 句柄 / path 探测。

**2.3.1 单扇区读**

步骤：`pread(buf, S, 0)`；与已知镜像/二次读对比

预期：返回 S；两次读一致；`status` 仍为 0

失败：短于 S；内容不一致；误 latch ERR_READ

**2.3.2 单扇区写再读**

步骤：1. 备份偏移 `off` 处原扇区；2. `pwrite` 已知模式；3. `fsync`；4. `pread` 校验；5. 写回备份

预期：写/读均返回 S；内容一致

失败：写失败；读回不符；未恢复导致后续用例污染

**2.3.3 多扇区连续读写**

步骤：长度 `N*S`（如 N=8），`offset` 为 S 整数倍；一次或分次 `pread`/`pwrite`

预期：完整传输；内容一致

失败：错位；部分成功却返回满长度

**2.3.4 `size==0`**

步骤：已 open；`pread`/`pwrite` 且 `size==0`

预期：返回 0；不做 I/O；`status`/`last_error` 不变

失败：误报 PARAM/IO；改写 status

**2.3.5 跨较大偏移读写**

步骤：在设备中后段选对齐 `offset`（仍在容量内），写一扇区再读回；测完恢复

预期：成功；内容正确

失败：偏移截断/符号错误导致写到错误位置

---

### 2.4 读写非扇区对齐数据

前置：缓冲 `open`（非 direct）；逻辑扇区 S（解析规则同 2.3：探测失败默认 512）。

说明：Win 缓冲 `open` 在 blkio 内用 head/tail 扇区缓存按逻辑扇区整齐 RMW，调用方允许未对齐 `offset`/`size`；`open_direct` 仍须对齐。Linux 缓冲模式通常本就可非对齐。

**2.4.1 非对齐 offset 读**

步骤：`offset = 1`（或 `S/2`），`size` 为合法正数（可对齐或非对齐）；`pread`

预期：缓冲模式返回 `size`；内容与从对齐读再切片一致

失败：误报 PARAM；内容错位；Win 上 RMW 失败却当成功

**2.4.2 非对齐 size 读**

步骤：`offset` 对齐，`size = S-1` 或 `S+1`；`pread`

预期：返回请求长度；内容正确

失败：强制按扇区取整导致越界或短读未报错

**2.4.3 非对齐 offset/size 写再读**

步骤：在可覆盖区写非对齐块 → 读回校验 → 恢复原数据

预期：写读一致；邻接未改字节保持原值（读邻接扇区核对）

失败：写穿邻接字节；静默对齐截断

**2.4.4 跨扇区边界的小块读写**

步骤：`offset = S-16`，`size=32`（跨一扇区边界）；写已知模式再读

预期：跨界正确；两端邻接区未破坏

失败：边界错乱；半扇区丢失

---

### 2.5 `open_direct` 与对齐约束

**2.5.1 `open_direct` 合法 align**

步骤：`sector_size` 得 S；`open_direct(..., RDWR, S)`（S 为 `ALIGN_512/1K/2K/4K` 之一）；`align_size()` 返回 S

预期：`PFS_BLKIO_RET_OK`；`is_open` 非 0

失败：合法扇区仍 PROBE/PARAM 失败

**2.5.2 非法 `align_size`**

步骤：`align_size` 取 0、256、3、非 2 幂等

预期：`PFS_BLKIO_RET_PARAM`；未 open

失败：误 open 成功

**2.5.3 direct 下对齐读写成功**

步骤：buf 按 `align_size` 对齐分配；`offset`/`size`/`buf` 均对齐；`pread`/`pwrite`

预期：返回 `size`；写读一致

失败：对齐正确仍 PARAM；或内容错误

**2.5.4 direct 下未对齐拒绝**

步骤：分别令 `offset`、`size`、`buf` 地址之一未对齐，其余合法

预期：`PFS_BLKIO_RET_PARAM`；**不** latch ERR_*；`status` 不变

失败：误 latch；或系统 IO 错误冒充 PARAM

**2.5.5 direct 与缓冲模式切换**

步骤：`open_direct` → close → `open(..., S)`（缓冲）→ 非对齐读写应允许；再 close → `open_direct`

预期：模式随 open 路径切换；`align_size` 在缓冲下为 0；Win 缓冲路径走扇区 RMW

失败：close 后仍按旧 align 约束

---

### 2.6 `fsync`、状态与错误 latch

**2.6.1 正常 `fsync`**

步骤：写若干扇区 → `fsync` → 关闭重开 → 读回

预期：`PFS_BLKIO_RET_OK`；数据可见

失败：fsync 失败却返回 OK；数据丢失（可测范围内）

**2.6.2 I/O 失败 latch**

步骤：对非法超大 `offset`（超出设备）或只读介质上 `pwrite`，触发 `PFS_BLKIO_RET_IO`

预期：`status` 含 `ERR_READ`/`ERR_WRITE`；`last_error` 非 0；随后 `pread`/`pwrite`/`fsync` 为 `PFS_BLKIO_RET_STATE`（直至恢复）

失败：IO 失败不 latch；或 latch 后仍继续写盘

**2.6.3 `set_status` 清除错误后恢复**

步骤：人为/触发 latch 后 `set_status(..., 0)`；再对齐读写

预期：清除成功；I/O 恢复；无 ERR_* 时 `last_error==0`

失败：无法清除；或已 open 时误允许置 `NOT_OPEN`/`CLOSED`

**2.6.4 `set_status` 非法组合**

步骤：已 open 时置 `NOT_OPEN`/`CLOSED`；未 open 时置 `status==0`

预期：`PFS_BLKIO_RET_PARAM`；原 status 不变

失败：状态机被破坏

---

### 2.7 访问模式与其它负例

**2.7.1 `O_RDONLY` 写拒绝**

步骤：`open`/`open_direct` 只读；`pwrite`

预期：失败（PARAM/IO/STATE，须稳定）；介质不被改写

失败：只读打开仍写出

**2.7.2 `O_WRONLY` 读拒绝**

步骤：只写打开后 `pread`

预期：失败；不崩溃

失败：误读成功或崩溃

**2.7.3 `O_RDWR` 正常读写**

步骤：读写打开后各做一次对齐 pread/pwrite

预期：均成功

失败：RDWR 下读或写失败

**2.7.4 超大单次 `size`**

步骤：`size > PFS_BLKIO_IO_SIZE_MAX` 或 `size < 0`

预期：`PFS_BLKIO_RET_PARAM`；不 latch

失败：截断静默传输；整数溢出

**2.7.5 `offset < 0` / 溢出范围**

步骤：负偏移；或 `offset+size` 超出 int64 可寻址约定

预期：`PFS_BLKIO_RET_PARAM`

失败：绕过检查进入系统调用

---

## 三 章 3：镜像容器（image）

实现：`portfs/src_test/pfs_test_image.c`。章 2–5 仅精确 doc_id。

### 3.1 VHD

**3.1.1 创建 Fixed 16G VHD**

步骤：调用 `pfs_vhd_create_fixed`，路径 `d:/work/tmp/fixed_16g.vhd`，大小 16G，扇区 512B；无需 device 参数

预期：返回成功；文件可随后被默认盘路径探测命中

失败：创建失败（路径不可写、磁盘不足等）

注：原数字 id `3` 已废止，请跑 `3.1.1`。

**3.1.2 创建 Dynamic 32G VHD**

步骤：调用 `pfs_vhd_create_dynamic`，路径 `d:/work/tmp/dynamic_32g.vhd`，大小 32G，扇区 512B；无需 device 参数

预期：返回成功；文件可随后被默认盘路径探测命中

失败：创建失败（路径不可写、磁盘不足等）

注：亦可数字 id `40`（与本章并存）；原 id `5` 已废止。

---

### 3.2 VHDX

**3.2.1 创建 Fixed 16G VHDX**

步骤：调用 `pfs_vhdx_create_fixed`，路径 `d:/work/tmp/fixed_16g.vhdx`，大小 16G（lss 512、block 2MiB）；无需 device 参数

预期：返回成功；文件可随后被默认盘路径探测命中

失败：创建失败（路径不可写、磁盘不足等）

注：请跑 `3.2.1`。

**3.2.2 创建 Dynamic 32G VHDX**

步骤：调用 `pfs_vhdx_create_dynamic`，路径 `d:/work/tmp/dynamic_32g.vhdx`，大小 32G（lss 512、block 2MiB）；无需 device 参数

预期：返回成功；文件可随后被默认盘路径探测命中

失败：创建失败（路径不可写、磁盘不足等）

注：亦可数字 id `41`（与本章并存）；原 id `6` 已废止。

---

### 3.3 QCOW2

**3.3.1 创建 Fixed 16G QCOW2**

步骤：调用 `pfs_qcow2_create`，路径 `d:/work/tmp/fixed_16g.qcow2`，大小 16G（虚拟容量，512B 对齐；空盘 v3，cluster 64KiB）；无需 device 参数

预期：返回成功；文件可随后被 blkio 探测打开

失败：创建失败（路径不可写、磁盘不足等）

注：qcow2 create 均为稀疏 v3，无独立 Fixed API；本节容量对齐 3.1.1 的 16G。标 `NO_DEVICE`；仅精确 `3.3.1`。

**3.3.2 创建 Dynamic 32G QCOW2**

步骤：调用 `pfs_qcow2_create`，路径 `d:/work/tmp/dynamic_32g.qcow2`，大小 32G（虚拟容量，512B 对齐；空盘 v3，cluster 64KiB）；无需 device 参数

预期：返回成功；文件可随后被 blkio 探测打开

失败：创建失败（路径不可写、磁盘不足等）

注：同上，稀疏 v3；容量对齐 3.1.2 的 32G。标 `NO_DEVICE`；仅精确 `3.3.2`。

---

### 3.4 VMDK

**3.4.1 创建 Fixed 16G VMDK**

步骤：创建 Fixed/FLAT 形态 VMDK，路径 `d:/work/tmp/fixed_16g.vmdk`，大小 16G，扇区 512B；无需 device 参数

预期：返回成功；文件可随后被 blkio 探测打开（描述符 FLAT）

失败：创建失败（路径不可写、磁盘不足等）

注：现行 **无** `pfs_vmdk_create*`（读写子集已落地）；用例与 API 待接。标 `NO_DEVICE`；仅精确 `3.4.1`。

**3.4.2 创建 Dynamic 32G VMDK**

步骤：创建 Dynamic/monolithicSparse 形态 VMDK，路径 `d:/work/tmp/dynamic_32g.vmdk`，大小 32G，扇区 512B；无需 device 参数

预期：返回成功；文件可随后被 blkio 探测打开（monolithicSparse）

失败：创建失败（路径不可写、磁盘不足等）

注：现行 **无** create API；用例与 API 待接。标 `NO_DEVICE`；仅精确 `3.4.2`。

---

### 3.5 VDI

**3.5.1 创建 Fixed 16G VDI**

步骤：创建 Fixed VDI，路径 `d:/work/tmp/fixed_16g.vdi`，大小 16G，扇区 512B；无需 device 参数

预期：返回成功；文件可随后被 blkio 探测打开（`PFS_VDI_TYPE_FIXED`）

失败：创建失败（路径不可写、磁盘不足等）

注：现行 **无** `pfs_vdi_create*`（Fixed/Dynamic 读写已落地）；用例与 API 待接。标 `NO_DEVICE`；仅精确 `3.5.1`。

**3.5.2 创建 Dynamic 32G VDI**

步骤：创建 Dynamic VDI，路径 `d:/work/tmp/dynamic_32g.vdi`，大小 32G，扇区 512B；无需 device 参数

预期：返回成功；文件可随后被 blkio 探测打开（`PFS_VDI_TYPE_DYNAMIC`）

失败：创建失败（路径不可写、磁盘不足等）

注：现行 **无** create API；用例与 API 待接。标 `NO_DEVICE`；仅精确 `3.5.2`。

---

## 四 章 4：diskfs（FAT12 / FAT16 / FAT32 / exFAT / NTFS / ISOFS / UDF / ext2 / ext3 / ext4）

实现：`portfs/src_test/pfs_test_diskfs.c`。对应代码支持的磁盘 FS（`PFS_FAT*` / `PFS_EXFAT` / `PFS_NTFS` / `PFS_ISOFS` / `PFS_UDF` / `PFS_EXT*`）。

约定：各节 **`4.x.1` = `pfs_mkfs`**；**`4.x.2` = `pfs_mkvol`**（**简单源** `pfs_t_mkvol_simple`：根文件 `hello.txt` + 一级空目录 `subdir/` + 三级目录文件 `l1/l2/l3/deep.txt`；读回核对三项）；**`4.x.3` = mkvol 后源/目标整树与文件对比**（**完整源** `pfs_t_mkvol_src`；目录存在 + **每个文件**长度与全文；失败打印路径、长度、首差异偏移）；**`4.x.4` = 源与目标整树与文件对比**（只读 `mount` 已有卷，**不** `pfs_mkfs`/`pfs_mkvol`；比对完整源，同 `4.x.3`）。`4.x.1`～`4.x.3` 为格式化/毁卷主题，标 **DESTRUCTIVE**（`a` 省略；仅精确 doc_id 可跑）；`4.x.4` **不**毁卷（章 2–5 仍仅精确 doc_id）。前置：blkio 已 open；`4.x.1`～`4.x.3` 时**未** mount 目标卷；`vol_bytes` 须扇区对齐（超 `PFS_*_VOL_BYTES_MAX` 时截断前缀）。简单源 Win：`d:/work/tmp/pfs_t_mkvol_simple`；完整源 Win：`d:/work/tmp/pfs_t_mkvol_src`；**禁止覆盖**目录内已有文件/子目录；缺默认项才创建。`opts` 可 NULL（默认卷标）。`4.x.4` 前置：卷已由 **`4.x.3`** 用完整源灌入。`4.1.1`～`4.10.4` 均已接线。

### 4.1 FAT12

**4.1.1 `pfs_mkfs` FAT12**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_FAT12)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_FAT12`

失败：mkfs 失败；挂载类型不符

**4.1.2 `pfs_mkvol` FAT12**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 4.1.2；与 `4.1.3` 完整源 **分离**。

**4.1.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 4.1.3。

**4.1.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 4.1.4；卷须已由 4.1.3（完整源）灌入。
---

### 4.2 FAT16

**4.2.1 `pfs_mkfs` FAT16**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_FAT16)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_FAT16`

失败：mkfs 失败；挂载类型不符

**4.2.2 `pfs_mkvol` FAT16**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 4.2.2；与 `4.2.3` 完整源 **分离**。

**4.2.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 4.2.3。

**4.2.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 4.2.4；卷须已由 4.2.3（完整源）灌入。
---

### 4.3 FAT32

**4.3.1 `pfs_mkfs` FAT32**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_FAT32)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_FAT32`

失败：mkfs 失败；挂载类型不符

**4.3.2 `pfs_mkvol` FAT32**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 4.3.2；与 `4.3.3` 完整源 **分离**。

**4.3.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 4.3.3。

**4.3.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 4.3.4；卷须已由 4.3.3（完整源）灌入。
---

### 4.4 exFAT

**4.4.1 `pfs_mkfs` exFAT**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_EXFAT)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_EXFAT`

失败：mkfs 失败；挂载类型不符

**4.4.2 `pfs_mkvol` exFAT**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 4.4.2；与 `4.4.3` 完整源 **分离**。

**4.4.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 4.4.3。

**4.4.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 4.4.4；卷须已由 4.4.3（完整源）灌入。
---

### 4.5 NTFS

**4.5.1 `pfs_mkfs` NTFS**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_NTFS)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_NTFS`

失败：mkfs 失败；挂载类型不符

**4.5.2 `pfs_mkvol` NTFS**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 4.5.2；与 `4.5.3` 完整源 **分离**。

**4.5.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 4.5.3。

**4.5.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 4.5.4；卷须已由 4.5.3（完整源）灌入。
---

### 4.6 ISOFS

介质约定：

| 场景 | 介质 | 打开 |
|------|------|------|
| **未指定** device（默认盘） | 宿主 **`d:/work/tmp/pfs_t_isofs.iso`**（Linux：`/tmp/pfs_t_isofs.iso`） | **raw**，`part_offset=0`，扇区 2048；缺文件时建约 256MiB |
| **显式** `-d` / 设备路径 | 调用方指定（如 U 盘 `\\.\PhysicalDriveN`、卷路径） | 常规探测打开；**不**改写为 iso；`-o` 仍为分区字节偏移 |

说明：pfs 可对 U 盘/`PhysicalDrive` 做 `pfs_mkfs`/`pfs_mkvol` ISOFS；Windows 资源管理器是否认盘符另论。写卷前请卸载占用并慎用（**DESTRUCTIVE**）。

**4.6.1 `pfs_mkfs` ISOFS**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_ISOFS)`（空卷）；再只读 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_ISOFS`

失败：mkfs 失败；挂载类型不符

注：标 **DESTRUCTIVE**；跑精确 `4.6.1`。

**4.6.2 `pfs_mkvol` ISOFS**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 `4.6.2`；与 `4.6.3` 完整源 **分离**。

**4.6.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 `4.6.3`。

**4.6.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 `4.6.4`；卷须已由 `4.6.3`（完整源）灌入。
---

### 4.7 UDF

**4.7.1 `pfs_mkfs` UDF**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_UDF)`（空卷）；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_UDF`

失败：mkfs 失败；挂载类型不符

**4.7.2 `pfs_mkvol` UDF**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 `4.7.2`；与 `4.7.3` 完整源 **分离**。

**4.7.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 `4.7.3`。

**4.7.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 `4.7.4`；卷须已由 `4.7.3`（完整源）灌入。
---

### 4.8 ext2

**4.8.1 `pfs_mkfs` ext2**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_EXT2)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_EXT2`

失败：mkfs 失败；挂载类型不符

**4.8.2 `pfs_mkvol` ext2**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 4.8.2；与 `4.8.3` 完整源 **分离**。

**4.8.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 4.8.3。

**4.8.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 4.8.4；卷须已由 4.8.3（完整源）灌入。
---

### 4.9 ext3

**4.9.1 `pfs_mkfs` ext3**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_EXT3)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_EXT3`

失败：mkfs 失败；挂载类型不符

**4.9.2 `pfs_mkvol` ext3**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 4.9.2；与 `4.9.3` 完整源 **分离**。

**4.9.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 4.9.3。

**4.9.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 4.9.4；卷须已由 4.9.3（完整源）灌入。
---

### 4.10 ext4

**4.10.1 `pfs_mkfs` ext4**

步骤：对空白或可覆盖镜像，`pfs_mkfs(..., PFS_EXT4)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFS_EXT4`

失败：mkfs 失败；挂载类型不符

**4.10.2 `pfs_mkvol` ext4**

步骤：准备简单源 `pfs_t_mkvol_simple`（根 `hello.txt`、空目录 `subdir/`、`l1/l2/l3/deep.txt`；缺项才建、不覆盖）；`pfs_mkvol(..., src_dir, opts)`；再只读 `mount`；核对 `fs_type`、根文件内容、`/subdir` 可 `opendir`、`/l1/l2/l3/deep.txt` 内容

预期：`PFS_OK`；`fs_type` 与本节一致；上述三项均通过

失败：mkvol 失败；挂载类型不符；根文件/目录/三级文件缺失或内容不符

注：标 **DESTRUCTIVE**；跑精确 4.10.2；与 `4.10.3` 完整源 **分离**。

**4.10.3 mkvol 后源/目标整树与文件对比**

步骤：准备完整源 `pfs_t_mkvol_src`（缺 `hello.txt` 才建默认；不覆盖已有）；`pfs_mkvol` + 只读 `mount`；再深度遍历宿主完整源：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：标 **DESTRUCTIVE**；跑精确 4.10.3。

**4.10.4 源与目标整树与文件对比**

步骤：只读 `mount` 已有目标卷（**不** `pfs_mkfs` / **不** `pfs_mkvol`）；再深度遍历宿主完整源 `pfs_t_mkvol_src`：核对每个目录存在；对每个文件逐字节比对长度与全文内容

预期：源树每个目录在目标卷存在；每个文件长度与全文内容一致

失败：缺失、长度不符或内容不符时打印 vfs/host 路径、长度与首差异偏移（可继续扫完再汇总 FAIL）

注：不毁卷；跑精确 4.10.4；卷须已由 4.10.3（完整源）灌入。
---

## 五 章 5：pfsmtd（squashfs / ubifs / jffs2 / yaffs / erofs / cramfs / romfs）

实现：`portfs/src_test/pfs_test_mtd.c`（合文件；用例表待接）。对应 `PFSMTD_REGISTER_FS_ALL` 七套目标 FS（`PFSMTD_*`）。须用 **`pfsmtd_mkfs` / `pfsmtd_mkvol`**（核心 `pfs_mkfs`/`pfs_mkvol` **不**认识 `PFSMTD_*`）。

约定：各节 **`5.x.1` = `pfsmtd_mkfs`**；**`5.x.2` = `pfsmtd_mkvol`**。均为格式化/毁卷主题，标 **MANUAL**（章 2–5 仅精确 doc_id；精确 id 默认 SKIP）。前置：blkio 已 open（含 PFUB/ubi 等可用后端）；**未** mount 目标卷；`opts` 可 NULL。固有只读（squashfs / erofs / cramfs / romfs）挂载用 `PFS_MNT_RDONLY`；可写（ubifs / jffs2 / yaffs）可用 `PFS_MNT_RDWR`。

### 5.1 squashfs

**5.1.1 `pfsmtd_mkfs` squashfs**

步骤：对空白或可覆盖镜像，`pfsmtd_mkfs(..., PFSMTD_SQUASHFS)`；再只读 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFSMTD_SQUASHFS`

失败：mkfs 失败；挂载类型不符

**5.1.2 `pfsmtd_mkvol` squashfs**

步骤：准备宿主 `src_dir`；`pfsmtd_mkvol(..., PFSMTD_SQUASHFS, src_dir, opts)`（一次打包）；再只读 `mount` + 核对内容与 `fs_type`

预期：`PFS_OK`；`fs_type == PFSMTD_SQUASHFS`；源目录树可按路径读出

失败：mkvol 失败；挂载类型不符；打包内容缺失

---

### 5.2 ubifs

**5.2.1 `pfsmtd_mkfs` ubifs**

步骤：对空白或可覆盖镜像，`pfsmtd_mkfs(..., PFSMTD_UBIFS)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFSMTD_UBIFS`

失败：mkfs 失败；挂载类型不符

**5.2.2 `pfsmtd_mkvol` ubifs**

步骤：准备宿主 `src_dir`；`pfsmtd_mkvol(..., PFSMTD_UBIFS, src_dir, opts)`；再 `mount` + 核对内容与 `fs_type`

预期：`PFS_OK`；`fs_type == PFSMTD_UBIFS`；源目录树可按路径读出

失败：mkvol 失败；挂载类型不符；灌入内容缺失

---

### 5.3 jffs2

**5.3.1 `pfsmtd_mkfs` jffs2**

步骤：对空白或可覆盖镜像，`pfsmtd_mkfs(..., PFSMTD_JFFS2)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFSMTD_JFFS2`

失败：mkfs 失败；挂载类型不符

**5.3.2 `pfsmtd_mkvol` jffs2**

步骤：准备宿主 `src_dir`；`pfsmtd_mkvol(..., PFSMTD_JFFS2, src_dir, opts)`；再 `mount` + 核对内容与 `fs_type`

预期：`PFS_OK`；`fs_type == PFSMTD_JFFS2`；源目录树可按路径读出

失败：mkvol 失败；挂载类型不符；灌入内容缺失

---

### 5.4 yaffs

**5.4.1 `pfsmtd_mkfs` yaffs**

步骤：对空白或可覆盖镜像，`pfsmtd_mkfs(..., PFSMTD_YAFFS)`；再 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFSMTD_YAFFS`

失败：mkfs 失败；挂载类型不符

**5.4.2 `pfsmtd_mkvol` yaffs**

步骤：准备宿主 `src_dir`；`pfsmtd_mkvol(..., PFSMTD_YAFFS, src_dir, opts)`；再 `mount` + 核对内容与 `fs_type`

预期：`PFS_OK`；`fs_type == PFSMTD_YAFFS`；源目录树可按路径读出

失败：mkvol 失败；挂载类型不符；灌入内容缺失

---

### 5.5 erofs

**5.5.1 `pfsmtd_mkfs` erofs**

步骤：对空白或可覆盖镜像，`pfsmtd_mkfs(..., PFSMTD_EROFS)`；再只读 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFSMTD_EROFS`

失败：mkfs 失败；挂载类型不符

**5.5.2 `pfsmtd_mkvol` erofs**

步骤：准备宿主 `src_dir`；`pfsmtd_mkvol(..., PFSMTD_EROFS, src_dir, opts)`；再只读 `mount` + 核对内容与 `fs_type`

预期：空 `src_dir` 时 `PFS_OK`（等同空卷 mkfs）；非空树现行可 `PFS_EOPNOTSUPP`；`fs_type == PFSMTD_EROFS`

失败：空树 mkvol 失败；挂载类型不符

注：非空灌树未做；内容打包优先 squashfs / romfs。

---

### 5.6 cramfs

**5.6.1 `pfsmtd_mkfs` cramfs**

步骤：对空白或可覆盖镜像，`pfsmtd_mkfs(..., PFSMTD_CRAMFS)`；再只读 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFSMTD_CRAMFS`

失败：mkfs 失败；挂载类型不符

**5.6.2 `pfsmtd_mkvol` cramfs**

步骤：准备宿主 `src_dir`；`pfsmtd_mkvol(..., PFSMTD_CRAMFS, src_dir, opts)`；再只读 `mount` + 核对内容与 `fs_type`

预期：空 `src_dir` 时 `PFS_OK`（等同空卷 mkfs）；非空树现行可 `PFS_EOPNOTSUPP`；`fs_type == PFSMTD_CRAMFS`

失败：空树 mkvol 失败；挂载类型不符

注：非空灌树未做；内容打包优先 squashfs / romfs。

---

### 5.7 romfs

**5.7.1 `pfsmtd_mkfs` romfs**

步骤：对空白或可覆盖镜像，`pfsmtd_mkfs(..., PFSMTD_ROMFS)`；再只读 `mount` + `fs_type` 核对

预期：`PFS_OK`；`fs_type == PFSMTD_ROMFS`

失败：mkfs 失败；挂载类型不符

**5.7.2 `pfsmtd_mkvol` romfs**

步骤：准备宿主 `src_dir`；`pfsmtd_mkvol(..., PFSMTD_ROMFS, src_dir, opts)`（一次打包）；再只读 `mount` + 核对内容与 `fs_type`

预期：`PFS_OK`；`fs_type == PFSMTD_ROMFS`；源目录树可按路径读出

失败：mkvol 失败；挂载类型不符；打包内容缺失

