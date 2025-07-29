# Wrong Optimizations


### Example 1: Constant Propagation Optimization and re-ordering optimization
```rust

//  ALl the Optimizations below are wrong... if raw pointers got involved.  

// using the aliasing rules, Rust will assume that x and y do not alias and thus, it
// can perform constant propagation. --- that would be wrong

// Under the same example, it can also decide to re-arrange the 2 instructions and there should be no problem (wrong also)
fn example1 (x: & mut i32 , y: & mut i32 ) -> i32 {
    * x = 42;
    *y = 13;
    return *x; // Has to read 42 , because x and y cannot alias !
}


// optimization 1
fn example1 (x: & mut i32 , y: & mut i32 ) -> i32 {
    * x = 42;
    *y = 13;
    return 42; // constant propagation avoids an entire memory read.
}


// Optimization 2
fn example1 (x: & mut i32 , y: & mut i32 ) -> i32 {
    
    *y = 13;
    * x = 42; // swapping of operations has happened, I don't know if that improves performance
    return *x; // Has to read 42 , because x and y cannot alias !

    // re-ordering with unsafe pointers would have caused the answer to be 13, can you see it?
}

```


If you wrote the code like this👇, then the answer would be 13 instead of 42: 
```Rust
fn main () {
    let mut local = 5;
    let raw_pointer = & mut local as * mut i32 ;
    let result = unsafe { example1 (& mut * raw_pointer , & mut * raw_pointer ) };
    println !( " {} " , result ); // Prints "13".
}
```

There are dozens of optimization mechanisms in a compiler... and new clever techniques get discovered from time to time.  
We do not know which optimizations affect which unsafe patterns in which way.  

I will focus on this page's sample from here on...  
I ain't covering the entire LLVM optimization.  