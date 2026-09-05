> **Source**: [How NTFS Works](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10)) (Microsoft Learn; Windows Server 2003 TechNet archive)
> **Local mirror**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25
> **Note**: Microsoft does **not** publish a single complete public NTFS on-disk specification comparable to exFAT / fatgen103.

## NTFS Processes and Interactions

The following sections describe NTFS processes and interactions.

### Mounting an NTFS Volume

When mounting an NTFS volume, the MBR executes code to start up the boot sector. The boot sector then executes additional code to mount the volume.

##### Master Boot Code Startup Process

The MBR contains a small amount of executable code called the master boot code, the disk signature, and the partition table for the disk. During startup, the master boot code performs the following activities:

1. Scans the partition table for the active partition.
2. Finds the starting sector of the active partition.
3. Loads a copy of the boot sector from the active partition into memory.
4. Transfers control to the executable code in the boot sector.

##### Boot Sector Startup Process

Computers use the boot sector to run instructions during startup. The initial startup process is summarized in the following steps:

1. The system BIOS and the CPU initiate the power-on self test (POST).
2. The BIOS finds the boot device, which is typically the first disk the BIOS finds, unless the controller is configured to boot from a different disk.
3. The BIOS loads the first physical sector of the boot device into memory and transfers CPU execution to that memory address.

If the boot device is on a hard disk, the BIOS loads the MBR. The master boot code in the MBR loads the boot sector of the active partition, and transfers CPU execution to that memory address. On computers that are running Windows Server 2003, the executable boot code in the boot sector finds Ntldr, loads it into memory, and transfers execution to that file.

**Note**

- Windows Server 2003 cannot start up from a spanned, striped, or RAID-5 volume on dynamic disks. These disk structures cannot be registered into the MBR partition table; therefore, a system volume that uses these structures cannot start.

If drive A contains a floppy disk, the system BIOS loads the first sector (the boot sector) of the disk into memory. If the disk is a startup disk (formatted by MS-DOS with core operating system files applied), the boot sector loads into memory and uses the executable boot code to transfer CPU execution to Io.sys, a core MS-DOS operating system file. If the floppy disk is not a startup disk, the executable boot code displays an error message.

**Note**

- These messages do not appear on normally functioning systems that are configured to look for the startup files on drive C first. On many computers, an option in the CMOS setup program allows the user to set the sequence of installed disks that the system searches to find the startup files.

If you get similar errors when trying to start the computer from the hard disk, the boot sector might be corrupted.

Initially, the startup process is independent of disk format and operating system. The unique characteristics of operating and file systems become important when the boot sector’s executable boot code starts.

### Formatting Volumes

During volume format, Windows Server 2003 places key NTFS file system structures on the volume, including the boot sector and the MFT as well as replacing Ntldr. Formatting also aligns clusters at the cluster size boundary.

Formatting a volume will check the integrity of all sectors on the volume during the process, as well as allow you to change the cluster size used on the volume. If a volume is formatted using Quick format, the file system structure on the volume is created, but the integrity of every sector in the volume is not checked.

### Converting Volumes

Windows Server 2003 can convert previous versions of NTFS to the new version of NTFS used in Windows Server 2003.

#### Converting NTFS Volumes Formatted By Using Windows 2000

When Windows Server 2003 first mounts an NTFS volume that was formatted by using Windows 2000, Windows Server 2003 converts the NTFS volume to NTFS 3.1. The conversion consists of changing the NTFS version from 3.0 to 3.1. No other changes are made to existing metadata or files on the volume. However, Windows Server 2003 uses a different header style for new files created on NTFS 3.1 volumes. As a result of this change, some non-Microsoft imaging programs cannot create images of NTFS 3.1 volumes. Contact the manufacturer of your imaging program to find out if a version is available that supports NTFS 3.1 volumes in Windows Server 2003.

Computers running Windows NT 4.0 with Service Pack 4 or later or Windows 2000 can access NTFS 3.1 volumes without any conversion or additional service packs. Also, note that NTFS 3.1 is identical in Windows XP and Windows Server 2003.

#### Converting NTFS Volumes Formatted By Using Windows NT 4.0 and Earlier

