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

## inodes 
On a Linux file system each file is denoted by a data structure called inode at the kernel level this data structure has the following information that it keeps: 
* file type (directory, regular files, fifo, character device or symbolic link) 
* Owner of the file (UID) 
* Group that the file belongs too (GID) 
* Access permissions for the three categories of user, group and others. 
* 3 time stamps about the files: 
	* last access time. 
	* last time the file was modified
	* time of the last status change. 
* Number of hard links to the file. 
* Size of the file in bytes 
* Number of blocks allocated to the file measured in units of 512 byte size. 
* Pointers to the data block. 

### inodes and data pointers on the ext2 system. 
Every file in Linux file system keeps track of the data blocks that are assigned to the file by keeping a list of pointers to the data blocks that are assgined to the file. 
It is important to note the that data on the files are never stored in contiguous memory locations as it is highly inefficient to do so in practice. Therefore blocks of data are assigned in incontiguous memory locations (buddy algorithm for assignment is used). This may lead to fragmentation of memory on the disk but this is very efficient in space usage. 

![ext2-fs-inode](images/inode-struct.png)

As you can see in the diagram above there are 15 pointers that the inode keeps for holding the pointers to the blocks of data. The first 12 pointers on the structure are directly to the first 12 blocks of the file. the 13th location on the inode data block poitns is a pointer to another set of pointers to data blocks that makes for the first level of indirection. The number of data blocks that the file system uses determines the number of datablocks that the 13th position can save e.g. if block size of ext2 is 1024 then the 13th position pointer can hold upto 256 blocks of memory and 1024 blocks in the case the size is 4096 bytes. 

If the file is still larger for the pointers in the 13th position then the 14th position pointers has even more locations to store the data as it is a double indirection pointers meaning it holds a pointer to pointer of blocks that hold pointers to the actual data block. If we still have a larger file on the system then the 15th location is used which is a 3 level of indirection. 

In a sense if the block size on the file system is 4096 then the largest file that ext2 system an hold is 4 Tera bytes. 


## Virtual File System (VFS) 
As the Linux operating system has been built to support multiple file sytems. To do this Linux has built a Virtual File system (VFS) interface which it enforces on to each file system that needs to work with Linux. Therefore the device drivers that actually interact with the file system at a lower level build primitives that the VFS exposes to applications that run on the Linux OS. There are several operations that the VFS supports that the file system device driver needs to support as well e.g. write(), read(), truncate(), mount(), unmount(), mmap(), link(), unlink(), symlink(), rename(), lseek(), close() etc. 

However not all file system support all these operations e.g. on Microsoft's VFAT file system there is no support to create symbolic links therefore the symlink() call to the device driver of VFAT system on Linux returns an error. 

![vfs](images/vfs-support.png)
