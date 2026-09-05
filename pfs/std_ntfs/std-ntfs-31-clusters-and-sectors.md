> **Source**: [How NTFS Works](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10)) (Microsoft Learn; Windows Server 2003 TechNet archive)
> **Local mirror**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25
> **Note**: Microsoft does **not** publish a single complete public NTFS on-disk specification comparable to exFAT / fatgen103.

## NTFS Physical Structure

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
