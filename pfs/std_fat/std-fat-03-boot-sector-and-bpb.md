> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: `portfs/doc/std_fat/` offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## Boot Sector and BPB

Boot Sector and BPB
The first important data structure on a FAT volume is called the BPB (BIOS Parameter Block), which is located in the first sector of the volume in the Reserved Region. This sector is sometimes called the
boot sector
 or the
reserved sector
 or the
0th sector,
 but the important fact is simply that it is the first sector of the volume.

This is the first thing about the FAT file system that sometimes causes confusion. In MS-DOS version 1.x, there was not a BPB in the boot sector. In this first version of the FAT file system, there were only two different formats, the one for single-sided and the one for double-sided 360K 5.25-inch floppy disks. The determination of which type was on the disk was done by looking at the first byte of the FAT (the low 8 bits of FAT[0]).

This type of media determination was superseded in MS-DOS version 2.x by putting a BPB in the boot sector, and the old style of media determination (done by looking at the first byte of the FAT) was no longer supported. All FAT volumes must have a BPB in the boot sector.

This brings us to the second point of confusion relating to FAT volume determination: What exactly does a BPB look like? The BPB in the boot sector defined for MS-DOS 2.x only allowed for a FAT volume with strictly less than 65,536 sectors (32 MB worth of 512-byte sectors). This limitation was due to the fact that the
total sectors
 field was only a 16-bit field. This limitation was addressed by MS-DOS 3.x, where the BPB was modified to include a new 32-bit field for the total sectors value.

The next BPB change occurred with the Microsoft Windows 95 operating system, specifically OEM Service Release 2 (OSR2), where the FAT32 type was introduced. FAT16 was limited by the maximum size of the FAT and the maximum valid cluster size to no more than a 2 GB volume if the disk had 512-byte sectors. FAT32 addressed this limitation on the amount of disk space that one FAT volume could occupy so that disks larger than 2
GB only had to have one partition defined.

The FAT32 BPB exactly matches the FAT12/FAT16 BPB up to and including the BPB_TotSec32 field. They differ starting at offset 36, depending on whether the media type is FAT12/FAT16 or FAT32 (see discussion below for determining FAT type). The relevant point here is that the BPB in the boot sector of a FAT volume should always be one that has all of the new BPB fields for either the FAT12/FAT16 or FAT32 BPB type. Doing it this way ensures the maximum compatibility of the FAT volume and ensures that all FAT file system drivers will understand and support the volume properly, because it always contains all of the currently defined fields.

NOTE: In the following description, all the fields whose names start with BPB_ are part of the BPB. All the fields whose names start with BS_ are part of the boot sector and not really part of the BPB. The following shows the start of sector 0 of a FAT volume, which contains the BPB:

 Boot Sector and BPB Structure
Name
Offset (byte)
Size (bytes)
Description
BS_jmpBoot
Jump instruction to boot code. This field has two allowed forms:
jmpBoot[0] = 0xEB, jmpBoot[1] = 0x??, jmpBoot[2] = 0x90
and
jmpBoot[0] = 0xE9, jmpBoot[1] = 0x??, jmpBoot[2] = 0x??

0x?? indicates that any 8-bit value is allowed in that byte. What this forms is a three-byte Intel x86 unconditional branch (jump) instruction that jumps to the start of the operating system bootstrap code. This code typically occupies the rest of sector 0 of the volume following the BPB and possibly other sectors. Either of these forms is acceptable. JmpBoot[0] = 0xEB is the more frequently used format.
BS_OEMName
MSWIN4.1
 There are many misconceptions about this field. It is only a name string. Microsoft operating systems don
t pay any attention to this field. Some FAT drivers do. This is the reason that the indicated string,
MSWIN4.1
, is the recommended setting, because it is the setting least likely to cause compatibility problems. If you want to put something else in here, that is your option, but the result may be that some FAT drivers might not recognize the volume. Typically this is some indication of what system formatted the volume.
BPB_BytsPerSec
Count of bytes per sector. This value may take on only the following values: 512, 1024, 2048 or 4096. If maximum compatibility with old implementations is desired, only the value 512 should be used. There is a lot of FAT code in the world that is basically
hard wired
 to 512 bytes per sector and doesn
