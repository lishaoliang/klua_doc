> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/) offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## FAT32 FSInfo Sector Structure and Backup Boot Sector

FAT32 FSInfo Sector Structure and Backup Boot Sector
On a FAT32 volume, the FAT can be a large data structure, unlike on FAT16 where it is limited to a maximum of 128K worth of sectors and FAT12 where it is limited to a maximum of 6K worth of sectors. For this reason, a provision is made to store the
last known
 free cluster count on the FAT32 volume so that it does not have to be computed as soon as an API call is made to ask how much free space there is on the volume (like at the end of a directory listing). The FSInfo sector number is the value in the BPB_FSInfo field; for Microsoft operating systems it is always set to 1. Here is the structure of the FSInfo sector:

FAT32 FSInfo Sector Structure and Backup Boot Sector
Name
Offset (byte)
Size (bytes)
Description
FSI_LeadSig
Value 0x41615252. This lead signature is used to validate that this is in fact an FSInfo sector.
FSI_Reserved1
This field is currently reserved for future expansion. FAT32 format code should always initialize all bytes of this field to 0. Bytes in this field must currently never be used.
FSI_StrucSig
Value 0x61417272. Another signature that is more localized in the sector to the location of the fields that are used.
FSI_Free_Count
Contains the last known free cluster count on the volume. If the value is 0xFFFFFFFF, then the free count is unknown and must be computed. Any other value can be used, but is not necessarily correct. It should be range checked at least to make sure it is <= volume cluster count.
FSI_Nxt_Free
This is a hint for the FAT driver. It indicates the cluster number at which the driver should start looking for free clusters. Because a FAT32 FAT is large, it can be rather time consuming if there are a lot of allocated clusters at the start of the FAT and the driver starts looking for a free cluster starting at cluster 2. Typically this value is set to the last cluster number that the driver allocated. If the value is 0xFFFFFFFF, then there is no hint and the driver should start looking at cluster 2. Any other value can be used, but should be checked first to make sure it is a valid cluster number for the volume.
FSI_Reserved2
This field is currently reserved for future expansion. FAT32 format code should always initialize all bytes of this field to 0. Bytes in this field must currently never be used.
FSI_TrailSig
Value 0xAA550000. This trail signature is used to validate that this is in fact an FSInfo sector. Note that the high 2 bytes of this value
which go into the bytes at offsets 510 and 511
match the signature bytes used at the same offsets in sector 0.

Another feature on FAT32 volumes that is not present on FAT16/FAT12 is the BPB_BkBootSec field. FAT16/FAT12 volumes can be totally lost if the contents of sector 0 of the volume are overwritten or sector 0 goes bad and cannot be read. This is a
single point of failure
 for FAT16 and FAT12 volumes. The BPB_BkBootSec field reduces the severity of this problem for FAT32 volumes, because starting at that sector number on the volume
there is a backup copy of the boot sector information including the volume
s BPB.

In the case where the sector 0 information has been accidentally overwritten, all a disk repair utility has to do is restore the boot sector(s) from the backup copy. In the case where sector 0 goes bad, this allows the volume to be mounted so that the user can access data before replacing the disk.

This second case
sector 0 goes bad
is the reason why no value other than 6 should ever be placed in the BPB_BkBootSec field. If sector 0 is unreadable, various operating systems are
hard wired
 to check for backup boot sector(s) starting at sector 6 of the FAT32 volume. Note that starting at the BPB_BkBootSec sector is a complete boot record. The Microsoft FAT32
boot sector
 is actually three 512-byte sectors long. There is a copy of all three of these sectors starting at the BPB_BkBootSec sector. A copy of the FSInfo sector is also there, even though the BPB_FSInfo field in this backup boot sector is set to the same value as is stored in the sector 0 BPB.

NOTE: All 3 of these sectors have the 0xAA55 signature in sector offsets 510 and 511, just like the first boot sector does (see the earlier discussion at the end of the BPB structure description).
