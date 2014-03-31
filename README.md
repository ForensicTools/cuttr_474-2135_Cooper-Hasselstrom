cuttr_474-2135_Cooper-Hasselstrom
=================================

Extensible Forensic Framework

License: Private, do not disclose (TBD)

Tool Name: cuttr

Tool description: The tool that we will create will be an extensible forensic framework that may be extended through Python scripts or modules, Perl scripts or modules, C/C++ and Bash scripts (among others).  Some of the initial built-in functions include:

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
