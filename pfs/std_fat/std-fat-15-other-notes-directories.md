> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/) offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## Other Notes Relating to FAT Directories

Other Notes Relating to FAT Directories
Long File Name directory entries are identical on all FAT types. See the preceeding sections for details.

DIR_FileSize is a 32-bit field. For FAT32 volumes, your FAT file system driver must not allow a cluster chain to be created that is longer than 0x100000000 bytes, and the last byte of the last cluster in a chain that long cannot be allocated to the file. This must be done so that no file has a file size > 0xFFFFFFFF bytes. This is a fundamental limit of all FAT file systems. The maximum allowed file size on a FAT volume is 0xFFFFFFFF (4,294,967,295) bytes.

Similarly, a FAT file system driver must not allow a directory (a file that is actually a container for other files) to be larger than 65,536 * 32 (2,097,152) bytes.
NOTE: This limit does not apply to the number of files in the directory. This limit is on the size of the directory itself and has nothing to do with the content of the directory. There are two reasons for this limit:

Because FAT directories are not sorted or indexed, it is a bad idea to create huge directories; otherwise, operations like creating a new entry (which requires every allocated directory entry to be checked to verify that the name doesn
t already exist in the directory) become very slow.
There are many FAT file system drivers and disk utilities, including Microsoft
s, that expect to be able to count the entries in a directory using a 16-bit WORD variable. For this reason, directories cannot have more than 16-bits worth of entries.

FAT: General Overview of On-Disk Format
Microsoft
Corporation. All rights reserved.

qmdXGXG>
|jc\cXcXcXcXcXcXcXcX
