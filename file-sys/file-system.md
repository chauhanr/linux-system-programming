# File Systems 

## Device files 
Every device on the Linux like the mouse, disk etc has to be represented as a file and this file is called a device file. The location of such a file is /dev folder. THe device files are managed by programs called device drivers. 
Device drivers are program that are unit of kernel code that implements a uniform interface that allow for the following functions: 
* open()
* close()
* write()
* read()
* mmap()
* ioctl()

Because this interface is consistant the devices can be controlled in similar fashion. 

### Devices are of two types 
* character device - these are devices that handle characters and example being the terminal devices. 
* block device - deal with data input and output on blocks e.g. disk devices. 


## Disks and Partitions 
Disk drives more often than not mean the hard disk that supports the entire system. Each disk is generaly divided into partitions. Each partition that is created on the disk drive is treated as a separate device on the linux system and can be seen under the /dev folder. 

The system admins use the fdisk command to create such partitions on the disk. Each disk partition consists of 3 areas: 
* a file system which holds regular files and directories. 
* data area - accesses as raw mode device 
* swap area - area on the partition that is used by the kernel to do memory management. 

## File Systems 
file system is nothing but an organised collection of regular files and directories and the file system can be cerated using mkfs command. Linux supports a lot of file system types. 
* ext2 - extendend file system 2 
* minix, system v and BSD fs 
* microsoft's  FAT, FAT32 and NTFS
* ISO 9660 
* Apple Macintosh HFS 
* range of network file systems like NFS, Novell's NCP, Coda file system etc. 
* journalling fs like ext3, ext4 , btrfs, xfs, jfs etc. 


The following diagram shows how the file system and partition look together 

![fs-partitions](images/file-system-partitions.png)

From the diagram above we have the following blocks on each parition: 
* boot block - this is always the first block on a partition and is never used by the file system it is for the kernel. Although kernel does not read from all partions but just one. So a lot of the boot block is not used. 
* superblock - This block has meta data about the file system and configurations. 
	* the size of the i-node table 
	* size of the logical block of the file system. 
	* the size of the file system in logical blocks. 
* i-node table - inode table is the place where information of all the files and directory in the system are stored. the inode table is also sometimes called the i-list. 
* data blocks - this is where the raw files and directory data lies. the structure of ext2 is a little different because on ext2 super, inode table and data blocks are all part of a block group which is made to reduce seek times. 


