> **Source**: [How NTFS Works](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10)) (Microsoft Learn; Windows Server 2003 TechNet archive)
> **Local mirror**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25
> **Note**: Microsoft does **not** publish a single complete public NTFS on-disk specification comparable to exFAT / fatgen103.

## NTFS Physical Structure

### Boot Sectors

On MBR disks, the boot sector, which is located at the first logical sector of each partition, is a critical disk structure for starting your computer. It contains executable code and the data required by the code, including information that the file system uses to access the volume. The boot sector is created when you format a volume. At the end of the boot sector is a 2-byte structure called a signature word or end of sector marker, which is always set to 0x55AA. On computers running Windows Server 2003, the boot sector on the active partition loads into memory and starts Ntldr, which loads the boot menu if multiple versions of Windows are installed, or loads the operating system if only one operating system is installed.

GUID partition table (GPT) disks are similar to MBR disks, except they use primary and backup partition structures to provide redundancy. These structures are located at the beginning and the end of the disk. GPT identifies these structures by their logical block address (LBA) rather than by their relative sectors.

A boot sector consists of the following elements:

- An x86-based CPU jump instruction.
- The original equipment manufacturer identification (OEM ID).
- The BIOS parameter block (BPB), a data structure.
- The extended BPB.
- The executable boot code (or bootstrap code) that starts the operating system.

All Windows Server 2003 boot sectors contain the preceding elements regardless of the type of disk (basic disk or dynamic disk).

#### Components of a Boot Sector

The MBR transfers CPU execution to the boot sector, so the first three bytes of the boot sector must be valid, executable x86-based CPU instructions. This includes a jump instruction that skips the next several nonexecutable bytes.

Following the jump instruction is the 8-byte OEM ID, a string of characters that identifies the name and version number of the operating system that formatted the volume. To preserve compatibility with MS-DOS, Windows Server 2003 records “NTFS” in this field.

**Note**

- You might also see the OEM ID “MSWIN4.0” on disks formatted by Windows 95 and “MSWIN4.1” on disks formatted by Windows 95 OEM Service Release 2 (OSR2), Windows 98, and Windows Millennium Edition. Windows Server 2003 does not use the OEM ID field in the boot sector except for verifying NTFS volumes.

Following the OEM ID is the BPB, which provides information that enables the executable boot code to locate Ntldr. The BPB always starts at the same offset, so standard parameters are in a known location. Disk size and geometry variables are encapsulated in the BPB. Because the first part of the boot sector is an x86 jump instruction, the BPB can be extended in the future by appending new information at the end. The jump instruction needs only a minor adjustment to accommodate this change. The BPB is stored in a packed (unaligned) format.

#### NTFS Boot Sector

The table Boot Sector Sections on an NTFS Volume describes the boot sector of a volume that is formatted with NTFS. When you format an NTFS volume, the format program allocates the first 16 sectors for the boot sector and the bootstrap code.

**Boot Sector Sections on an NTFS Volume**

| Byte Offset | Field Length | Field Name |
| --- | --- | --- |
| 0x00 | 3 bytes | Jump instruction |
| 0x03 | 8 bytes | OEM ID |
| 0x0B | 25 bytes | BPB |
| 0x24 | 48 bytes | Extended BPB |
| 0x54 | 426 bytes | Bootstrap code |
| 0x01FE | 2 bytes | End of sector marker |

On NTFS volumes, the data fields that follow the BPB form an extended BPB. The data in these fields enables Ntldr to find the MFT during startup. On NTFS volumes, the MFT is not located in a predefined sector. For this reason, NTFS can move the MFT if there is a bad sector in the current location of the MFT. However, if the data is corrupted, the MFT cannot be located, and Windows Server 2003 assumes that the volume has not been formatted.

The following example illustrates the boot sector of an NTFS volume that is formatted by using Windows Server 2003. The printout is formatted in three sections:

- Bytes 0x00– 0x0A are the jump instruction and the OEM ID (shown in bold print).
- Bytes 0x0B–0x53 are the BPB and the extended BPB.
- The remaining code is the bootstrap code and the end of sector marker (shown in bold print).

