# File Attributes 

File metadata that the linux operating system keeps for all files in the OS. The commands that allow
for this infromation to be retried are: 
* stat() - returns information about a named file 
* lstat() - this is used to return information about a link to a file not the file itself
* fstat() - return information of a file which is specified by file descriptor. 

All the 3 system calls above return a stat structure that returns the following set of information: 

* **Device id and inode number** defined the device on which the file recides (file system) and the
  inode number of the file. combination of this uniquely identifies the file across the Linux OS. 
* **file ownership** - the user and group id that the file belongs too. 
* **Link Count** - the number of hard links that are created and point to the file. 
* **file type and permission** - the st_mode attribute holds both the file type and file
  permissions. the file types can be either of
  	* regular 
	* directory 
	* character device 
	* FIFO or pipe 
	* socket 
	* symbolic link 
  * **file size, blocks, allocated, and optional I/O block size** 
  	* file size in the case of a regular file is the size of the file in bytes where as in the
	  case of symbolic links the file size is the length of the path name
	* blocks - this specifies the number of blocks thae file holds in 512 byte blocks. 
	* optimal block size - there is field that specifies the optimal file size for i/o on the
	  file system. 
	* file time stamps - these specify the last time the file was accessed, last file
	  modification and last file status change (last time the inode infromation was changed).

## File Timestamps 
based on the command that run on a file there various time stamps can be changed. e.g. 
	* mkdir - changes access, modification and status change times for a file 
	* mkfifo - also has the same changes as a mkdir 
	* open, close()- same as the earlier two.
	* read() - system call will only change the access time. 
	* removeattr - will only change the status time and not the access or modification times. 

#### Change file timestamps using utime() and utimes()
the last access and modification time stamps can be changed using the _utime()_ or _utimes()_ the
utime method takes the utimebuf which specifies the time since epoc when the file owas accessed. 
* if the utime is called with null then the current time is sent to the program 
* if the utime get its value then that is set at the time the file was accessed. 

The only different between utime and utimes is that with utimes you can specify the exact time in
milli seconds accuracy. 

The utimenstat and futimens() method also are similar but they allow for setting the time is nano
second precision. 


## File Ownership
When a file us created it needs to be set to have an owner and group that it belongs too. These are
specified by the files user and group id. However files user id and groups can be changed. The
commands that do this are: 

* chown - this takes the path of the file in questions and changes the user and group to the one
  specified. 
* lchown - this takes input as the link to the file and changes the ownership and group 
* fchown - this too changes ownership and group of a file but takes a file descriptor as input 

_only a previledged user can use the chown commands to change the user id of the file. An
unpreviledged process can just change the group of the file from its group to a group that it is a
part of only_ 


## File Permission 
One the file stat structure the st_mode field is made of 15 bits and the last 12 is what we call the
file permission bits. The following table respresents the constants of permission 

| Constants | Octal values | Permission bits | 
| ----------|:------------:| ----------------|
| S_ISUID   | 04000        | Set-user-ID     | 
| S_ISGID   | 02000        | Set-group-ID    | 
| S_ISVTX   | 01000        | Sticky          | 
| ----------| -------------| ----------------| 
| S_IRUSR   | 0400         | User-read       | 
| S_IWUSR   | 0200         | User-Write      | 
| S_IXUSR   | 0100         | User-execute    | 
| ----------| -------------| ----------------| 
| S_IRGRP   | 040          | Group-read      |
| S_IWGRP   | 020          | Group-write     | 
| S_IXGRP   | 010          | Group-execute   | 
| ----------| -------------| ----------------| 
| S_IROTH   | 04           | Other-read      | 
| S_IWOTH   | 02           | Other-write     | 
| S_IXWOTH  | 01           | Other-execute   | 
| ----------| -------------| ----------------| 

#### Permission on Directories 
The read, write and execute permissions on the directory are interpreted a little differently. 
* read - the contents of the directories (file and other dir) can be listed using the ls command. 
* write - the permission allows for the files in the directory to be created or removed. 
* execute - file within the directory may be accessed. execute permission is sometimes called search
  permission also. 

Some other points to remember: 
* when accessing a file execute permission is required at all level of the path. so to list /home/mtk/x
  we need read permission on home and mkt. 
* read permission allows only the file to be listed using the ls command but if the execute
  permission is not set then the file inode will not be accessible. 
* if the execute permission is set but read is not then we can access the file using the entire path but will not be able
  to list is using ls command. 
* to add and remove files in a directory we need execute and write permissions both.


#### set-user-ID, set-group-ID, sticky bit 
The set user id and set group id bits are used to create previledged programs. set-group-id also
helps in controlling group ownership of new files and enable mandatory locking of file. 

The sticky bit has some historical significance. During the earlier versions of UNIX for system
files that were run frequently this sticky flags were set for file to be kept in swap space so that
the file could be loaded faster in consequent uses (this use is now obsolete as the memory
management has become better. 

The sticky bit is still used but it uses is more on the directory. When this bit is set on the
directory it stands for restrictive deletion where an unpreviledged user can only modify the file
that he/she has access to or has created. This is a setting that has been done on the tmp folder in
Linux. 

The sticky bit can be set using the chmod command i.e.  chmod +t.

#### Process file mode creation mask (umask) 
When a new file or directory is created the premissions are carried from the mode that is passed to
the command or the premission the process has based on the shell or process that spawns it. This
however is changed is the file creation mode called umask is set. 
The umask defines the permission bit that must be switched off when the files or directories are
craeted by the process. The umask is a process level attribute. 

so for example of the umask is set at 022 (----w--w-) will mask the permission on the file to ensure
that the group and other users do not have write permissions. 


