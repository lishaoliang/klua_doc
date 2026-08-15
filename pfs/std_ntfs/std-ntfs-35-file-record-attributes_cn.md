> **来源**: Microsoft Learn / TechNet NTFS public docs (see std-ntfs-specification.md 拷贝来源)
> **本地镜像**: `portfs/doc/std_ntfs/`；权威以官方 Learn 英文页为准
> **译文说明**: 中文译本；字段名/元数据文件名保留英文；术语对齐 linux-7.1.2 `fs/ntfs/` 与 `pfs_ntfs_*`
> **Fetched**: 2026-07-25

## NTFS 物理结构

### NTFS File Record Attributes (NTFS 文件记录属性)

NTFS 卷上每个已分配扇区都属于某个文件. 连文件系统元数据也是文件的一部分. NTFS 将每个文件 (或文件夹) 视为一组 file attributes. 文件名、安全信息乃至数据等文件元素均为 file attribute. 每个 attribute 由 attribute type code 与可选 attribute name 标识.

File record 与 folder record 各为 1 KB, 存储在 MFT 中, 其 attribute 写入 MFT 中已分配空间. 除 file attribute 外, 每条 file record 还含该 record 在 MFT 中位置的信息.

当文件的 attribute 能放入该文件的 MFT file record 时, 称 resident attributes (驻留属性). 文件名、时间戳等 attribute 始终驻留. 当文件信息量超出其 MFT file record 容量时, 部分 file attribute 变为 nonresident (非驻留). 非驻留 attribute 分配一个或多个簇的磁盘空间. 非驻留 attribute 的一部分仍留在 MFT 中, 指向外部簇. NTFS 创建 Attribute List attribute 以描述所有 attribute record 的位置.

属性类型常量见 `NTFS_AT_*` (`pfs_ntfs_raw.h`, `fs/ntfs/layout.h`).

**NTFS File Attribute Types (NTFS 文件属性类型)**

| Attribute Type | 说明 |
| --- | --- |
| Standard Information (`NTFS_AT_STANDARD_INFORMATION`) | 访问模式 (只读、读/写等)、时间戳、link count 等信息. |
| Attribute List (`NTFS_AT_ATTRIBUTE_LIST`) | 无法放入 MFT record 的所有 attribute record 的位置. |
| File Name (`NTFS_AT_FILE_NAME`) | 长文件名与短文件名的可重复 attribute. 长文件名最多 255 个 Unicode 字符. 短文件名为 8.3、大小写不敏感名称. POSIX 所需的额外名称或 hard link 可作为额外 File Name attribute 包含. |
| Data (`NTFS_AT_DATA`) | 文件数据. NTFS 支持每个文件多个 Data attribute. 每个文件通常有一个未命名的 Data attribute. 文件也可有一个或多个命名的 Data attribute. |
| Object ID (`NTFS_AT_OBJECT_ID`) | 卷内唯一的文件标识符. 供 distributed link tracking service 使用. 并非所有文件都有 object identifier. |
| Logged Tool Stream (`NTFS_AT_LOGGED_UTILITY_STREAM`) | 类似 data stream, 但操作像 NTFS 元数据变更一样记录到 NTFS log file. EFS 使用此 attribute. |
| Reparse Point (`NTFS_AT_REPARSE_POINT`) | 用于挂载驱动器. 也供 Installable File System (IFS) filter driver 将特定文件标记为该驱动器专用. |
| Index Root (`NTFS_AT_INDEX_ROOT`) | 用于实现文件夹与其他索引. |
| Index Allocation (`NTFS_AT_INDEX_ALLOCATION`) | 用于实现大文件夹与其他大索引的 B-tree 结构. |
| Bitmap (`NTFS_AT_BITMAP`) | 用于实现大文件夹与其他大索引的 B-tree 结构. |
| Volume Information (`NTFS_AT_VOLUME_INFORMATION`) | 仅用于 $Volume 系统文件. 含卷版本. |

NTFS 为卷上创建的每个文件生成 file record, 为每个文件夹生成 folder record. MFT 含 MFT 自身的单独 file record. 这些 file record 与 folder record 各为 1 KB, 存储在 MFT 中. 文件的 attribute 写入 MFT 中已分配空间. 除 file attribute 外, 每条 file record 还含该 record 在 MFT 中位置的信息. 图 MFT Entry with Resident Record 展示小文件或文件夹的 MFT record 内容. 小文件与文件夹 (通常 900 字节或更小) 完全包含在该文件的 MFT record 内.

**MFT Entry with Resident Record (含驻留记录的 MFT 项)**

![MFT Entry with Resident Record](images/cc781134.86787c15-cf0a-4cb9-8ba1-ff1afd37aaf5(ws.10).gif)

