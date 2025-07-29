# Strict Provenance

This API provides the following solutions:  
1. It gives the dev **explicit** functions to cast between pointers and integers. You no longer have to do manual casting. This is important because these istructions explicitly indicate whether the casting preserves provenance or not. Tools like Miri can then recompile such code with ease and track when provenance was lost or regained.  
2. It provides a conceptual way of tracking and storing provenance of the pointers like a namespace (this has not been implemented in rustc but in tools like Miri)

Here is the API:  

1. `ptr::addr()` : extracts the address of a pointer without its provenance.
```rust
   let ptr: *const u8 = 15;  
   let addr = ptr.addr(); // Just the numeric address  

   // instead of using manual cast
```

2. `ptr::with_addr()` creates a new pointer with a given address but retains the provenance of the original pointer.
3. ptr::map_addr() : a sugar coat for `ptr::with_addr()` 


# Exposed Provenance  

There is a database stores exposed provenances and associated pointers. It is not a strict database. (Future me, just explain this verbally, I can't write now, pretty please)


> there is no algorithm that decides which provenance will be used. You can think of this as “guessing” the right provenance, and the guess will be “maximally in your favor”, in the sense that if there is any way to avoid undefined behavior, then that is the guess that will be taken  




