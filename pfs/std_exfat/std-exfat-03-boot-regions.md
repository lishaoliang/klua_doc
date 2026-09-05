> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 3 Main and Backup Boot Regions

The Main Boot region provides all the necessary boot-strapping instructions, identifying information, and file system parameters to enable an implementation to perform the following:

1. Boot-strap a computer system from an exFAT volume.
2. Identify the file system on the volume as exFAT.
3. Discover the location of the exFAT file system structures.

The Backup Boot region is a backup of the Main Boot region. It aids recovery of the exFAT volume in the event of the Main Boot region being in an inconsistent state. Except under infrequent circumstances, such as updating boot-strapping instructions, implementations should not modify the contents of the Backup Boot region.

### 3.1 Main and Backup Boot Sector Sub-regions

The Main Boot Sector contains code for boot-strapping from an exFAT volume and fundamental exFAT parameters which describe the volume structure (see Table 4). BIOS, MBR, or other boot-strapping agents may inspect this sector and may load and execute any boot-strapping instructions contained therein.

The Backup Boot Sector is a backup of the Main Boot Sector and has the same structure (see Table 4). The Backup Boot Sector may aid recovery operations; however, implementations shall treat the contents of the VolumeFlags and PercentInUse fields as stale.

Prior to using the contents of either the Main or Backup Boot Sector, implementations shall verify their contents by validating their respective Boot Checksum and ensuring all their fields are within their valid value range.

While the initial format operation will initialize the contents of both the Main and Backup Boot Sectors, implementations may update these sectors (and shall also update their respective Boot Checksum) as needed. However, implementations may update either the VolumeFlags or PercentInUse fields without updating their respective Boot Checksum (the checksum specifically excludes these two fields).

**Table 4 Main and Backup Boot Sector Structure**

| **Field Name** | **Offset** **(byte)** | **Size** **(bytes)** | **Comments** |
| --- | --- | --- | --- |
| JumpBoot | 0 | 3 | This field is mandatory and Section 3.1.1 defines its contents. |
| FileSystemName | 3 | 8 | This field is mandatory and Section 3.1.2 defines its contents. |
| MustBeZero | 11 | 53 | This field is mandatory and Section 3.1.3 defines its contents. |
| PartitionOffset | 64 | 8 | This field is mandatory and Section 3.1.4 defines its contents. |
| VolumeLength | 72 | 8 | This field is mandatory and Section 3.1.5 defines its contents. |
| FatOffset | 80 | 4 | This field is mandatory and Section 3.1.6 defines its contents. |
| FatLength | 84 | 4 | This field is mandatory and Section 3.1.7 defines its contents. |
| ClusterHeapOffset | 88 | 4 | This field is mandatory and Section 3.1.8 defines its contents. |
| ClusterCount | 92 | 4 | This field is mandatory and Section 3.1.9 defines its contents. |
| FirstClusterOfRootDirectory | 96 | 4 | This field is mandatory and Section 3.1.10 defines its contents. |
| VolumeSerialNumber | 100 | 4 | This field is mandatory and Section 3.1.11 defines its contents. |
| FileSystemRevision | 104 | 2 | This field is mandatory and Section 3.1.12 defines its contents. |
| VolumeFlags | 106 | 2 | This field is mandatory and Section 3.1.13 defines its contents. |
| BytesPerSectorShift | 108 | 1 | This field is mandatory and Section 3.1.14 defines its contents. |
| SectorsPerClusterShift | 109 | 1 | This field is mandatory and Section 3.1.15 defines its contents. |
| NumberOfFats | 110 | 1 | This field is mandatory and Section 3.1.16 defines its contents. |
| DriveSelect | 111 | 1 | This field is mandatory and Section 3.1.17 defines its contents. |
| PercentInUse | 112 | 1 | This field is mandatory and Section 3.1.18 defines its contents. |
| Reserved | 113 | 7 | This field is mandatory and its contents are reserved. |
| BootCode | 120 | 390 | This field is mandatory and Section 3.1.19 defines its contents. |
| BootSignature | 510 | 2 | This field is mandatory and Section 3.1.20 defines its contents. |
| ExcessSpace | 512 | 2^BytesPerSectorShift^ – 512 | This field is mandatory and its contents, if any, are undefined. Note: the Main and Backup Boot Sectors both contain the BytesPerSectorShift field. |

#### 3.1.1 JumpBoot Field

The JumpBoot field shall contain the jump instruction for CPUs common in personal computers, which, when executed, "jumps" the CPU to execute the boot-strapping instructions in the BootCode field.

