> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: `portfs/doc/std_exfat/` offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 6 Directory Structure

The exFAT file system uses a directory tree approach to manage the file system structures and files which exist in the Cluster Heap. Directories have a one-to-many relationship between parent and child in the directory tree.

The directory to which the FirstClusterOfRootDirectory field refers is the root of the directory tree. All other directories descend from the root directory in a singly-linked fashion.

Each directory consists of a series of directory entries (see Table 13).

One or more directory entries combine into a directory entry set which describes something of interest, such as a file system structure, sub-directory, or file.

**Table 13 Directory Structure**

| **Field Name** | **Offset** **(byte)** | **Size** **(byte)** | **Comments** |
| --- | --- | --- | --- |
| DirectoryEntry[0] | 0 | 32 | This field is mandatory and Section 6.1 defines its contents. |
| ... | ... | ... | ... |
| DirectoryEntry[N–1] | (N – 1) \* 32 | 32 | This field is mandatory and Section 6.1 defines its contents. N, the number of DirectoryEntry fields, is the size, in bytes, of the cluster chain which contains the given directory, divided by the size of a DirectoryEntry field, 32 bytes. |

### 6.1 DirectoryEntry[0] ... DirectoryEntry[N--1]

Each DirectoryEntry field in this array derives from the Generic DirectoryEntry template (see Section 6.2).

### 6.2 Generic DirectoryEntry Template

The Generic DirectoryEntry template provides the base definition for directory entries (see Table 14). All directory entry structures derive from this template and only Microsoft-defined directory entry structures are valid (exFAT does not have provisions for manufacturer-defined directory entry structures except as defined in Section 7.8 and Section 7.9). The ability to interpret the Generic DirectoryEntry template is mandatory.

**Table 14 Generic DirectoryEntry Template**

| **Field Name** | **Offset** **(byte)** | **Size** **(byte)** | **Comments** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | This field is mandatory and Section 6.2.1 defines its contents. |
| CustomDefined | 1 | 19 | This field is mandatory and structures which derive from this template may define its contents. |
| FirstCluster | 20 | 4 | This field is mandatory and Section 6.2.2 defines its contents. |
| DataLength | 24 | 8 | This field is mandatory and Section 6.2.3 defines its contents. |

#### 6.2.1 EntryType Field

The EntryType field has three modes of usage which the value of the field defines (see list below).

- 00h, which is an end-of-directory marker and the following conditions apply:
 - All other fields in the given DirectoryEntry are actually reserved
 - All subsequent directory entries in the given directory also are end-of-directory markers
 - End-of-directory markers are only valid outside directory entry sets
 - Implementations may overwrite end-of-directory markers as necessary
- Between 01h and 7Fh inclusively, which is an unused-directory-entry marker and the following conditions apply:
 - All other fields in the given DirectoryEntry are actually undefined
 - Unused directory entries are only valid outside of directory entry sets
 - Implementations may overwrite unused directory entries as necessary
 - This range of values corresponds to the InUse field (see Section 6.2.1.4) containing the value 0
- Between 81h and FFh inclusively, which is a regular directory entry and the following conditions apply:
 - The contents of the EntryType field (see Table 15) determine the layout of the remainder of the DirectoryEntry structure
 - This range of values, and only this range of values, are valid inside a directory entry set
 - This range of values directly corresponds to the InUse field (see Section 6.2.1.4) containing the value 1

To prevent modifications to the InUse field (see Section 6.2.1.4) erroneously resulting in an end-of-directory marker, the value 80h is invalid.

**Table 15 Generic EntryType Field Structure**

| **Field Name** | **Offset** **(bit)** | **Size** **(bits)** | **Comments** |
| --- | --- | --- | --- |
| TypeCode | 0 | 5 | This field is mandatory and Section 6.2.1.1 defines its contents. |
| TypeImportance | 5 | 1 | This field is mandatory and Section Section 6.2.1.2 defines its contents. |
| TypeCategory | 6 | 1 | This field is mandatory and Section 6.2.1.3 defines its contents. |
| InUse | 7 | 1 | This field is mandatory and Section 6.2.1.4 defines its contents. |

##### 6.2.1.1 TypeCode Field

The TypeCode field partially describes the specific type of the given directory entry. This field, plus the TypeImportance and TypeCategory fields (see Section 6.2.1.2 and Section 6.2.1.3, respectively) uniquely identify the type of the given directory entry.

