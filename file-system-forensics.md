# File System Forensics

List partition table: <br>
`mmls able2.dd` <br>

Check the contents of a specific partition: <br>
`fsstat -o 10260 able2.dd` <br>
This gives us the last time it was mounted, the last time it was written on.
<br>

List file names and directory:<br>
`fls -o 10260 able2.dd`<br>
This will automatically search on the root directory. But if you want to specify the directory you want to search in:<br>
`fls -o 10260 able2.dd 2`<br>

- d/d: directory
- v/v: virtual folder, represent unallocated metadata entries where there are no corresponding files.
- r/r: raw file
<br>

You can now, iteratively, go through every directory with the same command to check what they store:<br>

`fls -o 10260 able2.dd 11105` <br>

This will enter the directory on that offset and print the stuff inside it.

## uncover deleted files

`fls -o 10260 able2.dd -Frd able2.dd`<br>

- -F: only file entries
- -r: recursively
- -d: deleted

<br>

Getting all the names associated with an inode:<br>
`ffind -o 10260 -a able2.dd 2139`<br>

Gattering info about an inode:<br>
`istat -o 10260 able2.dd 2139`<br>
This command reads the inode statistics on the file system.

Getting the data from a deleted file and storing it on another file for analysics:<br>
`icat -o 10260 able2.dd 2139 > lrkn.tgz.2139`


In some file systems (like EXT3 and EXT4) the file metadata are cleared after deleting the file, and so the techniques seen at the top are not possible. We have to use different methods.

`mmls able_3/able_3.000`

checking the content of a file:
`icat -o 104448 able_3/able_3.000 21 | xxd | head -n 5`<br>

In deleted files, nothing wil output (for EXT file systems) cause there not a pointer to the file metadata after its deletion

## File Carving
For when we can't check the content of deleted files.
Works for when directory entries are corrupted or missing

### Scalpel
file-system independent tool that reads a database of header and footer definitions and extracts matching files or data fragments from a set of image files or raw device files.<br>
It works by having a config file that allows to name the type of files that we want to carve from unnalocated memory (also deleted files)<br>

To get that unnalocated part we use the command:<br>
`blkls -o 104448 able_3/able_3.000 > home.blkls`<br>
And run scalpel:<br>
`scalpel -c scalpel.conf -o scalpel_out -O home.blkls`