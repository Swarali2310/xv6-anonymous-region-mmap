# xv6-anonymous-region-mmap
Add anonymous mmap support to xv6

##  A memory allocator for the Kernel - kmalloc

### Problem

To build a memory allocator for the kernel in xv6.
Currently xv6 has only the user memory allocator, but Kernel cannot use it as memory returned by the user space allocator is freed when the process terminates.
Kernel requires memory that is stable beyond the lifetime of a process.

### Solution

Implement 2 system calls viz. kmalloc() and kmfree() for the kernel memory allocation and de-allocation.
In the kmalloc call, we use kalloc() instead of growproc() to allocate memory. 
At a time, kalloc() can only allocate a page ( = 4096 bytes). 
We panic the kernel in case kmalloc is called with size greater than page size. 
In the size parameter of the header, we allocate header sized chunks of the page allocated by kalloc() which turns out to be 4096/sizeof(Header) : as in every case, kalloc returns a single page of size 4096, unless its out of memory.
The other changes are wrapper functions to make kmalloc and kmfree callable from user programs.


## Anonymous memory map and unmap

### Problem

To implement mmap - create a new mapping in the calling process's address space.
To implement munmap - Unmap the region mapped through the mmap system call

### Solution

#### Infrastructure

We create a mmap_region linked list structure, and also add a nregions variable to keep the track of number of regions mapped by a process.

#### Mmap

• Based on the hint address, if it is greater than current size of the process, we allocate length sized memory to process, if it is less than the current virtual size of process, we try to find if there are any free areas in the process space by the munmap calls.

• Based on the minimum absolute distance, we pick the address for reuse.

• Populate the mmap linked list structure.

• If there is already a page mapped at the address specified by user, we do conflict resolution by finding the next page, depending on the length/ PGSIZE, whichever is smaller.

• Once we successfully find such an address, we add the node to the linked list and return the address or else -1 on error.

• Before erroring out with deallocate the memory allocated by allocuvm, and also by kmalloc - using kmfree.

#### Munmap

• We validate the address and length passed in the call.

• We walk the mapped_regions linked list structure to find the address and the length corresponding to the mmap call. Once found, we deallocate the mapped region using deallocuvm and remove the node from the linked list.

• In case the node is not found while traversing the list, we return an error.
