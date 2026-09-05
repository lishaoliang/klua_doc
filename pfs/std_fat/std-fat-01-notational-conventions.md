> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/) offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## Notational Conventions in this Document

Notational Conventions in this Document

Numbers that have the characters
 at the beginning of them are hexadecimal (base 16) numbers.

Any numbers that do not have the characters
 at the beginning are decimal (base 10) numbers.

The code fragments in this document are written in the
 programming language. Strict typing and syntax are not adhered to.

There are several code fragments in this document that freely mix 32-bit and 16-bit data elements. It is assumed that you are a programmer who understands how to properly type such operations so that data is not lost due to truncation of 32-bit values to 16-bit values. Also take note that all data types are UNSIGNED. Do not do FAT computations with signed integer types, because the computations will be wrong on some FAT volumes.
