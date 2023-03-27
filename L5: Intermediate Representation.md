# 6.035 L5: Intermediate Representation

## Motivation
After parsing, want to represent program in a **internal representation (IR)** that allows us to apply analyses such as:
* Semantic checks (ex. static type checking)
* Optimizations
* Machine Code generation

Often, the process of creating the IR is split into a **high-level IR** and a **low-level IR**. 

The goal of the high-level IR is to represent program in an intuitive way that supports future compilation tasks. 
* Preserves object structure (i.e. symbolic object fields/ variables, functions, etc.) 
* Preserves structured control flow (if/for blocks).
* Useful for the higher-level analyses and parsing of abstract expressions into more machine-friendly loads and stores from values. 

We then further transform this into the low-level IR which interfaces directly into assembly code. 
* Object structure is turned into memory offsets.
* No more variables- just values stored in some place in memory. 
* Structured control flow turned into jumps, etc. 
* Register/ stack optimizations is turned here. 

This lecture mostly discusses the high-level IR. 

## Symbol Tables
**Symbol Tables** map identifiers (strings, can be names of variables, functions, classes, etc.) to descriptive information about them.
* Ex: for static type checking, symbol tables are used to determine if variable usage or method call is consistent with its declared type, refusing to compile if it does not.
* For classes, symbol table might map class name onto its fields and methods for that class. 
* For low-level IR, symbol table may indicate the stack offset allocated to that variable.  

### Symbol Table Hierarchies
Often, symbol tables may include a pointer to "parent" symbol tables that are above it in a symbol table hierarchy. This naturally arises from:
* Nested Scoping- with a symbol table for each scope, the deeper symbol tables pointer to the shallower ones
* Inheritance- child class symbol table points to parent class symbol table

Lookup of an identifier proceeds up the hierarchy until a description is found. Ex. for a method in a class, there may be symbol tables for:
* Local variables of the method, which points to
* Parameter variables of the method, which points to
* Fields of the class, which points to
* Fields of parent class, and so on

Much more detailed example in slides. 

#### Dynamic Dispatch
While typically symbol tables are a compile-time construct, inheritance introduces a complication since subclasses may override methods. If the exact class of an object cannot be determined at compile time, dynamic symbol tables are required to invoke the correct method.

## Translation from Abstract Syntax Trees
As a preprocessing step, often parser works with a hacked grammar that removes ambiguity, has precedence, avoids left recursion, etc. at the cost of creating an unintuitive parse tree. It may be necessary to walk this tree and produce a more intuitive abstract syntax tree before creating the IR.

Then, the symbol tables and IR are created by recursively visiting this tree. 

Slides include suggestions of how each type of statement should be represented. 