t bother to check this field to make sure it is 512. Microsoft operating systems will properly support 1024, 2048, and 4096.

Note: Do not misinterpret these statements about maximum compatibility. If the media being recorded has a physical sector size N, you must use N and this must still be less than or equal to 4096. Maximum compatibility is achieved by only using media with specific sector sizes.
BPB_SecPerClus
Number of sectors per allocation unit. This value must be a power of 2 that is greater than 0. The legal values are 1, 2, 4, 8, 16, 32, 64, and 128. Note however, that a value should never be used that results in a
bytes per cluster
 value (BPB_BytsPerSec * BPB_SecPerClus) greater than 32K (32 * 1024). There is a misconception that values greater than this are OK. Values that cause a cluster size greater than 32K bytes do not work properly; do not try to define one. Some versions of some systems allow 64K bytes per cluster value. Many application setup programs will not work correctly on such a FAT volume.
BPB_RsvdSecCnt
Number of reserved sectors in the Reserved region of the volume starting at the first sector of the volume. This field must not be 0. For FAT12 and FAT16 volumes, this value should never be anything other than 1. For FAT32 volumes, this value is typically 32. There is a lot of FAT code in the world
hard wired
 to 1 reserved sector for FAT12 and FAT16 volumes and that doesn
t bother to check this field to make sure it is 1. Microsoft operating systems will properly support any non-zero value in this field.
BPB_NumFATs
The count of FAT data structures on the volume. This field should always contain the value 2 for any FAT volume of any type. Although any value greater than or equal to 1 is perfectly valid, many software programs and a few operating systems
 FAT file system drivers may not function properly if the value is something other than 2. All Microsoft file system drivers will support a value other than 2, but it is still highly recommended that no value other than 2 be used in this field.

The reason the standard value for this field is 2 is to provide redun
dancy for the FAT data structure so that if a sector goes bad in one of the FATs, that data is not lost because it is duplicated in the other FAT. On non-disk-based media, such as FLASH memory cards, where such redundancy is a useless feature, a value of 1 may be used to save the space that a second copy of the FAT uses, but some FAT file system drivers might not recognize such a volume properly.
BPB_RootEntCnt
For FAT12 and FAT16 volumes, this field contains the count of 32-byte directory entries in the root directory. For FAT32 volumes, this field must be set to 0. For FAT12 and FAT16 volumes, this value should always specify a count that when multiplied by 32 results in an even multiple of BPB_BytsPerSec. For maximum compatibility, FAT16 volumes should use the value 512.
BPB_TotSec16
This field is the old 16-bit total count of sectors on the volume. This count includes the count of all sectors in all four regions of the volume. This field can be 0; if it is 0, then BPB_TotSec32 must be non-zero. For FAT32 volumes, this field must be 0. For FAT12 and FAT16 volumes, this field contains the sector count, and BPB_TotSec32 is 0 if the total sector count
 (is less than 0x10000).
BPB_Media
0xF8 is the standard value for
 (non-removable) media. For removable media, 0xF0 is frequently used. The legal values for this field are 0xF0, 0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD, 0xFE, and 0xFF. The only other important point is that whatever value is put in here must also be put in the low byte of the FAT[0] entry. This dates back to the old MS-DOS 1.x media determination noted earlier and is no longer usually used for anything.
BPB_FATSz16
This field is the FAT12/FAT16 16-bit count of sectors occupied by ONE FAT. On FAT32 volumes this field must be 0, and BPB_FATSz32 contains the FAT size count.
BPB_SecPerTrk
Sectors per track for interrupt 0x13. This field is only relevant for media that have a geometry (volume is broken down into tracks by multiple heads and cylinders) and are visible on interrupt 0x13. This field contains the
sectors per track
 geometry value.
