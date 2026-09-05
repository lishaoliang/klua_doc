> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 2 Volume Structure

A volume is the set of all file system structures and data space necessary to store and retrieve user data. All exFAT volumes contain four regions (see Table 3).

**Table 3 Volume Structure**

| **Sub-region Name** | **Offset** **(sector)** | **Size** **(sectors)** | **Comments** |
| --- | --- | --- | --- |
| **Main Boot Region** | | | |
| Main Boot Sector | 0 | 1 | This sub-region is mandatory and Section 3.1 defines its contents. |
| Main Extended Boot Sectors | 1 | 8 | This sub-region is mandatory and Section 3.2) defines its contents. |
| Main OEM Parameters | 9 | 1 | This sub-region is mandatory and Section 3.3 defines its contents. |
| Main Reserved | 10 | 1 | This sub-region is mandatory and its contents are reserved. |
| Main Boot Checksum | 11 | 1 | This sub-region is mandatory and Section 3.4 defines its contents. |
| **Backup Boot Region** | | | |
| Backup Boot Sector | 12 | 1 | This sub-region is mandatory and Section 3.1 defines its contents. |
| Backup Extended Boot Sectors | 13 | 8 | This sub-region is mandatory and Section 3.2 defines its contents. |
| Backup OEM Parameters | 21 | 1 | This sub-region is mandatory and Section 3.3 defines its contents. |
| Backup Reserved | 22 | 1 | This sub-region is mandatory and its contents are reserved. |
| Backup Boot Checksum | 23 | 1 | This sub-region is mandatory and Section 3.4 defines its contents. |
| **FAT Region** | | | |
| FAT Alignment | 24 | FatOffset – 24 | This sub-region is mandatory and its contents, if any, are undefined. Note: the Main and Backup Boot Sectors both contain the FatOffset field. |
| First FAT | FatOffset | FatLength | This sub-region is mandatory and Section 4.1 defines its contents. Note: the Main and Backup Boot Sectors both contain the FatOffset and FatLength fields. |
| Second FAT | FatOffset + FatLength | FatLength \* (NumberOfFats – 1) | This sub-region is mandatory and Section 4.1 defines its contents, if any. Note: the Main and Backup Boot Sectors both contain the FatOffset, FatLength, and NumberOfFats fields. The NumberOfFats field may only hold values 1 and 2. |
| **Data Region** | | | |
| Cluster Heap Alignment | FatOffset + FatLength \* NumberOfFats | ClusterHeapOffset – (FatOffset + FatLength \* NumberOfFats) | This sub-region is mandatory and its contents, if any, are undefined. Note: the Main and Backup Boot Sectors both contain the FatOffset, FatLength, NumberOfFats, and ClusterHeapOffset fields. The NumberOfFats field’s valid values are 1 and 2. |
| Cluster Heap | ClusterHeapOffset | ClusterCount \* 2^SectorsPerClusterShift^ | This sub-region is mandatory and Section 5.1 defines its contents. Note: the Main and Backup Boot Sectors both contain the ClusterHeapOffset, ClusterCount, and SectorsPerClusterShift fields. |
| Excess Space | ClusterHeapOffset + ClusterCount \* 2^SectorsPerClusterShift^ | VolumeLength – (ClusterHeapOffset + ClusterCount \* 2^SectorsPerClusterShift^) | This sub-region is mandatory and its contents, if any, are undefined. Note: the Main and Backup Boot Sectors both contain the ClusterHeapOffset, ClusterCount, SectorsPerClusterShift, and VolumeLength fields. |
