# So Undeterminism

This is a Myth: 
>"If the rustc compiler is fed `code_x`, it will produce `binary_code_x`. If I give it `code_x` again, it will re-produce the exact copy of `binary_code_x`. "

Truth : 
1. Compilers produce different binaries for each architecture.  
2. Compilers produce different binaries for a specific architecture because of optimizations. Compilation 1 may produce a different artefact from compilation 2 in the same physical machine.  
3. Even if you run the same binary in similar machines, the execution environment will be different for each running binary.  

Main Point:  
Undeterminism is inevitable in a way... you can try to create an exact enironment... but only for critical software.  
One of the main cause of undeterminism in Optimization.  


# Optimizations
Code optimization happens in 2 levels : 
1. Compiler level
2. Hardware Level

Let's discuss this from a very very very very very very VERY high level.  
Let's begin with hardware-level optimization because we will handle only one hardware optimization.  

## Hardware optimization
This is when the hardware (CPU + memory) will try to make your code faster through methods such as : 
- Pipelining
- Branch prediction
- Out-of-order execution

Branch prediction and piplelining are both sub-enablers of Out-of-Order execution, so it makes sense to give an example on it.  
```Rust

// Arrangement 1
println!("{}", arr[0]);
let x = 2;
let y = 3;
let z = x + y;
println!("{}", arr[0])
```

```Rust

// Arrangement 2

// I/O operations on the same array have been put together, 
// The I/O response may get combined in order to reduce these 2 requests into a single request
println!("{}", arr[0]);
println!("{}", arr[0])

// the 2 operations below have been swapped, there is no consequence
let y = 3;
let x = 2;

let z = x + y;

```

# Compiler optimization
These boil down to 4 main goals :  
1. Maintain the exact behaviour of the initial program
2. Reduce unnecessary code eg Duplicates, dead-code and repetitie patterns
3. Re-order and modify code to suit memory-latency and other factors (eg usage of FPUs)

Examples : 
1. **Inlining**: Replaces function calls with the function's code itself
2. **Loop Unrolling**: Replicates loop bodies to reduce branching overhead. eg
   ```pseudo
   // this loop can get unrolled as...
   for i in 0..4 { sum += arr[i]; }

   // ...this
   sum += arr[0]; 
   sum += arr[1]; 
   sum += arr[2]; 
   sum += arr[3];
   ```
3. **Constant Propagation**: Replaces variables with known values at compile time. 
4. **Dead Code Elimination**: Removes unreachable code.  


(future-me, give live examples, I am not writing this... this is your problem)
Point is: in as much as this optimizations are formally verifiable, the optimizations implemented depend on heuristics, compiler versions, and compiler configs.  

Safe code is easy to optimize, but unsafe code is tricky ground.  
You could say that you will instruct your compiler to only optimize your safe blocks, but code is tightly coupled.  
In the end, the optimization heuristics of the compiler, and the undefined-behaviour of your unsafe code cause the compiler to produce wrong programs.  

Now, to deliver my end of the deal, let me show you wrong optimizations....


