> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: `portfs/doc/std_ntfs/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

## NTFS 进程与交互

下列各节说明 NTFS 相关进程与交互.

### 挂载 NTFS 卷

挂载 NTFS 卷时, MBR 执行代码以启动 boot sector. boot sector 再执行额外代码以挂载该卷.

##### 主引导代码启动过程

MBR 含少量可执行代码, 称 master boot code, 以及 disk signature 与磁盘 partition table. 启动时 master boot code 执行下列活动:

1. 扫描 partition table 查找 active partition.
2. 找到 active partition 的起始扇区.
3. 将 active partition 的 boot sector 副本载入内存.
4. 将控制权转交给 boot sector 中的可执行代码.

##### Boot Sector 启动过程

计算机在启动时使用 boot sector 运行指令. 初始启动过程概括如下:

1. 系统 BIOS 与 CPU 发起 power-on self test (POST).
2. BIOS 找到 boot device, 通常是 BIOS 找到的第一块磁盘, 除非控制器配置为从其他磁盘启动.
3. BIOS 将 boot device 的第一个物理扇区载入内存并将 CPU 执行转到该内存地址.

若 boot device 在硬盘上, BIOS 载入 MBR. MBR 中的 master boot code 载入 active partition 的 boot sector 并将 CPU 执行转到该地址. 在运行 Windows Server 2003 的计算机上, boot sector 中的可执行 boot code 找到 Ntldr, 将其载入内存并将执行转交给该文件.

**Note**

- Windows Server 2003 无法从动态磁盘上的 spanned, striped 或 RAID-5 卷启动. 这些磁盘结构无法登记到 MBR partition table; 因此使用这些结构的系统卷无法启动.

若驱动器 A 含软盘, 系统 BIOS 将磁盘第一个扇区 (boot sector) 载入内存. 若该盘为启动盘 (由 MS-DOS 格式化并写入核心操作系统文件), boot sector 载入内存并用可执行 boot code 将 CPU 执行转交给 Io.sys (MS-DOS 核心文件). 若软盘非启动盘, 可执行 boot code 显示错误消息.

**Note**

- 配置为先在驱动器 C 查找启动文件的正常系统不会出现这些消息. 许多计算机可在 CMOS setup 中设置系统搜索启动文件的磁盘顺序.

若从硬盘启动时出现类似错误, boot sector 可能已损坏.

初始启动过程与磁盘格式及操作系统无关. 当 boot sector 的可执行 boot code 开始运行时, 操作系统与文件系统的特性才变得重要.

### 格式化卷

格式化卷时, Windows Server 2003 在卷上放置关键 NTFS 文件系统结构, 包括 boot sector 与 MFT, 并替换 Ntldr. 格式化还会按簇大小边界对齐簇.

格式化会检查卷上全部扇区的完整性, 并允许更改卷所用簇大小. 若使用 Quick format, 仅创建卷上文件系统结构, 不检查每个扇区的完整性.

### 转换卷

Windows Server 2003 可将旧版 NTFS 转换为 Windows Server 2003 所用的新版 NTFS.

#### 转换 Windows 2000 格式化的 NTFS 卷

Windows Server 2003 首次挂载 Windows 2000 格式化的 NTFS 卷时, 将该卷转换为 NTFS 3.1. 转换仅将 NTFS 版本从 3.0 改为 3.1, 不更改卷上现有元数据或文件. 然而, Windows Server 2003 对 NTFS 3.1 卷上新创建文件使用不同 header 风格. 因此部分非 Microsoft  imaging 程序无法对 NTFS 3.1 卷创建镜像. 请联系 imaging 程序厂商确认是否有支持 Windows Server 2003 NTFS 3.1 卷的版本.

运行 Windows NT 4.0 Service Pack 4 或更高版本, 或 Windows 2000 的计算机无需转换或额外 service pack 即可访问 NTFS 3.1 卷. 另请注意 NTFS 3.1 在 Windows XP 与 Windows Server 2003 中相同.

#### 转换 Windows NT 4.0 及更早版本格式化的 NTFS 卷

从 Windows NT 4.0 升级到 Windows Server 2003 时, 所有用 Windows NT 4.0 及更早 NTFS 版本格式化的本地卷在 Windows Server 2003 Setup 完成后首次挂载时升级为 NTFS 3.1 (升级不在 Setup 期间进行). Setup 期间移除或关闭的 NTFS 卷, 或 Setup 后添加的卷, 在 Windows Server 2003 挂载时转换.

Ntfs.sys 驱动通过判定卷所用 NTFS 版本并在必要时转换来完成升级. 转换在任何大小卷上仅需数秒, 并在 master file table 中新增下列 record:

- $Secure, 含卷内所有文件的唯一 security descriptor.
- $Extend, 用于配额, reparse point 数据, object identifier 等扩展. 转换过程还向 $Extend 目录添加三个新文件:

 - $Quota, 用于 disk quota.
 - $Reparse, 用于 reparse point.
 - $ObjID, 用于 distributed link tracking.

$Secure 与 $Extend 占用原先未使用的 MFT record, 因此卷上总有足够空间容纳这两条 record. 然而 $Quota, $Reparse 与 $ObjID 是 MFT 的新增项, 卷上必须有足够空闲空间容纳这些文件, 否则转换失败.

若转换失败, 卷仍可用, 但只能执行 Windows NT 4.0 及更早版本可用的 NTFS 相关任务. 要转换为 NTFS 3.1, 须通过删除或移动文件释放磁盘空间, 然后 dismount 该卷.

**Note**

- 用旧版 NTFS 格式化的可移动介质在安装或升级完成后, 或在插入并由 Windows Server 2003 挂载时升级.

#### 卷转换的限制

转换是单向过程. 卷转换为 NTFS 后, 无法在不备份数据, 将卷重新格式化为 FAT 并恢复数据的情况下转回 FAT. 卷上应有一定空闲空间, 并有足够内存更新 cache.

从 FAT 转换为 NTFS 还适用下列限制:

- 在多系统启动配置中, 仅 Windows NT 4.0 Service Pack 4 或更高版本, Windows 2000, Windows XP 或 Windows Server 2003 可访问 NTFS 卷.
- 若在格式化为 FAT16 或 FAT32 的卷上安装 Recovery Console, 再将该卷转换为 NTFS, Recovery Console 将无法运行. 原因是系统卷 cmdcons 文件夹中用于运行 Recovery Console 的文件系统专用启动文件对已转换为 NTFS 的卷无效. 转换后可从 Windows Server 2003 操作系统光盘重新安装 Recovery Console, 亦可用该光盘启动 Recovery Console.
- 由于 Windows Server 2003 格式化时按簇大小边界对齐 FAT 数据簇, 转换可保留与该卷大小对应的簇大小 (最大 4 KB), 而非 Windows 2000 对转换卷使用的 512 字节簇. 下表 **Cluster Sizes for Volumes Converted to NTFS** 列出转换为 NTFS 后的簇大小.

 **Cluster Sizes for Volumes Converted to NTFS**

 | Original FAT Cluster Size | Converted NTFS Cluster Size |
 | --- | --- |
 | 512 bytes | 512 bytes |
 | 1 KB | 1 KB |
 | 2 KB | 2 KB |
 | 4 KB and larger | 4 KB |

### 文件命名

Windows Server 2003 在 NTFS 卷上同时支持长文件名与短文件名.

#### Windows Server 2003 中的文件名

每次创建带长文件名的文件时, NTFS 会创建第二条 file entry, 含类似的 8.3 短文件名. 8.3 短文件名由 1 至 8 个字符的文件名与 1 至 3 个字符的扩展名组成, 以句点分隔.

Windows Server 2003 中的文件名最长 255 字符, 可含空格, 多个句点以及 MS-DOS 文件名不允许的特殊字符. Windows Server 2003 为每个文件生成 MS-DOS 可读 (8.3) 名称, 使其他操作系统可访问长文件名文件. 这些 8.3 名称亦使基于 MS-DOS 与 Windows 3.x 的应用程序能识别并加载长文件名文件. 程序在 Windows Server 2003 上保存文件时, 8.3 文件名与长文件名均保留.

**Note**

- 8.3 格式表示文件名可有 1 至 8 个字符, 须以字母或数字开头, 除下列字符外可使用任意字符:
- . " / \ [ ] : ; | = , * ? (space)
- 8.3 文件名通常有 1 至 3 字符的扩展名, 受相同字符限制. 句点分隔文件名与扩展名.
- 系统保留若干特殊文件名, 不可用于文件或文件夹: CON, AUX, COM1, COM2, COM3, COM4, LPT1, LPT2, LPT3, PRN, NUL

#### NTFS 如何生成短文件名

在 Windows Server 2003 中, FAT 与 NTFS 均使用 Unicode 字符集命名, 其中含 MS-DOS 无法读取的若干禁止字符. 为生成 MS-DOS 可读短文件名, Windows Server 2003 从长文件名删除这些字符并移除空格. 因 MS-DOS 可读文件名只能有一个句点, Windows Server 2003 亦删除文件名中额外句点. 必要时将文件名截断为六个字符并追加波浪号 (**~**) 与数字. 例如, 每个非重复文件名追加 **~1**. 重复文件名依次为 **~2**, **~3** 等. 截断文件名后, 扩展名截断为最多三个字符. 最后在命令行显示文件名时, Windows Server 2003 将文件名与扩展名字符全部转为大写.

**Note**

- 可使用 **fsutil behavior set** 命令允许扩展字符. 该设置生效前须重启计算机. 有关 **fsutil behavior set** 的更多信息, 见 Windows Server 2003 Help and Support Center 中的 Fsutil: behavior 主题.

当存在五个或更多可能产生重复短文件名的文件时, Windows Server 2003 采用略不同的短文件名创建方法. 对第五个及后续文件, Windows Server 2003:

- 仅使用长文件名的前两个字母.
- 通过数学变换长文件名剩余字母生成短文件名的后四个字母.
- 在结果后追加 **~1** (或必要时其他数字以避免重复).

该方法在 Windows Server 2003 须为大量长文件名相似的文件创建短文件名时显著提升性能. Windows Server 2003 在 FAT 与 NTFS 卷上均用此方法创建短名.

下表展示六次测试创建文件的短文件名.

**Short File Names Created by Windows Server 2003 — Example One**

| Long File Name | Short File Name |
| --- | --- |
| This is test 1.txt | THISIS~1.TXT |
| This is test 2.txt | THISIS~2.TXT |
| This is test 3.txt | THISIS~3.TXT |
| This is test 4.txt | THISIS~4.TXT |
| This is test 5.txt | THA1CA~1.TXT |
| This is test 6.txt | THA1CE~1.TXT |

若上表长文件名以不同顺序创建, 短文件名不同, 见下表.

**Short File Names Created by Windows Server 2003 — Example Two**

| Long File Name | Short File Name |
| --- | --- |
| This is test 2.txt | THISIS~1.TXT |
| This is test 3.txt | THISIS~2.TXT |
| This is test 1.txt | THISIS~3.TXT |
| This is test 4.txt | THISIS~4.TXT |
| This is test 5.txt | THA1CA~1.TXT |
| This is test 6.txt | THA1CE~1.TXT |

删除文件时其短文件名亦删除. 在同一文件夹创建新文件时, Windows Server 2003 可能复用已删除的短文件名. 例如, 在示例 1 中若删除 "This is test 1.txt" 再创建 "This is test 7.txt", 其短文件名为 THISIS~1.TXT.

若文件夹内文件数量很大 (30 万或更多) 且长文件名前几个字符相同, 创建文件所需时间会增加. 原因是 NTFS 基于长文件名前六个字符生成短文件名. 超过 30 万个文件的文件夹中, NTFS 用尽与长文件名相似的 8.3 名后短文件名开始冲突. 生成的短文件名与现有短文件名反复冲突会导致 NTFS 将短文件名重新生成 6 至 8 次.

### 文件与文件夹压缩

NTFS 支持对单个文件, 文件夹内全部文件以及整个 NTFS 卷内全部文件进行压缩. 因压缩在 NTFS 内部实现, 任何基于 Windows 的程序均可读写压缩文件而无需判定压缩状态. 压缩在 file header 的位中设置, 压缩信息存储在 Data file attribute 中.

通过对 DeviceIoControl 传入 FSCTL_SET_COMPRESSION 对文件与目录进行压缩与解压. 压缩后的文件或目录关联 FILE_ATTRIBUTE_COMPRESSED 标志. 应用程序可用 GetFileAttributes 判定文件或目录的压缩状态.

程序打开压缩文件时, NTFS 仅解压正在读取的文件部分并复制到内存. 数据在内存中保持未压缩, 因此 NTFS 读写或修改内存中数据时性能不受影响. 数据稍后写入磁盘时, NTFS 压缩修改或新增的数据.

NTFS 压缩算法支持最大 4 KB 的簇大小. NTFS 卷簇大小大于 4 KB 时, NTFS 压缩功能均不可用.

#### 移动与复制文件或文件夹

移动与复制文件和文件夹可能改变其压缩状态. 结果取决于移动还是复制, 以及是否在 NTFS 卷之间或移动到 FAT 卷.

NTFS 支持压缩单个文件, 目录内全部文件或整卷全部文件. 压缩在 file header 的位中设置, 压缩信息存储在 Data file attribute 中. 位设置后, 系统在保存时压缩文件并在需要时解压.

压缩增加系统开销, 因为即使在同一计算机内复制, 压缩 NTFS 文件也会先解压, 复制, 再作为新文件重新压缩. 对移动或复制所指定文件的压缩 attribute 的任何更改均会应用. 若压缩卷内全部文件, 过程可能需数分钟, 取决于卷大小, 待压缩文件数量与计算机速度. 延迟是因为 Windows Server 2003 须更改卷上每个文件夹的压缩 attribute 并压缩或解压每个文件.

更改文件夹压缩状态相对较快, 因为对每个文件夹 Windows Server 2003 仅更改压缩 attribute. 然而压缩或解压卷上每个文件耗时更长, 因为 NTFS 须从磁盘以当前形式 (压缩或未压缩) 读取数据, 在内存中转换为新形式, 再写回磁盘.

##### 在 NTFS 卷内移动文件或文件夹

将未压缩文件或文件夹移动到 NTFS 卷上另一文件夹时, 文件保持未压缩. 图 **Moving an Uncompressed File to a Compressed Folder** 展示将未压缩文件移到压缩文件夹的结果.

**Moving an Uncompressed File to a Compressed Folder**

![Moving Uncompressed File to Compressed Folder](images/cc781134.25020930-608e-463d-bfaf-047d12da52c6(ws.10).gif)

将压缩文件移到未压缩文件夹时, 移动后文件仍保持压缩. 图 **Moving a Compressed File to an Uncompressed Folder** 展示将压缩文件移到未压缩文件夹的结果.

**Moving a Compressed File to an Uncompressed Folder**

![Moving a Compressed File to an Uncompressed Folder](images/cc781134.243edc64-3367-4cb5-8fe3-38b015cbe61d(ws.10).gif)

##### 在 NTFS 卷内复制文件或文件夹

复制文件到文件夹时, 文件采用目标文件夹的压缩 attribute.

若将压缩文件复制到未压缩文件夹, 复制到文件夹时文件被解压, 如图 **Copying a Compressed File to an Uncompressed Folder**.

**Copying a Compressed File to an Uncompressed Folder**

![Copying Compressed File to Uncompressed Folder](images/cc781134.72fa8c1b-be2a-405c-8ee5-750cc65c4c73(ws.10).gif)

若复制文件到已含同名文件的文件夹, 复制的文件采用目标文件的压缩 attribute, 如图 **Copying a File to a Folder that Contains a File of the Same Name**.

**Copying a File to a Folder that Contains a File of the Same Name**

![Copy File to Folder that Contains Same Name File](images/cc781134.f7fd237f-33e6-4a97-8e01-337e3f3dce5f(ws.10).gif)

##### 在 FAT 与 NTFS 卷之间复制文件

从 FAT 文件夹复制到 NTFS 文件夹的文件采用目标文件夹的压缩 attribute. 从 NTFS 卷复制到 FAT 卷或软盘的压缩文件会被解压.

### NTFS 卷上的 Mounted Drive

Mounted drive (亦称 volume mount point 或 drive path) 是挂载到 NTFS 卷空文件夹上的卷. mounted drive 功能与其他卷相同, 但分配 label 或名称而非驱动器号. mounted drive 对计算机添加或移除设备时的系统变更具有鲁棒性, 不受驱动器号 26 卷限制, 可用于访问超过 26 个卷.

宿主卷须使用 Windows Server 2003 所含 NTFS 版本. 被挂载的卷可用 Windows Server 2003 支持的任意文件系统格式化.

一个卷可托管多个 mounted drive, 便于在 Windows Server 2003 系统上扩展特定卷的存储容量. 本地或通过网络连接的用户可继续使用同一驱动器号访问该卷, 同时从该驱动器号同时使用多个卷.

仅 NTFS 卷可持有 mounted drive, 但任意本地驱动器均可挂载到一个 NTFS 卷.

#### 实现 Mounted Drive

NTFS mounted drive 通过 reparse point 实现, 并受其限制. reparse point 是一组用户定义数据. 数据格式由存储数据的应用程序与解释数据并处理文件的 file system filter 理解. 应用程序设置 reparse point 时存储该数据及唯一标识所存数据的 reparse tag. 文件系统打开含 reparse point 的文件时, 尝试找到与该 reparse tag 标识的数据格式关联的 file system filter. 若找到 filter, 则按 reparse data 指示处理文件. 若未找到, 文件打开失败.

reparse point 适用下列限制:

- 可为目录建立 reparse point, 但目录必须为空. 否则 NTFS 无法建立 reparse point. 此外, 不能在含 reparse point 的目录中创建子目录或文件.
- reparse point 与 extended attribute 互斥. NTFS 无法在含 extended attribute 的文件上创建 reparse point, 也无法在含 reparse point 的文件上创建 extended attribute.
- reparse point 数据不得超过 16 KB. 若待写入 reparse point 的数据超过该限制则设置失败.

### Hard Link

hard link 是指向给定文件的 NTFS 专用链接. 在 NTFS 卷上创建 hard link 时, NTFS 添加 hard link 的 directory entry 而不复制原文件. 通过 hard link 可以:

- 使用与原文件相同的文件名但出现在不同文件夹.
- 使用与原文件不同的文件名但出现在同一文件夹.
- 使用不同文件名且出现在不同文件夹.

hard link 是文件的 directory entry, 应用程序可通过任一 hard link 修改文件. 使用其他 hard link 的应用程序可检测到变更. 然而 hard link 的 directory entry 仅在用户通过该 hard link 访问文件时更新. 例如, 若用户通过 hard link 打开并修改文件且原文件大小改变, 用于访问的 hard link 亦显示新大小.

hard link 没有 security descriptor; security descriptor 属于 hard link 指向的原文件. 因此更改任一 hard link 的 security descriptor 实际更改底层文件的 security descriptor. 指向该文件的全部 hard link 均允许新指定的访问. 无法按 hard link 为文件设置不同 security descriptor.

hard link 使用 Win32 函数 CreateHardLink 在文件间创建链接.

### Distributed Link Tracking

Distributed link tracking 确保目标文件重命名或移动后 shell shortcut 与 OLE link 仍可用. 在 NTFS 卷上创建指向文件的 shortcut 时, distributed link tracking 在目标文件 (link source) 中写入唯一 object identifier (ID). 有关 object ID 的信息亦存储在引用文件 (link client) 中. distributed link tracking 在 Windows Server 2003 域内 NTFS 卷上发生下列任一组合事件时, 用此 object ID 定位 link source:

- link source 被重命名.
- link source 移到同卷另一文件夹或同计算机另一卷.
- link source 从某一共享网络文件夹移到域内不同计算机的另一共享网络文件夹.
- 含 link source 的计算机被重命名.
- 含 link source 的共享网络文件夹名称改变.
- 含 link source 的卷移到域内另一计算机.

**Note**

- Distributed link tracking 仅在运行 Windows 2000, Windows XP 或 Windows Server 2003 的计算机上的 NTFS 卷有效. 这些 NTFS 卷不能在可移动介质上.

Distributed link tracking 尝试维护非域内 link: 跨域, 工作组内或单机未联网. 当 link source 在计算机内移动或 link source 计算机上的网络共享文件夹改变时, 这些 link 通常可维护. link source 移到另一计算机时通常亦可维护, 但随时间推移可靠性较低.

Distributed link tracking 对客户端与服务器使用不同服务:

- Distributed Link Tracking Client 服务运行于全部 Windows 2000 与 Windows Server 2003 计算机. 未联网计算机上 Client 服务执行全部 link tracking 活动.
- Distributed Link Tracking Server 服务运行于 Windows 2000 与 Windows Server 2003 域控制器. Server 服务维护与 link source 移动相关的信息. 因此域内 link 比域外更可靠. 域内计算机上 Distributed Link Tracking Client 服务通过与 Distributed Link Tracking Server 服务通信利用这些信息.

Distributed Link Tracking Client 服务监视 NTFS 卷活动, 并在每个卷根目录隐藏文件夹 System Volume Information 内的 Tracking.log 文件中存储维护信息. 该文件夹受权限保护, 仅系统可访问. System Volume Information 亦被 Indexing Service 等其他 Windows Server 2003 服务使用.

### Sparse File

sparse file 提供一种节省磁盘空间的方法, 适用于既含有意义数据又含大量零字节区的文件. 若 NTFS 文件标记为 sparse, NTFS 仅为应用程序显式指定的数据分配磁盘簇. 文件中未指定范围在磁盘上以未分配空间表示. 从已分配范围读取时返回存储的数据; 从未分配范围读取时返回零.

文件系统 API 允许将文件按实际位与 sparse stream 范围复制或备份. 文件系统 API 亦允许查询已分配范围. 实现这些 API 的程序只需读取已分配范围即可恢复文件中全部数据. 结果是高效的文件系统存储与访问. 图 **Sparse Data Storage** 展示设置与未设置 sparse file attribute 时的数据存储.

**Sparse Data Storage**

![Sparse Data Storage](images/cc781134.eda85415-6d21-4b03-8a50-644dd718dd90(ws.10).gif)

例如, 文件属性可能显示为 1 GB sparse file. 尽管文件为 1 GB, 仅占用 64 KB 磁盘空间.

**Note**

- 仅 Windows 2000, Windows XP 或 Windows Server 2003 系列挂载的 NTFS 卷支持 sparse file. 若将 sparse file 复制或移动到 FAT 卷或由上述以外操作系统挂载的 NTFS 卷, 文件将展开为原始指定大小. 若无足够空间则操作失败.

### Disk Quota

可启用 disk quota 以限制用户在带 NTFS 文件系统的远程或本地计算机上占用的卷空间. disk quota 使用服务器所在域中的用户名. 管理员可针对域中用户设置 disk quota.

有关 Disk Quota 的更多信息, 见 [Disk Quotas Technical Reference](cc786969%28v=ws.10%29).

### NTFS Change Journal

当文件, 文件夹与其他 NTFS 对象被添加, 删除与修改时, NTFS 在 stream 中写入 change journal record, 每台计算机上每个卷各一条.

当前 journal 中全部 record 的总大小可变, 但有可配置的最大大小. change journal 可超过最大大小直至达到外部阈值, 此时删除最旧 record 的一部分直至 journal 恢复至最大大小. change journal 最大大小可配置但只能增大不能减小.

change journal 为否则需扫描整卷以确定变更的应用程序带来显著可扩展性收益. 文件系统 indexing, replication manager, 病毒扫描器与增量备份应用程序均可受益于 change journal.

change journal 在确定特定命名空间变更方面比 time stamp 或 file notification 高效得多. 须重新扫描整卷以确定变更的应用程序现在可扫描一次并随后参考 change journal. I/O 成本取决于变更文件数量, 而非卷上文件总数.

API 有完整文档, 独立软件供应商 (ISV) 可使用. Microsoft 在 Windows Server 2003 组件 (如 Indexing Service 与 File Replication Service) 中使用 change journal. ISV 可用此特性增强备份, 防病毒与审计工具等产品的可扩展性与鲁棒性.

### NTFS 文件系统可恢复性

NTFS 是可恢复文件系统, 通过标准 transaction logging 与 recovery 技术保证卷一致性. 系统故障时 NTFS 运行 recovery 过程, 访问 transaction log file 中存储的信息. NTFS recovery 过程保证卷恢复至一致状态. transaction logging 开销很小.

#### 恢复 NTFS 文件结构

NTFS 将修改卷上文件的每个操作视为 transaction 并作为整体单元管理. NTFS 亦可能将单个复杂操作拆成多个 transaction. transaction 开始后要么完成, 要么因导致操作失败的事件而 rollback, NTFS 卷回到 transaction 开始前的状态. 可能导致操作失败的事件包括坏扇区, 瞬时低内存条件与断开设备.

为确保 transaction 可完成或 rollback, NTFS 对每个 transaction 执行下列步骤:

1. 在内存 cache 的 log file 中记录 transaction 的 metadata 操作.
2. 在内存中记录实际的 metadata 操作.
3. 在 cache 的 log file 中将 transaction 标记为 committed.
4. 将 log file flush 到磁盘.
5. 将实际 metadata 操作 flush 到磁盘.

步骤 4 与 5 在 transaction 完成后 lazy 执行, 即 flush 操作不与 transaction 本身绑定. NTFS 快速在内存中修改 log 与 metadata, 随后在合适时机 flush 以提升性能.

NTFS 保证含 transaction metadata 操作的 log record 在 transaction 修改的 metadata 写入磁盘之前写入磁盘. NTFS 更新 cache 后通过在 cache 的 log file 中记录 transaction 完成来 commit transaction. cache 的 log file flush 到磁盘后, 即使系统在变更写入磁盘前失败, 全部 committed transaction 亦保证完成.

**Note**

- 应用程序可指定 FILE_FLAG_WRITE_THROUGH Win32 标志, 指示系统绕过中间 cache 直接写磁盘. 系统仍可 cache 写操作, 但不可 lazy flush.

若发生系统故障, NTFS 在 log 中有足够信息以完成或 abort 任何 partial NTFS transaction. recovery 期间 NTFS redo log file 中发现的每个 committed transaction, 然后定位系统故障时未 commit 的 transaction 并 undo log file 中记录的每个 metadata 操作. 因 NTFS 在任何 metadata 变更写入磁盘前 flush log, recovery 期间 NTFS 拥有 rollback 所需 metadata 变更的完整信息.

**Note**

- NTFS 使用 transaction logging 与 recovery 保证卷结构不损坏. 因此系统故障后全部文件系统数据仍可访问. NTFS 仅在创建数据的程序使用 FILE_FLAG_WRITE_THROUGH Win32 标志时才保证用户数据. 若程序未使用该标志, 系统故障可能导致用户数据丢失. 若发生系统故障, NTFS 显示先前数据, 新数据或零, 用户不会因崩溃而在卷上看到随机数据.

#### Cache 与数据恢复

cache 是 RAM 中含最近使用数据的区域. 向磁盘写数据时, Windows Server 2003 的 lazy-write 技术表示数据仍在 cache 中即视为已写入. cache 内存亦可能在磁盘控制器 (如 SCSI 控制器 cache) 或磁盘单元 (如 ATA 磁盘 cache) 上. 下列信息有助于决定是否启用磁盘或控制器 cache:

- write cache 提升磁盘性能, 尤其写入大量数据时.
- write-back cache 的控制由磁盘厂商 firmware 提供. 见磁盘或磁盘控制器文档. 无法从 Windows Server 2003 配置 write-back cache.
- 只要磁盘厂商 firmware 遵守 NTFS 驱动发出的 write-through 请求, write cache 不影响文件系统自身 metadata 可靠性. NTFS 指示磁盘设备驱动确保 metadata 写入无论是否启用 write cache. 非 metadata 通常写入磁盘且可被 cache.
- 磁盘中的 read cache 不影响文件系统可靠性.

#### Cluster Remapping

NTFS 检测到坏扇区时动态 remapping 含坏扇区的簇 (称 cluster remapping) 并为数据分配新簇. 若错误发生在读操作, NTFS 向调用程序返回读错误且数据丢失. 若错误发生在写操作, NTFS 将数据写入新簇且无数据丢失.

NTFS 将含坏扇区的簇地址放入 MFT 中 bad cluster file $BadClus, 使坏扇区不被复用.

#### 磁盘 Recovery 操作

NTFS 在计算机重启后或卷 dismount 后挂载卷时执行 disk recovery 操作, 以保证全部 NTFS 卷完整性. NTFS 亦使用 cluster remapping 技术最小化 NTFS 卷上坏扇区的影响.

NTFS 将修改卷上文件的每个操作视为 transaction 并作为整体单元管理. NTFS 亦可能将单个复杂操作拆成多个 transaction. transaction 开始后要么完成, 要么 rollback. 可能导致操作失败的事件包括坏扇区, 瞬时低内存条件与断开设备.

NTFS 保证含 transaction metadata 操作的 log record 在 transaction 修改的 metadata 写入磁盘前写入磁盘. NTFS 更新 cache 后 commit transaction. cache 的 log file flush 到磁盘后, 即使系统崩溃, 全部 committed transaction 亦保证完成.

若发生系统故障, NTFS 在 log 中有足够信息以完成或 abort partial transaction. recovery 期间 NTFS redo committed transaction 并 undo 未 commit transaction 的 metadata 操作. 因 NTFS 在 metadata 变更写入磁盘前 flush log, recovery 拥有 rollback 所需的完整信息.

### 基于 Windows NT 的卷上的 Cleanup 操作

Windows Server 2003 所含 NTFS 版本格式化的卷上的文件可由 Windows NT 4.0 Service Pack 4 或更高版本读写, 因此 Windows Server 2003 在基于 Windows NT 的计算机上挂载后可能需要执行 cleanup 操作以保证卷数据结构一致性.

Windows Server 2003 不对先前由 Windows 2000 或 Windows XP 挂载的卷执行 cleanup 操作.

Cleanup 操作影响下列特性:

##### Reparse point

运行 Windows NT 4.0 或更早版本的计算机无法访问含 reparse point 的文件, 因此无需 cleanup. reparse point 是关联 reparse data 数据块的文件或目录.

##### Disk quota

若 disk quota 关闭, Windows Server 2003 不执行 cleanup. 若 disk quota 开启, Windows Server 2003 通过重建 index 清理 quota 信息. 若用户在 NTFS 卷由 Windows NT 4.0 SP4 或更高版本系统挂载期间超出 disk quota 且严格强制执行 quota, 该用户使用 Windows Server 2003 进行的后续数据磁盘分配均失败. 用户仍可读写现有文件但不可增大文件. 然而用户可删除与缩小文件. 使用量降至分配的 disk quota 以下后可恢复数据磁盘分配.

##### Encryption

运行 Windows NT 4.0 或更早版本的计算机无法访问加密文件, 因此无需 cleanup.

##### Sparse file

运行 Windows NT 4.0 或更早版本的计算机无法访问 sparse file, 因此无需 cleanup.

##### Change journal

运行 Windows NT 4.0 或更早版本的计算机不在 change journal 中记录文件变更. Windows Server 2003 启动时, 由 Windows NT 访问的卷上的 change journal 重置为表示 journal 历史不完整. 使用 change journal 的应用程序须能接受不完整 journal.

##### Object identifier

Windows Server 2003 维护 object identifier 的两处引用: 文件上一处, 卷级 object identifier index 上一处. 若删除含 object identifier 的文件, Windows Server 2003 须扫描并清理 index 中的 entry.

### POSIX 兼容性

NTFS 提供若干特性以支持 Portable Operating System Interface (POSIX) 标准, 该标准由 Institute of Electrical and Electronic Engineers (IEEE) 标准 1003.1-1990 (亦称 ISO/IEC 9945-1:1990) 定义.

NTFS 包含下列 POSIX 兼容特性.

##### Case-sensitive naming

例如, POSIX 将 README.TXT, Readme.txt 与 readme.txt 视为不同文件.

##### Hard link

一个文件可有多个名称. 这允许同一卷上不同文件夹中的两个不同文件名指向相同数据.

##### Additional time stamp

显示文件上次访问或修改时间.

Windows NT 与 Windows 2000 所含的 POSIX 子系统未包含在 Windows Server 2003 中. 支持多数 UNIX 系统上超出 POSIX.1 标准广泛功能的新子系统作为 Interix 2.2 的一部分提供. Interix 子系统可认证至 NIST FIPS 151-2 POSIX Conformance Test Suite.

**Note**

- 须使用基于 Interix 的程序管理仅大小写不同的文件名. 不能使用标准 Windows Server 2003 命令行工具 (如 **copy**, **del**, **move**, 或 Windows Explorer / My Computer 中等价操作) 管理仅大小写不同的文件名.