BPB_NumHeads
Number of heads for interrupt 0x13. This field is relevant as discussed earlier for BPB_SecPerTrk. This field contains the one based
count of heads
. For example, on a 1.44 MB 3.5-inch floppy drive this value is 2.
BPB_HiddSec
Count of hidden sectors preceding the partition that contains this FAT volume. This field is generally only relevant for media visible on interrupt 0x13. This field should always be zero on media that are not partitioned. Exactly what value is appropriate is operating system specific.
BPB_TotSec32
This field is the new 32-bit total count of sectors on the volume. This count includes the count of all sectors in all four regions of the volume. This field can be 0; if it is 0, then BPB_TotSec16 must be non-zero. For FAT32 volumes, this field must be non-zero. For FAT12/FAT16 volumes, this field contains the sector count if BPB_TotSec16 is 0 (count is greater than or equal to 0x10000).

At this point, the BPB/boot sector for FAT12 and FAT16 differs from the BPB/boot sector for FAT32. The first table shows the structure for FAT12 and FAT16 starting at offset 36 of the boot sector.

Fat12 and Fat16 Structure Starting at Offset 36
Name
Offset (byte)
Size (bytes)
Description
BS_DrvNum
Int 0x13 drive number (e.g. 0x80). This field supports MS-DOS bootstrap and is set to the INT 0x13 drive number of the media (0x00 for floppy disks, 0x80 for hard disks).
NOTE: This field is actually operating system specific.
BS_Reserved1
Reserved (used by Windows NT). Code that formats FAT volumes should always set this byte to 0.
BS_BootSig
Extended boot signature (0x29). This is a signature byte that indicates that the following three fields in the boot sector are present.
BS_VolID
Volume serial number. This field, together with BS_VolLab, supports volume tracking on removable media. These values allow FAT file system drivers to detect that the wrong disk is inserted in a removable drive. This ID is usually generated by simply combining the current date and time into a 32-bit value.
BS_VolLab
Volume label. This field matches the 11-byte volume label recorded in the root directory.
NOTE: FAT file system drivers should make sure that they update this field when the volume label file in the root directory has its name changed or created. The setting for this field when there is no volume label is the string
NO NAME
BS_FilSysType
One of the strings
FAT12
FAT16
FAT
.  NOTE: Many people think that the string in this field has something to do with the determination of what type of FAT
FAT12, FAT16, or FAT32
that the volume has. This is not true. You will note from its name that this field is not actually part of the BPB. This string is informational only and is not used by Microsoft file system drivers to determine FAT typ,e because it is frequently not set correctly or is not present. See the FAT Type Determination section of this document. This string should be set based on the FAT type though, because some non-Microsoft FAT file system drivers do look at it.
Here is the structure for FAT32 starting at offset 36 of the boot sector.

FAT32 Structure Starting at Offset 36
Name
Offset (byte)
Size (bytes)
Description
BPB_FATSz32
This field is only defined for FAT32 media and does not exist on FAT12 and FAT16 media. This field is the FAT32 32-bit count of sectors occupied by ONE FAT. BPB_FATSz16 must be 0.
BPB_ExtFlags
This field is only defined for FAT32 media and does not exist on FAT12 and FAT16 media.
Bits 0-3	-- Zero-based number of active FAT. Only valid if mirroring is disabled.
Bits 4-6	-- Reserved.
Bit      7	-- 0 means the FAT is mirrored at runtime into all FATs.
	-- 1 means only one FAT is active; it is the one referenced in
