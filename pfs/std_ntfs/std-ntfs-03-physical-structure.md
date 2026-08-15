> **Source**: [How NTFS Works](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10)) (Microsoft Learn; Windows Server 2003 TechNet archive)
> **Local mirror**: `portfs/doc/std_ntfs/` offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25
> **Note**: Microsoft does **not** publish a single complete public NTFS on-disk specification comparable to exFAT / fatgen103.

## NTFS Physical Structure

The following information describes how clusters and sectors are organized on an NTFS volume, how the boot sector on the volume determines the file system, and how the Master File Table (MFT) organizes structures on the volume.

### Clusters and Sectors on an NTFS Volume

A cluster (or allocation unit) is the smallest amount of disk space that can be allocated to hold a file. All file systems used by Windows Server 2003 organize hard disks based on cluster size, which is determined by the number of sectors (units of storage on a hard disk) that the cluster contains. For example, on a disk that uses 512-byte sectors, a 512-byte cluster contains one sector, whereas a 4-kilobyte (KB) cluster contains eight sectors.

Computers access certain sectors on a hard disk during startup to determine which operating system to start and where the partitions are located. The data stored on these sectors varies depending on the computer platform.

#### Sequence of Clusters on an NTFS Volume

Clusters on an NTFS volume are numbered sequentially from the beginning of the partition into logical cluster numbers. NTFS stores all objects in the file system using a record called the Master File Table (MFT), similar in structure to a database.

On NTFS volumes, clusters start at sector zero; therefore, every cluster is aligned on the cluster boundary. Contiguous clusters for file storage allow for faster processing of a file.

**Note**

- Floppy disks do not use NTFS and are always formatted as FAT.

#### Limitations of Cluster Sizes on an NTFS Volume

Because NTFS uses different cluster sizes depending on the size of the volume, each file system has a maximum number of clusters it can support. The smaller the cluster size, the more efficiently a disk potentially stores information because unused space within a cluster cannot be used by other files. And the more clusters a file system supports, the larger the volumes you can create and format by using a particular file system. NTFS uses smaller cluster sizes, which makes it a more efficient file organization structure.

The table Default NTFS Cluster Sizes lists NTFS volume and default cluster sizes.

**Default NTFS Cluster Sizes**

| Volume Size | NTFS Cluster Size |
| --- | --- |
| 7 megabytes (MB)–512 MB | 512 bytes |
| 513 MB–1,024 MB | 1 KB |
| 1,025 MB–2 GB | 2 KB |
| 2 GB–2 terabytes | 4 KB |

#### Maximum Sizes on an NTFS Volume

Before you format an NTFS volume, evaluate the types of files to be stored on the volume so that you can determine whether to use the default cluster size.

When formatting NTFS volumes, you can specify a cluster size of up to 64 KB using the Disk Management snap-in. If you format a volume, but do not specify a cluster size, default values are used. If you want to change the cluster size after the volume is formatted, you must reformat the volume.

Before you choose a cluster size other than the default, note the following important limitations:

- For Microsoft Windows NT, Windows 2000, Windows XP, and Windows Server 2003, the cluster size of FAT16 volumes ranging from 2 gigabytes (GB) through 4 GB is 64 KB, which can create compatibility issues with some applications. For example, setup programs do not compute free space properly on a volume with 64-KB clusters and cannot run because of a perceived lack of free space. For this reason, you can use either NTFS or FAT32 to format volumes larger than 2 GB.
- Because file compression is not supported on cluster sizes greater than 4 KB, the default NTFS cluster size for Windows Server 2003 never exceeds 4 KB.

In theory, the maximum NTFS volume size is 264 clusters minus 1 cluster. However, the maximum NTFS volume size as implemented in Windows Server 2003 is 232 clusters minus 1 cluster. For example, using 64-KB clusters, the maximum NTFS volume size is 256 terabytes minus 64 KB. Using the default cluster size of 4 KB, the maximum NTFS volume size is 16 terabytes minus 4 KB.

**Note**

- If you use large numbers of files in an NTFS folder (300,000 or more), disable short-file name generation for better performance, and especially if the first six characters of the long file names are similar.

The table NTFS Size Limits lists NTFS size limits.

**NTFS Size Limits**