The valid value for this field is (in order of low-order byte to high-order byte) EBh 76h 90h.

#### 3.1.2 FileSystemName Field

The FileSystemName field shall contain the name of the file system on the volume.

The valid value for this field is, in ASCII characters, "EXFAT ", which includes three trailing white spaces.

#### 3.1.3 MustBeZero Field

The MustBeZero field shall directly correspond with the range of bytes the packed BIOS parameter block consumes on FAT12/16/32 volumes.

The valid value for this field is 0, which helps to prevent FAT12/16/32 implementations from mistakenly mounting an exFAT volume.

#### 3.1.4 PartitionOffset Field

The PartitionOffset field shall describe the media-relative sector offset of the partition which hosts the given exFAT volume. This field aids boot-strapping from the volume using extended INT 13h on personal computers.

All possible values for this field are valid; however, the value 0 indicates implementations shall ignore this field.

#### 3.1.5 VolumeLength Field

The VolumeLength field shall describe the size of the given exFAT volume in sectors.

The valid range of values for this field shall be:

- At least 2^20^/ 2^BytesPerSectorShift^, which ensures the smallest volume is no less than 1MB
- At most 2^64^- 1, the largest value this field can describe.

 However, if the size of the Excess Space sub-region is 0, then the largest value of this field is ClusterHeapOffset + (2^32^- 11) \*2^SectorsPerClusterShift^.

#### 3.1.6 FatOffset Field

The FatOffset field shall describe the volume-relative sector offset of the First FAT. This field enables implementations to align the First FAT to the characteristics of the underlying storage media.

The valid range of values for this field shall be:

- At least 24, which accounts for the sectors the Main Boot and Backup Boot regions consume
- At most ClusterHeapOffset - (FatLength \* NumberOfFats), which accounts for the sectors the Cluster Heap consumes

#### 3.1.7 FatLength Field

The FatLength field shall describe the length, in sectors, of each FAT table (the volume may contain up to two FATs).

The valid range of values for this field shall be:

- At least (ClusterCount + 2) \* 2^2^/ 2^BytesPerSectorShift^rounded up to the nearest integer, which ensures each FAT has sufficient space for describing all the clusters in the Cluster Heap
- At most (ClusterHeapOffset - FatOffset) / NumberOfFats rounded down to the nearest integer, which ensures the FATs exist before the Cluster Heap

This field may contain a value in excess of its lower bound (as described above) to enable the Second FAT, if present, to also be aligned to the characteristics of the underlying storage media. The contents of the space which exceeds what the FAT itself requires, if any, are undefined.

#### 3.1.8 ClusterHeapOffset Field

The ClusterHeapOffset field shall describe the volume-relative sector offset of the Cluster Heap. This field enables implementations to align the Cluster Heap to the characteristics of the underlying storage media.

The valid range of values for this field shall be:

- At least FatOffset + FatLength \* NumberOfFats, to account for the sectors all the preceding regions consume
- At most 2^32^- 1 or VolumeLength - (ClusterCount \* 2^SectorsPerClusterShift^), whichever calculation is less

#### 3.1.9 ClusterCount Field

The ClusterCount field shall describe the number of clusters the Cluster Heap contains.

The valid value for this field shall be the lesser of the following:

- (VolumeLength - ClusterHeapOffset) / 2^SectorsPerClusterShift^rounded down to the nearest integer, which is exactly the number of clusters which can fit between the beginning of the Cluster Heap and the end of the volume
- 2^32^- 11, which is the maximum number of clusters a FAT can describe

The value of the ClusterCount field determines the minimum size of a FAT. To avoid extremely large FATs, implementations can control the number of clusters in the Cluster Heap by increasing the cluster size (via the SectorsPerClusterShift field). This specification recommends no more than 2^24^- 2 clusters in the Cluster Heap. However, implementations shall be able to handle volumes with up to 2^32^- 11 clusters in the Cluster Heap.

#### 3.1.10 FirstClusterOfRootDirectory Field

The FirstClusterOfRootDirectory field shall contain the cluster index of the first cluster of the root directory. The root directory shall always be described with a cluster chain in the active FAT, as if the root directory were described with a directory entry with the GeneralPrimaryFlags field's NoFatChain flag (see section 6.3.4.2) equal to zero. The data length of the root directory shall always be determined by loading the cluster chain. Implementations should make every effort to place the first cluster of the root directory in the first non-bad cluster after the clusters the Allocation Bitmap and Up-case Table consume.