bits 0-3.
Bits 8-15 	-- Reserved.
BPB_FSVer
This field is only defined for FAT32 media and does not exist on FAT12 and FAT16 media. High byte is major revision number. Low byte is minor revision number. This is the version number of the FAT32 volume. This supports the ability to extend the FAT32 media type in the future without worrying about old FAT32 drivers mounting the volume. This document defines the version to 0:0.  If this field is non-zero, back-level Windows versions will not mount the volume.
NOTE:  Disk utilities should respect this field and not operate on volumes with a higher major or minor version number than that for which they were designed. FAT32 file system drivers must check this field and not mount the volume if it does not contain a version number that was defined at the time the driver was written.
BPB_RootClus
This field is only defined for FAT32 media and does not exist on FAT12 and FAT16 media. This is set to the cluster number of the first cluster of the root directory, usually 2 but not required to be 2.
NOTE:  Disk utilities that change the location of the root directory should make every effort to place the first cluster of the root directory in the first non-bad cluster on the drive (i.e., in cluster 2, unless it
s marked bad). This is specified so that disk repair utilities can easily find the root directory if this field accidentally gets zeroed.
BPB_FSInfo
This field is only defined for FAT32 media and does not exist on FAT12 and FAT16 media. Sector number of FSINFO structure in the reserved area of the FAT32 volume. Usually 1.
NOTE: There will be a copy of the FSINFO structure in BackupBoot, but only the copy pointed to by this field will be kept up to date (i.e., both the primary and backup boot record will point to the same FSINFO sector).
BPB_BkBootSec
This field is only defined for FAT32 media and does not exist on FAT12 and FAT16 media. If non-zero, indicates the sector number in the reserved area of the volume of a copy of the boot record. Usually 6. No value other than 6 is recommended.
BPB_Reserved
This field is only defined for FAT32 media and does not exist on FAT12 and FAT16 media. Reserved for future expansion. Code that formats FAT32 volumes should always set all of the bytes of this field to 0.
BS_DrvNum
This field has the same definition as it does for FAT12 and FAT16 media. The only difference for FAT32 media is that the field is at a different offset in the boot sector.
BS_Reserved1
This field has the same definition as it does for FAT12 and FAT16 media. The only difference for FAT32 media is that the field is at a different offset in the boot sector.
BS_BootSig
This field has the same definition as it does for FAT12 and FAT16 media. The only difference for FAT32 media is that the field is at a different offset in the boot sector.
BS_VolID
This field has the same definition as it does for FAT12 and FAT16 media. The only difference for FAT32 media is that the field is at a different offset in the boot sector.
BS_VolLab
This field has the same definition as it does for FAT12 and FAT16 media. The only difference for FAT32 media is that the field is at a different offset in the boot sector.
BS_FilSysType
Always set to the string
FAT32
.  Please see the note for this field in the FAT12/FAT16 section earlier. This field has nothing to do with FAT type determination.

There is one other important note about Sector 0 of a FAT volume. If we consider the contents of the sector as a byte array, it must be true that sector[510] equals 0x55, and sector[511] equals 0xAA.

NOTE: Many FAT documents mistakenly say that this 0xAA55 signature occupies the
last 2 bytes of the boot sector
. This statement is correct if
 and only if
 BPB_BytsPerSec is 512. If BPB_BytsPerSec is greater than 512, the offsets of these signature bytes do not change (although it is perfectly OK for the last two bytes at the end of the boot sector to also contain this signature).

Check your assumptions about the value in the BPB_TotSec16/32 field. Assume we have a disk or partition of size in sectors DskSz. If the BPB TotSec field (either BPB_TotSec16 or BPB_TotSec32
 whichever is non-zero) is less than or equal to DskSz, there is nothing whatsoever wrong with the FAT volume. In fact, it is not at all unusual to have a BPB_TotSec16/32 value that is slightly smaller than DskSz. It is also perfectly OK for the BPB_TotSec16/32 value to be considerably smaller than DskSz.

All this means is that disk space is being wasted. It does not by itself mean that the FAT volume is damaged in some way. However, if BPB_TotSec16/32 is larger than DskSz, the volume is seriously damaged or malformed because it extends past the end of the media or overlaps data that follows it on the disk. Treating a volume for which the BPB_TotSec16/32 value is
too large
 for the media or partition as valid can lead to catastrophic data loss.
