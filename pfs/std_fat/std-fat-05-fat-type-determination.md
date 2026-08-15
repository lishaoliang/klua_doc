> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: `portfs/doc/std_fat/` offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## FAT Type Determination

FAT Type Determination
There is considerable confusion over exactly how this works, which leads to many
off by 1
off by 2
off by 10
massively off
 errors. It is really quite simple how this works. The FAT type
one of FAT12, FAT16, or FAT32
is determined by the count of clusters on the volume and nothing else.

Please read everything in this section carefully, all of the words are important. For example, note that the statement was
count of clusters.
 This is not the same thing as
maximum valid cluster number,
 because the first data cluster is 2 and not 0 or 1.

To begin, let
s discuss exactly how the
count of clusters
 value is determined. This is all done using the BPB fields for the volume. First, we determine the count of sectors occupied by the root directory as noted earlier.

RootDirSectors = ((BPB_RootEntCnt * 32) + (BPB_BytsPerSec
 1)) / BPB_BytsPerSec;

Note that on a FAT32 volume, the BPB_RootEntCnt value is always 0; so on a FAT32 volume, RootDirSectors is always 0.

Next, we determine the count of sectors in the data region of the volume:

If(BPB_FATSz16 != 0)
    FATSz = BPB_FATSz16;
Else
    FATSz = BPB_FATSz32;

If(BPB_TotSec16 != 0)
    TotSec = BPB_TotSec16;
Else
    TotSec = BPB_TotSec32;

DataSec = TotSec
 (BPB_ResvdSecCnt + (BPB_NumFATs * FATSz) + RootDirSectors);

Now we determine the count of clusters:

CountofClusters = DataSec / BPB_SecPerClus;

Please note that this computation rounds down.

Now we can determine the FAT type. Please note carefully or you will commit an off-by-one error!

In the following example, when it says <, it does not mean <=. Note also that the numbers are correct. The first number for FAT12 is 4085; the second number for FAT16 is 65525. These numbers and the
 signs are not wrong.

If(CountofClusters < 4085) {
/* Volume is FAT12 */
} else if(CountofClusters < 65525) {
    /* Volume is FAT16 */
} else {
    /* Volume is FAT32 */
}

This is the one and only way that FAT type is determined. There is no such thing as a FAT12 volume that has more than 4084 clusters. There is no such thing as a FAT16 volume that has less than 4085 clusters or more than 65,524 clusters. There is no such thing as a FAT32 volume that has less than 65,525 clusters. If you try to make a FAT volume that violates this rule, Microsoft operating systems will not handle them correctly because they will think the volume has a different type of FAT than what you think it does.

NOTE: As is noted numerous times earlier, the world is full of FAT code that is wrong. There is a lot of FAT type code that is off by 1 or 2 or 8 or 10 or 16. For this reason, it is highly recommended that if you are formatting a FAT volume which has maximum compatibility with all existing FAT code, then you should you avoid making volumes of any type that have close to 4,085 or 65,525 clusters. Stay at least 16 clusters on each side away from these cut-over cluster counts.

Note also that the CountofClusters value is exactly that
the count of data clusters starting at cluster 2. The maximum valid cluster number for the volume is CountofClusters + 1, and the
count of clusters including the two reserved clusters
 is CountofClusters + 2.

There is one more important computation related to the FAT. Given any valid cluster number N, where in the FAT(s) is the entry for that cluster number? The only FAT type for which this is complex is FAT12. For FAT16 and FAT32, the computation is simple:

If(BPB_FATSz16 != 0)
     FATSz = BPB_FATSz16;
 Else
    FATSz = BPB_FATSz32;

If(FATType == FAT16)
    FATOffset = N * 2;
Else if (FATType == FAT32)
    FATOffset = N * 4;

ThisFATSecNum = BPB_ResvdSecCnt + (FATOffset / BPB_BytsPerSec);
ThisFATEntOffset = REM(FATOffset / BPB_BytsPerSec);

REM(
) is the remainder operator. That means the remainder after division of FATOffset by BPB_BytsPerSec. ThisFATSecNum is the sector number of the FAT sector that contains the entry for cluster N in the first FAT. If you want the sector number in the second FAT, you add FATSz to ThisFATSecNum; for the third FAT, you add 2*FATSz, and so on.

You now read sector number ThisFATSecNum (remember this is a sector number relative to sector 0 of the FAT volume). Assume this is read into an 8-bit byte array named SecBuff. Also assume that the type WORD is a 16-bit unsigned and that the type DWORD is a 32-bit unsigned.

If(FATType == FAT16)
    FAT16ClusEntryVal = *((WORD *) &SecBuff[ThisFATEntOffset]);
Else
    FAT32ClusEntryVal = (*((DWORD *) &SecBuff[ThisFATEntOffset])) & 0x0FFFFFFF;

Fetches the contents of that cluster. To set the contents of this same cluster you do the following:

If(FATType == FAT16)
    *((WORD *) &SecBuff[ThisFATEntOffset]) = FAT16ClusEntryVal;
