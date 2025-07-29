# rough_paper

Pointers primer

- pointers are not integers. They should not be semantically treated as integers, that is a wrong abstraction of what they are. This wrong abstraction leads to semantical errors/faults. This is where provenance comes in...  
- Abstraction should maintain the semantic meaning of a pointer. Helps in correct compiler optimization and semantic checks



- This “extra information” that distinguishes different pointers to the same address is typically called provenance. 



- "However, to make the argument that an optimization is correct, the exact semantics of LLVM IR (what the behavior of all possible programs is and when they have UB) needs to be documented. All involved optimizations need to exactly agree on what is and is not UB, to ensure that whatever code they produce will not be considered UB by a later optimization. This is exactly what we also expect from the specification of a programming language such as C, which is why I think we should consider compiler IRs as proper programming languages in their own right, and specify them with the same diligence as we would specify “normal” languages.4 Sure, no human is going to write many programs in LLVM IR, so their syntax barely matters, but clang and rustc produce LLVM IR programs all the time, and as we have seen understanding the exact rules governing the behavior of programs is crucial to ensuring that the optimizations LLVM performs do not change program behavior.

Take-away: If we want to be able to justify the correctness of a compiler in a modular way, considering only one optimization at a time, we need to perform these optimizations in an IR that has a precise specification of all aspects of program behavior, including UB. Then we can, for each optimization separately, consider the question: does the optimization ever change program behavior, and does it ever introduce UB into UB-free programs? For a correct optimization, the answer to these questions is “no”." - explain this intently


"how precise a language specification needs to be." - enough to avoid undefined behaviour/ variation in semantics











This is my breakdown of the paper `Stacked Borrows: An Aliasing Model for Rust` and the official docs for `Provenance` in Rust lang.  
I write, therefore I understand. I understand therefore I write.  

I write in prose form. I am not concise.  
I believe that a technical book/blog/paper is best written and read as a novel. In prose.  
One simple sentence after another. The reader should not think a lot, and neither should the writer. So I will be chaining one obvious sentence after another obvious sentence... you're about to die of boredom. At the end of it you will realize that you've read the whole thing without using your head. Boring is good. (for me at least)


# Stacked Borrows  

"Type systems are useful not just for the structural safety-guarantees they provide, but also for helping compilers generate
more efficient code by simplifying important program analysis." - a hard sentence, sigh. (let's call this the `1st obvious sentence`).  

