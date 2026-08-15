> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: `portfs/doc/std_fat/` offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## FAT Directory Structure

FAT Directory Structure
We will first talk about short directory entries and ignore long directory entries for the moment.

A FAT directory is nothing but a
 composed of a linear list of 32-byte structures. The only special directory, which must always be present, is the root directory. For FAT12 and FAT16 media, the root directory is located in a fixed location on the disk immediately following the last FAT and is of a fixed size in sectors computed from the BPB_RootEntCnt value (see computations for RootDirSectors earlier in this document). For FAT12 and FAT16 media, the first sector of the root directory is sector number relative to the first sector of the FAT volume:

FirstRootDirSecNum = BPB_ResvdSecCnt + (BPB_NumFATs * BPB_FATSz16);

For FAT32, the root directory can be of variable size and is a cluster chain, just like any other directory is. The first cluster of the root directory on a FAT32 volume is stored in BPB_RootClus. Unlike other directories, the root directory itself on any FAT type does not have any date or time stamps, does not have a file name (other than the implied file name
), and does not contain
 files as the first two directory entries in the directory. The only other special aspect of the root directory is that it is the only directory on the FAT volume for which it is valid to have a file that has only the ATTR_VOLUME_ID attribute bit set (see below).

FAT 32 Byte Directory Entry Structure
Name
Offset (byte)
Size (bytes)
Description
DIR_Name
Short name.
DIR_Attr
File attributes:
ATTR_READ_ONLY   	0x01
ATTR_HIDDEN 	0x02
ATTR_SYSTEM 	0x04
ATTR_VOLUME_ID 	0x08
ATTR_DIRECTORY	0x10
ATTR_ARCHIVE  	0x20
ATTR_LONG_NAME 	ATTR_READ_ONLY | ATTR_HIDDEN | ATTR_SYSTEM | ATTR_VOLUME_ID
The upper two bits of the attribute byte are reserved and should always be set to 0 when a file is created and never modified or looked at after that.
DIR_NTRes
Reserved for use by Windows NT. Set value to 0 when a file is created and never modify or look at it after that.
DIR_CrtTimeTenth
Millisecond stamp at file creation time. This field actually contains a count of tenths of a second. The granularity of the seconds part of DIR_CrtTime is 2 seconds so this field is a count of tenths of a second and its valid value range is 0-199 inclusive.

DIR_CrtTime
Time file was created.
DIR_CrtDate
Date file was created.
DIR_LstAccDate
Last access date. Note that there is no last access time, only a date. This is the date of last read or write. In the case of a write, this should be set to the same date as DIR_WrtDate.

DIR_FstClusHI
High word of this entry
s first cluster number (always 0 for a FAT12 or FAT16 volume).
DIR_WrtTime
Time of last write. Note that file creation is considered a write.
DIR_WrtDate
Date of last write. Note that file creation is considered a write.
DIR_FstClusLO
Low word of this entry
s first cluster number.
DIR_FileSize
32-bit DWORD holding this file
s size in bytes.

DIR_Name[0]
Special notes about the first byte (DIR_Name[0]) of a FAT directory entry:

If DIR_Name[0] == 0xE5, then the directory entry is free (there is no file or directory name in this entry).

If DIR_Name[0] == 0x00, then the directory entry is free (same as for 0xE5), and there are no allocated directory entries after this one (all of the DIR_Name[0] bytes in all of the entries after this one are also set to 0).

The special 0 value, rather than the 0xE5 value, indicates to FAT file system driver code that the rest of the entries in this directory do not need to be examined because they are all free.

If DIR_Name[0] == 0x05, then the actual file name character for this byte is 0xE5. 0xE5 is actually a valid KANJI lead byte value for the character set used in Japan. The special 0x05 value is used so that this special file name case for Japan can be handled properly and not cause FAT file system code to think that the entry is free.

The DIR_Name field is actually broken into two parts+ the 8-character main part of the name, and the 3-character extension. These two parts are
trailing space padded
 with bytes of 0x20.

DIR_Name[0] may not equal 0x20. There is an implied
 character between the main part of the name and the extension part of the name that is not present in DIR_Name. Lower case characters are not allowed in DIR_Name (what these characters are is country specific).

The following characters are not legal in any bytes of DIR_Name:
Values less than 0x20 except for the special case of 0x05 in DIR_Name[0] described above.
0x22, 0x2A, 0x2B, 0x2C, 0x2E, 0x2F, 0x3A, 0x3B, 0x3C, 0x3D, 0x3E, 0x3F, 0x5B, 0x5C, 0x5D, and 0x7C.

