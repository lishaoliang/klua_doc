> **Source**: [How NTFS Works](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10)) (Microsoft Learn; Windows Server 2003 TechNet archive)
> **Local mirror**: `portfs/doc/std_ntfs/` offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25
> **Note**: Microsoft does **not** publish a single complete public NTFS on-disk specification comparable to exFAT / fatgen103.

## NTFS Physical Structure

### Organization of an NTFS Volume

The figure Organization of an NTFS Volume illustrates how NTFS organizes structures on a volume.

**Organization of an NTFS Volume**

![Organization of an NTFS Volume](images/cc781134.737c1f18-1bbc-45c7-9cb7-d61387d78324(ws.10).gif)

The following table describes each of the organizational structures on the NTFS volume.

**NTFS Volume Components**

| Component | Description |
| --- | --- |
| NTFS Boot Sector | Contains the BIOS parameter block that stores information about the layout of the volume and the file system structures, as well as the boot code that loads Windows Server 2003. |
| Master File Table | Contains the information necessary to retrieve files from the NTFS partition, such as the attributes of a file. |
| File System Data | Stores data that is not contained within the Master File Table. |
| Master File Table Copy | Includes copies of the records essential for the recovery of the file system if there is a problem with the original copy. |
