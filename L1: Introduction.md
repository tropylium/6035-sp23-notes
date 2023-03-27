# 6.035 L1: Introduction

## Compilers: What/Why
Assembly (machine) is not very human readable or understandable, and thus it's hard to work with and build things in
* Soln: program in a high-level language (i.e. C, Java, Python, etc).

Compilers bridge, taking high-level code from human programmers as input and outputting low-level assembly language that can be executed by computers.
* Must read/ understand program
* Precisely translate program actions into computer instructions

Compilers thus are an indispensible programmer productivity tool. 

### Input
Most imperative languages convey:
* Some form of state, in the form of
  * Variables
  * Structures
  * Arrays
* Computation
  * Expression (arithmetic and logical)
  * Assignments
  * Flow control (i.e. conditionals/ loops)
  * Procedures

### Output
From the perspective of the machine
* State
  * Registers
  * Main Memory with flat, 1D, linear address space
* Instructions
  * Arithmetic and logical ops on registers
  * Load and store to registers
  * Branch instructions

### Optimizations
Rarely thus the most literal possible translation from high-level code to assembly result in a very performant program- in terms of speed, code size, etc.

Compilers thus perform numerous optimizations during translation:
* Dead code elimination: no need to execute useless code.
* Constant Propagation/ Algebraic Simplification: compute things at compile time instead of at run time as much as possible.
* Common Subexpression Elimination: compute a common subexpression once rather than recomputing it every time it appears.
* Strength reduction: downgrade expensive operations (like mul/ divide) to less expensive but equivalent operations (like shift/ add) if possible, i.e. mul by 4 becomes shift left by 2.

One of the most common targets of optimization are loops:
* Loop invariant: compute invariants once instead of recomputing at each iteration.
* Strength reduction: take advantage of loop structure to downgrade some operations.

Numerous other optimizations are possible.

After all is said and done, the assembly code often logically works very differently compared to the high-level code, even if they produce the same result. 