Here are some examples of how a user-entered name maps into DIR_Name:

            ->
FOO     BAR
            ->
FOO     BAR
            ->
FOO     BAR
                ->
FOO
               ->
FOO
PICKLE.A
           ->
PICKLE  A
prettybg.big
       ->
PRETTYBGBIG
               -> illegal, DIR_Name[0] cannot be 0x20

In FAT directories all names are unique. Look at the first three examples earlier. Those different names all refer to the same file, and there can only be one file with DIR_Name set to
FOO     BAR
 in any directory.

DIR_Attr specifies attributes of the file:

ATTR_READ_ONLY 	Indicates that writes to the file should fail.
ATTR_HIDDEN 	Indicates that normal directory listings should not show this file.
ATTR_SYSTEM 	Indicates that this is an operating system file.
ATTR_VOLUME_ID 	There should only be one
 on the volume that has this attribute set, and that file must be in the root directory. This name of this file is actually the label for the volume. DIR_FstClusHI and DIR_FstClusLO must always be 0 for the volume label (no data clusters are allocated to the volume label file).
ATTR_DIRECTORY	Indicates that this file is actually a container for other files.
ATTR_ARCHIVE  	This attribute supports backup utilities. This bit is set by the FAT file system driver when a file is created, renamed, or written to. Backup utilities may use this attribute to indicate which files on the volume have been modified since the last time that a backup was performed.

Note that the ATTR_LONG_NAME attribute bit combination indicates that the
 is actually part of the long name entry for some other file. See the next section for more information on this attribute combination.

When a directory is created, a file with the ATTR_DIRECTORY bit set in its DIR_Attr field, you set its DIR_FileSize to 0. DIR_FileSize is not used and is always 0 on a file with the ATTR_DIRECTORY attribute (directories are sized by simply following their cluster chains to the EOC mark). One cluster is allocated to the directory (unless it is the root directory on a FAT16/FAT12 volume), and you set DIR_FstClusLO and DIR_FstClusHI to that cluster number and place an EOC mark in that clusters entry in the FAT. Next, you initialize all bytes of that cluster to 0. If the directory is the root directory, you are done (there are no dot or dotdot entries in the root directory). If the directory is not the root directory, you need to create two special entries in the first two 32-byte directory entries of the directory (the first two 32 byte entries in the data region of the cluster you just allocated).

The first directory entry has DIR_Name set to:
.

The second has DIR_Name set to:
..

These are called the dot and dotdot entries. The DIR_FileSize field on both entries is set to 0, and all of the date and time fields in both of these entries are set to the same values as they were in the directory entry for the directory that you just created. You now set DIR_FstClusLO and DIR_FstClusHI for the dot entry (the first entry) to the same values you put in those fields for the directories directory entry (the cluster number of the cluster that contains the dot and dotdot entries).

Finally, you set DIR_FstClusLO and DIR_FstClusHI for the dotdot entry (the second entry) to the first cluster number of the directory in which you just created the directory (value is 0 if this directory is the root directory even for FAT32 volumes).

Here is the summary for the dot and dotdot entries:
The dot entry is a directory that points to itself.
The dotdot entry points to the starting cluster of the parent of this directory (which is 0 if this directories parent is the root directory).

Date and Time Formats
Many FAT file systems do not support Date/Time other than DIR_WrtTime and DIR_WrtDate. For this reason, DIR_CrtTimeMil, DIR_CrtTime, DIR_CrtDate, and DIR_LstAccDate are actually optional fields. DIR_WrtTime and DIR_WrtDate must be supported, however. If the other date and time fields are not supported, they should be set to 0 on file create and ignored on other file operations.

Date Format. A FAT directory entry date stamp is a 16-bit field that is basically a date relative to the MS-DOS epoch of 01/01/1980. Here is the format (bit 0 is the LSB of the 16-bit word, bit 15 is the MSB of the 16-bit word):

Bits 0
4: Day of month, valid value range 1-31 inclusive.
Bits 5
8: Month of year, 1 = January, valid value range 1
12 inclusive.
Bits 9
15: Count of years from 1980, valid value range 0
127 inclusive (1980
2107).

Time Format. A FAT directory entry time stamp is a 16-bit field that has a granularity of 2 seconds. Here is the format (bit 0 is the LSB of the 16-bit word, bit 15 is the MSB of the 16-bit word).

Bits 0
4: 2-second count, valid value range 0
29 inclusive (0
 58 seconds).
Bits 5
10: Minutes, valid value range 0
59 inclusive.
Bits 11
15: Hours, valid value range 0
23 inclusive.

The valid time range is from Midnight 00:00:00 to 23:59:58.