All possible values of this field are valid, unless the TypeImportance and TypeCategory fields both contain the value 0; in that case, the value 0 is invalid for this field.

##### 6.2.1.2 TypeImportance Field

The TypeImportance field shall describe the importance of the given directory entry.

The valid values for this field shall be:

- 0, which means the given directory entry is critical (see Section 6.3.1.2.1 and Section 6.4.1.2.1 for critical primary and critical secondary directory entries, respectively)
- 1, which means the given directory entry is benign (see Section 6.3.1.2.2 and Section 6.4.1.2.2 for benign primary and benign secondary directory entries, respectively)

##### 6.2.1.3 TypeCategory Field

The TypeCategory field shall describe the category of the given directory entry.

The valid values for this field shall be:

- 0, which means the given directory entry is primary (see Section 6.3)
- 1, which means the given directory entry is secondary (see Section 6.4)

##### 6.2.1.4 InUse Field

The InUse field shall describe whether the given directory entry in use or not.

The valid values for this field shall be:

- 0, which means the given directory entry is not in use; this means the given structure actually is an unused directory entry
- 1, which means the given directory entry is in use; this means the given structure is a regular directory entry

#### 6.2.2 FirstCluster Field

The FirstCluster field shall contain the index of the first cluster of an allocation in the Cluster Heap associated with the given directory entry.

The valid range of values for this field shall be:

- Exactly 0, which means no cluster allocation exists
- Between 2 and ClusterCount + 1, which is the range of valid cluster indices

Structures which derive from this template may redefine both the FirstCluster and DataLength fields, if a cluster allocation is not compatible with the derivative structure.

#### 6.2.3 DataLength Field

The DataLength field describes the size, in bytes, of the data the associated cluster allocation contains.

The valid range of value for this field is:

- At least 0; if the FirstCluster field contains the value 0, then this field's only valid value is 0
- At most ClusterCount \* 2^SectorsPerClusterShift^\* 2^BytesPerSectorShift^

Structures which derive from this template may redefine both the FirstCluster and DataLength fields, if a cluster allocation is not possible for the derivative structure.

### 6.3 Generic Primary DirectoryEntry Template

The first directory entry in a directory entry set shall be a primary directory entry. All subsequent directory entries, if any, in the directory entry set shall be secondary directory entries (see Section 6.4).

The ability to interpret the Generic Primary DirectoryEntry template is mandatory.

All primary directory entry structures derive from the Generic Primary DirectoryEntry template (see Table 16), which derives from the Generic DirectoryEntry template (see Section 6.2).

**Table 16 Generic Primary DirectoryEntry Template**

| **Field Name** | **Offset** **(byte)** | **Size** **(byte)** | **Comments** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | This field is mandatory and Section 6.3.1 defines its contents. |
| SecondaryCount | 1 | 1 | This field is mandatory and Section 6.3.2 defines its contents. |
| SetChecksum | 2 | 2 | This field is mandatory and Section 6.3.3 defines its contents. |
| GeneralPrimaryFlags | 4 | 2 | This field is mandatory and Section 6.3.4 defines its contents. |
| CustomDefined | 6 | 14 | This field is mandatory and structures which derive from this template define its contents. |
| FirstCluster | 20 | 4 | This field is mandatory and Section 6.3.5 defines its contents. |
| DataLength | 24 | 8 | This field is mandatory and Section 6.3.6 defines its contents. |

#### 6.3.1 EntryType Field

The EntryType field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1).

##### 6.3.1.1 TypeCode Field

The TypeCode field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1.1).

##### 6.3.1.2 TypeImportance Field

The TypeImportance field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1.2).

###### 6.3.1.2.1 Critical Primary Directory Entries

Critical primary directory entries contain information which is critical to the proper management of an exFAT volume. Only the root directory contains critical primary directory entries (File directory entries are an exception, see Section 7.4).

The definition of critical primary directory entries correlates to the major exFAT revision number. Implementations shall support all critical primary directory entries and shall only record the critical primary directory entry structures this specification defines.

###### 6.3.1.2.2 Benign Primary Directory Entries

Benign primary directory entries contain additional information which may be useful for managing an exFAT volume. Any directory may contain benign primary directory entries.

The definition of benign primary directory entries correlates to the minor exFAT revision number. Support for any benign primary directory entry this specification, or any subsequent specification, defines is optional. An unrecognized benign primary directory entry renders the entire directory entry set as unrecognized (beyond the definition of the applicable directory entry templates).

