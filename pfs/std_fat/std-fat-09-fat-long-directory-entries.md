> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/) offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## FAT Long Directory Entries

FAT Long Directory Entries
In adding long directory entries to the FAT file system it was crucial that their addition to the FAT file system's existing design:

symbol 183 \f "Symbol" \s 10 \h
	Be essentially transparent on earlier versions of MS-DOS.  The primary goal being that existing MS-DOS APIs on previous versions of MS-DOS/Windows do not easily "find" long directory entries.  The only MS-DOS APIs that can "find" long directory entries are the FCB-based-find APIs when used with a full meta-character matching pattern (i.e. *.*) and full attribute matching bits (i.e. matching attributes are FFh).  On post-Windows 95 versions of MS-DOS/Windows, no MS-DOS API can accidentally "find" a single long directory entry.
symbol 183 \f "Symbol" \s 10 \h
	Be located in close physical proximity, on the media, to the short directory entries they are associated with.  As will be evident, long directory entries are immediately contiguous to the short directory entry they are associated with and their existence imposes an unnoticeable performance impact on the file system.
symbol 183 \f "Symbol" \s 10 \h
	If detected by disk maintenance utilities, they do not jeopardize the integrity of existing file data.  Disk maintenance utilities typically do not use MS-DOS APIs to access on-media file-system-specific data structures.  Rather they read physical or logical sector information from the disk and judge for themselves what the directory entries contain.  Based on the heuristics employed in the utilities, the utility may take various steps to "repair" what it perceives to be "damaged" file-system-specific data structures.  Long directory entries were added to the FAT file system in such a way as to not cause the loss of file data if a disk containing long directory entries was "repaired" by a pre-Windows 95-compatible disk utility on a previous version of MS-DOS/Windows.

In order to meet the goals of locality-of-access and transparency, the long directory entry is defined as a short directory entry with a special attribute.  As described previously, a long directory entry is just a regular directory entry in which the attribute field has a value of:

ATTR_LONG_NAME	ATTR_READ_ONLY |
ATTR_HIDDEN |
ATTR_SYSTEM |
ATTR_VOLUME_ID

A mask for determining whether an entry is a long-name sub-component should also be defined:

ATTR_LONG_NAME_MASK	ATTR_READ_ONLY |
ATTR_HIDDEN |
ATTR_SYSTEM |
ATTR_VOLUME_ID |
ATTR_DIRECTORY |
ATTR_ARCHIVE

When such a directory entry is encountered it is given special treatment by the file system.  It is treated as part of a set of directory entries that are associated with a single short directory entry. Each long directory entry has the following structure:

FAT Long Directory Entry Structure
Name
Offset
(byte)
Size
(bytes)
Description
LDIR_Ord
The order of this entry in the sequence of long dir entries associated with the short dir entry at the end of the long dir set.
If masked with 0x40 (LAST_LONG_ENTRY), this indicates the entry is the last long dir entry in a set of long dir entries. All valid sets of long dir entries must begin with an entry having this mask.
LDIR_Name1
Characters 1-5 of the long-name sub-component in this dir entry.
LDIR_Attr
Attributes - must be ATTR_LONG_NAME
LDIR_Type
If zero, indicates a directory entry that is a sub-component of a long name.  NOTE: Other values reserved for future extensions.

Non-zero implies other dirent types.
LDIR_Chksum
Checksum of name in the short dir entry at the end of the long dir set.
LDIR_Name2
Characters 6-11 of the long-name sub-component in this dir entry.
LDIR_FstClusLO
Must be ZERO. This is an artifact of the FAT "first cluster" and must be zero for compatibility with existing disk utilities.  It's meaningless in the context of a long dir entry.
LDIR_Name3
Characters 12-13 of the long-name sub-component in this dir entry.

Organization and Association of Short & Long Directory Entries

