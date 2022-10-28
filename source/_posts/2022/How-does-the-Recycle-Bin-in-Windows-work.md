---
title: How does the Recycle Bin in Windows work?
date: 2022-10-28 09:28:39
updated: 2022-10-28 09:28:39
tags: [Windows]
categories: Windows
---

> References from:
>
> https://superuser.com/questions/368890/how-does-the-recycle-bin-in-windows-work/1736690#1736690
>
> https://en.wikipedia.org/wiki/Trash_(computing)#Microsoft_Windows

<!-- more -->

The actual location of the Recycle Bin depends on the type of operating system and file system. On older [FAT](https://en.wikipedia.org/wiki/File_Allocation_Table) file systems (typically Windows 98 and prior), it is located in *Drive:\RECYCLED*. In the [NTFS](https://en.wikipedia.org/wiki/NTFS) filesystem (Windows 2000, XP, NT) it is *Drive:\RECYCLER*. On Windows Vista and above it is *Drive:\$Recycle.Bin* folder.[[35\]](https://en.wikipedia.org/wiki/Trash_(computing)#cite_note-35)

The Recycle Bin can be accessed from the desktop or Windows Explorer,[*[how?](https://en.wikipedia.org/wiki/Wikipedia:Please_clarify)*] or by typing "shell:RecycleBinFolder" in the [Run dialog box](https://en.wikipedia.org/wiki/Run_command) (⊞ Win+R). It is the only icon shown by default on the Windows XP desktop. When accessed from the desktop, the Recycle Bin options and information are different from those of the physical Recycle Bin folders seen on each partition in [Windows Explorer](https://en.wikipedia.org/wiki/Windows_Explorer). From [Windows XP](https://en.wikipedia.org/wiki/Windows_XP) onwards, with [NTFS](https://en.wikipedia.org/wiki/NTFS), different users cannot see the contents of each other's Recycle Bins.

Prior to Windows Vista, a file in the Recycle Bin is stored in its physical location and renamed as `D<original drive letter of file><#>.<original extension>`.[[16\]](https://en.wikipedia.org/wiki/Trash_(computing)#cite_note-RBOverview-16) A hidden file called *info2* (*info* in Windows 95 without the [Windows Desktop Update](https://en.wikipedia.org/wiki/Windows_Desktop_Update)) stores the file's original path and original name in binary format.[[16\]](https://en.wikipedia.org/wiki/Trash_(computing)#cite_note-RBOverview-16) Since Windows Vista, the "meta" information of each file is saved as `$I<number>.<original extension>` and the original file is renamed to `$R<number>.<original extension>`.

When the user views the Recycle Bin, the files are displayed with their original names. When the file is "Restored" from the Recycle Bin, it is returned to its original directory and name.[[16\]](https://en.wikipedia.org/wiki/Trash_(computing)#cite_note-RBOverview-16)

------

When you delete file in the Bin, it is not actually moved to it.

First, lets take a look at the subfolders of `$Recycle.Bin`:

- `C:\$Recycle.Bin\S-1-5-18` is folder for built-in SYSTEM account
- `C:\$Recycle.Bin\S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-1XXX\`, starting from 1000 are non-built in user folders. You should first check which one of them is the one you need by typing `whoami /all` in command prompt to get your [SID](https://learn.microsoft.com/en-us/windows/win32/secauthz/well-known-sids), or `wmic useraccount get name,sid` to get all local accounts SIDs, then choose the folder matching with SID.

> The subfolders name such as `S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-1XXX` is a User [SID](https://learn.microsoft.com/en-us/windows/win32/secauthz/well-known-sids)

If you give security rights to your account to all this folders, Explorer might show your deleted files in every folder, however its just Explorer bug. If you navigate to, say, `C:\$Recycle.Bin\S-1-5-18` folder and type `dir /a`, you will see that its actually empty, and only folder that match with your SID contains your deleted files.

Note that even if you are the only user on the PC, you might still have folder `C:\$Recycle.Bin\S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-1000\` that most likely will be empty. Then your actual folder will be `...-1001\`. User "1000" seems to be dangling folder for user that was created automatically by Windows on installation or some updates, then deleted. But `Recycle.Bin` folder wasn't destroyed and just stayed there. I assume that its safe to delete this dangling folder. I'm not so sure about `C:\$Recycle.Bin\S-1-5-18`, but I'm fairly sure its safe too, as there is no need for OS SYSTEM account to use the Bin, anyway.

So, here is deletion algorithm:

1. System creates [hardlink](https://docs.microsoft.com/en-us/windows/win32/fileio/hard-links-and-junctions) of the deleted file in `C:\$Recycle.Bin\S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-1XXX\` folder with the name `$RXXXXXX.<file_ext>`, where `XXXXXX` is 6 symbols HASH calculated based on the file contents, as I assume.
2. Metadata file is created in the same folder with the name `$IXXXXXX.<file_ext>`. I will show you the content of this file later.
3. Original file is deleted, but because of extra hardlink in `$Recycle.Bin`, actual file's data stays in the same location on the drive as if file where never touched. It is not moved anywhere. This is why each logical volume must have it's own `$Recycle.Bin` folder, as hardlinks only work within the same volume.

That's about it. Restoration algorithm:

1. Metadata file is read and hardlink of `$RXXXXXX.<file_ext>` is created based on `$IXXXXXX.<file_ext>`'s info in original file location, with original file name.
2. Both metadata and backup file are deleted from `$Recycle.Bin`

Pretty simple, ain't it?

Now, the most interesting part is that metadata file:

```
0000  02 00 00 00 00 00 00 00  <-- File header (QW)
0008  00 7C 0A 00 00 00 00 00  <-- File size (QW)
0010  90 83 72 44 28 9C D8 01  <-- File deletion date (QW)
0018  18 00 00 00|43 00 3A 00  <-- File path string length (DW)
0020  5C 00 24 00 52 00 65 00  
0028  63 00 79 00 63 00 6C 00
0030  65 00 2E 00 42 00 69 00
0038  6E 00 5C 00 66 00 73 00
0040  73 00 2E 00 65 00 78 00
0048  65 00 00 00|             <-- |Null-terminated path string| (wchar_t)
```

All values are in [Little Endian](https://en.wikipedia.org/wiki/Endianness) format.

Header is fixed and its identical for all the files. Be warned that some `$I` files might have some junk bytes `FF FE` appear before the Header. I have no idea what are this for, so you should check for full header before reading further.

Deletion date is in [FILETIME](https://docs.microsoft.com/en-us/windows/win32/api/minwinbase/ns-minwinbase-filetime) format and can be converted to usual [SYSTEMTIME](https://docs.microsoft.com/en-us/windows/win32/api/minwinbase/ns-minwinbase-systemtime) via [FileTimeToSystemTime](https://docs.microsoft.com/en-us/windows/win32/api/timezoneapi/nf-timezoneapi-filetimetosystemtime). It represents 100-nanosecond intervals since January 1, 1601 (UTC).

So it's not super complicated file format, but quite interesting design.

To reclaim all of that storage, We can open an elevated command prompt and execute the following command:

```
rd /s /q c:\$Recycle.Bin
```

Following a reboot, the `$RECYCLE.BIN` folder will be recreated and each Windows user profile will now have an empty recycle bin.
