> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: `portfs/doc/std_fat/` offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## Effect of Long Directory Entries on Down Level Versions of FAT

Effect of Long Directory Entries on Down Level Versions of FAT
The support of long names is most important on the hard disk, however it will be supported on removable media as well.  The implementation provides support for long names without breaking compatibility with the existing FAT format.  A disk can be read by a down level system without any compatibility problems.  An existing disk does not go through a conversion process before it can start using long names.  All of the current files remain unmodified.  The long name directory entries are added when a long name is created.  The addition of a long name to an existing file may require the 8.3 directory entry to be moved if the required adjacent directory entries are not available.

The long name entries are as hidden as hidden or system files are on a down level system.  This is enough to keep the casual user from causing problems.  The user can copy the files off using the 8.3 name, and put new files on without any side effects

The interesting part of this is what happens when the disk is taken to a down level FAT system and the directory is changed.  This can affect the long name entries since the down level system ignores these long names and will not ensure they are properly associated with the 8.3 names.

A down level system will only see the long name entries when searching for a label.  On a down level system, the volume label will be incorrectly reported if the true volume label does not come before all of the long name entries in the root directory.  This is because the long name entries also have the volume label bit set.  This is unfortunate, but is not a critical problem.

If an attempt is made to remove the volume label, one of the long name directory entries may be deleted.  This would be a rare occurrence.  It is easily detected on an aware system.  The long name entry will no longer be a valid file entry, since one or more of the long entries is marked as deleted.  If the deleted entry is reused, then the attribute byte will not have the proper value for a long name entry.

If a file is renamed on a down level system, then only the short name will be renamed.  The long name will not be affected.  Since the long and short names must be kept consistent across the name space, it is desirable to have the long name become invalid as a result of this rename.  The checksum of the 8.3 name that is kept in the long name directory provides the ability to detect this type of change.  This checksum will be checked to validate the long name before it is used.  Rename will cause problems only if the renamed 8.3 file name happens to have the same checksum.  The checksum algorithm chosen has a relatively flat distribution across the short name space.

This rename of the 8.3 name must also not conflict with any of the long names.  Otherwise a down level system could create a short name in one file that matches a long name, when case is ignored, in a different file.  To prevent this, the automatic creation of an 8.3 name from a long name, that has an 8.3 format, will directly map the long name to the 8.3 name by converting the characters to upper case.

If the file is deleted, then the long name is simply orphaned.  If a new file is created, the long name may be incorrectly associated with the new file name.  As in the case of a rename the checksum of the 8.3 name will help prevent this incorrect association.
