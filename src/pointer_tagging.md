Pointer tagging is a technique where you store metadata in the unused bits of a pointer in order to save memory.  

Pointers are typically aligned (e.g 8-byte alignment on 64-bit systems), their lower bits are always zero ; so we can hijack them to store data. When a pointer is aligned by 8, it will always have at least 3 zeroes at its tail.  

Let us see that in Rust: 

>We will try as much as possible to avoid using primitive types or collections or smart pointers in our examples in order to keep things simple and without 'tricks'.  eg Collections come with hundreds of custom functions. Primitives implement the `Copy` trait and thus will make it hard to demonstrate borrow-checking in action.  
```rust
struct Point{
    x: f64
    y: f64
}


fn main(){
    let origin = 
}
```