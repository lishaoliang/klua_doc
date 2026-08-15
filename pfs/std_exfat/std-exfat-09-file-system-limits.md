> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: `portfs/doc/std_exfat/` offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 9 File System Limits

### 9.1 Sector Size Limits

The BytesPerSectorShift field defines the lower and upper sector size limits (which evaluates to **lower limit: 512 bytes; upper limit: 4,096 bytes**).

### 9.2 Cluster Size Limits

The SectorsPerClusterShift field defines the lower and upper cluster size limits (**lower limit: 1 sector; upper limit: 25 -- BytesPerSectorShift sectors**, which evaluates to 32MB).

### 9.3 Cluster Heap Size Limits

The Cluster Heap shall contain at least enough space to host the following basic file system structures: the root directory, all Allocation Bitmaps, and the Up-case Table.

The lower Cluster Heap size limit is a function of the lower size limit of each of the basic file system structures which reside in the Cluster Heap. Even given the smallest possible cluster (512 bytes), each of the basic file system structures needs no more than one cluster. Therefore, the **lower limit is: 2 + NumberOfFats clusters**, which evaluates to either 3 or 4 clusters, depending on the value the NumberOfFats field.

The upper Cluster Heap size limit is a simple function of the maximum possible number of clusters, which the ClusterCount field defines (**upper limit: 2^32^- 11 clusters**). Regardless of the cluster size, such a cluster heap has enough space to at least host the basic file system structures.

### 9.4 Volume Size Limits

The VolumeLength field defines the lower and upper volume size limits (lower limit: **2^20^/ 2^BytesPerSectorShift^sectors**, which evaluates to 1MB; **upper limit: 2^64^- 1 sectors**, which, given the largest possible sector size, evaluates to approximately 64ZB). However, this specification recommends no more than 2^24^- 2 clusters in the Cluster Heap (see Section 3.1.9). Therefore, the recommended upper limit of a volume is: ClusterHeapOffset + (2^24^- 2) \*2^SectorsPerClusterShift^. Given the largest possible cluster size, 32MB, and assuming ClusterHeapOffset is 96MB (enough space for the Main and Backup Boot regions and only the First FAT), the recommended upper limit of a volume evaluates to approximately 512TB.

### 9.5 Directory Size Limits

The DataLength field of the Stream Extension directory entry defines the lower and upper directory size limits (**lower limit: 0 bytes; upper limit: 256MB**). This means a directory may host up to 8,388,608 directory entries (each directory entry consumes 32 bytes). Given the smallest possible File directory entry set, three directory entries, a directory may host up to 2,796,202 files.
