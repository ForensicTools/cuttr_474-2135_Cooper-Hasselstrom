cuttr_474-2135_Cooper-Hasselstrom
=================================

Extensible Forensic Framework

License: Private, do not disclose (TBD)

Tool Name: cuttr

Tool description: The tool that we will create will be an extensible forensic framework that may be extended through Python scripts or modules, Perl scripts or modules, C/C++ and Bash scripts (among others).  Some of the initial built-in functions **HOPE TO** include:

    Automated carving of disk partitions using fdisk
    Automated carving of disk partitions using mmls (Sleuth Kit)
    Automated carving of disk partitions using MBR analysis
    Automated File recovery using Sleuth Kit utilities
    Other functions as they arise throughout the development process


Licenses of tools used:

    fdisk: GNU LGPL
    fsstat: GNU LGPL
    dd: GNU GPL
    Sleuth Kit utilities are licensed under a number of licenses, including: http://www.sleuthkit.org/sleuthkit/licenses.php
        IBM Public License: http://www.sleuthkit.org/sleuthkit/ibmpl.php
        Common Public License: http://www.sleuthkit.org/sleuthkit/cpl10.php
        GNU GPL: http://www.sleuthkit.org/sleuthkit/gpl2.php
    Python: PSF LICENSE AGREEMENT FOR PYTHON 2.7.6
    Bash: GNU GPL
    Perl: GNU GPL


Step 1:  Parse the output of fdisk -l, storing output in python variables.

For example: 

    root@esio:~# fdisk -l data/input/sdb.dd

    Disk data/input/sdb.dd: 536 MB, 536870912 bytes
    255 heads, 63 sectors/track, 65 cylinders, total 1048576 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x7b28ae0c

                Device Boot    Start         End      Blocks   Id  System
    data/input/sdb.dd1          2048      261631      129792   83  Linux
    data/input/sdb.dd2        261632      523263      130816   83  Linux
    data/input/sdb.dd3        523264     1048575      262656    5  Extended
    data/input/sdb.dd5        525312      786943      130816   83  Linux
    data/input/sdb.dd6        788992     1048575      129792   83  Linux

The partitions begin at blocks 2048, 261632, 523264, 525312, and 788992.  However, since the 3rd partition (523264) is an extended partition, carving the disk from 523264 to 523264+262656 will result in a disk containing the remaining partitions, plus 4096 blocks of unallocated space.

Step 2. Parse the output of mmls <disk>, storing output in python variables.

