> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: `portfs/doc/std_fat/` offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## Name Matching In Short & Long Names

Name Matching In Short & Long Names
The names contained in the set of all short directory entries are termed the "short name space".  The names contained in the set of all long directory entries are termed the "long name space".  Together, they form a single unified name space in which no duplicate names can exist.  That is: any name within a specific directory, whether it is a short name or a long name, can occur only once in the name space.  Furthermore, although the case of a name is preserved in a long name, no two names can have the same name although the names on the media actually differ by case.  That is names like "foobar" cannot be created if there is already a short entry with a name of "FOOBAR" or a long name with a name of "FooBar".

All types of search operations within the file system (i.e. find, open, create, delete, rename) are case-insensitive.  An open of "FOOBAR" will open either "FooBar" or "foobar" if one or the other exists.  A find using "FOOBAR" as a pattern will find the same files mentioned.  The same rules are also true for extended characters that are accented.

A short name search operation checks only the names of the short directory entries for a match.  A long name search operation checks both the long and short directory entries.  As the file system traverses a directory, it caches the long-name sub-components contained in long directory entries.  As soon as a short directory entry is encountered that is associated with the cached long name, the long name search operation will check the cached long name first and then the short name for a match.

When a character on the media, whether it is stored in the OEM character set or in UNICODE, cannot be translated into the appropriate character in the OEM or ANSI code page, it is always "translated" to the "_" (underscore) character as it is returned to the user
 it is NOT modified on the disk.  This character is the same in all OEM code pages and ANSI.
