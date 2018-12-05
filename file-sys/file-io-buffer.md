# File IO Buffering

As write operations are slow when compared to reads because of the disk. A technique of storing the data in a cache that will later be written to the disk is called buffering. There are times when the buffering can be let go and technique called direct io can help. 

## Buffer Cache 
In order to make the read and write operations in the Kernel faster Linux has a datastructure called buffer cache where data is either written to temporarily or read from disk (read ahead) so that the I/O are not slow.
* In the case of write the kernel will write the data to cache and write method will return immediately but the data written to the cache will be written at once. Similarly in read operations too the data can be stored in the buffer cache and in order to keep data in cache read ahead operations are performed. 

Benchmarks do show that the larger the buffer cache the better the performance but we need to make a balance. : 