```
Physical Sector: Cyl 0, Side 1, Sector 1
00000000: EB 52 90 4E 54 46 53 20 - 20 20 20 00 02 08 00 00 .R.NTFS ..... ..
00000010: 00 00 00 00 00 F8 00 00 - 3F 00 FF 00 3F 00 00 00 ........?...?...
00000020: 00 00 00 00 80 00 80 00 - 1C 91 11 01 00 00 00 00 ................
00000030: 00 00 04 00 00 00 00 00 - 11 19 11 00 00 00 00 00 ................
00000040: F6 00 00 00 01 00 00 00 - 3A B2 7B 82 CD 7B 82 14 ........:.{..{..
00000050: 00 00 00 00 FA 33 C0 8E - D0 BC 00 7C FB B8 C0 07 .....3.....|....
```

The table BPB and Extended BPB Fields on NTFS Volumes describes the fields in the BPB and the extended BPB on NTFS volumes. The fields starting at 0x0B, 0x0D, 0x15, 0x18, 0x1A, and 0x1C match those on FAT16 and FAT32 volumes. The sample values correspond to the data in this example.

**BPB and Extended BPB Fields on NTFS Volumes**

| Byte Offset | Field Length | Sample Value | Field Name and Definition |
| --- | --- | --- | --- |
| 0x0B | 2 bytes | 00 02 | **Bytes Per Sector**. The size of a hardware sector. For most disks used in the United States, the value of this field is 512. |
| 0x0D | 1 byte | 08 | **Sectors Per Cluster**.The number of sectors in a cluster. |
| 0x0E | 2 bytes | 00 00 | **Reserved Sectors**. Always 0 because NTFS places the boot sector at the beginning of the partition. If the value is not 0, NTFS fails to mount the volume. |
| 0x10 | 3 bytes | 00 00 00 | Value must be 0 or NTFS fails to mount the volume. |
| 0x13 | 2 bytes | 00 00 | Value must be 0 or NTFS fails to mount the volume. |
| 0x15 | 1 byte | F8 | **Media Descriptor**. Provides information about the media being used. A value of F8 indicates a hard disk and F0 indicates a high-density 3.5-inch floppy disk. Media descriptor entries are a legacy of MS-DOS FAT16 disks and are not used in Windows Server 2003. |
| 0x16 | 2 bytes | 00 00 | Value must be 0 or NTFS fails to mount the volume. |
| 0x18 | 2 bytes | 3F 00 | Not used or checked by NTFS. |
| 0x1A | 2 bytes | FF 00 | Not used or checked by NTFS. |
| 0x1C | 4 bytes | 3F 00 00 00 | Not used or checked by NTFS. |
| 0x20 | 4 bytes | 00 00 00 00 | The value must be 0 or NTFS fails to mount the volume. |
| 0x24 | 4 bytes | 80 00 80 00 | Not used or checked by NTFS. |
| 0x28 | 8 bytes | 1C 91 11 01 00 00 00 00 | **Total Sectors**. The total number of sectors on the hard disk. |
| 0x30 | 8 bytes | 00 00 04 00 00 00 00 00 | **Logical Cluster Number for the File $MFT**. Identifies the location of the MFT by using its logical cluster number. |
| 0x38 | 8 bytes | 11 19 11 00 00 00 00 00 | **Logical Cluster Number for the File $MFTMirr**. Identifies the location of the mirrored copy of the MFT by using its logical cluster number. |
| 0x40 | 1 byte | F6 | **Clusters Per MFT Record**. The size of each record. NTFS creates a file record for each file and a folder record for each folder that is created on an NTFS volume. Files and folders smaller than this size are contained within the MFT. If this number is positive (up to 7F), then it represents clusters per MFT record. If the number is negative (80 to FF), then the size of the file record is 2 raised to the absolute value of this number. |
| 0x41 | 3 bytes | 00 00 00 | Not used by NTFS. |
| 0x44 | 1 byte | 01 | **Clusters Per Index Buffer**. The size of each index buffer, which is used to allocate space for directories. If this number is positive (up to 7F), then it represents clusters per MFT record. If the number is negative (80 to FF), then the size of the file record is 2 raised to the absolute value of this number. |
| 0x45 | 3 bytes | 00 00 00 | Not used by NTFS. |
| 0x48 | 8 bytes | 3A B2 7B 82 CD 7B 82 14 | **Volume Serial Number**. The volume’s serial number. |
| 0x50 | 4 bytes | 00 00 00 00 | Not used by NTFS. |
