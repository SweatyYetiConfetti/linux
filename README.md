# Learning the Linux Kernel

## Basic Introduction Notes

### Source Tree

<ins>**arch**</ins><br />
This dir is for architecture specific functionality.<br /><br />

The notes in this repo will be mainly concered with the x86 and riscv instruction sets.<br /><br />

Both AMD and Intel use the x86 instruction set making it the most widely used instruction set in personal computers.<br /><br />

This dir has a Kconfig file used to specify configuration for the systems architecute dependent options.<br /> <br />

Inside each subdir specifying a specific architecture you will find a Makefile and a Kconfig file that will specify that architectures specific configuration and make process.<br /><br />


<ins>**block**</ins><br />
This dir is for block devices (block special file) and their associated functionality.<br /><br />

What is a block device? A block device is a file that provides an abstracted access to a hardware device (hard drive, usb,...etc) that allows reading and writing data in blocks of specified size.<br /><br />

Block devices allow themselves to be mounted by the OS and are ideal for random and sequential data access by addressing each block individually.<br /><br />

Block devices make use of system buffering for data transfers and this allows a reduction in the number of read/write operations leading to a performance improvement.<br /><br />

Compatible with structured file systems and provide wear leveling to reduce overuse of certain regions of the block device improving longevity. <br /><br />


Error checking and correction algorithms can also be used to improve data integrity, transaction accuracy and stability of systems depending on block devices.<br /><br />

Block devices support booting an OS directly from them.<br /><br />


<ins>certs</ins>
This subdir is

<ins>crypto</ins>
This subdir is

<ins>Documentation</ins>
This subdir is

<ins>drivers</ins>
This subdir is

<ins>fs</ins>
This subdir is

<ins>include</ins>
This subdir is
<ins>init</ins>
This subdir is
<ins>io_uring</ins>
This subdir is
<ins>ipc</ins>
This subdir is
<ins>Kbuild</ins>
This subdir is
<ins>Kconfig</ins>
This subdir is
<ins>kernel</ins>
This subdir is
<ins>lib</ins>
This subdir is
<ins>LICENSES</ins>
This subdir is
<ins>MAINTAINERS</ins>
This subdir is
<ins>Makefile</ins>
This subdir is
<ins>mm</ins>
This subdir is
<ins>net</ins>
This subdir is
<ins>README</ins>
This subdir is
<ins>rust</ins>
This subdir is
<ins>samples</ins>
This subdir is
<ins>scripts</ins>
This subdir is
<ins>security</ins>
This subdir is
<ins>sound</ins>
This subdir is
<ins>tools</ins>
This subdir is
<ins>usr</ins>
This subdir is
<ins>virt</ins>
This subdir is

