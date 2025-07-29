# Bad news  

So Rust people came up with the idea of provenance.  
Where, we very allocation made through `alloc` is given a special ID.  

When an allocation is created, that allocation has a unique Original Pointer. For `alloc` APIs this is literally the pointer the call returns, and for local variables and statics, this is the name of the variable/static.  

You can derive new pointers and references through borrowing or casting. 

Every pointer derived from that allocation inherits provenance from the direct parent. They either get the same provenance, or less-provenance ie Borrowing immutably from an origial pointer will make sure the new pointer gets read-access instead of full-read-write access. 

Like a sandbox.  
> Some operations may shrink the permissions of the derived provenance, limiting how much memory it can access or how long it’s valid for (i.e. borrowing a subfield and subslicing can shrink the spatial component of provenance, and all borrowing can shrink the temporal component of provenance). However, no operation can ever grow the permissions of the derived provenance: even if you “know” there is a larger allocation, you can’t derive a pointer with a larger provenance. Similarly, you cannot “recombine” two contiguous provenances back into one (i.e. with a fn merge(&[T], &[T]) -> &[T])

There is no escalation.  

So they came up with 2 new APIs : [`Strict Provenance API`](https://doc.rust-lang.org/stable/std/ptr/index.html#strict-provenance) and [`Exposed Provenance API`](https://doc.rust-lang.org/stable/std/ptr/index.html#exposed-provenance)

## The Bad news

People tried to make the tracking static. But 2 facts remained:
1. **Pointers in the real world are too dynamic.** Raw pointers can come from anywhere (*eg* FFI, memory-mapped I/O), and their behavior may depend on runtime conditions that may be too unique for a static analyzer to analyse. Static analysis is not comprehensive enough to account for these arbitrary runtime situationships.  

2. **Compiler Optimizations are also many (and increasing)**  

So far, the Rust compiler cannot compile with Provenance-checks in a **STATIC** manner. However, the rust people came up with  unsafe-code-guidelines, Lints, [miri](https://github.com/rust-lang/miri?tab=readme-ov-file) and [stacked borrows](https://plv.mpi-sws.org/rustbelt/stacked-borrows/), [tree-borrows](https://perso.crans.org/vanille/treebor/)  


## So what are these APIs?  

Here we go...