##### 6.3.1.3 TypeCategory Field

The TypeCategory field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1.3).

For this template, the valid value for this field shall be 0.

##### 6.3.1.4 InUse Field

The InUse field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1.4).

#### 6.3.2 SecondaryCount Field

The SecondaryCount field shall describe the number of secondary directory entries which immediately follow the given primary directory entry. These secondary directory entries, along with the given primary directory entry, comprise the directory entry set.

The valid range of values for this field shall be:

- At least 0, which means this primary directory entry is the only entry in the directory entry set
- At most 255, which means the next 255 directory entries and this primary directory entry comprise the directory entry set

Critical primary directory entry structures which derive from this template may redefine both the SecondaryCount and SetChecksum fields.

#### 6.3.3 SetChecksum Field

The SetChecksum field shall contain the checksum of all directory entries in the given directory entry set. However, the checksum excludes this field (see Figure 2). Implementations shall verify the contents of this field are valid prior to using any other directory entry in the given directory entry set.

Critical primary directory entry structures which derive from this template may redefine both the SecondaryCount and SetChecksum fields.

**Figure 2 EntrySetChecksum Computation**

```C
UInt16 EntrySetChecksum
(
    UCHAR * Entries,       // points to an in-memory copy of the directory entry set
    UCHAR   SecondaryCount
)
{
    UInt16 NumberOfBytes = ((UInt16)SecondaryCount + 1) * 32;
    UInt16 Checksum = 0;
    UInt16 Index;

    for (Index = 0; Index < NumberOfBytes; Index++)
    {
        if ((Index == 2) || (Index == 3))
        {
            continue;
        }
        Checksum = ((Checksum&1) ? 0x8000 : 0) + (Checksum>>1) +  (UInt16)Entries[Index];
    }
    return Checksum;
}
```

#### 6.3.4 GeneralPrimaryFlags Field

The GeneralPrimaryFlags field contains flags (see Table 17).

Critical primary directory entry structures which derive from this template may redefine this field.

**Table 17 Generic GeneralPrimaryFlags Field Structure**

| **Field Name** | **Offset** **(bit)** | **Size** **(bits)** | **Comments** |
| --- | --- | --- | --- |
| AllocationPossible | 0 | 1 | This field is mandatory and Section 6.3.4.1 defines its contents. |
| NoFatChain | 1 | 1 | This field is mandatory and Section 6.3.4.2 defines its contents. |
| CustomDefined | 2 | 14 | This field is mandatory and structures which derive from this template may define this field. |

##### 6.3.4.1 AllocationPossible Field

The AllocationPossible field shall describe whether or not an allocation in the Cluster Heap is possible for the given directory entry.

The valid values for this field shall be:

- 0, which means an associated allocation of clusters is not possible and the FirstCluster and DataLength fields are actually undefined (structures which derive from this template may redefine those fields)
- 1, which means an associated allocation of clusters is possible and the FirstCluster and DataLength fields are as defined

##### 6.3.4.2 NoFatChain Field

The NoFatChain field shall indicate whether or not the active FAT describes the given allocation's cluster chain.

The valid values for this field shall be:

- 0, which means the corresponding FAT entries for the allocation's cluster chain are valid and implementations shall interpret them; if the AllocationPossible field contains the value 0, or if the AllocationPossible field contains the value 1 and the FirstCluster field contains the value 0, then this field's only valid value is 0
- 1, which means the associated allocation is one contiguous series of clusters; the corresponding FAT entries for the clusters are invalid and implementations shall not interpret them; implementations may use the following equation to calculate the size of the associated allocation: DataLength / (2^SectorsPerClusterShift^\* 2^BytesPerSectorShift^) rounded up to the nearest integer

If critical primary directory entry structures which derive from this template redefine the GeneralPrimaryFlags field, then the corresponding FAT entries for any associated allocation's cluster chain are valid.

#### 6.3.5 FirstCluster Field

The FirstCluster field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.2).

If the NoFatChain bit is 1 then FirstCluster must point to a valid cluster in the cluster heap.

Critical primary directory entry structures which derive from this template may redefine the FirstCluster and DataLength fields. Other structures which derive from this template may redefine the FirstCluster and DataLength fields only if the AllocationPossible field contains the value 0.

#### 6.3.6 DataLength Field

The DataLength field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.3).

