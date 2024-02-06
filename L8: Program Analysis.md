# 6.035 L8: Program Analysis

## Why
Program analysis is compile-time reasoning about the run-time behavior of the program, to discover things that are always true/ infer things that are likely to be true
* want to use analysis results to improve some aspect of program, including:
  * number of executed instructions
  * code size
  * power consumption
  * memory usage
  * cache hit rate
  * etc.

We focus mostly on basic block optimizations.

## Basic Block Analysis framework
We simulate a symbolic (logical, virtual) execution of the basic block. We reason about values of variables (or other aspects) to derive property of interest (hence the name dataflow analysis).

### Common Sub-expression Elimination
Essentially, want to discover which variables and expressions have the same value at some point, replace redundant code with code that computes those values once to avoid wasting recomputation.

**Value Numbering**: we symbolic forward execute a basic block, assigning each new value discovered to a temporary (even if we don't discover any duplicates later- this will be addressed). Point is to preserve this value for possible use later in program, even this value gets discarded/ overwritten by the original formulation.

Uses several mappings:
* Variable to value- specify symbolic value for each variable
  * used to compute symbolic value of variables while processing statement of form `x = y+z`
* Expression to Temporary- determine which temporary if discovered expression that was previously computed
* Expression to value- used to update Var to Val after doing the common subexpression elimination

Combining with flattening, can find arbitrarily complex common subexpressions

Issues:
* Introduces lots of temporaries, copy statements to temporaries which may be unnecessary
* Eliminate with the next 2 analyses

Issues 2:
* Expressions have to be identical (doesn't know about commutativity/ associativity)
* Solution: use canonicalization (put expressions in a predetermined order) or algebraic simplification

### Copy Propagation
Want to use original value instead of temporary; i.e. transform statements such as:
```
a = b + c
tmp1 = a
d = tmp1
```
into 
```
a = b + c
tmp1 = a
d = a
```
Key idea: have to determine whether or not original variable has been overwritten between its assignment and use of the value; if not overwritten, use original variable

Uses two maps:
* Temporary to variable: tells which variable to use rather than the temporary
* Variable to set: inverse of above- tells which temps are mapped to a given variable

### Dead Code Elimination
Copy propagation keeps all temps around- including temps that are never read. DCE eliminates them and more. 

Basic idea- 
* process code in *reverse* execution order. 
* Maintain a set of variables that are needed later in computation. 
* If encounter an assignment to temp that is not needed, remove assignment

### Algebraic Simplification
* Apply knowledge from algebra, etc. to simplify some expressions
* May need to consider some effects about numerical stability