When you upgrade the operating system from Windows NT 4.0 to Windows Server 2003, all local volumes formatted by using the version of NTFS used in Windows NT 4.0 and earlier are upgraded to NTFS 3.1. The upgrade occurs when Windows Server 2003 mounts the volume for the first time after Windows Server 2003 Setup is completed. (The upgrade does not take place during Setup.) Any NTFS volumes that are removed or turned off during Setup, or added after Setup, are converted when Windows Server 2003 mounts the volumes.

The Ntfs.sys driver performs the conversion by determining which version of NTFS is used on the volume and converting the volume if necessary. The conversion takes only a few seconds on any size volume and consists of the following new records in the master file table:

- $Secure, which contains unique security descriptors for all files within a volume.
- $Extend, which is used for extensions such as quotas, reparse point data, and object identifiers. The conversion process also adds three new files the to $Extend directory:

 - $Quota, used for disk quotas.
 - $Reparse, used for reparse points.
 - $ObjID, used for distributed link tracking.

Both $Secure and $Extend take the place of previously unused master file table (MFT) records, so sufficient space always exists in the volume for these two records. However, $Quota, $Reparse, and $ObjID are new additions to the MFT, and you must have enough free space in the volume to contain these files, or the conversion fails.

If the conversion fails, the volume is still available, but you can only perform NTFS-related tasks that were available in Windows NT 4.0 or earlier. To convert the volume to NTFS 3.1, you must free disk space by deleting or moving files and then dismount the volume.

**Note**

- Removable media that is formatted by using the previous version of NTFS is upgraded after the installation or upgrade process, or when you insert the media and Windows Server 2003 mounts it.

#### Limitations in Converting Volumes

The conversion is a one-way process. After you convert a volume to NTFS, you cannot reconvert the volume to FAT without backing up your data, reformatting the volume as FAT, and then restoring your data. There should also be a certain amount of free space on the volume and sufficient memory to update the cache.

The following limitations also apply to conversion of a volume from FAT to NTFS:

- In multiple-boot configurations, NTFS volumes are accessible only by using Windows NT 4.0 with Service Pack 4 or later, Windows 2000, Windows XP, or Windows Server 2003.
- When you install Recovery Console onto a volume that is formatted for either the FAT16 or FAT32 file systems, and then convert the volume to NTFS, the Recovery Console no longer runs. This problem occurs because the file-system-specific boot files (in the cmdcons folder of the system volume) that are used to run Recovery Console are not valid for a volume that has been converted to NTFS. You can re-install Recovery Console from the Windows Server 2003 operating system disk after the conversion. You can also use the Windows Server 2003 operating system disk to start Recovery Console.
- Because formatting in Windows Server 2003 aligns FAT data clusters at the cluster size boundary, conversion can preserve the cluster size for the size of the volume (up to 4 KB) instead of using the 512-byte cluster size used in Windows 2000 for converted volumes. The table Cluster Sizes for Volumes Converted to NTFS lists cluster sizes for volumes converted to NTFS.

 **Cluster Sizes for Volumes Converted to NTFS**

 | Original FAT Cluster Size | Converted NTFS Cluster Size |
 | --- | --- |
 | 512 bytes | 512 bytes |
 | 1 KB | 1 KB |
 | 2 KB | 2 KB |
 | 4 KB and larger | 4 KB |

### File Naming

Windows Server 2003 supports both long and short file names on NTFS volumes.

#### File Names in Windows Server 2003

Every time you create a file with a long file name, NTFS creates a second file entry that has a similar 8.3 short file name. A file with an 8.3 short file name has a file name containing 1 to 8 characters and a file name extension containing 1 to 3 characters. The file name and file name extension are separated by a period.

File names in Windows Server 2003 can be up to 255 characters and can contain spaces, multiple periods, and special characters that are not allowed in MS-DOS file names. Windows Server 2003 makes it possible for other operating systems to access files that have long names by generating an MS-DOS-readable (8.3) name for each file. These MS-DOS-readable names also enable MS-DOS-based and Windows 3.*x*–based applications to recognize and load files that have long file names. When a program saves a file on a computer running Windows Server 2003, both the 8.3 file name and long file name are retained.

**Note**

