---
layout: page
---
# Need For Custom Allocators

- avoid fragmentation
- 1 => better use of memory
- Avoid book keeping 
- make better deallocations
- make allocated memory volatile for multi threaded application


## Avoid Fragmentation

 say we use new for all allocations, then we allocate a vectorint of size n
 then we allocate a set and insert some elements
 then allocate a map then insert some more than elements
 then insert some more set elements and map elements.
 and say set goes out of scope then now our memory looks very fragmented.
 so we can use a pool allocater which is an allocater for objects of fixed size
 this would greatly reduce fragmentation
end


## Better Use of Memory 


- Less fragmentation => better use of memory


## Avoid book Keeping


say when we are deallocating an object we don't specify its size 
so the allocater is storing this on heap for most part

- not totally sure as it may be something we could figure out at compile time or may be use __vptr to find out at runtime 
- but not totally sure on this one but mostly true for array's but that won't be solved using a pool allocater
end


## Make better deallocations 

- say we know we have setunique_ptrA* that goes out of scope. it calls deallocate for every ptr.. 
- we can just note that and deallocate all of them at once when its done


## Making them Voloatile 

making sure that these are volatile.. i guess don't know enough

Links

://drdobbs.com/cpp/improving-performance-with-custom-pool-a/184406243}

An Estimate of the Store Size Necessary for ␍
Dynamic Storage Allocation 

https://dl.acm.org/doi/pdf/10.1145/321650.321658 **didn't read yet**

https://stackoverflow.com/questions/826569/compelling-examples-of-custom-c-allocators **didn't read yet**


## Some Allocators[some-allocators.md](#some-allocatorsmd)