通常每个文件使用一条 file record. 然而, 若文件 attribute 数量很多或高度碎片化, 可能需要多条 file record. 此时文件的第一条 record (base file record) 存储该文件所需其他 file record 的位置.

Folder record 含 index 信息. 小 folder record 完全位于 MFT 结构内, 而大文件夹组织为 B-tree 结构, 其 record 含指向外部簇的指针, 外部簇存放无法放入 MFT 结构的 folder entry.

使用 B-tree 结构的好处在大文件夹枚举文件时明显. B-tree 允许 NTFS 对相似文件名分组或索引, 然后只搜索含该文件的组, 最小化查找特定文件所需的磁盘访问次数, 尤其对大文件夹. 由于 B-tree 结构, NTFS 在大文件夹上优于 FAT, 因为 FAT 必须先扫描大文件夹中所有文件名才能列出全部文件.

#### Last Access Time (最后访问时间)

NTFS 卷上每个文件与文件夹含称 Last Access Time 的 attribute. 它显示文件或文件夹最后被访问的时间, 例如用户列出文件夹、向文件夹添加文件、读取文件或修改文件时. 最新的 Last Access Time 始终保存在内存中, 最终会写入磁盘两处:

- 文件的 attribute, 即其 MFT record 的一部分.
- 文件的 directory entry. directory entry 存储于包含该文件的文件夹中. 具有多个 hard link 的文件有多个 directory entry.

磁盘上的 Last Access Time 并不总是最新, 因为 NTFS 在强制将 Last Access Time 更新写入磁盘前会等待约一小时. 当用户或程序对文件或文件夹执行只读操作 (如列出文件夹内容或读取但不修改文件夹中的文件) 时, NTFS 也会延迟将 Last Access Time 写入磁盘. 若每次读操作都使磁盘上的 Last Access Time 保持最新, 所有读操作都会变成写操作, 影响 NTFS 性能.

**注意**

- 基于文件的 Last Access Time 查询即使磁盘上并非所有值都是最新也仍然准确. NTFS 在查询时返回正确值, 因为准确值保存在内存中.

NTFS 最终按下列方式将内存中的 Last Access Time 写入磁盘.

##### 在文件的 attribute 内

若内存中当前 Last Access Time 与磁盘上 Last Access Time 相差超过一小时, 或该文件的所有内存引用都已消失 (以较晚者为准), NTFS 通常更新磁盘上文件的 attribute. 例如, 若文件当前 Last Access Time 为下午 1:00, 你在下午 1:30 读取文件, NTFS 不更新 Last Access Time. 若你在下午 2:00 再次读取, NTFS 将 attribute 中的 Last Access Time 更新为下午 2:00, 因为 attribute 显示下午 1:00 而内存中为下午 2:00.

##### 在文件的 directory entry 内

NTFS 在下列事件期间更新文件的 directory entry:

- 当 NTFS 更新文件的 Last Access Time 并检测到与 directory entry 中存储的 Last Access Time 相差超过一小时时. 这通常发生在程序关闭用于访问文件夹内文件的句柄之后. 若程序长时间保持句柄打开, directory entry 中的变更会有延迟.
- 当 NTFS 更新其他文件 attribute (如 Last Modify Time) 且 Last Access Time 更新待处理时. 此时 NTFS 与其他更新一并更新 Last Access Time, 无额外性能影响.

**注意**

- 当文件的所有内存引用消失时, NTFS 不更新该文件的 directory entry.

若 NTFS 卷含大量文件夹或文件, 且有程序依次短暂访问每个项, 用于生成 Last Access Time 更新的 I/O 带宽可能占总体 I/O 带宽的显著比例.

#### Multiple Data Streams (多数据流)

Data stream 是字节序列. 应用程序通过在 stream 内特定偏移写入数据来填充 stream, 然后在读路径中读取相同偏移来读取数据. 无论使用何种文件系统, 每个文件都有一个与之关联的主、未命名 stream.

然而, NTFS 支持额外的 named data streams, 每个 data stream 是另一段字节序列, 如图 Unnamed and Named Streams 所示. 应用程序可创建额外 named streams 并通过名称访问. 此特性允许将相关数据作为单一单元管理. 例如, 图形程序可在含图像的 NTFS 文件的 named data stream 中存储位图的缩略图.

**Unnamed and Named Streams (未命名与命名流)**

![Unnamed and Named Streams](images/cc781134.71c1e60b-b8a4-4e65-8bfc-a50995dbcfa8(ws.10).gif)

FAT 卷仅支持主、未命名 stream, 因此若尝试将 Streamexample.doc 复制或移动到 FAT 卷或软盘, 会收到错误消息.