- The 8.3 format means that files can have between 1 and 8 characters in the file name. The name must start with a letter or a number and can contain any characters except the following:
- . " / \ [ ] : ; | = , \* ? (space)
- An 8.3 file name typically has a file name extension that is from one to three characters long and has the same character restrictions. A period separates the file name from the file name extension.
- Several special file names are reserved by the system and cannot be used for files or folders: CON, AUX, COM1, COM2, COM3, COM4, LPT1, LPT2, LPT3, PRN, NUL

#### How NTFS Generates Short File Names

In Windows Server 2003, both FAT and NTFS use the Unicode character set, which contains several prohibited characters that MS-DOS cannot read, for their names. To generate a short MS-DOS-readable file name, Windows Server 2003 deletes all of these characters from the long file name and removes any spaces. Because an MS-DOS-readable file name can have only one period, Windows Server 2003 also removes extra periods from the file name. If necessary, Windows Server 2003 truncates the file name to six characters and appends a tilde (**~**) and a number. For example, each non-duplicate file name is appended with **~1**. Duplicate file names end with **~2**, then **~3**, and so on. After the file names are truncated, the file name extensions are truncated to three or fewer characters. Finally, when displaying file names at the command line, Windows Server 2003 translates all characters in the file name and extension to uppercase.

**Note**

- You can permit extended characters by using the **fsutil behavior set** command. You must restart the computer before this setting takes effect. For more information about using the **fsutil behavior set** command, see the topic Fsutil: behavior in Help and Support Center in Windows Server 2003.

When five or more files exist that can result in duplicate short file names, Windows Server 2003 uses a slightly different method for creating short file names. For the fifth and subsequent files, Windows Server 2003:

- Uses only the first two letters of the long file name.
- Generates the next four letters of the short file name by mathematically manipulating the remaining letters of the long file name.
- Appends **~1** (or another number, if necessary, to avoid a duplicate file name) to the result.

This method substantially improves performance when Windows Server 2003 must create short file names for a large number of files with similar long file names. Windows Server 2003 uses this method to create short names for files on both FAT and NTFS volumes.

The following table shows the short file names for files created by six tests.

**Short File Names Created by Windows Server 2003 — Example One**

| Long File Name | Short File Name |
| --- | --- |
| This is test 1.txt | THISIS~1.TXT |
| This is test 2.txt | THISIS~2.TXT |
| This is test 3.txt | THISIS~3.TXT |
| This is test 4.txt | THISIS~4.TXT |
| This is test 5.txt | THA1CA~1.TXT |
| This is test 6.txt | THA1CE~1.TXT |

If the long file names in the preceding table are created in a different order, their short file names are different, as shown in the following table.

**Short File Names Created by Windows Server 2003 — Example Two**

| Long File Name | Short File Name |
| --- | --- |
| This is test 2.txt | THISIS~1.TXT |
| This is test 3.txt | THISIS~2.TXT |
| This is test 1.txt | THISIS~3.TXT |
| This is test 4.txt | THISIS~4.TXT |
| This is test 5.txt | THA1CA~1.TXT |
| This is test 6.txt | THA1CE~1.TXT |

When you delete a file, its short file name is also deleted. When you create new files in the same folder, Windows Server 2003 might re-use short file names that have been deleted. For instance, in Example 1, if you delete the file “This is test 1.txt,” and then create a new file called “This is test 7.txt,” its short file name becomes THISIS~1.TXT.

If you have a large number of files (300,000 or more) in a folder, and the files have long file names with the same initial characters, the time required to create the files increases. The increase occurs because NTFS bases the short file name on the first six characters of the long file name. In folders with more than 300,000 files, the short file names start to conflict after NTFS uses all of the 8.3 names that are similar to the long file names. Repeated conflicts between a generated short file name and existing short file names cause NTFS to regenerate the short file name from 6 to 8 times.

### Compression of Files and Folders

NTFS supports compression on individual files, all files within a folder, and all files within NTFS volumes. Because compression is implemented within NTFS, any Windows-based program can read and write compressed files without determining the compression state of the file. Compression is set in a bit within the file header, while information about compression is stored in the Data file attribute.