For example:

    root@esio:~# mmls data/input/sdb.dd 
    DOS Partition Table
    Offset Sector: 0
    Units are in 512-byte sectors

         Slot    Start        End          Length       Description
    00:  Meta    0000000000   0000000000   0000000001   Primary Table (#0)
    01:  -----   0000000000   0000002047   0000002048   Unallocated
    02:  00:00   0000002048   0000261631   0000259584   Linux (0x83)
    03:  00:01   0000261632   0000523263   0000261632   Linux (0x83)
    04:  Meta    0000523264   0001048575   0000525312   DOS Extended (0x05)
    05:  Meta    0000523264   0000523264   0000000001   Extended Table (#1)
    06:  -----   0000523264   0000525311   0000002048   Unallocated
    07:  01:00   0000525312   0000786943   0000261632   Linux (0x83)
    08:  Meta    0000786944   0001048575   0000261632   DOS Extended (0x05)
    09:  Meta    0000786944   0000786944   0000000001   Extended Table (#2)
    10:  -----   0000786944   0000788991   0000002048   Unallocated
    11:  02:00   0000788992   0001048575   0000259584   Linux (0x83)


From this output:

    0000000000 - 0000000001 = MBR

        this is the record that states where each of the primary partitions 
        begin and end

    0000000000 - 0000002047 = unallocated space

        Unallocated space, dependent upon fdisk, gparted, ect.

    0000261632 - 0000523263 = Linux formatted (ext..ish) filesystem

        0x83 indicates that this partition is indeed an allocated filesystem

    0000523264 - 0001048575 = extended partition (entire)

        0x05 indicates that this is an extended partition, beginning at 
        beginning at 0000261632, ending at 0000523263.  However this does not
        indicate that the entire balance of this space is allocated

    0000523264 - 0000523265 = 1st extended boot record

        extended boot record, defining where the beginning and end of the first 
        extended partition is located, as well as the location of the second
        extended boot record

    0000523264 - 0000525311 = unallocated space

        Unallocated space, dependent upon fdisk, gparted, ect.

    0000525312 - 0000786943 = Linux formatted (ext..ish) filesystem

        0x83 indicates that this partition is indeed an allocated filesystem

    0000786944 - 0001048575 = 2nd extended partition (entire)

        0x05 indicates that this is an extended partition, beginning at 
        beginning at 0000786944, ending at 0001048575.  However this does not
        indicate that the entire balance of this space is allocated

    0000786944 - 0000786945 = 2nd extended boot record

        extended boot record, defining where the beginning and end of the first 
        extended partition is located, no additional extended boot record

    0000786944 - 0000788991 = unallocated space

        Unallocated space, dependent upon fdisk, gparted, ect.

    0000788992 - 0001048575 = Linux formated (ext..ish) filesystem

        0x83 indicates that this partition is indeed an allocated filesystem

Step 3: carve the partitions from the disk

Now, the disk may be carved into partitions using outputs of fdisk as follows:

    dd if=data/input/disks/sdb.dd of=data/output/sdb.dd1 bs=512 skip=2048 count=259584
    259584+0 records in
    259584+0 records out
    132907008 bytes (133 MB) copied, 2.61187 s, 50.9 MB/s


    dd if=data/input/disks/sdb.dd of=data/output/sdb.dd2 bs=512 skip=261632 count=261632
    261632+0 records in
    261632+0 records out
    133955584 bytes (134 MB) copied, 2.79361 s, 48.0 MB/s

    dd if=data/input/disks/sdb.dd of=data/output/sdb.dd3 bs=512 skip=525312 count=261632
    261632+0 records in
    261632+0 records out
    133955584 bytes (134 MB) copied, 2.42183 s, 55.3 MB/s

    dd if=data/input/disks/sdb.dd of=data/output/sdb.dd4 bs=512 skip=788992 count=259584
    259584+0 records in
    259584+0 records out
    132907008 bytes (133 MB) copied, 2.08099 s, 63.9 MB/s

The same may be accomplished using the output of mmls, however, the unallocated
is more clearly marked, giving more options.

===============================================================================
    Extended Scriptability
===============================================================================

Given that this project is undertaken by two people who speak entirely
different programming languages, there arises a need to be able to extend this
Python framework to include Perl, Bash, C/C++ and various other language
libraries.

The first script contributed to this library accepts a file system as its first
argument and attempts to reconstitute JPEG format files using magic number
patterns.  This method works well with image files smaller than 8M.  Larger
images require more massaging of data blocks.


===============================================================================
    File Recovery
===============================================================================

Given a filesystem, the output of fls provides the inode information that is 
necessary to recover NON-deleted files from the file system.

    root@esio:$ fls -r data/input/disks/sdb.dd1
    d/d 11: lost+found
    d/d 4065:   archive
    + r/r 4066: man.zip
    + r/r 4067: man.tar
    d/d 8129:   doc
    + r/r 8130: rit-State_of_the_Institute_2012_Final.docx
    + r/r 8131: DoD_timeline_transcript.doc
    d/d 12193:  exe
    + r/r 12194:    putty.exe
    + r/r 12195:    hello.c
    + r/r 12196:    hello.o
    + r/r 12197:    hello.sh
    d/d 28449:  img
    + r/r 28450:    img-zip.jpeg
    + r/r 28451:    img-tar.jpeg
    + r/r 28452:    img-pdf.jpeg
    + r/r 28453:    img-drew.png
    + r/r 28454:    img-rit.jpeg
    + r/r 28455:    img-earl.gif
    d/d 16257:  links
    + r/r 16258:    img-pdf.jpeg
    + l/l 16259:    hello.o
    + l/l 16260:    img-rit.jpeg
    + r/r 16261:    hello.sh
    d/d 18289:  pdf
    + r/r 18290:    extrasensory_perception_part01.pdf
    d/d 10161:  ppt
    + r/r 10162:    rit-STANYS_2010.pptx
    + r/r 10163:    rit-CMPseminar.ppt
    d/d 24385:  text
    + r/r 24386:    man-fsstat.txt
    + r/r 24387:    man-man.txt
    + r/r 24388:    man-ls.txt
    + r/r 24389:    man-fdisk.txt
    + r/r 24390:    man-grep.txt
    d/d 26417:  xls
    + r/r 26418:    rit-edu-System_Level_Specifications.xlsx
    + r/r 26419:    healthcare-gov-Insurance_data_for_me__Pinellas.xls
    d/d 14225:  deleted
    + r/r * 14226:  img-drew.png
    + r/r * 14227:  img-earl.gif
    + r/r * 14228:  img-pdf.jpeg
    + r/r * 14229:  img-rit.jpeg
    + r/r * 14230:  img-tar.jpeg
    + r/r * 14231:  img-zip.jpeg
    d/d 32513:  $OrphanFiles

Parsing the output of fls provides a list of inodes containing data blocks
which are either directly or indirectly allocated to the file.  The icat
utility can then be used to recover each of the files listed (those without
the asterisk).  For example, the following command recovers the file hello.sh:

    icat data/input/disks/sdb.dd1 16261

