[<< I want total control over vectors and lists](./problem_31.md) | [**Home**](../README.md) 

Problem 32: A fixed-size object allocator
**2017-11-29**

A custom allocator can be fast.
**Fixed size allocator:** all allocated "chunks" are the same size (ie. customaized code for one class) - no need to keep track of sizes

(aside - many many traditional allocators store the size of the block before the pointer so that the allocator knows how much space is allocated to that pointer.)

**Fixed size:**
- Saves space (no hidden size field)
- Saves time (no hunting for a block of the right size)

**Approach:** create a pool of memory  - an array large enough to hold `n` `T` objects.