Files and directories are compressed and decompressed by passing FSCTL\_SET\_COMPRESSION code to DeviceIoControl. A compressed file or directory then has the flag FILE\_ATTRIBUTE\_COMPRESSED associated with it. Applications can determine a file or directory’s compression state using GetFileAttributes.

When a program opens a compressed file, NTFS decompresses only the portion of the file being read and then copies that data to memory. By leaving data in memory uncompressed, NTFS performance is not impacted when it reads or modifies data in memory. NTFS compresses the modified or new data in the file when the data is later written to disk.

The compression algorithms in NTFS support cluster sizes of up to 4 KB. When the cluster size is greater than 4 KB on an NTFS volume, none of the NTFS compression features are available.

#### Moving and Copying Files or Folders

Moving and copying files and folders can change their compression state. The resulting compression state depends on whether you move or copy the files and whether you move the files between NTFS volumes or to FAT volumes.

NTFS supports compression on one file, all files in a directory, or all files on a volume. Compression is set in a bit within the file header, while information about compression is stored in the Data file attribute. When the bit is set, the system compresses the file when it is saved and decompresses it as needed.

Compression adds overhead to the system because a compressed NTFS file is decompressed, copied, and then recompressed as a new file even when the file is copied in the same computer. Any change to the compression attribute is applied to the files you specify for moving or copying. If you compress all files in the volume, the process might take a few minutes to finish, depending on the size of the volume, the number of files to compress, and the speed of the computer. The delay occurs because Windows Server 2003 must change the compression state of every folder on the volume and compress or uncompress every file on the volume.

Changing the compression state of folders is relatively fast because for each folder Windows Server 2003 changes only the compression attribute. However, compressing or uncompressing every file on the volume takes longer because NTFS must read data in its current form (compressed or uncompressed) from the disk, convert the data to its new form in memory, and then write the data back to disk.

##### Moving Files or Folders within an NTFS Volume

When you move an uncompressed file or folder to another folder on the NTFS volume, the file remains uncompressed. The figure Moving an Uncompressed File to a Compressed Folder illustrates the result of moving an uncompressed file to a compressed folder.

**Moving an Uncompressed File to a Compressed Folder**

![Moving Uncompressed File to Compressed Folder](images/cc781134.25020930-608e-463d-bfaf-047d12da52c6(ws.10).gif)

When you move a compressed file to an uncompressed folder, the file remains uncompressed after the move. The figure Moving a Compressed File to an Uncompressed Folder illustrates the result of moving a compressed file or folder to a compressed folder.

**Moving a Compressed File to an Uncompressed Folder**

![Moving a Compressed File to an Uncompressed Folder](images/cc781134.243edc64-3367-4cb5-8fe3-38b015cbe61d(ws.10).gif)

##### Copying Files or Folders within an NTFS Volume

Copying a file to a folder takes on the compression attribute of the target folder.

If you copy a compressed file to an uncompressed folder, the file is uncompressed when it is copied to the folder, as shown in the figure Copying a Compressed File to an Uncompressed Folder.

**Copying a Compressed File to an Uncompressed Folder**

![Copying Compressed File to Uncompressed Folder](images/cc781134.72fa8c1b-be2a-405c-8ee5-750cc65c4c73(ws.10).gif)

When you copy a file to a folder that already contains a file of the same name, the copied file takes on the compression attribute of the target file, as shown in the figure Copying a File to a Folder that Contains a File of the Same Name.

**Copying a File to a Folder that Contains a File of the Same Name**

![Copy File to Folder that Contains Same Name File](images/cc781134.f7fd237f-33e6-4a97-8e01-337e3f3dce5f(ws.10).gif)

##### Copying Files between FAT and NTFS Volumes

Files copied from a FAT folder to an NTFS folder take on the compression attribute of the target folder. Compressed files copied from an NTFS volume to a FAT volume or floppy disk are uncompressed.

### Mounted Drives on NTFS Volumes

Mounted drives, also known as volume mount points or drive paths, are volumes attached to an empty folder on an NTFS volume. Mounted drives function the same way as any other volume, but are assigned a label or name instead of a drive letter. Mounted drives are robust against system changes that occur when devices are added or removed from a computer. They are not subject to the 26-volume limit imposed by drive letters, so you can use them for access to more than 26 volumes on your computer.