The valid range of values for this field shall be:

- At least 2, the index of the first cluster in the Cluster Heap
- At most ClusterCount + 1, the index of the last cluster in the Cluster Heap

#### 3.1.11 VolumeSerialNumber Field

The VolumeSerialNumber field shall contain a unique serial number. This assists implementations to distinguish among different exFAT volumes. Implementations should generate the serial number by combining the date and time of formatting the exFAT volume. The mechanism for combining date and time to form a serial number is implementation-specific.

All possible values for this field are valid.

#### 3.1.12 FileSystemRevision Field

The FileSystemRevision field shall describe the major and minor revision numbers of the exFAT structures on the given volume.

The high-order byte is the major revision number and the low-order byte is the minor revision number. For example, if the high-order byte contains the value 01h and if the low-order byte contains the value 05h, then the FileSystemRevision field describes the revision number 1.05. Likewise, if the high-order byte contains the value 0Ah and if the low-order byte contains the value 0Fh, then the FileSystemRevision field describes the revision number 10.15.

The valid range of values for this field shall be:

- At least 0 for the low-order byte and 1 for the high-order byte
- At most 99 for the low-order byte and 99 for the high-order byte

The revision number of exFAT this specification describes is 1.00. Implementations of this specification should mount any exFAT volume with major revision number 1 and shall not mount any exFAT volume with any other major revision number. Implementations shall honor the minor revision number and shall not perform operations or create any file system structures not described in the given minor revision number's corresponding specification.

#### 3.1.13 VolumeFlags Field

The VolumeFlags field shall contain flags which indicate the status of various file system structures on the exFAT volume (see Table 5).

Implementations shall not include this field when computing its respective Main Boot or Backup Boot region checksum. When referring to the Backup Boot Sector, implementations shall treat this field as stale.

**Table 5 VolumeFlags Field Structure**

| **Field Name** | **Offset** **(bit)** | **Size** **(bits)** | **Comments** |
| --- | --- | --- | --- |
| ActiveFat | 0 | 1 | This field is mandatory and Section 3.1.13.1 defines its contents. |
| VolumeDirty | 1 | 1 | This field is mandatory and Section 3.1.13.2 defines its contents. |
| MediaFailure | 2 | 1 | This field is mandatory and Section 3.1.13.3 defines its contents. |
| ClearToZero | 3 | 1 | This field is mandatory and Section 3.1.13.4 defines its contents. |
| Reserved | 4 | 12 | This field is mandatory and its contents are reserved. |

##### 3.1.13.1 ActiveFat Field

The ActiveFat field shall describe which FAT and Allocation Bitmap are active (and implementations shall use), as follows:

- 0, which means the First FAT and First Allocation Bitmap are active
- 1, which means the Second FAT and Second Allocation Bitmap are active and is possible only when the NumberOfFats field contains the value 2

Implementations shall consider the inactive FAT and Allocation Bitmap as stale. Only TexFAT-aware implementations shall switch the active FAT and Allocation Bitmaps (see Section 7.1).

##### 3.1.13.2 VolumeDirty Field

The VolumeDirty field shall describe whether the volume is dirty or not, as follows:

- 0, which means the volume is probably in a consistent state
- 1, which means the volume is probably in an inconsistent state

Implementations should set the value of this field to 1 upon encountering file system metadata inconsistencies which they do not resolve. If, upon mounting a volume, the value of this field is 1, only implementations which resolve file system metadata inconsistencies may clear the value of this field to 0. Such implementations shall only clear the value of this field to 0 after ensuring the file system is in a consistent state.

If, upon mounting a volume, the value of this field is 0, implementations should set this field to 1 before updating file system metadata and clear this field to 0 afterwards, similar to the recommended write ordering described in Section 8.1.

##### 3.1.13.3 MediaFailure Field

The MediaFailure field shall describe whether an implementation has discovered media failures or not, as follows:

- 0, which means the hosting media has not reported failures or any known failures are already recorded in the FAT as "bad" clusters
- 1, which means the hosting media has reported failures (i.e. has failed read or write operations)

An implementation should set this field to 1 when:

1. The hosting media fails access attempts to any region in the volume
2. The implementation has exhausted access retry algorithms, if any

If, upon mounting a volume, the value of this field is 1, implementations which scan the entire volume for media failures and record all failures as "bad" clusters in the FAT (or otherwise resolve media failures) may clear the value of this field to 0.

##### 3.1.13.4 ClearToZero Field

The ClearToZero field does not have significant meaning in this specification.

The valid values for this field are:

