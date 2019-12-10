In the early days, there is no memory abstraction. Every program simply saw the physical memory. A program may have multiple threads, but it cannot have multiple unrelated programs running together. One way to run multiple programs is to swap the one in memory to disk and load another process into memory. IBM 360 segmented memory into 2-KB blocks each of which is assigned a protection key, with one in the PSW to indicate the active memory block. Only the OS could change the key. Absolute address in a program was modified to reflect the actual address when loading the program. However, this slowed down the loading, and sometimes it was difficult to determine whether an integer is an address.

_Protection_ and _relocation_ are the two major problems with running multiple programs in memory.

# Address Space

- _address space_: the set of addresses that a process can use to address memory

- _dynamic relocation_: a simple way is to use base and limit registers. A program are loaded into consecutive memory location. The base register is loaded with the physical address where its program begins in memory and the limit register is loaded with the length of the program. Every time a process reference memory, the CPU hardware automatically adds the base value to the address generated by the process before sending the address out on the memory bus. Only the OS can modify the base and limit registers. 8088 doesn't even have them to prevent out-of-range memory references.

However, not all programs can be stuffed into main memory. 

- _swapping_: consists of bringing in each process in its entirety, running it for a while, then putting it back on the disk. Swapping sometimes requires memory compaction to eliminate unused holes inside memory. A process's data/stack segment can grow. It's difficult to predict how much memory a process will need. If a process requires more memory than it has, the whole process must be moved to a hole large enough for it. If this cannot be done, and swapping back to disk is impossible, this process has to be suspended.

- _virtual memory_: allows programs to run even when they are only partially in main memory.

The OS must manage memory allocation:

- _bitmaps_: memory is divided into allocation units with each unit assigned a bit to indicate its allocation state. However, searching through bimaps to modified allocation states is slow.

- _free list_: a linked list of allocated and free memory segments, where a segment either contains a process or is an empty hole between two processes. First fit (the first hole that is big enough), next fit (search the list from the place where it left off last time instead of always at the beginning; worse than first fit), best fit (take the smallest appropriate hole among the entire list; resulting in more wasted memory than first/next fit), worst fit (take the largest available hole so that the new hole will be big enough to be useful; not very good either), quick fit (maintains separate lists for some of the more common sizes requested). Allocated segments and unallocated holes can be maintained in separate lists.

# Virtual Memory

The problem of programs larger than memory has been around since the beginning of computing. A solution in the '60s was to split programs into little pieces (overlays). The overlay manager loaded each overlay when it was needed. The overlays were kept on the disk and swapped in and out of memory by the overlay manager. The work of splitting the program had to be done manually by the programmer. The automatic version of this splitting method is known as _virtual memory_. The basic idea is that each program has its own address space, which is broken up into chunks called _pages_. 

- _page_: a contiguous range of addresses. These pages are mapped onto physical memory, but not all pages have to be in physical memory at the same time to run the program.

- _page frame_: the corresponding units in the physical memory.

Virtual memory is a generalization of the base-and-limit-register idea (with respect to address mapping). The entire address space is mapped onto physical memory in small units.

- virtual address: program-generated address.

When virtual memory is used, the virtual addresses do not go directly to the memory bus. Instead, they go to an MMU that maps the virtual addresses onto the physical memory addresses. A present/absent bit keeps track of which pages are physically present in memory. A page fault causes the OS to bring the unmapped page into physical memory. The original page that uses this frame, if any, is set to absent and the faulting page is set to present.

## Page Table

The virtual address is split into a virtual page number and an offset. The purpose of the page table is to map virtual pages onto page frames.

```
            Caching
            disabled    Modified          Present/Absent
+-----------|-----|-----|-----|----------|-----|-----------------------------+
| || || |-| |     |     |     |          |     |                             |
| || || |-| |     |     |     |Protection|     |       Page Frame Number     |
| || || |-| |     |     |     |          |     |                             |
+-----------|-----|-----|-----|----------|-----|-----------------------------+
                  Referenced
```

- protection bits: what kinds of access are permitted

- modified bit/dirty bit: of value when the OS decides to reclaim a frame

- referenced bit: helps the OS choose a page to evict when a page fault occurs. Pages that are not being used are far better candidates than pages that are.

- caching bit: important for pages that map onto device registers rather than memory.

Virtual memory fundamentally does is create a new abstraction - the address space which is an abstraction of physical memory, just as a process is an abstraction of the physical processor.

The major issue in any paging system is

1. the mapping must be fast;

In widely implemented schemes for speeding up paging and handling large virtual address spaces, the page table is in memory. Most programs tend to make a large number of references to a small number of pages. Computers are equipped with a small hardware device for mapping virtual address to physical addresses without going through the page table - _translation lookaside buffer_, also called an _associative memory_, which is usually inside the MMU and consists of a small number of entries, rarely more than 256. When the virtual page number is not in the TLB. The MMU detects the miss and does an ordinary page table lookup. It then evicts one of the entries from the TLB and replaces it with the page table just looked up.

Many RISC machines do nearly all of page management in software. The TLB entries are explicitly loaded by the OS. When a TLB miss occurs, a TLB fault causes the OS to buffer the page entry in the TLB. If the TLB is moderately large to reduce the miss rate, software management of the TLB turns out to be acceptably efficient. The OS can use some prediction methods to cache related pages. However, the page that holding the page table may not be in the TLB, causing additional TLB faults. The OS can maintain a large software cache of TLB entries in a fixed location whose page is always kept in the TLB (multi-level?). 

A _soft miss_ occurs when the page referenced is not in the TLB. A _hard miss_ occurs when the page itself is not in memory. Looking up the mapping in the page table hierarchy is known as a _page table walk_. Some misses are slightly softer than other misses. _Minor page fault_ is that the page is not in the page table the current process, but brought into memory by another process. A major page fault occurs if the page needs to be brought in from disk. The virtual address may not map to any storage, resulting in segmentation fault.

2. if the virtual address space is large, the page table will be large. And EACH process needs its own page table.

_multilevel page table_: avoid keeping all the page tables in memory all the time. The unused entries of the top level  or intermediate levels can be marked as absent, saving a lot of space for low-level page tables. Not each process need to have a full multi-level page table. Intel 386 uses a two-level page table (10-10-12). Pentium Pro extended each entry in each level of the page table from 32 bits to 64 bits. AMD64 uses a four-level page table (9-9-9-9-12).

_inverted page tables_ (Itanium): one entry per page frame in real memory. The entry keeps track of which (process, virutal page) is located in the page frame. It's hard to perform virtual-to-physical translation. TLB and hashing are used.

# Page Replacement Algorithms

TODO

# 
