> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/) offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## General Comments (Applicable to FAT File System All Types)

General Comments (Applicable to FAT File System All Types)
All of the FAT file systems were originally developed for the IBM PC machine architecture. The importance of this is that FAT file system on disk data structure is all
little endian.
 If we look at one 32-bit FAT entry stored on disk as a series of four 8-bit bytes
the first being byte[0] and the last being byte[4]
here is where the 32 bits numbered 00 through 31 are (00 being the least significant bit):

byte[3]		3 3 2 2 2 2 2 2
		1 0 9 8 7 6 5 4

byte[2]		2 2 2 2 1 1 1 1
		3 2 1 0 9 8 7 6

byte[1]		1 1 1 1 1 1 0 0
		5 4 3 2 1 0 9 8

byte[0]		0 0 0 0 0 0 0 0
7 6 5 4 3 2 1 0

This is important if your machine is a
big endian
 machine, because you will have to translate between big and little endian as you move data to and from the disk.

A FAT file system volume is composed of four basic regions, which are laid out in this order on the volume:
	0
 Reserved Region
	1
 FAT Region
	2
 Root Directory Region (doesn
t exist on FAT32 volumes)
	3
 File and Directory Data Region
