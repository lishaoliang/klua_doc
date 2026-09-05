> **Source**: [How NTFS Works](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10)) (Microsoft Learn; Windows Server 2003 TechNet archive)
> **Local mirror**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25
> **Note**: Microsoft does **not** publish a single complete public NTFS on-disk specification comparable to exFAT / fatgen103.

## NTFS Architecture

During format and setup of a volume file system on a hard disk, a master boot record (MBR) is created. The MBR contains a small amount of executable code called the master boot code as well as a partition table for the disk. When a volume is mounted, the MBR executes the master boot code and transfers control to the boot sector on the disk, allowing the server to boot the operating system on the file system of that specific volume.

**Note**

- The partition table contains a number of fields used to describe the partition. One of these fields is the System ID field, which defines the file system, such as NTFS, on the partition. For NTFS volumes, the system ID is 0x07.

The figure NTFS Architecture shows the architecture of this process.

**NTFS Architecture**

![NTFS Architecture](images/cc781134.91d1303a-c92d-4e1e-a98e-aca7bfa54bf4(ws.10).gif)

The following table describes the components of an NTFS file system.

**NTFS Architecture Components on an x86-based System**

| Component | Component Description |
| --- | --- |
| Hard disk | Contains one or more partitions. |
| Boot sector | Bootable partition that stores information about the layout of the volume and the file system structures, as well as the boot code that loads Ntdlr. |
| Master Boot Record | Contains executable code that the system BIOS loads into memory. The code scans the MBR to find the partition table to determine which partition is the active, or bootable, partition. |
| Ntldlr.dll | Switches the CPU to protected mode, starts the file system, and then reads the contents of the Boot.ini file. This information determines the startup options and initial boot menu selections. |
| Ntfs.sys | System file driver for NTFS. |
| Ntoskrnl.exe | Extracts information about which system device drivers to load and the load order. |
| Kernel mode | The processing mode that allows code to have direct access to all hardware and memory in the system. |
| User mode | The processing mode in which applications run. |
