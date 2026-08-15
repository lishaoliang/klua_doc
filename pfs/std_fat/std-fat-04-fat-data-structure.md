> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: `portfs/doc/std_fat/` offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## FAT Data Structure

FAT Data Structure
The next data structure that is important is the FAT itself. What this data structure does is define a singly linked list of the
 (clusters) of a file. Note at this point that a FAT directory or file container is nothing but a regular file that has a special attribute indicating it is a directory. The only other special thing about a directory is that the data or contents of the
 is a series of 32=byte FAT directory entries (see discussion below). In all other respects, a directory is just like a file. The FAT maps the data region of the volume by cluster number. The first data cluster is cluster 2.

The first sector of cluster 2 (the data region of the disk) is computed using the BPB fields for the volume as follows. First, we determine the count of sectors occupied by the root directory:

RootDirSectors = ((BPB_RootEntCnt * 32) + (BPB_BytsPerSec
 1)) / BPB_BytsPerSec;

Note that on a FAT32 volume the BPB_RootEntCnt value is always 0, so on a FAT32 volume RootDirSectors is always 0. The 32 in the above is the size of one FAT directory entry in bytes. Note
also that this computation rounds up.

The start of the data region, the first sector of cluster 2, is computed as follows:

If(BPB_FATSz16 != 0)
    FATSz = BPB_FATSz16;
Else
    FATSz = BPB_FATSz32;

FirstDataSector = BPB_ResvdSecCnt + (BPB_NumFATs * FATSz) + RootDirSectors;

NOTE: This sector number is relative to the first sector of the volume that contains the BPB (the sector that contains the BPB is sector number 0). This does not necessarily map directly onto the drive, because sector 0 of the volume is not necessarily sector 0 of the drive due to partitioning.

Given any valid data cluster number N, the sector number of the first sector of that cluster (again relative to sector 0 of the FAT volume) is computed as follows:

FirstSectorofCluster = ((N
 2) * BPB_SecPerClus) + FirstDataSector;

NOTE: Because BPB_SecPerClus is restricted to powers of 2 (1,2,4,8,16,32
.), this means that division and multiplication by BPB_SecPerClus can actually be performed via SHIFT operations on 2s complement architectures that are usually faster instructions than MULT and DIV instructions. On current Intel X86 processors, this is largely irrelevant though because the MULT and DIV machine instructions are heavily optimized for multiplication and division by powers of 2.
