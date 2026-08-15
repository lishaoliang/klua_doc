> **Source**: [Master File Table (Developer Notes)](https://learn.microsoft.com/en-us/windows/win32/devnotes/master-file-table) (Microsoft Learn)
> **Also**: https://docs.microsoft.com/en-us/windows/win32/devnotes/master-file-table
> **Local mirror**: `portfs/doc/std_ntfs/` offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25

# Master File Table (Developer Notes)

[This document applies only to version 3 of NTFS volumes.]

The master file table (MFT) stores the information required to retrieve files from an NTFS partition.

A file may have one or more MFT records, and can contain one or more attributes. In NTFS, a file reference is the MFT segment reference of the base file record. For more information, see [MFT_SEGMENT_REFERENCE](https://learn.microsoft.com/en-us/windows/win32/devnotes/mft-segment-reference).

The MFT contains file record segments; the first 16 of these are reserved for special files, such as the following:

- 0: MFT ($Mft)
- 5: root directory (\)
- 6: volume cluster allocation file ($Bitmap)
- 8: bad-cluster file ($BadClus)

Each file record segment starts with a file record segment header. For more information, see [FILE_RECORD_SEGMENT_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/file-record-segment-header). Each file record segment is followed by one or more attributes. Each attribute starts with an attribute record header. For more information, see [ATTRIBUTE_RECORD_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-record-header). The attribute record includes the attribute type (such as $DATA or $BITMAP), an optional name, and the attribute value. The user data stream is an attribute, as are all streams. The attribute list is terminated with 0xFFFFFFFF ($END).

The following are some example attributes.

- The $Mft file contains an unnamed $DATA attribute that is the sequence of MFT record segments, in order.
- The $Mft file contains an unnamed $BITMAP attribute that indicates which MFT records are in use.
- The $Bitmap file contains an unnamed $DATA attribute that indicates which clusters are in use.
- The $BadClus file contains a $DATA attribute named $BAD that contains an entry that corresponds to each bad cluster.

When there is no more space for storing attributes in the file record segment, additional file record segments are allocated and inserted in the first (or base) file record segment in an attribute called the attribute list. The attribute list indicates where each attribute associated with the file can be found. This includes all attributes in the base file record, except for the attribute list itself. For more information, see [ATTRIBUTE_LIST_ENTRY](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-list-entry).

Structures related to the MFT include the following:

- [ATTRIBUTE_LIST_ENTRY](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-list-entry)
- [ATTRIBUTE_RECORD_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/attribute-record-header)
- [FILE_NAME](https://learn.microsoft.com/en-us/windows/win32/devnotes/file-name)
- [FILE_RECORD_SEGMENT_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/file-record-segment-header)
- [MFT_SEGMENT_REFERENCE](https://learn.microsoft.com/en-us/windows/win32/devnotes/mft-segment-reference)
- [MULTI_SECTOR_HEADER](https://learn.microsoft.com/en-us/windows/win32/devnotes/multi-sector-header)
- [STANDARD_INFORMATION](https://learn.microsoft.com/en-us/windows/win32/devnotes/standard-information)