A set of long entries is always associated with a short entry that they always immediately precede.  Long entries are paired with short entries for one reason: only short directory entries are visible to previous versions of MS-DOS/Windows.  Without a short entry to accompany it, a long directory entry would be completely invisible on previous versions of MS-DOS/Windows.  A long entry never legally exists all by itself.  If long entries are found without being paired with a valid short entry, they are termed orphans.  The following figure depicts a set of n long directory entries associated with it's single short entry.

Long entries always immediately precede and are physically contiguous with, the short entry they are associated with.  The file system makes a few other checks to ensure that a set of long entries is actually associated with a short entry.

Sequence Of Long Directory Entries
Entry
Nth Long entry
LAST_LONG_ENTRY (0x40) | N
 Additional Long Entries
1st Long entry
Short Entry Associated With Preceding Long Entries
(not applicable)

First, every member of a set of long entries is uniquely numbered and the last member of the set is or'd with a flag indicating that it is, in fact, the last member of the set.  The LDIR_Ord field is used to make this determination.  The first member of a set has an LDIR_Ord value of one.  The nth long member of the set has a value of (n OR LAST_LONG_ENTRY).  Note that the LDIR_Ord field cannot have values of 0xE5 or 0x00.  These values have always been used by the file system to indicate a "free" directory entry, or the "last" directory entry in a cluster.  Values for LDIR_Ord do not take on these two values over their range.  Values for LDIR_Ord must run from 1 to (n OR LAST_LONG_ENTRY).  If they do not, the long entries are "damaged" and are treated as orphans by the file system.

Second, an 8-bit checksum is computed on the name contained in the short directory entry at the time the short and long directory entries are created.  All 11 characters of the name in the short entry are used in the checksum calculation.  The check sum is placed in every long entry.  If any of the check sums in the set of long entries do not agree with the computed checksum of the name contained in the short entry, then the long entries are treated as orphans.  This can occur if a disk containing long and short entries is taken to a previous version of MS-DOS/Windows and only the short name of a file or directory with a long entries is renamed.

The algorithm, implemented in C, for computing the checksum is:

	//-----------------------------------------------------------------------------
	//	ChkSum()
	//	Returns an unsigned byte checksum computed on an unsigned byte
	//	array.  The array must be 11 bytes long and is assumed to contain
	//	a name stored in the format of a MS-DOS directory entry.
	//	Passed:	 pFcbName    Pointer to an unsigned byte array assumed to be
//                          11 bytes long.
	//	Returns: Sum         An 8-bit unsigned checksum of the array pointed
//                           to by pFcbName.
	//------------------------------------------------------------------------------
	unsigned char ChkSum (unsigned char *pFcbName)
		short FcbNameLen;
		unsigned char Sum;
		Sum = 0;
		for (FcbNameLen=11; FcbNameLen!=0; FcbNameLen--) {
			// NOTE: The operation is an unsigned char rotate right
			Sum = ((Sum & 1) ? 0x80 : 0) + (Sum >> 1) + *pFcbName++;
		return (Sum);
	}

As a consequence of this pairing, the short directory entry serves as the structure that contains fields like: last access date, creation time, creation date, first cluster, and size.  It also holds a name that is visible on previous versions of MS-DOS/Windows.  The long directory entries are free to contain new information and need not replicate information already available in the short entry.  Principally, the long entries contain the long name of a file.  The name contained in a short entry which is associated with a set of long entries is termed the alias name, or simply alias, of the file.

Storage of a Long-Name Within Long Directory Entries

A long name can consist of more characters than can fit in a single long directory entry.  When this occurs the name is stored in more than one long entry.  In any event, the name fields themselves within the long entries are disjoint.  The following example is provided to illustrate how a long name is stored across several long directory entries.  Names are also NUL terminated and padded with 0xFFFF characters in order to detect corruption of long name fields by errant disk utilities.  A name that fits exactly in a n long directory entries (i.e. is an integer multiple of 13) is not NUL terminated and not padded with 0xFFFFs.

Suppose a file is created with the name: "The quick brown.fox".  The following example illustrates how the name is packed into long and short directory entries.  Most fields in the directory entries are also filled in as well.

The heuristics used to "auto-generate" a short name from a long name are explained in a later section.
