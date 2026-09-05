> **Source**: [How NTFS Works](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc781134(v=ws.10)) (Microsoft Learn; Windows Server 2003 TechNet archive)
> **Local mirror**: [portfs/doc/std_ntfs/](https://gitee.com/klua/portfs/tree/trunk/doc/std_ntfs/) offline reference for pfs; official Learn page is authoritative.
> **Fetched**: 2026-07-25
> **Note**: Microsoft does **not** publish a single complete public NTFS on-disk specification comparable to exFAT / fatgen103.

## NTFS Physical Structure

### NTFS File Record Attributes

Every allocated sector on an NTFS volume belongs to a file. Even the file system metadata is part of a file. NTFS views each file (or folder) as a set of file attributes. File elements such as its name, its security information, and even its data are file attributes. Each attribute is identified by an attribute type code and an optional attribute name.

File and folder records are 1 KB each and are stored in the MFT, the attributes of which are written to the allocated space in the MFT. Besides file attributes, each file record contains information about the position of the file record in the MFT.

When a file’s attributes can fit within the MFT file record for that file, they are called resident attributes. Attributes such as file name and time stamp are always resident. When the amount of information for a file does not fit in its MFT file record, some file attributes become nonresident. Nonresident attributes are allocated one or more clusters of disk space. A portion of the nonresident attribute remains in the MFT and points to the external clusters. NTFS creates the Attribute List attribute to describe the location of all attribute records. The table NTFS File Attribute Types lists the file attributes currently defined by NTFS.

**NTFS File Attribute Types**

| Attribute Type | Description |
| --- | --- |
| Standard Information | Information such as access mode (read-only, read/write, and so forth) timestamp, and link count. |
| Attribute List | Locations of all attribute records that do not fit in the MFT record. |
| File Name | A repeatable attribute for both long and short file names. The long name of the file can be up to 255 Unicode characters. The short name is the 8.3, case-insensitive name for the file. Additional names, or hard links, required by POSIX can be included as additional file name attributes. |
| Data | File data. NTFS supports multiple data attributes per file. Each file typically has one unnamed data attribute. A file can also have one or more named data attributes. |
| Object ID | A volume-unique file identifier. Used by the distributed link tracking service. Not all files have object identifiers. |
| Logged Tool Stream | Similar to a data stream, but operations are logged to the NTFS log file just like NTFS metadata changes. This attribute is used by EFS. |
| Reparse Point | Used for mounted drives. This is also used by Installable File System (IFS) filter drivers to mark certain files as special to that driver. |
| Index Root | Used to implement folders and other indexes. |
| Index Allocation | Used to implement the B-tree structure for large folders and other large indexes. |
| Bitmap | Used to implement the B-tree structure for large folders and other large indexes. |
| Volume Information | Used only in the $Volume system file. Contains the volume version. |

NTFS creates a file record for each file and a folder record for each folder created on an NTFS volume. The MFT includes a separate file record for the MFT itself. These file and folder records are 1 KB each and are stored in the MFT. The attributes of the file are written to the allocated space in the MFT. Besides file attributes, each file record contains information about the position of the file record in the MFT. The figure MFT Entry with Resident Record shows the contents of an MFT record for a small file or folder. Small files and folders (typically, 900 bytes or smaller) are entirely contained within the file’s MFT record.

**MFT Entry with Resident Record**

![MFT Entry with Resident Record](images/cc781134.86787c15-cf0a-4cb9-8ba1-ff1afd37aaf5(ws.10).gif)

Typically, each file uses one file record. However, if a file has a large number of attributes or becomes highly fragmented, it might need more than one file record. If this is the case, the first record for the file, the base file record, stores the location of the other file records required by the file.

Folder records contain index information. Small folder records reside entirely within the MFT structure, while large folders are organized B-tree structures and have records with pointers to external clusters that contain folder entries that cannot be contained within the MFT structure.

The benefit of using B-tree structures is evident when NTFS enumerates files in a large folder. The B-tree structure allows NTFS to group, or index, similar file names and then search only the group that contains the file, minimizing the number of disk accesses needed to find a particular file, especially for large folders. Because of the B-tree structure, NTFS outperforms FAT for large folders because FAT must scan all file names in a large folder before listing all of the files.

#### Last Access Time

Each file and folder on an NTFS volume contains an attribute called Last Access Time. This attribute shows when the file or folder was last accessed, such as when a user performs a folder listing, adds files to a folder, reads a file, or makes changes to a file. The most up-to-date Last Access Time is always stored in memory and is eventually written to disk within two places:

- The file’s attribute, which is part of its MFT record.
- A directory entry for the file. The directory entry is stored in the folder that contains the file. Files with multiple hard links have multiple directory entries.

The Last Access Time on disk is not always current because NTFS looks for a one-hour interval before forcing the Last Access Time updates to disk. NTFS also delays writing the Last Access Time to disk when users or programs perform read-only operations on a file or folder, such as listing the folder’s contents or reading (but not changing) a file in the folder. If the Last Access Time is kept current on disk for read operations, all read operations become write operations, which impacts NTFS performance.

**Note**

- File-based queries of Last Access Time are accurate even if all on-disk values are not current. NTFS returns the correct value on queries because the accurate value is stored in memory.

NTFS eventually writes the in-memory Last Access Time to disk as follows.

##### Within the file’s attribute

NTFS typically updates a file’s attribute on disk if the current Last Access Time in memory differs by more than an hour from the Last Access Time stored on disk, or when all in-memory references to that file are gone, whichever is more recent. For example, if a file’s current Last Access Time is 1:00 P.M., and you read the file at 1:30 P.M., NTFS does not update the Last Access Time. If you read the file again at 2:00 P.M., NTFS updates the Last Access Time in the file’s attribute to reflect 2:00 P.M. because the file’s attribute shows 1:00 P.M. and the in-memory Last Access Time shows 2:00 P.M.

##### Within a directory entry for a file

NTFS updates the directory entry for a file during the following events:

- When NTFS updates the file’s Last Access Time and detects that the Last Access Time for the file differs by more than an hour from the Last Access Time stored in the file’s directory entry. This update typically occurs after a program closes the handle used to access a file within the directory. If the program holds the handle open for an extended time, a lag occurs before the change appears in the directory entry.
- When NTFS updates other file attributes such as Last Modify Time, and a Last Access Time update is pending. In this case, NTFS updates the Last Access Time along with the other updates without additional performance impact.

**Note**

- NTFS does not update a file’s directory entry when all in-memory references to that file are gone.

If you have an NTFS volume with a high number of folders or files, and a program is running that briefly accesses each of these in turn, the I/O bandwidth used to generate the Last Access Time updates can be a significant percentage of the overall I/O bandwidth.

#### Multiple Data Streams

A data stream is a sequence of bytes. An application populates the stream by writing data at specific offsets within the stream. The application can then read the data by reading the same offsets in the read path. Every file has a main, unnamed stream associated with it, regardless of the file system used.

However, NTFS supports additional named data streams in which each data stream is an alternate sequence of bytes as illustrated in the figure Unnamed and Named Streams. Applications can create additional named streams and access the streams by referring to their names. This feature permits related data to be managed as a single unit. For example, a graphics program can store a thumbnail image of bitmap in a named data stream within the NTFS file containing the image.

**Unnamed and Named Streams**

![Unnamed and Named Streams](images/cc781134.71c1e60b-b8a4-4e65-8bfc-a50995dbcfa8(ws.10).gif)

FAT volumes support only the main, unnamed stream, so if you try to copy or move Streamexample.doc to a FAT volume or floppy disk, you receive an error message.
