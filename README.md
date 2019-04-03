# Port of unrar for MorphOS and AmigaOS 4

<h2>State of work:</h2>

<h3>Basic functions</h3>

All the basic functions like listing (verbose, base, technical), unpacking (w/ or w/o path, selected files or all), testing or printing works fine. Note, that wherever a path is needed it has to be in amiga format not POSIX ie. //test.rar not ../../test.rar.

<h3>National characters support</h3>

RAR stores data in UTF-8 in an archive file, then read it into Unicode (wchar_t) and finally it uses it (prints, create files, find files, ...) in UTF-8. Unfortunately no Amiga operating system supports UTF-8 today, therefore in order to handle national characters in file names, comments, options, etc. it needs to be converted to OS local encoding sets in OS prefs. Unrar reads what was set using environment variables:
* on Morphos: CODEPAGE
* on AmigaOS4: Charset
Characters that cannot be converted, because are not present in selected codepage, are replaced with '?'. That means that national characters are fully supported but only for selected locale, i.e. if I have Polish locale and have a rar archive with Spanish characters, they will not appeart and will be replaced by '?'. That in turns mean that some files may have the same names even if they are different files. In that case only one will be unpacked. To deal with this, you can use special environment variable RAR_CODEPAGE that allows overriding codepage (for both MorhOS and AmigaOS4) without changing system prefs. For an example above, if you set RAR_CODEPAGE, you will still not see Spanish characters (unless you have the right font) but at least you should get all the files unpacked.


<h4>National characters in file names</h4>

National characters in in file names are handled as described above - this covers:
* name of files and directories in an archive
* archive names given to unrar command
* names in list files (if -sc option is not used)

Obviously, in order to see national characters in shell or when browsing unpacked files in Ambient/Workbench you need to have fonts with the right encoding installed in OS and set as system font and/or in Ambient.

<h4>National characters in password</h4>

National characters are supported in passwords. A password can be given in the following forms:
* from shell when asked
* from shell command as -p 
* from RAR envoronment variable

In each of the case above, only passwords containing national characters of currently selected locale are supported and should be given using locale encoding (as for file names described above).

<h4>-sc switch</h4>

Unrar has -sc<chr>[obj] switch described as "Specify the character set" that is not very useful and not very well documented. From the source code I read that <chr> can be one of: 
* "a" for ANSI, 
* "o" for OEM, 
* "u" for Unicode and 
* "f" for UTF-8. 

For non-Windows systems, only "u" and "f" matters (BTW: "u" is UTF-16 not UTF-32).

[obj] is one of:
* 'c' for comment - does not apply unrar, used only in 'full' rar utility
* 'r' for redirect - applies to Windows only
* 'l' for file lists - applies to file list to unpack (specified after "@"), file list to exclude (specified after -x@) and filter file list (specified after -n@).
  
So for option -sc is not used, unrar will try to autodetect UTF-16 and UTF-8 and when it fails it will assume local encoding. The latter will also happen when "a" or "o" was used.

All above is standard unrar behavior - I'm only documenting it here.

<h3>Wildcards</h3>

Unrar supports wildcard in Windows style:
* "?" = any character
* "*" = any charasters sequence

Amiga style is not supported for this moment.

<h3>File modification dates</h3>

When unpacking, created files get modification date set as stored in RAR archive. The data is set in the local time zone. As there is no special attribute for creation data in AmigaOS, this value is always ignored.

<h3>File owner and group</h3>

When unpacking from RAR archive with option -ow, unrar will try to set owner name and group. It will only success if user with given name and group with given name is present in local system. Otherwise an error will be printed. When -ow is not used, owner/group is ignored.

<h3>Symbolic links in an archive</h3>

RAR can store symbolic links and this unrar port supports that. When unpacking, targets for unix symbolic links will be changed into Amiga links (Makelink function) provided that filesystem into which unpack is done supports that (SFS does). When the filesystem does not support symbolic links, an error will be printed and the sympolic link will be ignored.

As RAR stores symbolic link target in the form that is right on OS where an archive was created, before making links in Amiga, the path need to be converted (for instance ../test.txt need to be changed to /test.txt). This unrar port does such conversion.

Note: unrar checks if a link is safe to create and will skip links targeting outside of enpacking directory. In order to make it create them, you need to use -ola switch when unpacking.

<h3>Hard links in an archive</h3>

Hard links are supported in a similar way as symbilic links. Again, they will only work if target file system supports hard links. Otherwise (like in case of SFS that does not support hard links) an error will be printed and hardlink will be ignored. There is no option in unrar that would allow the user to unpack hardlinks and regular files or symbolic links - so if target file system does not support hard links it is not possible to get them unpacked. 

<h3>Reference files in an archive</h3>

RAR can pack identical files as references inside of an archive - that reduces size of the archive file. In order to make such archive -oi switch should be used when packing. When unpacking, duplicates are re-created as separate files. That is fully supported in this unrar port.

<h3>NTFS Junction Points in an archive</h3>

NTFS Junction Points are like symbolic links but their target is always absolute path with a drive letter in front. In Unrar version 5.7 for Unix-like systems (including Amiga, at least from devenv stand point) these kind of links are always skipped silently (!). While is a different behaviour from how it was in 5.0, I decided to leave it unchanged for Amiga. So in practice, junction points are always skipped when unpacking.

<h3>Configuration file</h3>

Unrar has a config file in which you can store switches that will always be used when unrar command is called. In order to do that, "switches=" should be present in the file with all the switches to add (for instance "switches=-ad -ola"). This port of unrar looks for configuration file in s:rar.conf.

There is also "RAR" environment variable that can have switches to use (like "switches=" in configuration file). The priority of resolving options to use is: directly given in command, RAR environment variable, configuration file.

<h3>Support for locatization</h3>

This unrar port complies with locatization rules of MorphOS and AmigaOS. unrar.cs file is included in the bundle, and can be base for making translations using tools like SimpleCat. 

<h2>Notes about porting</h2>

This port is based directly on unrar source code version 5.7.3 (unrarsrc-5.7.3.tar.gz) from rarlab.com:
https://www.rarlab.com/rar/unrarsrc-5.7.3.tar.gz

For normalizing UTF, I used utf8proc. I did not need to port it for Amiga - it compiles without any change:
https://juliastrings.github.io/utf8proc/

For vfwprintf-like functions from libc, that are not implemented on Amiga at all, I ported the code I took from newlib 3.1.0:
http://sourceware.org/newlib/

ftp://sourceware.org/pub/newlib/newlib-3.1.0.tar.gz

MorphOS version of unrar has been compiled using gcc 4.4.5 (part of SDK 3.12).
AmigaOS4 version of unrar has been compiled using gcc 4.x.x (part of SDK x.xx).