"To make pointer provenance part of the type-system, we enforce the concept of stacked borrows." - another hard sentence. sigh some more. (let us call this `the 2)

The first sentence sounds obvious. I have lots of time, so I am going to over-explain that obvious statement because I am at the beginning of the paper and I have not yet got in my zone. You can skip to [this section](undone) to avoid my rumblings.  

## explanation of the 1st obvious sentence 

A type-system in a programming language is a system that defines **and** enforces:
  - the types (kinds of data layouts) *eg* a `&mut usize`  
  - the operations allowed on the types *eg* you can dereference a `&mut usize`.
  - the conditions and contexts needed to perform those operations on the types : *eg* you cannot create two `&mut usize` types that reference the same data in the same scope.  

Rust has a static type-system... one that enforces it's rules at compile time.  


**Structural safety** refers to the ability of a program to avoid undefined behavior, memory corruption, or logical inconsistencies due to improper manipulation of data in the data-types.  
A type system ensures structural safety, by construction. You could say, the type-system **IS** the structure itself. 


### The Rust type system and how it ensures structural safety
This is not comprehensive description of the rust type-system... you can find that in the language reference.  
In fact, I am just going to list things so that you can get the idea that the type-system is made up of many mechanisms and that we can improve it by adding more mechanisms.  

1. **The Ownership mechanism**: defines aliasing rules and eliminates data-races
2. **Lifetime annotation mechanism**: statically takes care of dangling pointers...
3. **Algebraic Data Types (ADTs) and Exhaustiveness Checking**: forces programmer to handle all arms of an enum or match statement thereby reducing logical bugs. 
4. **No Null or Uninitialized Values**: Null drama is avoided. Rust’s `Option<T>` forces explicit handling of absence, eliminating null pointer dereferences (a billion-dollar mistake)

This list could go on forever... i just hope you get it.  

### Code optimization and Efficient code generation
For the compiler to generate efficient code, it has to do some optimizations. Simple.  
Largely, optimization is about re-ordering and eliminating reducing memory operations.  
Optimizations should not change the behaviour of the program...nor should they introduce undefined behaviour. They should only make the code more efficient.  

For the compliler to enforce rigorous optimization techniques, it needs metadata on how the program works... and the type-system provides such info. The more precise info the compiler has, the more optimizations it can confidently implement. (ps, that is why type-dependent languages are sometimes seen as a holy-grail for optimization, quite an interesting off-topic)  

In the name of verbosity, here are some examples of how the rust-type system assists in providing info for optimization: 

1. The ownership mechanism affirms to the compiler that it can **re-order** instructions that contain references in certain contexts *eg* the compiler can freely re-order instructions that contain immutable borrows to some variable `x` in the same scope... this is because it is sure that the value in `x` could not have possibly changed so each memory-read in the same scope can be interchanged.  
   
2. **Dead code elimination**: The type system can prove certain code paths are unreachable (*eg* match on an enum where some variants are statically impossible), allowing the compiler to remove such code entirely. (type-state programming, I wish I knew this stuff)
   
3. **Memory Layout Optimization**: the type system can provide info on types such that the compiler can allocate on the stack instead of the heap when lifetimes are provably short (via escape analysis).  
   
4. **Compile time monomorphization**: generic implementations get implemented at compile-time for each associated type such that dynamic dispatch (indirection) is totally eliminated. This als*eg*o ensures `type-specific` optimizations ...*eg*: `Vec<i32>` and `Vec<f64>` generate separate code, allowing for type-specific optimizations (*eg* CPU vectorization for numeric types)

### Nonsense
So I am gonna repeat the `1st obvious statement` and I hope it will be obvious: "Type systems are needed for both **structural safety** and **optimization** of the program."


Now, time for the `2nd obvious statement`.  

## The 2nd Obvious statement
"To make pointer provenance part of the type-system, we enforce the concept of stacked borrows."  

This statement is actually not obvious, it will be obvious after the paper. hehehe.  

But it goes along the lines of: The type-system needs to be able to catch errors concerning pointers in unsafe code... provenance is a way to track pointers in such a way that it provides the compiler with enough information for it to perform correct code-optimizations. Stacked borrowing is an abstraction above memory accesses that makes this pointer-tracking easier.  

If that did not make sense, then don't try to understand it now. Keep it at the back of your head. I am sorry.  

## Pointer aliasing problems in unsafe code  

>"Type correctness does not always equal program correctness" - anonymous  

**Pointer aliasing** in Rust refers to the situation where two or more pointers(inclusive of references) refer to the same memory location.  

Aliasing may result in undefined behaviours:
   - when using alias A to read from memory that is being simultaneously written two through alias B
   - when dereferencing memory using alias A, when that memory had already been freed via alias B
Let us call such problematic aliasing a wonderful name: **unsafe aliasing**.  

Rust prevents **unsafe aliasing** at compile time using the ownership model: "At any given time, you can have either one mutable reference &mut T or any number of immutable references &T, but not both." This is achieved with the help of static lifetime annotations.  

If you stick to pure safe Rust and use references only, you are safe from **unsafe aliasing** because of the borrow-checker mechanism. But if you start using pointers(under unsafe blocks), you may have problems. This is because unsafe blocks allow you to use pointers outside the aliasing rules. Eg you may end up having 2 mutable pointers to the same or overlapping memory allocations.  

I am told that unsafe code is inevitable. I don't know meh. Somewhere deep inside me, I am totally convinced that with good design, unsafe code can be totally eliminated. I know I am wrong... but it's my truth. Sigh.  

So we are going to learn about **provenance** and **stacked borrows** and **memory models** and a lot of other things so that we can write safe unsafe-code.  


## Stacked-Borrows : the mechanism  
The borrowchecker is able to do static analysis because of the static lifetime annotations. The lifetime annotations get denoted explicitly by the programmer or implicitly by the compiler.  


### Why not build a static analyzer? Why not extend the default borrow-checker?  

> Because it is hard(maybe impossible) to do comprehensive static analysis on unsafe pointers in a general-computing environment.  

The stack-borrow paper had the following reasons:  

1. Raw pointers can come from anywhere (*eg* FFI, memory-mapped I/O), and their behavior may depend on runtime conditions that may be too unique for a static analyzer to analyse. Static analysis is not comprehensive enough to account for these arbitrary runtime situationships. 

2. The borrow-checker is built on the concept of lifetime annotations...which has a couple of problems :
   1. **Lifetime inference mechanisms are not constant** : The lifetime annotations get denoted explicitly by the programmer or implicitly by the compiler. Funny part is that the compiler also infers lifetimes within unsafe blocks. This infered annotations are fine for safe code... but not always-correct for unsafe code. Moreover, the compiler lifetime-inference mechanism is fickle because it is subject to change: eg "it switched from the old AST-based borrow checker to non-lexical lifetimes and it is planned to change again as part of an ongoing project called `Polonius`."  

   2. **Lifetime annotations are erased from the program before the optimization phase**: lifetimes are erased from the program during the Rust compilation process immediately after borrow checking is done, so the optimizer does not even have access to that information

I hear you, the above reasons are not convincing...personally, I am not convinced.  
But those are the reasons.  
I will accept them since I dont know much AND because the mechanisms proposed by the stack-paper made sense:

1. A dynamic checker will be able to analyze the situation better. 
2. Stack borrow method applies to both the safe and unsafe code without much drama. (As explained in the previous section)




* why is MIri an interpreter?



## Limitations of Stacked Borrows
Are we wasting our time learning Stacked borrows and Provenance? ha ha ha. Oh my! Please, let's not get personal here...  
Never tell a researcher that they are wasting their time. You could trigger despair.  

I see developers as mages. There is no harm in learning many spells. Even the most escentric ones.  
I love Frieren's philosophy, despite my currently short lifespan.  
This is no loss. It can't be. right?  

But anyway, here is how I have wasted your time and mental energy.  

A. There exists other different mechanisms
B. Dynamic nature is not exhaustive

1. Stacked Borrows are build for sequential single-threaded code. They are not tested for concurret code.  
2. Technical [problems](https://www.ralfj.de/blog/2023/06/02/tree-borrows.html) that I don't yet understand. 
3. There is a newer model called **[Tree Borrows](https://www.ralfj.de/blog/2023/06/02/tree-borrows.html)** that improves upon Stacked Borrows. And the good news is that it is also not concurrency-aware.
4. The "[RustBelt](https://plv.mpi-sws.org/rustbeln process immediately after borrow checking is done, so the optimizer does not even have access to that information
t/#project)" exists and there are other proposals besides stack-borrows: These project explored formalizing Rust’s concurrency guarantees (*eg* for Mutex, Atomic types). They did not directly extend Stacked Borrows but provided foundations for handling shared-state concurrency.
5. Some research tools (*eg* [Viper at ETH Zurich](https://www.pm.inf.ethz.ch/research/viper.html) and [Prusti](https://github.com/viperproject/prusti-dev)) experimented with concurrency-aware borrow checking, but these are not part of Rust’s official semantics. It would have been better if you learnt them. Like I n process immediately after borrow checking is done, so the optimizer does not even have access to that information
said, I wasted your time.
6. Tools like Miri exist and there will always be some crazy devs who build free tools, you need not learn the internal workings of such tools...because each mechanism is worth a phD.  
7. Stacked Borrows has not yet been used in the official Rust lang. Rust doesn't even have an official memory model specification. SB is used in tools like Miri (Rust’s UB detector) and guides unsafe code best practices

I am betting that the Rust folks will extend their current model to use stacked borrows for the sequential bit and another model for the concurrent bit. I do not know.  

The dynamic aspect is not a limitation. You can count that as a compile-time expense.  