The version of NTFS included with Windows Server 2003 must be used on the host volume. However, the volume to be mounted can be formatted in any file system supported by Windows Server 2003.

One volume can host multiple mounted drives, providing a way for you to easily extend the storage capacity of any particular volume on a Windows Server 2003 system. Users on the local computer or users who connect to it over a network can continue to use the same drive letter for access to the volume, but multiple volumes can be in use simultaneously from that drive letter.

Only NTFS volumes can hold a mounted drive, although any local drive can be mounted on one.

#### Implementing Mounted Drives

NTFS mounted drives are implemented by using reparse points and are subject to their restrictions. Reparse points are a collection of user-defined data. The format of this data is understood by the application which stores the data, and a file system filter, which you install to interpret the data and process the file. When an application sets a reparse point, it stores this data, plus a reparse tag, which uniquely identifies the data it is storing. When the file system opens a file with a reparse point, it attempts to find the file system filter associated with the data format identified by the reparse tag. If a file system filter is found, the filter processes the file as directed by the reparse data. If a file system filter is not found, the file open operation fails.

The following restrictions apply to reparse points:

- Reparse points can be established for a directory, but the directory must be empty. Otherwise, NTFS fails to establish the reparse point. In addition, you cannot create directories or files in a directory that contains a reparse point.
- Reparse points and extended attributes are mutually exclusive. NTFS cannot create a reparse point when the file contains extended attributes, and it cannot create extended attributes on a file that contains a reparse point.
- Reparse point data cannot exceed 16 kilobytes. Setting a reparse point fails if the amount of data to be placed in the reparse point exceeds this limit.

### Hard Links

A hard link is an NTFS-only based link to a given file. When you create a hard link to a file on an NTFS volume, NTFS adds a directory entry for the hard link without duplicating the original file. By creating hard links you can:

- Use the same file name as the original file but appear in different folders.
- Use different file names from the original file but appear in the same folder.
- Use different file names from the original file and appear in different folders.

Because a hard link is a directory entry for a file, an application can modify a file by using any of its hard links. Applications that use any other hard link can detect the changes. However, directory entries for hard links are updated only when a user accesses a file by using the hard link. For example, if a user opens and modifies a file by using its hard link, and the size of the original file changes, the hard link that is used to access the file also shows the new size.

Hard links do not have security descriptors; instead, the security descriptor belongs to the original file to which the hard link points. Thus, if you change the security descriptor of any hard link, you actually change the underlying file’s security descriptor. All hard links that point to the file allow the newly specified access. You cannot give a file different security descriptors on a per-hard-link basis.

Hard links use the Win32 function CreateHardLink to create hard links between files.

### Distributed Link Tracking

Distributed link tracking ensures that shell shortcuts and OLE links continue to work after the target file is renamed or moved. When you create a shortcut to a file on an NTFS volume, distributed link tracking stamps a unique object identifier (ID) into the target file, known as the link source. Information about the object ID is also stored within the referring file, known as the link client. Distributed link tracking uses this object ID to locate the link source in any combination of the following events that occur on NTFS volumes within a Windows Server 2003-based domain:

- The link source is renamed.
- The link source is moved to another folder on the same volume or to a different volume on the same computer.
- The link source is moved from one shared network folder to another shared network folder on different computers within the same domain.
- The computer containing the link source is renamed.
- The name of the shared network folder containing the link source has changed.
- The volume containing the link source is moved to another computer within the same domain.

**Note**

- Distributed link tracking works only on NTFS volumes in computers running Windows 2000, Windows XP, or Windows Server 2003. The NTFS volumes cannot be on removable media.

Distributed link tracking attempts to maintain even those links that do not occur within a domain: cross-domain, within a workgroup, or on a single computer that is not connected to a network. Links can always be maintained in these events when a link source is moved within a computer, or when the network shared folder on the link source computer is changed. Typically, links can be maintained when the link source is moved to another computer; however, this form of tracking is less reliable over time.

Distributed link tracking uses different services for client and server:

