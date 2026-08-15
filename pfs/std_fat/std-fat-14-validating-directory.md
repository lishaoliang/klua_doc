> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: `portfs/doc/std_fat/` offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## Validating The Contents of a Directory

Validating The Contents of a Directory
These guidelines are provided so that disk maintenance utilities can verify individual directory entries for 'correctness' while maintaining compatibility with future enhancements to the directory structure.

1.	DO NOT look at the content of directory entry fields marked 'reserved' and assume that, if they are any value other than zero, that they are 'bad'.
2.	DO NOT reset the content of directory entry fields marked reserved to zero when they contain non-zero values (under the assumption that they are "bad").  Directory entry fields are designated reserved, rather than must-be-zero.  They should be ignored by your application..  These fields are intended for future extensions of the file system.  By ignoring them an utility can continue to run on future versions of the operating system.
3.	DO use the A_LONG attribute first when determining whether a directory entry is a long directory entry or a short directory entry.  The following algorithm is the correct algorithm for making this determination:

	if  (((LDIR_attr & ATTR_LONG_NAME_MASK) == ATTR_LONG_NAME) && (LDIR_Ord != 0xE5))
	/*   Found an active long name sub-component.   */
}

4.	DO use bits 4 and 3 of a short entry together when determining what type of short directory entry is being inspected.    The following algorithm is the correct algorithm for making this determination:

	if  (((LDIR_attr & ATTR_LONG_NAME_MASK) != ATTR_LONG_NAME) && (LDIR_Ord != 0xE5))
	if        ((DIR_Attr & (ATTR_DIRECTORY | ATTR_VOLUME_ID)) == 0x00)
		/*   Found a file.   */
	else if ((DIR_Attr & (ATTR_DIRECTORY | ATTR_VOLUME_ID)) == ATTR_DIRECTORY)
		/*   Found a directory.   */
	else if ((DIR_Attr & (ATTR_DIRECTORY | ATTR_VOLUME_ID)) == ATTR_VOLUME_ID)
		/*   Found a volume label.   */
		/*   Found an invalid directory entry.   */
}

5.	DO NOT assume that a non-zero value in the "type" field indicates a bad directory entry.  Do not force the "type" field to zero.
6.	Use the "checksum" field as a value to validate the directory entry.  The "first cluster" field is currently being set to zero, though this might change in future.
