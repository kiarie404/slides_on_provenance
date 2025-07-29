# Pointer Validity

***When is a pointer valid?***  

An operation is **safe** if is does what it is expected to do without any chance of undefined behaviour.  
An operation involving a raw-pointer can only be **safe** if the pointer is **valid** for the given memory opration. For example, writing through a `*const T` is invalid, and therefore unsafe.  


Whether a pointer is valid depends on 
1. The operation it is used for (read or write)
2. The extent of the memory that is accessed through that pointer(i.e., how many bytes are read/written)  

Pointers for Zero types are an exception because all their memory operations don't interact with actual memory, those operations end up as `Nop` (No-operation). The exception is that all their pointers are valid [undone](explain this further).  

From here on out, we will talk about Non-Zero-sized types ONLY.  