| Description | Limit |
| --- | --- |
| Maximum file size | Architecturally: 16 exabytes minus 1 KB (264 bytes minus 1 KB) Implementation: 16 terabytes minus 64 KB (244 bytes minus 64 KB) |
| Maximum volume size | Architecturally: 264 clusters minus 1 cluster Implementation: 256 terabytes minus 64 KB ( 232 clusters minus 1 cluster) |
| Files per volume | 4,294,967,295 (232 minus 1 file) |

#### Partition Tables on MBR and GUID disks

Master boot record (MBR) disks use both basic and dynamic volumes. Because partition tables on MBR disks support partition sizes only up to 2 terabytes, you must use dynamic volumes to create NTFS volumes over 2 terabytes. Windows Server 2003 manages dynamic volumes in a special database instead of in the partition table; therefore dynamic volumes are not subject to the 2-terabyte physical limit imposed by the partition table. Dynamic NTFS volumes can be as large as the maximum volume size supported by NTFS. Itanium-based computers that use GUID partition table (GPT) disks also support NTFS volumes larger than 2 terabytes.

### Organization of an NTFS Volume

The figure Organization of an NTFS Volume illustrates how NTFS organizes structures on a volume.

**Organization of an NTFS Volume**

![Organization of an NTFS Volume](images/cc781134.737c1f18-1bbc-45c7-9cb7-d61387d78324(ws.10).gif)

The following table describes each of the organizational structures on the NTFS volume.

**NTFS Volume Components**

| Component | Description |
| --- | --- |
| NTFS Boot Sector | Contains the BIOS parameter block that stores information about the layout of the volume and the file system structures, as well as the boot code that loads Windows Server 2003. |
| Master File Table | Contains the information necessary to retrieve files from the NTFS partition, such as the attributes of a file. |
| File System Data | Stores data that is not contained within the Master File Table. |
| Master File Table Copy | Includes copies of the records essential for the recovery of the file system if there is a problem with the original copy. |

### Boot Sectors

On MBR disks, the boot sector, which is located at the first logical sector of each partition, is a critical disk structure for starting your computer. It contains executable code and the data required by the code, including information that the file system uses to access the volume. The boot sector is created when you format a volume. At the end of the boot sector is a 2-byte structure called a signature word or end of sector marker, which is always set to 0x55AA. On computers running Windows Server 2003, the boot sector on the active partition loads into memory and starts Ntldr, which loads the boot menu if multiple versions of Windows are installed, or loads the operating system if only one operating system is installed.

GUID partition table (GPT) disks are similar to MBR disks, except they use primary and backup partition structures to provide redundancy. These structures are located at the beginning and the end of the disk. GPT identifies these structures by their logical block address (LBA) rather than by their relative sectors.

A boot sector consists of the following elements:

- An x86-based CPU jump instruction.
- The original equipment manufacturer identification (OEM ID).
- The BIOS parameter block (BPB), a data structure.
- The extended BPB.
- The executable boot code (or bootstrap code) that starts the operating system.

All Windows Server 2003 boot sectors contain the preceding elements regardless of the type of disk (basic disk or dynamic disk).

#### Components of a Boot Sector

The MBR transfers CPU execution to the boot sector, so the first three bytes of the boot sector must be valid, executable x86-based CPU instructions. This includes a jump instruction that skips the next several nonexecutable bytes.

Following the jump instruction is the 8-byte OEM ID, a string of characters that identifies the name and version number of the operating system that formatted the volume. To preserve compatibility with MS-DOS, Windows Server 2003 records “NTFS” in this field.

**Note**

- You might also see the OEM ID “MSWIN4.0” on disks formatted by Windows 95 and “MSWIN4.1” on disks formatted by Windows 95 OEM Service Release 2 (OSR2), Windows 98, and Windows Millennium Edition. Windows Server 2003 does not use the OEM ID field in the boot sector except for verifying NTFS volumes.

Following the OEM ID is the BPB, which provides information that enables the executable boot code to locate Ntldr. The BPB always starts at the same offset, so standard parameters are in a known location. Disk size and geometry variables are encapsulated in the BPB. Because the first part of the boot sector is an x86 jump instruction, the BPB can be extended in the future by appending new information at the end. The jump instruction needs only a minor adjustment to accommodate this change. The BPB is stored in a packed (unaligned) format.

#### NTFS Boot Sector