Else {
     FAT32ClusEntryVal = FAT32ClusEntryVal & 0x0FFFFFFF;
    *((DWORD *) &SecBuff[ThisFATEntOffset]) =
        (*((DWORD *) &SecBuff[ThisFATEntOffset])) & 0xF0000000;
    *((DWORD *) &SecBuff[ThisFATEntOffset]) =
        (*((DWORD *) &SecBuff[ThisFATEntOffset])) | FAT32ClusEntryVal;
}

Note how the FAT32 code above works. A FAT32 FAT entry is actually only a 28-bit entry. The high 4 bits of a FAT32 FAT entry are reserved. The only time that the high 4 bits of FAT32 FAT entries should ever be changed is when the volume is formatted, at which time the whole 32-bit FAT entry should be zeroed, including the high 4 bits.

A bit more explanation is in order here, because this point about FAT32 FAT entries seems to cause a great deal of confusion. Basically 32-bit FAT entries are not really 32-bit values; they are only 28-bit values. For example, all of these 32-bit cluster entry values: 0x10000000, 0xF0000000, and 0x00000000 all indicate that the cluster is FREE, because you ignore the high 4 bits when you read the cluster entry value. If the 32-bit free cluster value is currently 0x30000000 and you want to mark this cluster as bad by storing the value 0x0FFFFFF7 in it. Then the 32-bit entry will contain the value 0x3FFFFFF7 when you are done, because you must preserve the high 4 bits when you write in the 0x0FFFFFF7 bad cluster mark.

Take note that because the BPB_BytsPerSec value is always divisible by 2 and 4, you never have to worry about a FAT16 or FAT32 FAT entry spanning over a sector boundary (this is not true of FAT12).

The code for FAT12 is more complicated because there are 1.5 bytes (12-bits) per FAT entry.

    if (FATType == FAT12)
        FATOffset = N + (N / 2);
/* Multiply by 1.5 without using floating point, the divide by 2 rounds DOWN */

    ThisFATSecNum = BPB_ResvdSecCnt + (FATOffset / BPB_BytsPerSec);
    ThisFATEntOffset = REM(FATOffset / BPB_BytsPerSec);

We now have to check for the sector boundary case:

If(ThisFATEntOffset == (BPB_BytsPerSec
 1)) {
    /* This cluster access spans a sector boundary in the FAT      */
    /* There are a number of strategies to handling this. The      */
    /* easiest is to always load FAT sectors into memory           */
    /* in pairs if the volume is FAT12 (if you want to load        */
    /* FAT sector N, you also load FAT sector N+1 immediately      */
    /* following it in memory unless sector N is the last FAT      */
    /* sector). It is assumed that this is the strategy used here  */
    /* which makes this if test for a sector boundary span         */
    /* unnecessary.                                                */
}

We now access the FAT entry as a WORD just as we do for FAT16, but if the cluster number is EVEN, we only want the low 12-bits of the 16-bits we fetch; and if the cluster number is ODD, we only want the high 12-bits of the 16-bits we fetch.

 FAT12ClusEntryVal = *((WORD *) &SecBuff[ThisFATEntOffset]);
 If(N & 0x0001)
     FAT12ClusEntryVal = FAT12ClusEntryVal >> 4;	/* Cluster number is ODD */
 Else
     FAT12ClusEntryVal = FAT12ClusEntryVal & 0x0FFF;	/* Cluster number is EVEN */

Fetches the contents of that cluster. To set the contents of this same cluster you do the following:

If(N & 0x0001) {
    FAT12ClusEntryVal = FAT12ClusEntryVal << 4;	/* Cluster number is ODD */
    *((WORD *) &SecBuff[ThisFATEntOffset]) =
        (*((WORD *) &SecBuff[ThisFATEntOffset])) & 0x000F;
} Else {
    FAT12ClusEntryVal = FAT12ClusEntryVal & 0x0FFF;	/* Cluster number is EVEN */
    *((WORD *) &SecBuff[ThisFATEntOffset]) =
        (*((WORD *) &SecBuff[ThisFATEntOffset])) & 0xF000;
}
*((WORD *) &SecBuff[ThisFATEntOffset]) =
    (*((WORD *) &SecBuff[ThisFATEntOffset])) | FAT12ClusEntryVal;

NOTE: It is assumed that the >> operator shifts a bit value of 0 into the high 4 bits and that the << operator shifts a bit value of 0 into the low 4 bits.

The way the data of a file is associated with the file is as follows. In the directory entry, the cluster number of the first cluster of the file is recorded. The first cluster (extent) of the file is the data associated with this first cluster number, and the location of that data on the volume is computed from the cluster number as described earlier (computation of FirstSectorofCluster).

Note that a zero-length file
a file that has no data allocated to it
has a first cluster number of 0 placed in its directory entry. This cluster location in the FAT (see earlier computation of ThisFATSecNum and ThisFATEntOffset) contains either an EOC mark (End Of Clusterchain) or the cluster number of the next cluster of the file. The EOC value is FAT type dependant (assume FATContent is the contents of the cluster entry in the FAT being checked to see whether it is an EOC mark):

