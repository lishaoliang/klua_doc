> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 10 Appendix

### 10.1 Globally Unique Identifiers (GUIDs)

A GUID is the Microsoft implementation of a universally unique identifier. A GUID is a 128-bit value consisting of one group of 8 hexadecimal digits, followed by three groups of 4 hexadecimal digits each, and followed by one group of 12 hexadecimal digits, for example {6B29FC40-CA47-1067-B31D-00DD010662DA}, (see Table 38).

**Table 38 GUID Structure**

| **Field Name** | **Offset** **(byte)** | **Size** **(bytes)** | **Comments** |
| --- | --- | --- | --- |
| Data1 | 0 | 4 | This field is mandatory and contains the four bytes from the first group of the GUID (6B29FC40h from the example). |
| Data2 | 4 | 2 | This field is mandatory and contains the two bytes from the second group of the GUID (CA47h from the example). |
| Data3 | 6 | 2 | This field is mandatory and contains the two bytes from the third group of the GUID (1067h from the example). |
| Data4[0] | 8 | 1 | This field is mandatory and contains the most significant byte from fourth group of the GUID (B3h from the example). |
| Data4[1] | 9 | 1 | This field is mandatory and contains the least significant byte from the fourth group of the GUID (1Dh from the example). |
| Data4[2] | 10 | 1 | This field is mandatory and contains the first byte from the fifth group of the GUID (00h from the example). |
| Data4[3] | 11 | 1 | This field is mandatory and contains the second byte from the fifth group of the GUID (DDh from the example). |
| Data4[4] | 12 | 1 | This field is mandatory and contains the third byte from the fifth group of the GUID (01h from the example). |
| Data4[5] | 13 | 1 | This field is mandatory and contains the fourth byte from the fifth group of the GUID (06h from the example). |
| Data4[6] | 14 | 1 | This field is mandatory and contains the fifth byte from the fifth group of the GUID (62h from the example). |
| Data4[7] | 15 | 1 | This field is mandatory and contains the sixth byte from the fifth group of the GUID (DAh from the example). |

### 10.2 Partition Tables

To ensure interoperability of exFAT volumes in a broad set of usage scenarios, implementations should use partition type 07h for MBR partitioned storage and partition GUID {EBD0A0A2-B9E5-4433-87C0-68B6B72699C7} for GPT partitioned storage.
