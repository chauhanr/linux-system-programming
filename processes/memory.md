# Memory Allocation 

## Allocating memory to the heap 
Current limit to the heap size is called the program break inorder to increase the size of heap we need to use the brk and sbrk system calls 

```
#include <unistd.h> 

int brk(void *end_data_segment); 
/*  The brk function will move the current program break to the location specified by the end_data_segment 
    position. since the virtual memory is calculated in page units the end_data_segment will be rounded off 
    to the next available page unit. 
*/

int sbrk(intptr_t increment); 
/* sbrk on the other hand adds segments based on the increments we send it. So sbrk(0) will return the current
   program break location without changing it. This is useful to track the size of the heap. 
*/
```

## malloc() and free()
These are the c languages methods to allocation and de allocate memory on the heap. 

```
void *malloc(size_t size); 
/* returns a pointer to the allocated memory on heap. If malloc() cannot allocated memory it returns a NULL
   and errno is set to a non zero value. 
*/

void free(void *ptr); 
/*  free function deallocates the block of memory which is represented by the pointer. The free function does 
    not lower the program break, but instead adds the block of memory to a list of free blocks that are 
    recycled when malloc() is called. 
    Use of the ptr passed to free can be done but make lead to unexpected results so should be avoided.
*/
```

## Other methods allocation of memory.

* **calloc()** - allocates memory for an array of identical items. It takes 2 arguments the number of items and the size fo each of the item.
* **realloc()** - function resizes a previous block of memory that was allocated using the malloc(). The input to the method are the ptr to the previous memory location and new size of the memory block. 
* **alloca()** - this method assigns memory on the stack and therefore is used to increase the size of the stack frame. As the memory allocated by alloca() is on the stack the memory is reclaimed by the kernel when the functiona that called the alloca returns. 