IsEOF = FALSE;
If(FATType == FAT12) {
    If(FATContent >= 0x0FF8)
        IsEOF = TRUE;
} else if(FATType == FAT16) {
    If(FATContent >= 0xFFF8)
        IsEOF = TRUE;
} else if (FATType == FAT32) {
    If(FATContent >= 0x0FFFFFF8)
        IsEOF = TRUE;
}

Note that the cluster number whose cluster entry in the FAT contains the EOC mark is allocated to the file and is also the last cluster allocated to the file. Microsoft operating system FAT drivers use the EOC value 0x0FFF for FAT12, 0xFFFF for FAT16, and 0x0FFFFFFF for FAT32 when they set the contents of a cluster to the EOC mark. There are various disk utilities for Microsoft operating systems that use a different value, however.

There is also a special
BAD CLUSTER
 mark. Any cluster that contains the
BAD CLUSTER
 value in its FAT entry is a cluster that should not be placed on the free list because it is prone to disk errors. The
BAD CLUSTER
 value is 0x0FF7 for FAT12, 0xFFF7 for FAT16, and 0x0FFFFFF7 for FAT32. The other relevant note here is that these bad clusters are also lost clusters
clusters that appear to be allocated because they contain a non-zero value but which are not part of any files allocation chain. Disk repair utilities must recognize lost clusters that contain this special value as bad clusters and not change the content of the cluster entry.

NOTE: It is not possible for the bad cluster mark to be an allocatable cluster number on FAT12 and FAT16 volumes, but it is feasible for 0x0FFFFFF7 to be an allocatable cluster number on FAT32 volumes. To avoid possible confusion by disk utilities, no FAT32 volume should ever be configured such that 0x0FFFFFF7 is an allocatable cluster number.

The list of free clusters in the FAT is nothing more than the list of all clusters that contain the value 0 in their FAT cluster entry. Note that this value must be fetched as described earlier as for any other FAT entry that is not free. This list of free clusters is not stored anywhere on the volume; it must be computed when the volume is mounted by scanning the FAT for entries that contain the value 0. On FAT32 volumes, the BPB_FSInfo sector may contain a valid count of free clusters on the volume. See the documentation of the FAT32 FSInfo sector.

What are the two reserved clusters at the start of the FAT for? The first reserved cluster, FAT[0], contains the BPB_Media byte value in its low 8 bits, and all other bits are set to 1. For example, if the BPB_Media value is 0xF8, for FAT12 FAT[0] = 0x0FF8, for FAT16 FAT[0] = 0xFFF8, and for FAT32 FAT[0] = 0x0FFFFFF8. The second reserved cluster, FAT[1], is set by FORMAT to the EOC mark. On FAT12 volumes, it is not used and is simply always contains an EOC mark. For FAT16 and FAT32, the file system driver may use the high two bits of the FAT[1] entry for dirty volume flags (all other bits, are always left set to 1). Note that the bit location is different for FAT16 and FAT32, because they are the high 2 bits of the entry.

For FAT16:
    ClnShutBitMask  = 0x8000;
    HrdErrBitMask   = 0x4000;

For FAT32:
    ClnShutBitMask  = 0x08000000;
    HrdErrBitMask   = 0x04000000;

Bit ClnShutBitMask
	If bit is 1, volume is
If bit is 0, volume is
. This indicates that the file system driver did not Dismount the volume properly the last time it had the volume mounted. It would be a good idea to run a Chkdsk/Scandisk disk repair utility on it, because it may be damaged.
Bit HrdErrBitMask
	If this bit is 1, no disk read/write errors were encountered.
If this bit is 0, the file system driver encountered a disk I/O error on the Volume the last time it was mounted, which is an indicator that some sectors may have gone bad on the volume. It would be a good idea to run a Chkdsk/Scandisk disk repair utility that does surface analysis on it to look for new bad sectors.

Here are two more important notes about the FAT region of a FAT volume:
The last sector of the FAT is not necessarily all part of the FAT. The FAT stops at the cluster number in the last FAT sector that corresponds to the entry for cluster number CountofClusters + 1 (see the CountofClusters computation earlier), and this entry is not necessarily at the end of the last FAT sector. FAT code should not make any assumptions about what the contents of the last FAT sector are after the CountofClusters + 1 entry. FAT format code should zero the bytes after this entry though.
The BPB_FATSz16 (BPB_FATSz32 for FAT32 volumes) value may be bigger than it needs to be. In other words, there may be totally unused FAT sectors at the end of each FAT in the FAT region of the volume. For this reason, the last sector of the FAT is always computed using the CountofClusters + 1 value, never from the BPB_FATSz16/32 value. FAT code should not make any assumptions about what the contents of these
 FAT sectors are. FAT format code should zero the contents of these extra FAT sectors though.
