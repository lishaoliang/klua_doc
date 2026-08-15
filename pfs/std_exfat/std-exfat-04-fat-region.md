> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: `portfs/doc/std_exfat/` offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 4 File Allocation Table Region

The File Allocation Table (FAT) region may contain up to two FATs, one in the First FAT sub-region and another in the Second FAT sub-region. The NumberOfFats field describes how many FATs this region contains. The valid values for the NumberOfFats field are 1 and 2. Therefore, the First FAT sub-region always contains a FAT. If the NumberOfFats field is two, then the Second FAT sub-region also contains a FAT.

The ActiveFat field of the VolumeFlags field describes which FAT is active. Only the VolumeFlags field in the Main Boot Sector is current. Implementations shall treat the FAT which is not active as stale. Use of the inactive FAT and switching between FATs is implementation specific.

### 4.1 First and Second FAT Sub-regions

A FAT shall describe cluster chains in the Cluster Heap (see Table 11). A cluster chain is a series of clusters which provides space for recording the contents of files, directories, and other file system structures. A FAT represents a cluster chain as a singly-linked list of cluster indices. With the exception of the first two entries, every entry in a FAT represents exactly one cluster.

**Table 11 File Allocation Table Structure**

| **Field Name** | **Offset** **(byte)** | **Size** **(bytes)** | **Comments** |
| --- | --- | --- | --- |
| FatEntry[0] | 0 | 4 | This field is mandatory and Section 4.1.1 defines its contents. |
| FatEntry[1] | 4 | 4 | This field is mandatory and Section 4.1.2 defines its contents. |
| FatEntry[2] | 8 | 4 | This field is mandatory and Section 4.1.3 defines its contents. |
| ... | ... | ... | ... |
| FatEntry[ClusterCount+1] | (ClusterCount + 1) \* 4 | 4 | This field is mandatory and Section 4.1.3 defines its contents. ClusterCount + 1 can never exceed FFFFFFF6h. Note: the Main and Backup Boot Sectors both contain the ClusterCount field. |
| ExcessSpace | (ClusterCount + 2) \* 4 | (FatLength \* 2^BytesPerSectorShift^) – ((ClusterCount + 2) \* 4) | This field is mandatory and its contents, if any, are undefined. Note: the Main and Backup Boot Sectors both contain the ClusterCount, FatLength, and BytesPerSectorShift fields. |

#### 4.1.1 FatEntry[0] Field

The FatEntry[0] field shall describe the media type in the first byte (the lowest order byte) and shall contain FFh in the remaining three bytes.

The media type (the first byte) should be F8h.

#### 4.1.2 FatEntry[1] Field

The FatEntry[1] field only exists due to historical precedence and does not describe anything of interest.

The valid value for this field is FFFFFFFFh. Implementations shall initialize this field to its prescribed value and should not use this field for any purpose. Implementations should not interpret this field and shall preserve its contents across operations which modify surrounding fields.

#### 4.1.3 FatEntry[2] ... FatEntry[ClusterCount+1] Fields

Each FatEntry field in this array shall represent a cluster in the Cluster Heap. FatEntry[2] represents the first cluster in the Cluster Heap and FatEntry[ClusterCount+1] represents the last cluster in the Cluster Heap.

The valid range of values for these fields shall be:

- Between 2 and ClusterCount + 1, inclusively, which points to the next FatEntry in the given cluster chain; the given FatEntry shall not point to any FatEntry which precedes it in the given cluster chain
- Exactly FFFFFFF7h, which marks the given FatEntry's corresponding cluster as "bad"
- Exactly FFFFFFFFh, which marks the given FatEntry's corresponding cluster as the last cluster of a cluster chain; this is the only valid value for the last FatEntry of any given cluster chain