- 0, which does not have any particular meaning
- 1, which means implementations shall clear this field to 0 prior to modifying any file system structures, directories, or files

#### 3.1.14 BytesPerSectorShift Field

The BytesPerSectorShift field shall describe the bytes per sector expressed as log2(N), where N is the number of bytes per sector. For example, for 512 bytes per sector, the value of this field is 9.

The valid range of values for this field shall be:

- At least 9 (sector size of 512 bytes), which is the smallest sector possible for an exFAT volume
- At most 12 (sector size of 4096 bytes), which is the memory page size of CPUs common in personal computers

#### 3.1.15 SectorsPerClusterShift Field

The SectorsPerClusterShift field shall describe the sectors per cluster expressed as log2(N), where N is number of sectors per cluster. For example, for 8 sectors per cluster, the value of this field is 3.

The valid range of values for this field shall be:

- At least 0 (1 sector per cluster), which is the smallest cluster possible
- At most 25 - BytesPerSectorShift, which evaluates to a cluster size of 32MB

#### 3.1.16 NumberOfFats Field

The NumberOfFats field shall describe the number of FATs and Allocation Bitmaps the volume contains.

The valid range of values for this field shall be:

- 1, which indicates the volume only contains the First FAT and First Allocation Bitmap
- 2, which indicates the volume contains the First FAT, Second FAT, First Allocation Bitmap, and Second Allocation Bitmap; this value is only valid for TexFAT volumes

#### 3.1.17 DriveSelect Field

The DriveSelect field shall contain the extended INT 13h drive number, which aids boot-strapping from this volume using extended INT 13h on personal computers.

All possible values for this field are valid. Similar fields in previous FAT-based file systems frequently contained the value 80h.

#### 3.1.18 PercentInUse Field

The PercentInUse field shall describe the percentage of clusters in the Cluster Heap which are allocated.

The valid range of values for this field shall be:

- Between 0 and 100 inclusively, which is the percentage of allocated clusters in the Cluster Heap, rounded down to the nearest integer
- Exactly FFh, which indicates the percentage of allocated clusters in the Cluster Heap is not available

Implementations shall change the value of this field to reflect changes in the allocation of clusters in the Cluster Heap or shall change it to FFh.

Implementations shall not include this field when computing its respective Main Boot or Backup Boot region checksum. When referring to the Backup Boot Sector, implementations shall treat this field as stale.

#### 3.1.19 BootCode Field

The BootCode field shall contain boot-strapping instructions. Implementations may populate this field with the CPU instructions necessary for boot-strapping a computer system. Implementations which don't provide boot-strapping instructions shall initialize each byte in this field to F4h (the halt instruction for CPUs common in personal computers) as part of their format operation.

#### 3.1.20 BootSignature Field

The BootSignature field shall describe whether the intent of a given sector is for it to be a Boot Sector or not.

The valid value for this field is AA55h. Any other value in this field invalidates its respective Boot Sector. Implementations should verify the contents of this field prior to depending on any other field in its respective Boot Sector.

### 3.2 Main and Backup Extended Boot Sectors Sub-regions

Each sector of the Main Extended Boot Sectors has the same structure; however, each sector may hold distinct boot-strapping instructions (see Table 6). Boot-strapping agents, such as the boot-strapping instructions in the Main Boot Sector, alternate BIOS implementations, or an embedded system's firmware, may load these sectors and execute the instructions they contain.

The Backup Extended Boot Sectors is a backup of the Main Extended Boot Sectors and has the same structure (see Table 6).

Prior to executing the instructions of either the Main or Backup Extended Boot Sectors, implementations should verify their contents by ensuring each sector's ExtendedBootSignature field contains its prescribed value.

While the initial format operation will initialize the contents of both the Main and Backup Extended Boot Sectors, implementations may update these sectors (and shall also update their respective Boot Checksum) as needed.

**Table 6 Extended Boot Sector Structure**

| **Field Name** | **Offset** **(byte)** | **Size** **(bytes)** | **Comments** |
| --- | --- | --- | --- |
| ExtendedBootCode | 0 | 2^BytesPerSectorShift^ – 4 | This field is mandatory and Section 3.2.1 defines its contents. Note: the Main and Backup Boot Sectors both contain the BytesPerSectorShift field. |
| ExtendedBootSignature | 2^BytesPerSectorShift^ – 4 | 4 | This field is mandatory and Section 3.2.2 defines its contents. Note: the Main and Backup Boot Sectors both contain the BytesPerSectorShift field. |

