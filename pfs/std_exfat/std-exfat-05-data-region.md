> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 5 Data Region

The Data region contains the Cluster Heap, which provides managed space for file system structures, directories, and files.

### 5.1 Cluster Heap Sub-region

The Cluster Heap's structure is very simple (see Table 12); each consecutive series of sectors describes one cluster, as the SectorsPerClusterShift field defines. Importantly, the first cluster of the Cluster Heap has index two, which directly corresponds to the index of FatEntry[2].

In an exFAT volume, an Allocation Bitmap (see Section 7.1.5) maintains the record of the allocation state of all clusters. This is a significant difference from exFAT's predecessors (FAT12, FAT16, and FAT32), in which a FAT maintained a record of the allocation state of all clusters in the Cluster Heap.

**Table 12 Cluster Heap Structure**

| **Field Name** | **Offset** **(sector)** | **Size** **(sectors)** | **Comments** |
| --- | --- | --- | --- |
| Cluster[2] | ClusterHeapOffset | 2^SectorsPerClusterShift^ | This field is mandatory and Section 5.1.1 defines its contents. Note: the Main and Backup Boot Sectors both contain the ClusterHeapOffset and SectorsPerClusterShift fields. |
| ... | ... | ... | ... |
| Cluster[ClusterCount+1] | ClusterHeapOffset + (ClusterCount – 1) \* 2^SectorsPerClusterShift^ | 2^SectorsPerClusterShift^ | This field is mandatory and Section 5.1.1 defines its contents. Note: the Main and Backup Boot Sectors both contain the ClusterCount, ClusterHeapOffset, and SectorsPerClusterShift fields. |

#### 5.1.1 Cluster[2] ... Cluster[ClusterCount+1] Fields

Each Cluster field in this array is a series of contiguous sectors, whose size is defined by the SectorsPerClusterShift field.
