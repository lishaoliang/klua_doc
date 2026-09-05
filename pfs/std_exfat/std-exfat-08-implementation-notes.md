> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 8 Implementation Notes

### 8.1 Recommended Write Ordering

Implementations should ensure the volume is as resilient as possible to power faults and other unavoidable failures. When creating new directory entries or modifying cluster allocations, implementations should generally follow this writing order:

1. Set the value of the VolumeDirty field to 1
2. Update the active FAT, if necessary
3. Update the active Allocation Bitmap
4. Create or update the directory entry, if necessary
5. Clear the value of the VolumeDirty field to 0, if its value prior to the first step was 0

When deleting directory entries or freeing cluster allocations, implementations should follow this writing order:

1. Set the value of the VolumeDirty field to 1
2. Delete or update the directory entry, if necessary
3. Update the active FAT, if necessary
4. Update the active Allocation Bitmap
5. Clear the value of the VolumeDirty field to 0, if its value prior to the first step was 0

### 8.2 Implications of Unrecognized Directory Entries

Future exFAT specifications of the same major revision number, 1, and minor revision number higher than 0, may define new benign primary, critical secondary, and benign secondary directory entries. Only exFAT specifications of a higher major revision number may define new critical primary directory entries. Implementations of this specification, exFAT Revision 1.00 File System Basic Specification, should be able to mount and access any exFAT volume of major revision number 1 and any minor revision number. This presents scenarios in which an implementation may encounter directory entries which it does not recognize. The following describe implications of these scenarios:

1. The presence of unrecognized critical primary directory entries in the root directory renders the volume invalid. The presence of any critical primary directory entry, except File directory entries, in any non-root directory, renders the hosting directory invalid.
2. Implementations shall not modify unrecognized benign primary directory entries or their associated cluster allocations. However, when deleting a directory, and only when deleting a directory, implementations shall delete unrecognized benign primary directory entries and free all associated cluster allocations, if any.
3. Implementations shall not modify unrecognized critical secondary directory entries or their associated cluster allocations. The presence of one or more unrecognized critical secondary directory entries in a directory entry set renders the entire directory entry set unrecognized. When deleting a directory entry set which contains one or more unrecognized critical secondary directory entries, implementations shall free all cluster allocations, if any, associated with unrecognized critical secondary directory entries. Further, if the directory entry set describes a directory, implementations may:

 - Traverse into the directory
 - Enumerate the directory entries it contains
 - Delete contained directory entries
 - Move contained directory entries to a different directory

 However, implementations shall not:

 - Modify contained directory entries, except delete, as noted
 - Create new contained directory entries
 - Open contained directory entries, except traverse and enumerate, as noted
4. Implementations shall not modify unrecognized benign secondary directory entries or their associated cluster allocations. Implementations should ignore unrecognized benign secondary directory entries. When deleting a directory entry set, implementations shall free all cluster allocations, if any, associated with unrecognized benign secondary directory entries.
