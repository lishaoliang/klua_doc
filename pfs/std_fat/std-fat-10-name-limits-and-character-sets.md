> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/) offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## Name Limits and Character Sets

Name Limits and Character Sets

Short Directory Entries

Short names are limited to 8 characters followed by an optional period (.) and extension of up to 3 characters.  The total path length of a short name cannot exceed 80 characters (64 char path + 3 drive letter + 12 for 8.3 name + NUL) including the trailing NUL.  The characters may be any combination of letters, digits, or characters with code point values greater than 127.  The following special characters are also allowed:

$   %   '   -   _   @   ~    `   !   (    )   {   }  ^  #  &

Names are stored in a short directory entry in the OEM code page that the system is configured for at the time the directory entry is created.  Short directory entries remain in OEM for compatibility with previous versions of MS-DOS/Windows.  OEM characters are single 8-bit characters or can be DBCS character pairs for certain code pages.

Short names passed to the file system are always converted to upper case and their original case value is lost.  One problem that is generally true of most OEM code pages is that they map lower to upper case extended characters in a non-unique fashion.  That is, they map multiple extended characters to a single upper case character.  This creates problems because it does not preserve the information that the extended character provides.  This mapping also prevents the creation of some file names that would normally differ, but because of the mapping to upper case they become the same file name.

Long Directory Entries

Long names are limited to 255 characters, not including the trailing NUL.  The total path length of a long name cannot exceed 260 characters, including the trailing NUL.  The characters may be any combination of those defined for short names with the addition of the period (.) character used multiple times within the long name.  A space is also a valid character in a long name as it always has been for a short name.  However, in short names it typically is not used.  The following six special characters are now allowed in a long name.  They are not legal in a short name.

+   ,   ;   =   [   ]

Embedded spaces within a long name are allowed.  Leading and trailing spaces in a long name are ignored.

Leading and embedded periods are allowed in a name and are stored in the long name.  Trailing periods are ignored.

Long names are stored in long directory entries in UNICODE.  UNICODE characters are 16-bit characters.  It is not be possible to store UNICODE in short directory entries since the names stored there are 8-bit characters or DBCS characters.

Long names passed to the file system are not converted to upper case and their original case value is preserved.  UNICODE solves the case mapping problem prevalent in some OEM code pages by always providing a translation for lower case characters to a single, unique upper case character.
