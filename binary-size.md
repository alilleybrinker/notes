
# Binary Size

- Three stories:
  - Frame pointers
  - Compact C Type Format (CTF)
  - Rust binary size complaints

- Frame pointers seem good
- They're needed for stack unwinding for at least _some_ functions
- Lots of platforms have historically ommitted them whenever possible to save on code size and performance
- When present, you pay a cost to put the frame info into the appropriate register and to restore it when a function returns, plus you lose a register that the register allocator of the compiler could otherwise use
- So maybe more stack spilling during register allocation, and more code size per function
- HOWEVER
- Without frame pointers, debugging becomes more difficult!

I am going to tell three stories about binary size and the trade-offs involved, and I hope
at the end that I will have convinced you of the idea that historically a lot of projects
have set defaults which emphasize small binary size to the detriment of debuggability, and
this has hampered the development of debuggable systems in meaningful ways we're still
coming to grips with.

The following post is specifically about binary size in the context of programs which are
ahead-of-time compiled into assembly language and thereafter to machine code. It's possible
similar issues exist in the world of languages which compile to bytecode, or are interpreted,
but I'm not as familiar with those languages and don't want to generalize beyond my area of
knowledge.

## Frame Pointers

A frame pointer is a pointer to the base address of a stack frame for a function. Historically,
popular C and C++ compiler have opted to elide frame pointers from functions whenever possible,
for a couple of reasons:

- To track the frame pointer for the current frame during execution of a program, that pointer
  needs to be persisted to a register. This means there is one less register available to use
  for parameters to the function, which can result in more spilling of parameters to the stack
  instead of passing through registers. Given that registers are much faster than accesses to
  memory or instruction caches, this can result in worse performance for function calls. The
  code for a function is also now larger, which may mean more instruction cache misses.
- Setting the frame pointer in the correct register for a function call, and then restoring
  it when the function returns, means more instructions to execute for each function call. While
  an optimizing compiler may inline functions (thus removing the function from the resulting
  assembly code, meaning there's no frame to record a frame pointer for), you still generally
  end up with larger binary size. This also has the effect of potentially more instruction
  cache misses, same as the first reason noted above.

So basically, they've been historically ommitted because of slower performance and larger
binary size.

You might ask, given that there's a clear negative return on performance for having frame
pointers all of the time, what the benefit of frame pointers even is. First, frame pointers
are _necessary_ to fully support two key pieces of functionality: backtraces and stack
unwinding.

Backtraces, as you're likely away, are readouts of the current status of the call stack at a
moment in time during the execution of a program, and are a popular tool for debugging
programs which are misbehaving. To produce a backtrace, you have to walk up the callstack
of the program, and frame pointers are essential to how you do that walk and identify the
boundaries of each frame.

Stack unwinding is a mechanism used for exceptions in C++ and panics in Rust. This also
walks through the call stack, but unlike backtraces actually modifies the callstack by
cleaning up and removing each stack frame as it goes. To do so, it needs frame pointer
data to know the stack frame boundaries.

You may have noticed early in the article that I said frame pointers are by default
ommitted when possible for popular systems. The "when possible" phrase is meant to
capture that at least some frame pointers will generally be present to support the two
use cases I've just identified; but many frame pointers _will_ be ommitted because they're
not needed to support unwinding.

The other, equally important thing we miss out on without frame pointers is _debuggability_.


## Compact C Type Format (CTF)

## Rust Binary Sizes


