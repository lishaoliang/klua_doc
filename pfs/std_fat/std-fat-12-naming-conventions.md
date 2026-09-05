> **Source**: Microsoft Extensible Firmware Initiative FAT32 File System Specification (fatgen103), Version 1.03, December 6, 2000
> **Local mirror**: [portfs/doc/std_fat/](https://gitee.com/klua/portfs/tree/trunk/doc/std_fat/) offline reference for pfs; official Microsoft download is authoritative.
> **Fetched**: 2026-07-25

## Naming Conventions and Long Names

Naming Conventions and Long Names
An API allows the caller to specify the long name to be assigned to a file or directory.  They do not allow the caller to independently specify the short name.  The reason for this prohibition is that the short and long names are considered to be a single unified name space.  As should be obvious the file system's name space does not support duplicate names.  In other words, a long name for a file may not contain the same name, ignoring case, as the short name in a different file.  This restriction is intended to prevent confusion among users, and applications, regarding the proper name of a file or directory.  To make this restriction transparent, whenever a long name is created and the no matching long name exists, the short name is automatically generated from the long name in such a way that it does not collide with an existing short name.

The technique chosen to auto-generate short names from long names is modeled after Windows NT.  Auto-generated short names are composed of the basis-name and an optional numeric-tail.

The Basis-Name Generation Algorithm

The basis-name generation algorithm is outlined below.  This is a sample algorithm and serves to illustrate how short names can be auto-generated from long names. An implementation should follow this basic sequence of steps.

1.	The UNICODE name passed to the file system is converted to upper case.
2.	The upper cased UNICODE name is converted to OEM.
if		(the uppercased UNICODE glyph does not exist as an OEM glyph in the OEM code page)
	or	(the OEM glyph is invalid in an 8.3 name)
	Replace the glyph to an OEM '_' (underscore) character.
	Set a "lossy conversion" flag.
}
3.	Strip all leading and embedded spaces from the long name.
4.	Strip all leading periods from the long name.
5.	While		(not at end of the long name)
	and	(char is not a period)
	and	(total chars copied < 8)
	Copy characters into primary portion of the basis name
}
6.	Insert a dot at the end of the primary components of the basis-name iff the basis name has an extension after the last period in the name.
7.	Scan for the last embedded period in the long name.
If	(the last embedded period was found)
	While		(not at end of the long name)
		and	(total chars copied < 3)
		Copy characters into extension portion of the basis name
}
Proceed to numeric-tail generation.

The Numeric-Tail Generation Algorithm

	If			(a "lossy conversion" was not flagged)
	and	(the long name fits within the 8.3 naming conventions)
	and	(the basis-name does not collide with any existing short name)
	The short name is only the basis-name without the numeric tail.
	Insert a numeric-tail "~n" to the end of the primary name such that the value of the "~n" is chosen so that the
	name thus formed does not collide with any existing short name and that the primary name does not exceed eight 	characters in length.
}
The "~n" string can range from "~1" to "~999999".  The number "n" is chosen so that it is the next number in a sequence of files with similar basis-names.  For example, assume the following short names existed: LETTER~1.DOC and LETTER~2.DOC.  As expected, the next auto-generated name of name of this type would be LETTER~3.DOC.  Assume the following short names existed: LETTER~1.DOC, LETTER~3.DOC.  Again, the next auto-generated name of name of this type would be LETTER~2.DOC.  However, one absolutely cannot count on this behavior.  In a directory with a very large mix of names of this type, the selection algorithm is optimized for speed and may select another "n" based on the characteristics of short names that end in "~n" and have similar leading name patterns.
