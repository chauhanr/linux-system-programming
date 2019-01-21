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



