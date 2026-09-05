> **Source**: [exFAT File System Specification](https://learn.microsoft.com/en-us/windows/win32/fileio/exfat-specification) (Microsoft Learn; also docs.microsoft.com redirect)
> **Local mirror**: [portfs/doc/std_exfat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_exfat/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25; ms.date 2025-07-08

## 1 Introduction

The exFAT file system is the successor to FAT32 in the FAT family of file systems. This specification describes the exFAT file system and provides all the information necessary for implementing the exFAT file system.

### 1.1 Design Goals

The exFAT file system has three central design goals (see list below).

1. *Retain the simplicity of FAT-based file systems.*

>
> Two of the strengths of FAT-based file systems are their relative simplicity and ease of implementation. In the spirit of its predecessors, implementers should find exFAT relatively simple and easy to implement.
2. *Enable very large files and storage devices.*

>
> The exFAT file system uses 64 bits to describe file size, thereby enabling applications which depend on very large files. The exFAT file system also allows for clusters as large as 32MB, effectively enabling very large storage devices.
3. *Incorporate extensibility for future innovation.*

>
> The exFAT file system incorporates extensibility into its design, enabling the file system to keep pace with innovations in storage and changes in usage.

### 1.2 Specific Terminology

In the context of this specification, certain terms (see Table 1) carry specific meaning for the design and implementation of the exFAT file system.

**Table 1 Definition of Terms Which Carry Very Specific Meaning**

| **Term** | **Definition** |
| --- | --- |
| Shall | This specification uses the term “shall” to describe a behavior which is mandatory. |
| Should | This specification uses the term “should” to describe a behavior which it strongly recommends, but does not make mandatory. |
| May | This specification uses the term “may” to describe a behavior which is optional. |
| Mandatory | This term describes a field or structure which an implementation shall modify and shall interpret as this specification describes. |
| Optional | This term describes a field or structure which an implementation may or may not support. If an implementation supports a given optional field or structure, it shall modify and shall interpret the field or structure as this specification describes. |
| Undefined | This term describes field or structure contents which an implementation may modify as necessary (i.e. clear to zero when setting surrounding fields or structures) and shall not interpret to hold any specific meaning. |
| Reserved | This term describes field or structure contents which implementations: 1. Shall initialize to zero and should not use for any purpose 2. Should not interpret, except when computing checksums 3. Shall preserve across operations which modify surrounding fields or structures |

### 1.3 Full Text of Common Acronyms

This specification uses acronyms in common use in the personal computer industry (see Table 2).

**Table 2 Full Text of Common Acronyms**

| **Acronym** | **Full Text** |
| --- | --- |
| ASCII | American Standard Code for Information Interchange |
| BIOS | Basic Input Output System |
| CPU | Central Processing Unit |
| exFAT | extensible File Allocation Table |
| FAT | File Allocation Table |
| FAT12 | File Allocation Table, 12-bit cluster indices |
| FAT16 | File Allocation Table, 16-bit cluster indices |
| FAT32 | File Allocation Table, 32-bit cluster indices |
| GPT | GUID Partition Table |
| GUID | Globally Unique Identifier (see Section 10.1) |
| INT | Interrupt |
| MBR | Master Boot Record |
| texFAT | Transaction-safe exFAT |
| UTC | Coordinated Universal Time |

### 1.4 Default Field and Structure Qualifiers

Fields and structures in this specification have the following qualifiers (see list below), unless the specification notes otherwise.

1. Are unsigned
2. Use decimal notation to describe values, where not otherwise noted; this specification uses the post-fix letter "h" to denote hexadecimal numbers and encloses GUIDs in curly braces
3. Are in little-endian format
4. Do not require a null-terminating character for strings

### 1.5 Windows CE and TexFAT

TexFAT is an extension to exFAT that adds transaction-safe operational semantics on top of the base file system. TexFAT is used by Windows CE. TexFAT requires the use of the two FATs and allocation bitmaps for use in transactions. It also defines several additional structures including padding descriptors and security descriptors.