The table Boot Sector Sections on an NTFS Volume describes the boot sector of a volume that is formatted with NTFS. When you format an NTFS volume, the format program allocates the first 16 sectors for the boot sector and the bootstrap code.

**Boot Sector Sections on an NTFS Volume**

| Byte Offset | Field Length | Field Name |
| --- | --- | --- |
| 0x00 | 3 bytes | Jump instruction |
| 0x03 | 8 bytes | OEM ID |
| 0x0B | 25 bytes | BPB |
| 0x24 | 48 bytes | Extended BPB |
| 0x54 | 426 bytes | Bootstrap code |
| 0x01FE | 2 bytes | End of sector marker |

On NTFS volumes, the data fields that follow the BPB form an extended BPB. The data in these fields enables Ntldr to find the MFT during startup. On NTFS volumes, the MFT is not located in a predefined sector. For this reason, NTFS can move the MFT if there is a bad sector in the current location of the MFT. However, if the data is corrupted, the MFT cannot be located, and Windows Server 2003 assumes that the volume has not been formatted.

The following example illustrates the boot sector of an NTFS volume that is formatted by using Windows Server 2003. The printout is formatted in three sections:

- Bytes 0x00– 0x0A are the jump instruction and the OEM ID (shown in bold print).
- Bytes 0x0B–0x53 are the BPB and the extended BPB.
- The remaining code is the bootstrap code and the end of sector marker (shown in bold print).

```
Physical Sector: Cyl 0, Side 1, Sector 1
00000000: EB 52 90 4E 54 46 53 20 - 20 20 20 00 02 08 00 00 .R.NTFS ..... ..
00000010: 00 00 00 00 00 F8 00 00 - 3F 00 FF 00 3F 00 00 00 ........?...?...
00000020: 00 00 00 00 80 00 80 00 - 1C 91 11 01 00 00 00 00 ................
00000030: 00 00 04 00 00 00 00 00 - 11 19 11 00 00 00 00 00 ................
00000040: F6 00 00 00 01 00 00 00 - 3A B2 7B 82 CD 7B 82 14 ........:.{..{..
00000050: 00 00 00 00 FA 33 C0 8E - D0 BC 00 7C FB B8 C0 07 .....3.....|....
```

The table BPB and Extended BPB Fields on NTFS Volumes describes the fields in the BPB and the extended BPB on NTFS volumes. The fields starting at 0x0B, 0x0D, 0x15, 0x18, 0x1A, and 0x1C match those on FAT16 and FAT32 volumes. The sample values correspond to the data in this example.

**BPB and Extended BPB Fields on NTFS Volumes**

| Byte Offset | Field Length | Sample Value | Field Name and Definition |
| --- | --- | --- | --- |
| 0x0B | 2 bytes | 00 02 | **Bytes Per Sector**. The size of a hardware sector. For most disks used in the United States, the value of this field is 512. |
| 0x0D | 1 byte | 08 | **Sectors Per Cluster**.The number of sectors in a cluster. |
| 0x0E | 2 bytes | 00 00 | **Reserved Sectors**. Always 0 because NTFS places the boot sector at the beginning of the partition. If the value is not 0, NTFS fails to mount the volume. |
| 0x10 | 3 bytes | 00 00 00 | Value must be 0 or NTFS fails to mount the volume. |
| 0x13 | 2 bytes | 00 00 | Value must be 0 or NTFS fails to mount the volume. |
| 0x15 | 1 byte | F8 | **Media Descriptor**. Provides information about the media being used. A value of F8 indicates a hard disk and F0 indicates a high-density 3.5-inch floppy disk. Media descriptor entries are a legacy of MS-DOS FAT16 disks and are not used in Windows Server 2003. |
| 0x16 | 2 bytes | 00 00 | Value must be 0 or NTFS fails to mount the volume. |
| 0x18 | 2 bytes | 3F 00 | Not used or checked by NTFS. |
| 0x1A | 2 bytes | FF 00 | Not used or checked by NTFS. |
| 0x1C | 4 bytes | 3F 00 00 00 | Not used or checked by NTFS. |
| 0x20 | 4 bytes | 00 00 00 00 | The value must be 0 or NTFS fails to mount the volume. |
| 0x24 | 4 bytes | 80 00 80 00 | Not used or checked by NTFS. |
| 0x28 | 8 bytes | 1C 91 11 01 00 00 00 00 | **Total Sectors**. The total number of sectors on the hard disk. |
| 0x30 | 8 bytes | 00 00 04 00 00 00 00 00 | **Logical Cluster Number for the File $MFT**. Identifies the location of the MFT by using its logical cluster number. |
| 0x38 | 8 bytes | 11 19 11 00 00 00 00 00 | **Logical Cluster Number for the File $MFTMirr**. Identifies the location of the mirrored copy of the MFT by using its logical cluster number. |
| 0x40 | 1 byte | F6 | **Clusters Per MFT Record**. The size of each record. NTFS creates a file record for each file and a folder record for each folder that is created on an NTFS volume. Files and folders smaller than this size are contained within the MFT. If this number is positive (up to 7F), then it represents clusters per MFT record. If the number is negative (80 to FF), then the size of the file record is 2 raised to the absolute value of this number. |
| 0x41 | 3 bytes | 00 00 00 | Not used by NTFS. |
| 0x44 | 1 byte | 01 | **Clusters Per Index Buffer**. The size of each index buffer, which is used to allocate space for directories. If this number is positive (up to 7F), then it represents clusters per MFT record. If the number is negative (80 to FF), then the size of the file record is 2 raised to the absolute value of this number. |
| 0x45 | 3 bytes | 00 00 00 | Not used by NTFS. |
| 0x48 | 8 bytes | 3A B2 7B 82 CD 7B 82 14 | **Volume Serial Number**. The volume’s serial number. |
| 0x50 | 4 bytes | 00 00 00 00 | Not used by NTFS. |