#### 3.2.1 ExtendedBootCode Field

The ExtendedBootCode field shall contain boot-strapping instructions. Implementations may populate this field with the CPU instructions necessary for boot-strapping a computer system. Implementations which don't provide boot-strapping instructions shall initialize each byte in this field to 00h as part of their format operation.

#### 3.2.2 ExtendedBootSignature Field

The ExtendedBootSignature field shall describe whether the intent of given sector is for it to be an Extended Boot Sector or not.

The valid value for this field is AA550000h. Any other value in this field invalidates its respective Main or Backup Extended Boot Sector. Implementations should verify the contents of this field prior to depending on any other field in its respective Extended Boot Sector.

### 3.3 Main and Backup OEM Parameters Sub-regions

The Main OEM Parameters sub-region contains ten parameters structures which may contain manufacturer-specific information (see Table 7). Each of the ten parameters structures derives from the Generic Parameters template (see Section 3.3.2). Manufacturers may derive their own custom parameters structures from the Generic Parameters template. This specification itself defines two parameters structures: Null Parameters (see Section 3.3.3) and Flash Parameters (see Section 3.3.4).

The Backup OEM Parameters is a backup of the Main OEM Parameters and has the same structure (see Table 7).

Prior to using the contents of either the Main or Backup OEM Parameters, implementations shall verify their contents by validating their respective Boot Checksum.

Manufacturers should populate the Main and Backup OEM Parameters with their own custom parameters structures, if any, and any other parameter structures. Subsequent format operations shall preserve the contents of the Main and Backup OEM Parameters.

Implementations may update the Main and Backup OEM Parameters as needed (and shall also update their respective Boot Checksum).

**Table 7 OEM Parameters Structure**

| **Field Name** | **Offset** **(byte)** | **Size** **(bytes)** | **Comments** |
| --- | --- | --- | --- |
| Parameters[0] | 0 | 48 | This field is mandatory and Section 3.3.1 defines its contents. |
| ... | ... | ... | ... |
| Parameters[9] | 432 | 48 | This field is mandatory and Section 3.3.1 defines its contents. |
| Reserved | 480 | 2^BytesPerSectorShift^ – 480 | This field is mandatory and its contents are reserved. Note: the Main and Backup Boot Sectors both contain the BytesPerSectorShift field. |

#### 3.3.1 Parameters[0] ... Parameters[9]

Each Parameters field in this array contains a parameters structure, which derives from the Generic Parameters template (see Section 3.3.2). Any unused Parameters field shall be described as containing a Null Parameters structure (see Section 3.3.3).

#### 3.3.2 Generic Parameters Template

The Generic Parameters template provides the base definition of a parameters structure (see Table 8). All parameters structures derive from this template. Support for this Generic Parameters template is mandatory.

**Table 8 Generic Parameters Template**

| **Field Name** | **Offset** **(byte)** | **Size** **(bytes)** | **Comments** |
| --- | --- | --- | --- |
| ParametersGuid | 0 | 16 | This field is mandatory and Section 3.3.2.1 defines its contents. |
| CustomDefined | 16 | 32 | This field is mandatory and the structures which derive from this template define its contents. |

##### 3.3.2.1 ParametersGuid Field

The ParametersGuid field shall describe a GUID, which determines the layout of the remainder of the given parameters structure.

All possible values for this field are valid; however, manufacturers should use a GUID-generating tool, such as GuidGen.exe, to select a GUID when deriving custom parameters structures from this template.

#### 3.3.3 Null Parameters

The Null Parameters structure derives from the Generic Parameters template (see Section 3.3.2) and shall describe an unused Parameters field (see Table 9). When creating or updating the OEM Parameters structure, implementations shall populate unused Parameters fields with the Null Parameters structure. Also, when creating or updating the OEM Parameters structure, implementations should consolidate Null Parameters structures at the end of the array, thereby leaving all other Parameters structures at the beginning of the OEM Parameters structure.

Support for the Null Parameters structure is mandatory.

**Table 9 Null Parameters Structure**

| **Field Name** | **Offset** **(byte)** | **Size** **(bytes)** | **Comments** |
| --- | --- | --- | --- |
| ParametersGuid | 0 | 16 | This field is mandatory and Section 3.3.3.1 defines its contents. |
| Reserved | 16 | 32 | This field is mandatory and its contents are reserved. |

##### 3.3.3.1 ParametersGuid Field

The ParametersGuid field shall conform to the definition provided by the Generic Parameters template (see Section 3.3.2.1).