- The Distributed Link Tracking Client service runs on all Windows 2000-based and Windows Server 2003-based computers. In computers that are not part of a network, the Client service performs all activities related to link tracking.
- The Distributed Link Tracking Server service runs on Windows 2000 and Windows Server 2003 domain controllers. The Server service maintains information relating to the movement of link sources. Because of this service and the information it maintains, links within a domain are more reliable than those outside a domain. For computers that run in a domain, the Distributed Link Tracking Client service takes advantage of this information by communicating with the Distributed Link Tracking Server service.

The Distributed Link Tracking Client service monitors activity on NTFS volumes and stores maintenance information in a file called Tracking.log, which is located at the root of each volume in a hidden folder called System Volume Information. This folder is protected by permissions that allow only the system to have access to it. The System Volume Information folder is also used by other Windows Server 2003 services such as Indexing Service.

### Sparse Files

Sparse files provide a method of saving disk space for files that contain meaningful data, as well as large sections of data composed of zeros. If an NTFS file is marked as sparse, then NTFS allocates disk clusters only for the data explicitly specified by the application. Non-specified ranges of the file are represented by non-allocated space on the disk. When a sparse file is read from allocated ranges, the data is returned as it was stored. Data read from non-allocated ranges is returned as zeros.

File system application programming interfaces (APIs) allow for the file to be copied or backed as actual bits and sparse stream ranges.File system APIs also allow for querying allocated ranges. Programs that implement these APIs then need only to read allocated ranges to recover all data in the file. The result is efficient file system storage and access. The figure Sparse Data Storage shows how data is stored with and without the sparse file attribute set.

**Sparse Data Storage**

![Sparse Data Storage](images/cc781134.eda85415-6d21-4b03-8a50-644dd718dd90(ws.10).gif)

For example, the properties of a file might show that the file is a 1-GB sparse file. Although the file is 1 GB, it occupies only 64 KB of disk space.

**Note**

- Only NTFS volumes mounted by Windows 2000, Windows XP, or the Windows Server 2003 family support sparse files. If you copy or move a sparse file to a FAT volume or an NTFS volume mounted by an operating system other than those listed previously, the file is built to its originally specified size. If the required space is not available, the operation fails.

### Disk Quotas

You can enable disk quotas to restrict the amount of volume space users take up on remote or local computers with NTFS file systems. Disk quotas uses names from the domain in which the server resides. An administrator can then set disk quotas against those users in the domain.

For additional information about Disk Quotas, see the [Disk Quotas Technical Reference](cc786969%28v=ws.10%29).

### NTFS Change Journal

As files, folders, and other NTFS objects are added, deleted, and modified, NTFS enters change journal records in streams, one for each volume on the computer.

The total size of all the records currently in the journal varies, but there is a configurable maximum size. The change journal can exceed the maximum size until the size reaches an outer threshold, at which point a portion of the oldest records are deleted until the change journal is restored to its maximum size. The maximum size of the change journal is configurable but cannot be reduced, only increased.

The change journal conveys significant scalability benefits to applications that might otherwise need to scan an entire volume for changes. File system indexing, replication managers, virus scanners, and incremental backup applications can benefit from using the change journal.

The change journal is much more efficient than time stamps or file notifications for determining changes in a particular namespace. Applications that must rescan an entire volume to determine changes can now scan once and subsequently refer to the change journal. The I/O cost depends on how many files have changed, not on how many files exist on the volume.

The APIs are fully documented and can be leveraged by independent software vendors (ISVs). Microsoft uses the change journal in Windows Server 2003 components such as the Indexing Service and File Replication Service. ISVs can use this feature to enhance the scalability and robustness of a range of products including backup, antivirus, and auditing tools.

### NTFS File System Recoverability

NTFS is a recoverable file system that guarantees the consistency of the volume by using standard transaction logging and recovery techniques. In the event of a system failure, NTFS runs a recovery procedure that accesses information stored in a transaction log file. The NTFS recovery procedure guarantees that the volume is restored to a consistent state. Transaction logging requires very little overhead.

#### Recovering NTFS File Structures

