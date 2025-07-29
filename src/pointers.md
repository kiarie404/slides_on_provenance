## src
- core::ptr module [docs](https://doc.rust-lang.org/core/ptr/index.html)
- primitive Type pointer [docs](https://doc.rust-lang.org/core/primitive.pointer.html)

## soft src
- To mess around with pointers, read the book "[Learn Rust With Entirely Too Many Linked Lists](https://rust-unofficial.github.io/too-many-lists/)" by [Aria Desires](https://faultlore.com/blah/)  


## Intro to 'pointers'
Pointers are variables that store memory addresses. I know I have just over-simplified it. Kindly, just let me say things without consequences, we are all devs here. Please, I just want to teach the intuitive part, not the technical part.    

There are many kinds of pointers: smart-pointers, raw-pointers, reference-pointers... others that the author doesn't know yet...  


1. **Raw-pointers** in Rust are denoted with the symbols `*const T` and `*mut T`. These are pointers that are so bare that they literally store an integer.  
2. **Reference-pointers** are denoted by `&mut T` and `& T`. My assumption is that you already know what they are and how to use them.
3. **Smart Pointers** are pointers that don't just store addresses, they have metadata and extra control mechanisms. You can read about smart pointers somewhere else, sorry.  



### Problems introduced when working with pointers  

The underlying superpower of raw-pointers over reference-pointers is that they can point to **ANY** memory address. You are free to access any memory address within your program's address space. That is their super-power.  

This consequently means that the pointer gains 3 caveats that references don't have to put up:  

1. raw-pointers can point to memory that is out-of-bounds (unlike reference-pointers).  
2. raw-pointers can point to unaligned addresses (unlike reference pointers)
3. raw-pointers can point to null

These 3 statements are worth explaining. You can skip the next section if they already make sense to you.  

### Caveat 1: Pointing to unaligned memory

### Caveat 2: Pointing to null

### Caveat 3: Pointing to Out-of-bounds address


### Explanation of the great 3 loopholes

### How to create raw-pointers
Here are some of the ways to make raw-pointers:  

1. **Making raw pointers from references.**  
This is the safest way to create pointers because you are assured that the resultant pointer points to a ***valid*** memory address. You can misuse the pointer after this, but at least you'll be sure of its validity at the start of its sorry lifetime.  

`&/&mut` is only allowed if the pointer is properly aligned and points to initialized data.
```rust  
fn main(){

    let numbers : Vec<String> = ["one".to_string(), "two".to_string()];
    let numbers_mut_pointer = &mut numbers as *mut Vec<String>; // create *mut T pointer
    let numbers_const_pointer = &numbers as *const Vec<String>; // create *const T pointer
}
```

1. Using `core::ptr::addr_of!` and `core::ptr::addr_of_mut!` (Avoids Reference Creation)


rustc does not and cannot track the provenance of raw pointers (*const T/*mut T) at compile time. T
heheheh :  between unsafe-code-guidelines, miri and stacked borrows, tree-borrows are all we got
bad mixing ref & ptrs


stable polyfill
Strict provenace
exposed provenance

feature gates eg : #![feature(strict_provenance)]

"This is purely a set of library APIs to make your code more clear/reliable, so that we can better understand what Rust code is actually trying to do and what it actually needs help with. It is overwhelmingly framed as a memory model because we are doing a bit of Roleplay here. We are roleplaying that this is a real memory model and seeing what code doesn't conform to it already. Then we are seeing how trivial it is to make that code "conform".

This cannot and will not "break your code" because the lang and compiler teams are wholy uninvolved with this. Your code cannot be "run under strict provenance" because there isn't a compiler flag for "enabling" it. Although it would be nice to have a lint to make it easier to quickly migrate code that wants to play along." - from the strict provenance tracking issue


Strict Provenance is an experimental set of APIs that help tools that try to validate the memory-safety of your program’s execution. Notably this includes Miri and CHERI which can detect when you access out of bounds memory or otherwise violate Rust’s memory model.  

but specifying a model for provenance in a way that is useful to both compilers and programmers is an ongoing challenge

Strict provenance : "what if we just said you couldn’t do all the nasty operations that make provenance so messy?" 
- what APIs to remove?
- what APIs to introduce?
- How much would code have to change?
- Would any patterns become truly inexpressible?
- Could we carve out special exceptions for those patterns? 


Pointers are not simply an “integer” or “address”. We got to seperate them.  

When an allocation is created, that allocation has a unique Original Pointer. For alloc APIs this is literally the pointer the call returns, and for local variables and statics, this is the name of the variable/static. 

How is it a sandbox?

Spatial provenance makes sure you don’t go beyond your sandbox, while temporal provenance makes sure that you can’t “get lucky” after your permission to access some memory has been revoked (either through deallocations or borrows expiring).  

Demo this: "Provenance is implicitly shared with all pointers transitively derived from The Original Pointer through operations like offset, borrowing, and pointer casts. Some operations may shrink the derived provenance, limiting how much memory it can access or how long it’s valid for (i.e. borrowing a subfield and subslicing)."  

Demo this and explain why it is wrong: "Shrinking provenance cannot be undone: even if you “know” there is a larger allocation, you can’t derive a pointer with a larger provenance. Similarly, you cannot “recombine” two contiguous provenances back into one (i.e. with a fn merge(&[T], &[T]) -> &[T])."  

If an allocation is deallocated, all pointers with provenance to that allocation become invalidated, and effectively lose their provenance.  

Stacked Borrows is necessary to properly describe what borrows are allowed to do and when they become invalidated.  
The strict provenance experiment is mostly only interested in exploring stricter spatial provenance. In this sense it can be thought of as a subset of the more ambitious and formal Stacked Borrows research project. This necessarily involves much more complex temporal reasoning than simply identifying allocations.  

The issue of:
  - pointers in segmented architecture
  - pointers in Havard architecture (wasm)


Situations where a valid pointer must be created from just an address, such as baremetal code accessing a memory-mapped interface at a fixed address, are an open question on how to support. These situations will still be allowed, but we might require some kind of “I know what I’m doing” annotation to explain the situation to the compiler. It’s also possible they need no special attention at all, because they’re generally accessing memory outside the scope of “the abstract machine”, or already using “I know what I’m doing” annotations like “volatile


explain each : 
Under Strict Provenance is is Undefined Behaviour to:

    Access memory through a pointer that does not have provenance over that memory.

    offset a pointer to or from an address it doesn’t have provenance over. This means it’s always UB to offset a pointer derived from something deallocated, even if the offset is 0. Note that a pointer “one past the end” of its provenance is not actually outside its provenance, it just has 0 bytes it can load/store.

But it is still sound to:

    Create an invalid pointer from just an address (see ptr::invalid). This can be used for sentinel values like null or to represent a tagged pointer that will never be dereferencable. In general, it is always sound for an integer to pretend to be a pointer “for fun” as long as you don’t use operations on it which require it to be valid (offset, read, write, etc).

    Forge an allocation of size zero at any sufficiently aligned non-null address. i.e. the usual “ZSTs are fake, do what you want” rules apply but this only applies for actual forgery (integers cast to pointers). If you borrow some struct’s field that happens to be zero-sized, the resulting pointer will have provenance tied to that allocation and it will still get invalidated if the allocation gets deallocated. In the future we may introduce an API to make such a forged allocation explicit.

    wrapping_offset a pointer outside its provenance. This includes invalid pointers which have “no” provenance. Unfortunately there may be practical limits on this for a particular platform, and it’s an open question as to how to specify this (if at all). Notably, CHERI relies on a compression scheme that can’t handle a pointer getting offset “too far” out of bounds. If this happens, the address returned by addr will be the value you expect, but the provenance will get invalidated and using it to read/write will fault. The details of this are architecture-specific and based on alignment, but the buffer on either side of the pointer’s range is pretty generous (think kilobytes, not bytes).

    Compare arbitrary pointers by address. Addresses are just integers and so there is always a coherent answer, even if the pointers are invalid or from different address-spaces/provenances. Of course, comparing addresses from different address-spaces is generally going to be meaningless, but so is comparing Kilograms to Meters, and Rust doesn’t prevent that either. Similarly, if you get “lucky” and notice that a pointer one-past-the-end is the “same” address as the start of an unrelated allocation, anything you do with that fact is probably going to be gibberish. The scope of that gibberish is kept under control by the fact that the two pointers still aren’t allowed to access the other’s allocation (bytes), because they still have different provenance.

    Perform pointer tagging tricks. This falls out of wrapping_offset but is worth mentioning in more detail because of the limitations of CHERI. Low-bit tagging is very robust, and often doesn’t even go out of bounds because types ensure size >= align (and over-aligning actually gives CHERI more flexibility). Anything more complex than this rapidly enters “extremely platform-specific” territory as certain things may or may not be allowed based on specific supported operations. For instance, ARM explicitly supports high-bit tagging, and so CHERI on ARM inherits that and should support it.



disclaimers:
- not good at FFI
- not conversant with CHERi
- not clean on Zero sized types
- not clean on un-initialized types
- not clean on exposed provenace
- not clean on Tree-borrows modifications on Miri
- not clean on stack-borrows modifications on Miri