If the NoFatChain bit is 1 then DataLength must not be zero. If the FirstCluster field is zero, then DataLength must also be zero.

Critical primary directory entry structures which derive from this template may redefine the FirstCluster and DataLength fields. Other structures which derive from this template may redefine the FirstCluster and DataLength fields only if the AllocationPossible field contains the value 0.

### 6.4 Generic Secondary DirectoryEntry Template

The central purpose of secondary directory entries is to provide additional information about a directory entry set. The ability to interpret the Generic Secondary DirectoryEntry template is mandatory.

The definition of both critical and benign secondary directory entries correlates to the minor exFAT revision number. Support for any critical or benign secondary directory entry this specification, or subsequent specifications, defines is optional.

All secondary directory entry structures derive from the Generic Secondary DirectoryEntry template (see Table 18), which derives from the Generic DirectoryEntry template (see Section 6.2).

**Table 18 Generic Secondary DirectoryEntry Template**

| **Field Name** | **Offset** **(byte)** | **Size** **(byte)** | **Comments** |
| --- | --- | --- | --- |
| EntryType | 0 | 1 | This field is mandatory and Section Section 6.4.1 defines its contents. |
| GeneralSecondaryFlags | 1 | 1 | This field is mandatory and Section 6.4.2 defines its contents. |
| CustomDefined | 2 | 18 | This field is mandatory and structures which derive from this template define its contents. |
| FirstCluster | 20 | 4 | This field is mandatory and Section 6.4.3 defines its contents. |
| DataLength | 24 | 8 | This field is mandatory and Section 6.4.4 defines its contents. |

#### 6.4.1 EntryType Field

The EntryType field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1)

##### 6.4.1.1 TypeCode Field

The TypeCode field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1.1).

##### 6.4.1.2 TypeImportance Field

The TypeImportance field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1.2).

###### 6.4.1.2.1 Critical Secondary Directory Entries

Critical secondary directory entries contain information which is critical to the proper management of its containing directory entry set. While support for any specific critical secondary directory entry is optional, an unrecognized critical directory entry renders the entire directory entry set as unrecognized (beyond the definition of the applicable directory entry templates).

However, if a directory entry set contains at least one critical secondary directory entry which an implementation does not recognize, then the implementation shall at most interpret the templates of the directory entries in the directory entry set and not the data any allocation associated with any directory entry in the directory entry set contains (File directory entries are an exception, see Section 7.4).

###### 6.4.1.2.2 Benign Secondary Directory Entries

Benign secondary directory entries contain additional information which may be useful for managing its containing directory entry set. Support for any specific benign secondary directory entry is optional. Unrecognized benign secondary directory entries do not render the entire directory entry set as unrecognized.

Implementations may ignore any benign secondary entry it does not recognize.

##### 6.4.1.3 TypeCategory Field

The TypeCategory field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1.3).

For this template, the valid value for this field is 1.

##### 6.4.1.4 InUse Field

The InUse field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.1.4).

#### 6.4.2 GeneralSecondaryFlags Field

The GeneralSecondaryFlags field contains flags (see Table 19).

**Table 19 Generic GeneralSecondaryFlags Field Structure**

| **Field Name** | **Offset** **(bit)** | **Size** **(bits)** | **Comments** |
| --- | --- | --- | --- |
| AllocationPossible | 0 | 1 | This field is mandatory and Section 6.4.2.1 defines its contents. |
| NoFatChain | 1 | 1 | This field is mandatory and Section 6.4.2.2 defines its contents. |
| CustomDefined | 2 | 6 | This field is mandatory and structures which derive from this template may define this field. |

##### 6.4.2.1 AllocationPossible Field

The AllocationPossible field shall have the same definition as the same-named field in the Generic Primary DirectoryEntry template (see Section 6.3.4.1).

##### 6.4.2.2 NoFatChain Field

The NoFatChain field shall have the same definition as the same-named field in the Generic Primary DirectoryEntry template (see Section 6.3.4.2).

#### 6.4.3 FirstCluster Field

The FirstCluster field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.2).

If the NoFatChain bit is 1 then FirstCluster must point to a valid cluster in the cluster heap.

#### 6.4.4 DataLength Field

The DataLength field shall conform to the definition provided in the Generic DirectoryEntry template (see Section 6.2.3).

If the NoFatChain bit is 1 then DataLength must not be zero. If the FirstCluster field is zero, then DataLength must also be zero.