NTFS views each operation that modifies a file on a volume as a transaction and manages each one as an integral unit. NTFS might also break a single complex operation into multiple transactions. After a transaction is started, it is either completed, or if an event occurs that causes the operation to fail, it is rolled back, and the NTFS volume returns to its state before the transaction began. Events that can cause an operation to fail include bad sectors, transient low-memory conditions, and disconnected devices.

To ensure that a transaction can either be completed or rolled back, NTFS performs the following steps for each transaction:

1. Records the metadata operations of a transaction in a log file cached in memory.
2. Records the actual metadata operations in memory.
3. Marks the transaction in the cached log file as committed.
4. Flushes the log file to disk.
5. Flushes the actual metadata operations to disk.

Steps 4 and 5 occur in a lazy fashion after the transaction is completed, meaning that the flush operations are not tied to the transaction itself. Instead, NTFS modifies the log and metadata quickly in memory, and then flushes later at a convenient time to boost performance.

NTFS guarantees that the log records containing the metadata operations of the transaction are written to disk before the metadata that is modified in the transaction is written to disk. After NTFS updates the cache, NTFS commits the transaction by recording in the cached log file that the transaction is complete. After the cached log file is flushed to disk, all committed transactions are guaranteed to be completed, even if the system fails before the changes are written to disk.

**Note**

- Applications can specify the FILE\_FLAG\_WRITE\_THROUGH Win32 flag to instruct the system to write through any intermediate cache and go directly to disk. The system can still cache write operations, but cannot lazily flush them.

If a system failure occurs, NTFS has enough information in the log to complete or abort any partial NTFS transaction. During recovery operations, NTFS redoes each committed transaction found in the log file. Then NTFS locates in the log file the transactions that were not committed at the time of the system failure and undoes each metadata operation recorded in the log file. Because NTFS flushes the log to disk before any metadata changes are written to disk, NTFS has complete information available about any metadata changes that need to be rolled back during recovery.

**Note**

- NTFS uses transaction logging and recovery to guarantee that the volume structure is not corrupted. For this reason, all file system data is accessible after a system failure. NTFS guarantees user data only if the program used to create the data uses the FILE\_FLAG\_WRITE\_THROUGH Win32 flag. If the program does not use this flag, user data can be lost due to a system failure. If a system failure does occur, NTFS shows either the previous data, the new data, or zeros. Users do not see random data on the volume as the result of a crash.

#### Caching and Data Recovery

The cache is the area of RAM that contains the most recently used data. When you write data to disk, the lazy-writetechnique in Windows Server 2003 indicates that the data is written when it is still in the cache. Cache memory can also be on the disk controller, such as cache memory available on SCSI controllers, or on the disk unit, such as cache memory available on Advanced Technology Attachment (ATA) disks. The following information can help you decide whether to enable the disk or the controller cache:

- Write caching improves disk performance, particularly if large amounts of data are being written to the disk.
- Control of the write-back cache is a firmware function provided by the disk manufacturer. See the documentation supplied with the disk or disk controller. You cannot configure the write-back cache from Windows Server 2003.
- Write caching does not impact the reliability of the file system’s own metadata as long as the firmware provided by the disk manufacturer honors write-through requests issued by the NTFS driver. NTFS instructs the disk device driver to ensure that metadata is written whether or not write caching is enabled. Non-metadata is typically written to disk and can be cached.
- Read caching in the disk does not affect the reliability of a file system.

#### Cluster Remapping

When NTFS detects a bad sector, NTFS dynamically remaps the cluster containing the bad sector — a recovery technique called cluster remapping — and allocates a new cluster for the data. If the error occurred during a read, NTFS returns a read error to the calling program, and the data is lost. If the error occurs during a write, NTFS writes the data to the new cluster, and no data is lost.

NTFS puts the address of the cluster containing the bad sector in the bad cluster file, $BadClus, in the MFT so that the bad sector is not reused.

#### Disk Recovery Operations

NTFS ensures the integrity of all NTFS volumes by performing disk recovery operations whenever a volume is mounted after the computer is restarted or after the volume is dismounted. NTFS also uses a technique called cluster remapping to minimize the effects of a bad sector on an NTFS volume.