The valid value for this field, in GUID notation, is {00000000-0000-0000-0000-000000000000}.

#### 3.3.4 Flash Parameters

The Flash Parameter structure derives from the Generic Parameters template (see Section 3.3.2) and contains parameters for flash media (see Table 10). Manufacturers of flash-based storage devices may populate a Parameters field (preferably the Parameters[0] field) with this parameters structure. Implementations may use the information in the Flash Parameters structure to optimize access operations during reads/writes and for alignment of file system structures during formatting of the media.

Support for the Flash Parameters structure is optional.

**Table 10 Flash Parameters Structure**

| **Field Name** | **Offset** **(byte)** | **Size** **(bytes)** | **Comments** |
| --- | --- | --- | --- |
| ParametersGuid | 0 | 16 | This field is mandatory and Section 3.3.4.1 defines its contents. |
| EraseBlockSize | 16 | 4 | This field is mandatory and Section 3.3.4.2 defines its contents. |
| PageSize | 20 | 4 | This field is mandatory and Section 3.3.4.3 defines its contents. |
| SpareSectors | 24 | 4 | This field is mandatory and Section 3.3.4.4 defines its contents. |
| RandomAccessTime | 28 | 4 | This field is mandatory and Section 3.3.4.5 defines its contents. |
| ProgrammingTime | 32 | 4 | This field is mandatory and Section 3.3.4.6 defines its contents. |
| ReadCycle | 36 | 4 | This field is mandatory and Section 3.3.4.7 defines its contents. |
| WriteCycle | 40 | 4 | This field is mandatory and Section 3.3.4.8 defines its contents. |
| Reserved | 44 | 4 | This field is mandatory and its contents are reserved. |

All possible values for all Flash Parameters fields, except for the ParametersGuid field, are valid. However, the value 0 indicates the field is actually meaningless (implementations shall ignore the given field).

##### 3.3.4.1 ParametersGuid Field

The ParametersGuid field shall conform to the definition provided in the Generic Parameters template (see Section 3.3.2.1).

The valid value for this field, in GUID notation, is {0A0C7E46-3399-4021-90C8-FA6D389C4BA2}.

##### 3.3.4.2 EraseBlockSize Field

The EraseBlockSize field shall describe the size, in bytes, of the flash media's erase block.

##### 3.3.4.3 PageSize Field

The PageSize field shall describe the size, in bytes of the flash media's page.

##### 3.3.4.4 SpareSectors Field

The SpareSectors field shall describe the number of sectors the flash media has available for its internal sparing operations.

##### 3.3.4.5 RandomAccessTime Field

The RandomAccessTime field shall describe the flash media's average random access time, in nanoseconds.

##### 3.3.4.6 ProgrammingTime Field

The ProgrammingTime field shall describe the flash media's average programming time, in nanoseconds.

##### 3.3.4.7 ReadCycle Field

The ReadCycle field shall describe the flash media's average read cycle time, in nanoseconds.

##### 3.3.4.8 WriteCycle Field

The WriteCycle field shall describe the average write cycle time, in nanoseconds.

### 3.4 Main and Backup Boot Checksum Sub-regions

The Main and Backup Boot Checksums each contain a repeating pattern of the four-byte checksum of the contents of all other sub-regions in their respective Boot regions. The checksum calculation shall not include the VolumeFlags and PercentInUse fields in their respective Boot Sector (see Figure 1). The repeating pattern of the four-byte checksum fills its respective Boot Checksum sub-region from the beginning to the end of the sub-region.

Prior to using the contents of any of the other sub-regions in either the Main or Backup Boot regions, implementations shall verify their contents by validating their respective Boot Checksum.

While the initial format operation will populate both the Main and Backup Boot Checksums with the repeating checksum pattern, implementations shall update these sectors as the contents of the other sectors in their respective Boot regions change.

**Figure 1 Boot Checksum Computation**

```c
UInt32 BootChecksum
(
    UCHAR  * Sectors,        // points to an in-memory copy of the 11 sectors
    USHORT   BytesPerSector
)
{
    UInt32 NumberOfBytes = (UInt32)BytesPerSector * 11;
    UInt32 Checksum = 0;
    UInt32 Index;

    for (Index = 0; Index < NumberOfBytes; Index++)
    {
        if ((Index == 106) || (Index == 107) || (Index == 112))
        {
            continue;
        }
        Checksum = ((Checksum&1) ? 0x80000000 : 0) + (Checksum>>1) + (UInt32)Sectors[Index];
    }

    return Checksum;
}
```