### Master File Table

When you format a volume with NTFS, Windows Server 2003 creates an MFT and metadata files on the partition. The MFT is a relational database that consists of rows of file records and columns of file attributes. It contains at least one entry for every file on an NTFS volume, including the MFT itself.

The MFT stores the information required to retrieve files from the NTFS partition.

#### MFT and Metadata Files

Because the MFT stores information about itself, NTFS reserves the first 16 records of the MFT for metadata files (approximately 16 KB), which are used to describe the MFT. Metadata files that begin with a dollar sign ($) are described in the table Metadata Files Stored in the MFT. The remaining records of the MFT contain the file and folder records for each file and folder on the volume.

**Metadata Files Stored in the MFT**

| System File | File Name | MFT Record | Purpose of the File |
| --- | --- | --- | --- |
| Master file table | $Mft | 0 | Contains one base file record for each file and folder on an NTFS volume. If the allocation information for a file or folder is too large to fit within a single record, other file records are allocated as well. |
| Master file table mirror | $MftMirr | 1 | Guarantees access to the MFT in case of a single-sector failure. It is a duplicate image of the first four records of the MFT. |
| Log file | $LogFile | 2 | Contains information used by NTFS for faster recoverability. The log file is used by Windows Server 2003 to restore metadata consistency to NTFS after a system failure. The size of the log file depends on the size of the volume, but you can increase the size of the log file by using the Chkdsk command. |
| Volume | $Volume | 3 | Contains information about the volume, such as the volume label and the volume version. |
| Attribute definitions | $AttrDef | 4 | Lists attribute names, numbers, and descriptions. |
| Root file name index | . | 5 | The root folder. |
| Cluster bitmap | $Bitmap | 6 | Represents the volume by showing free and unused clusters. |
| Boot sector | $Boot | 7 | Includes the BPB used to mount the volume and additional bootstrap loader code used if the volume is bootable. |
| Bad cluster file | $BadClus | 8 | Contains bad clusters for a volume. |
| Security file | $Secure | 9 | Contains unique security descriptors for all files within a volume. |
| Upcase table | $Upcase | 10 | Converts lowercase characters to matching Unicode uppercase characters. |
| NTFS extension file | $Extend | 11 | Used for various optional extensions such as quotas, reparse point data, and object identifiers. |
| | | 12–15 | Reserved for future use. |

The data segment locations for both the MFT and the backup MFT, $Mft and $MftMirr, respectively, are recorded in the boot sector. The $MftMirr is a duplicate image of either the first four records of the $Mft or the first cluster of the $Mft, whichever is larger. If any MFT records in the mirrored range are corrupted or unreadable, NTFS reads the boot sector to find the location of the $MftMirr. NTFS then reads the $MftMirr and uses the information in $MftMirr instead of the information in the MFT. If possible, the correct data from the $MftMirr is written back to the corresponding location in the $Mft.

