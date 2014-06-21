psypkg
======

Unpack, list and mount [Psychonauts](http://www.psychonauts.com/) .pkg archives. Only
the uncompressed version is currently supported, because I only have access to such
files with my Steam copy of Psychonauts.

Basic usage:

	psypkg.py list <archive>                 - list contens of .pkg archive
	psypkg.py unpack <archive>               - extract .pkg archive
	psypkg.py mount <archive> <mount-point>  - mount archive as read-only file system

The `mount` command depends on the [llfuse](https://code.google.com/p/python-llfuse/)
Python package. If it's not available the rest is still working.

This script is compatible with Python 2.7 and 3 (tested with 2.7.5 and 3.3.2).

File Format
-----------

I used [this](http://quickandeasysoftware.net/readmes/PsychonautsExplorerHelp/index.html?pkgfileformat.htm)
description of the file format to implement this script. Here follows a summary of the
file format.

There are many doubled file names in the `PsychonautsData2.pkg` file distributed with
Psychonauts for Steam. I guess the later file records are updates on the earlier ones?

For these file entries the second occurance will overwrite the first when unpacking.
When the archive is mounted as a file system name conflict resolution is a bit different.
In this case `~number` is added between the file name and the extension. `number` is the
first number that doesn't produce a confict.

	┌────────────────────────────────┐
	│                                │
	│  Header                        │
	│                                │
	│    file magic ("ZPKG")         │
	│    version                     │
	│    file data offset            │
	│    number of files             │
	│    unknown data offset         │
	│    ?                           │
	│    name directory offset       │
	│    file type directory offset  │
	│    zero-padding                │
	│                                │
	├────────────────────────────────┤
	│                                │
	│  File Records                  │
	│                                │
	│ ┌────────────────────────────┐ │
	│ │                            │ │
	│ │  Record                    │ │
	│ │                            │ │
	│ │    null                    │ │
	│ │    file type offset        │ │
	│ │    null                    │ │
	│ │    file name offset        │ │
	│ │    file data offset        │ │
	│ │    file data size          │ │
	│ │                            │ │
	│ └────────────────────────────┘ │
	│                                │
	│  ...                           │
	│                                │
	├────────────────────────────────┤
	│                                │
	│  ???                           │
	│                                │
	├────────────────────────────────┤
	│                                │
	│  Name Directory                │
	│                                │
	├────────────────────────────────┤
	│                                │
	│  File Type Directory           │
	│                                │
	├────────────────────────────────┤
	│                                │
	│  File Data                     │
	│                                │
	└────────────────────────────────┘

ZSTR is a zero-terminated string. All values are encoded in little endian byte order.

	Size  Type        Description
	 512  Header      archive header
	16*N  Record[N]   file records
       ?  ?           unknown data
	   ?  ZSTR[*]     name directory: sequence of zero-terminated strings
	   ?  ZSTR[*]     file type directory: sequence of zero-terminated strings
	   ?  uint8_t[*]  file data

### Header

	Offset  Size  Type          Description
	     0     4  char[4]       file magic ("ZPKG")
	     4     4  uint32_t      version (1)
	     8     4  uint32_t      file data offset
	    12     4  uint32_t      number of files
	    16     4  uint32_t      unknown data offset (end of file records)
        20     4  uint32_t      ?
	    24     4  uint32_t      name directory offset
	    28     4  uint32_t      file type directory offset
	    32   480  uint8_t[480]  zero-padding (maybe reversed for use in version >1?)

### Record

	Offset  Size  Type          Description
	     0     1  uint8_t       null (0)
	     1     2  uint16_t      file type offset (relative to file type direectory offset)
	     3     1  uint8_t       null (0)
	     4     4  uint32_t      file name offset (relative to name direectory offset)
	     8     4  uint32_t      file data offset
	    12     4  uint32_t      file data size

Related Projects
----------------

 * [fezpak](https://github.com/panzi/fezpak): pack, unpack, list and mount FEZ .pak archives
 * [bgebf](https://github.com/panzi/bgebf): pack, unpack, list and mount Beyond Good and Evil .bf archives
 * [unvpk](https://bitbucket.org/panzi/unvpk): extract, list, check and mount Valve .vpk archives

BSD License
-----------
Copyright (c) 2014 Mathias Panzenböck

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
