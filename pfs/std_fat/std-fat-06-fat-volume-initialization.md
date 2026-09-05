> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/) offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## FAT Volume Initialization

FAT Volume Initialization
At this point, the careful reader should have one very interesting question. Given that the FAT type (FAT12, FAT16, or FAT32) is dependant on the number of clusters
and that the sectors available in the data area of a FAT volume is dependant on the size of the FAT
when handed an unformatted volume that does not yet have a BPB, how do you determine all this and compute the proper values to put in BPB_SecPerClus and either BPB_FATSz16 or BPB_FATSz32? The way Microsoft operating systems do this is with a fixed value, several tables, and a clever piece of arithmetic.

Microsoft operating systems only do FAT12 on floppy disks. Because there is a limited number of floppy formats that all have a fixed size, this is done with a simple table:

If it is a floppy of this type, then the BPB looks like this.

There is no dynamic computation for FAT12. For the FAT12 formats, all the computation for BPB_SecPerClus and BPB_FATSz16 was worked out by hand on a piece of paper and recorded in the table (being careful of course that the resultant cluster count was always less than 4085). If your media is larger than 4 MB, do not bother with FAT12. Use smaller BPB_SecPerClus values so that the volume will be FAT16.

The rest of this section is totally specific to drives that have 512 bytes per sector. You cannot use these tables, or the clever arithmetic, with drives that have a different sector size. The
fixed value
 is simply a volume size that is the
FAT16 to FAT32 cutover value
. Any volume size smaller than this is FAT16 and any volume of this size or larger is FAT32. For Windows, this value is 512 MB. Any FAT volume smaller than 512 MB is FAT16, and any FAT volume of 512 MB or larger is FAT32.

Please don
t draw an incorrect conclusion here.

There are many FAT16 volumes out there that are larger than 512 MB. There are various ways to force the format to be FAT16 rather than the default of FAT32, and there is a great deal of code that implements different limits. All we are talking about here is the default cutover value for MS-DOS and Windows on volumes that have not yet been formatted. There are two tables
one is for FAT16 and the other is for FAT32. An entry in these tables is selected based on the size of the volume in 512 byte sectors (the value that will go in BPB_TotSec16 or BPB_TotSec32), and the value that this table sets is the BPB_SecPerClus value.

    struct DSKSZTOSECPERCLUS {
	DWORD	DiskSize;
	BYTE	SecPerClusVal;
    };

 /*
*This is the table for FAT16 drives. NOTE that this table includes
* entries for disk sizes larger than 512 MB even though typically
* only the entries for disks < 512 MB in size are used.
* The way this table is accessed is to look for the first entry
* in the table for which the disk size is less than or equal
* to the DiskSize field in that table entry.  For this table to
* work properly BPB_RsvdSecCnt must be 1, BPB_NumFATs
* must be 2, and BPB_RootEntCnt must be 512. Any of these values
* being different may require the first table entries DiskSize value
* to be changed otherwise the cluster count may be to low for FAT16.
   */
    DSKSZTOSECPERCLUS DskTableFAT16 [] = {
        {        8400,   0}, /* disks up to  4.1 MB, the 0 value for SecPerClusVal trips an error */
        {      32680,   2},  /* disks up to   16 MB,  1k cluster */
        {    262144,   4},   /* disks up to 128 MB,  2k cluster */
        {   524288,    8},   /* disks up to 256 MB,  4k cluster */
        { 1048576,  16},     /* disks up to 512 MB,  8k cluster */
        /* The entries after this point are not used unless FAT16 is forced */
        { 2097152,  32},     /* disks up to     1 GB, 16k cluster */
        { 4194304,  64},     /* disks up to     2 GB, 32k cluster */
        { 0xFFFFFFFF, 0} /* any disk greater than 2GB, 0 value for SecPerClusVal trips an error */
    };

/*
* This is the table for FAT32 drives. NOTE that this table includes
* entries for disk sizes smaller than 512 MB even though typically
* only the entries for disks >= 512 MB in size are used.
* The way this table is accessed is to look for the first entry
* in the table for which the disk size is less than or equal
* to the DiskSize field in that table entry. For this table to
* work properly BPB_RsvdSecCnt must be 32, and BPB_NumFATs
* must be 2. Any of these values being different may require the first
* table entries DiskSize value to be changed otherwise the cluster count
* may be to low for FAT32.
*/
    DSKSZTOSECPERCLUS DskTableFAT32 [] = {
        {       66600,   0},  /* disks up to 32.5 MB, the 0 value for SecPerClusVal trips an error */
        {     532480,   1},   /* disks up to 260 MB,  .5k cluster */
        { 16777216,   8},     /* disks up to     8 GB,    4k cluster */
        { 33554432, 16},      /* disks up to   16 GB,    8k cluster */
        { 67108864, 32},      /* disks up to   32 GB,  16k cluster */
        { 0xFFFFFFFF, 64}/* disks greater than 32GB, 32k cluster */
    };

So given a disk size and a FAT type of FAT16 or FAT32, we now have a BPB_SecPerClus value. The only thing we have left is do is to compute how many sectors the FAT takes up so that we can set BPB_FATSz16 or BPB_FATSz32. Note that at this point we assume that BPB_RootEntCnt, BPB_RsvdSecCnt, and BPB_NumFATs are appropriately set. We also assume that DskSize is the size of the volume that we are either going to put in BPB_TotSec32 or BPB_TotSec16.

RootDirSectors = ((BPB_RootEntCnt * 32) + (BPB_BytsPerSec
 1)) / BPB_BytsPerSec;
TmpVal1 = DskSize
 (BPB_ResvdSecCnt + RootDirSectors);
TmpVal2 = (256 * BPB_SecPerClus) + BPB_NumFATs;
If(FATType == FAT32)
    TmpVal2 = TmpVal2 / 2;
FATSz = (TMPVal1 + (TmpVal2
 1)) / TmpVal2;
If(FATType == FAT32) {
    BPB_FATSz16 = 0;
    BPB_FATSz32 = FATSz;
} else {
    BPB_FATSz16 = LOWORD(FATSz);
    /* there is no BPB_FATSz32 in a FAT16 BPB */
}

Do not spend too much time trying to figure out why this math works. The basis for the computation is complicated; the important point is that this is how Microsoft operating systems do it, and it works. Note, however, that this math does not work perfectly. It will occasionally set a FATSz that is up to 2
sectors too large for FAT16, and occasionally up to 8 sectors too large for FAT32. It will never compute a FATSz value that is too small, however. Because it is OK to have a FATSz that is too large, at the expense of wasting a few sectors, the fact that this computation is surprisingly simple more than makes up for it being off in a safe way in some cases.