#### MFT Zone

To prevent the MFT from becoming fragmented, NTFS reserves 12.5 percent of volume by default for exclusive use of the MFT. This space, known as the MFT zone, is not used to store data unless the remainder of the volume becomes full.

Depending on the average file size and other variables, as the volume fills to capacity, either the MFT zone or the unreserved space on the volume becomes full first.

- Volumes that have a small number of large files exhaust the unreserved space first.
- Volumes with a large number of small files exhaust the MFT zone space first.

In either case, fragmentation of the MFT occurs when one region or the other becomes full. You can change the size of the MFT zone for newly created volumes by to correspond to a percentage of the volume to be used as the MFT zone. The MFT zone sizes follow:

- Setting 1, the default, reserves approximately 12.5 percent of the volume.
- Setting 2 reserves approximately 25 percent.
- Setting 3 reserves approximately 37.5 percent.
- Setting 4 reserves approximately 50 percent.

In most computers, the default setting of 1 is adequate. The default setting accommodates volumes with an average file size of 8 KB. Storing a large number of smaller files might necessitate that you increase the size of the MFT zone for new volumes.

After you increase the size of the MFT zone, NTFS does not immediately allocate space to accommodate the size of the new MFT zone. Instead, NTFS exhausts the original reserved space before increasing the size of the MFT zone. When the original space is exhausted, NTFS looks for the next contiguous space large enough to hold the additional MFT zone, which can cause the MFT to become fragmented. You can adjust the zone size for the MFT if the defaults do not fit your needs.

### NTFS File Record Attributes

Every allocated sector on an NTFS volume belongs to a file. Even the file system metadata is part of a file. NTFS views each file (or folder) as a set of file attributes. File elements such as its name, its security information, and even its data are file attributes. Each attribute is identified by an attribute type code and an optional attribute name.

File and folder records are 1 KB each and are stored in the MFT, the attributes of which are written to the allocated space in the MFT. Besides file attributes, each file record contains information about the position of the file record in the MFT.

When a file’s attributes can fit within the MFT file record for that file, they are called resident attributes. Attributes such as file name and time stamp are always resident. When the amount of information for a file does not fit in its MFT file record, some file attributes become nonresident. Nonresident attributes are allocated one or more clusters of disk space. A portion of the nonresident attribute remains in the MFT and points to the external clusters. NTFS creates the Attribute List attribute to describe the location of all attribute records. The table NTFS File Attribute Types lists the file attributes currently defined by NTFS.

**NTFS File Attribute Types**

| Attribute Type | Description |
| --- | --- |
| Standard Information | Information such as access mode (read-only, read/write, and so forth) timestamp, and link count. |
| Attribute List | Locations of all attribute records that do not fit in the MFT record. |
| File Name | A repeatable attribute for both long and short file names. The long name of the file can be up to 255 Unicode characters. The short name is the 8.3, case-insensitive name for the file. Additional names, or hard links, required by POSIX can be included as additional file name attributes. |
| Data | File data. NTFS supports multiple data attributes per file. Each file typically has one unnamed data attribute. A file can also have one or more named data attributes. |
| Object ID | A volume-unique file identifier. Used by the distributed link tracking service. Not all files have object identifiers. |
| Logged Tool Stream | Similar to a data stream, but operations are logged to the NTFS log file just like NTFS metadata changes. This attribute is used by EFS. |
| Reparse Point | Used for mounted drives. This is also used by Installable File System (IFS) filter drivers to mark certain files as special to that driver. |
| Index Root | Used to implement folders and other indexes. |
| Index Allocation | Used to implement the B-tree structure for large folders and other large indexes. |
| Bitmap | Used to implement the B-tree structure for large folders and other large indexes. |
| Volume Information | Used only in the $Volume system file. Contains the volume version. |

NTFS creates a file record for each file and a folder record for each folder created on an NTFS volume. The MFT includes a separate file record for the MFT itself. These file and folder records are 1 KB each and are stored in the MFT. The attributes of the file are written to the allocated space in the MFT. Besides file attributes, each file record contains information about the position of the file record in the MFT. The figure MFT Entry with Resident Record shows the contents of an MFT record for a small file or folder. Small files and folders (typically, 900 bytes or smaller) are entirely contained within the file’s MFT record.

**MFT Entry with Resident Record**