NTFS views each operation that modifies a file on a volume as a transaction and manages each one as an integral unit. NTFS might also break a single complex operation into multiple transactions. After a transaction is started, it is either completed, or if an event occurs that causes the operation to fail, it is rolled back, and the NTFS volume returns to its state before the transaction began. Events that can cause an operation to fail include bad sectors, transient low-memory conditions, and disconnected devices.

NTFS guarantees that the log records containing the metadata operations of the transaction are written to disk before the metadata that is modified in the transaction is written to disk. After NTFS updates the cache, NTFS commits the transaction by recording in the cached log file that the transaction is complete. After the cached log file is flushed to disk, all committed transactions are guaranteed to be completed, even if the system crashes before the changes are written to disk.

If a system failure occurs, NTFS has enough information in the log to complete or abort any partial NTFS transaction. During recovery operations, NTFS redoes each committed transaction found in the log file. Then NTFS locates in the log file the transactions that were not committed at the time of the system failure and undoes each metadata operation recorded in the log file. Because NTFS flushes the log to disk before any metadata changes are written to disk, NTFS has complete information available about any metadata changes that need to be rolled back during recovery.

### Cleanup Operations on Windows NT–Based Volumes

Because files on volumes formatted by using the version of NTFS included with Windows Server 2003 can be read and written to by Windows NT 4.0 Service Pack 4 or later, Windows Server 2003 might need to perform cleanup operations to ensure the consistency of the data structures of a volume after it is mounted on a computer running Windows NT.

Windows Server 2003 does not perform cleanup operations on volumes previously mounted by using Windows 2000 or Windows XP.

Cleanup operations affect the following features:

##### Reparse points

Computers running Windows NT 4.0 or earlier cannot access files that have reparse points, so no cleanup operations are necessary. Reparse points are files or directories that have blocks of data called reparse data associated with them.

##### Disk quotas

If disk quotas are turned off, Windows Server 2003 performs no cleanup operations. If disk quotas are turned on, Windows Server 2003 cleans up the quota information by rebuilding the index. If a user exceeds the disk quota while the NTFS volume is mounted by a Windows NT 4.0 SP4 or later system, and disk quotas are strictly enforced, all further disk allocations of data by that user using Windows Server 2003 fail. The user can still read and write data to any existing file but cannot increase the size of a file. However, the user can delete and shrink files. When usage falls below the assigned disk quota, disk allocations of data can resume.

##### Encryption

Encrypted files cannot be accessed by computers that are running Windows NT 4.0 or earlier, so no cleanup operations are necessary.

##### Sparse files

Computers running Windows NT 4.0 or earlier cannot access sparse files, so no cleanup operations are necessary.

##### Change journal

Computers that are running Windows NT 4.0 or earlier do not log file changes in the change journal. When Windows Server 2003 starts, the change journals on volumes accessed by Windows NT are reset to indicate that the journal history is incomplete. Applications that use the change journal must be able to accept incomplete journals.

##### Object identifiers

Windows Server 2003 maintains two references to the object identifier: one on the file and one in the volume-wide object identifier index. If you delete a file that has an object identifier, Windows Server 2003 must scan and clean up the entry in the index.

### POSIX Compliance

NTFS provides a several features to support the Portable Operating System Interface (POSIX) standard, which is defined by the Institute of Electrical and Electronic Engineers (IEEE) standard 1003.1-1990 (also known as ISO/IEC 9945-1:1990).

NTFS includes the following POSIX-compliant features.

##### Case-sensitive naming

For example, POSIX interprets README.TXT, Readme.txt, and readme.txt as separate files.

##### Hard links

A file can have more than one name. This allows two different file names, which can be in different folders on the same volume, to point to the same data.

##### Additional time stamps

These show when the file was last accessed or modified.

The POSIX subsystem included with Windows NT and Windows 2000 is not included with Windows Server 2003. A new subsystem supporting the broad functionality found on most UNIX systems beyond the POSIX.1 standard is shipped as part of Interix 2.2. The Interix subsystem can be certified to the NIST FIPS 151-2 POSIX Conformance Test Suite.

**Note**

- You must use Interix-based programs to manage file names that differ only in case. You cannot use standard Windows Server 2003 command-line tools (such as **copy**, **del**, and **move**, or their equivalents in Windows Explorer or My Computer) to manage file names that differ only in case.