![MFT Entry with Resident Record](images/cc781134.86787c15-cf0a-4cb9-8ba1-ff1afd37aaf5(ws.10).gif)

Typically, each file uses one file record. However, if a file has a large number of attributes or becomes highly fragmented, it might need more than one file record. If this is the case, the first record for the file, the base file record, stores the location of the other file records required by the file.

Folder records contain index information. Small folder records reside entirely within the MFT structure, while large folders are organized B-tree structures and have records with pointers to external clusters that contain folder entries that cannot be contained within the MFT structure.

The benefit of using B-tree structures is evident when NTFS enumerates files in a large folder. The B-tree structure allows NTFS to group, or index, similar file names and then search only the group that contains the file, minimizing the number of disk accesses needed to find a particular file, especially for large folders. Because of the B-tree structure, NTFS outperforms FAT for large folders because FAT must scan all file names in a large folder before listing all of the files.

#### Last Access Time

Each file and folder on an NTFS volume contains an attribute called Last Access Time. This attribute shows when the file or folder was last accessed, such as when a user performs a folder listing, adds files to a folder, reads a file, or makes changes to a file. The most up-to-date Last Access Time is always stored in memory and is eventually written to disk within two places:

- The file’s attribute, which is part of its MFT record.
- A directory entry for the file. The directory entry is stored in the folder that contains the file. Files with multiple hard links have multiple directory entries.

The Last Access Time on disk is not always current because NTFS looks for a one-hour interval before forcing the Last Access Time updates to disk. NTFS also delays writing the Last Access Time to disk when users or programs perform read-only operations on a file or folder, such as listing the folder’s contents or reading (but not changing) a file in the folder. If the Last Access Time is kept current on disk for read operations, all read operations become write operations, which impacts NTFS performance.

**Note**

- File-based queries of Last Access Time are accurate even if all on-disk values are not current. NTFS returns the correct value on queries because the accurate value is stored in memory.

NTFS eventually writes the in-memory Last Access Time to disk as follows.

##### Within the file’s attribute

NTFS typically updates a file’s attribute on disk if the current Last Access Time in memory differs by more than an hour from the Last Access Time stored on disk, or when all in-memory references to that file are gone, whichever is more recent. For example, if a file’s current Last Access Time is 1:00 P.M., and you read the file at 1:30 P.M., NTFS does not update the Last Access Time. If you read the file again at 2:00 P.M., NTFS updates the Last Access Time in the file’s attribute to reflect 2:00 P.M. because the file’s attribute shows 1:00 P.M. and the in-memory Last Access Time shows 2:00 P.M.

##### Within a directory entry for a file

NTFS updates the directory entry for a file during the following events:

- When NTFS updates the file’s Last Access Time and detects that the Last Access Time for the file differs by more than an hour from the Last Access Time stored in the file’s directory entry. This update typically occurs after a program closes the handle used to access a file within the directory. If the program holds the handle open for an extended time, a lag occurs before the change appears in the directory entry.
- When NTFS updates other file attributes such as Last Modify Time, and a Last Access Time update is pending. In this case, NTFS updates the Last Access Time along with the other updates without additional performance impact.

**Note**

- NTFS does not update a file’s directory entry when all in-memory references to that file are gone.

If you have an NTFS volume with a high number of folders or files, and a program is running that briefly accesses each of these in turn, the I/O bandwidth used to generate the Last Access Time updates can be a significant percentage of the overall I/O bandwidth.

#### Multiple Data Streams

A data stream is a sequence of bytes. An application populates the stream by writing data at specific offsets within the stream. The application can then read the data by reading the same offsets in the read path. Every file has a main, unnamed stream associated with it, regardless of the file system used.

However, NTFS supports additional named data streams in which each data stream is an alternate sequence of bytes as illustrated in the figure Unnamed and Named Streams. Applications can create additional named streams and access the streams by referring to their names. This feature permits related data to be managed as a single unit. For example, a graphics program can store a thumbnail image of bitmap in a named data stream within the NTFS file containing the image.

**Unnamed and Named Streams**

![Unnamed and Named Streams](images/cc781134.71c1e60b-b8a4-4e65-8bfc-a50995dbcfa8(ws.10).gif)

FAT volumes support only the main, unnamed stream, so if you try to copy or move Streamexample.doc to a FAT volume or floppy disk, you receive an error message.
