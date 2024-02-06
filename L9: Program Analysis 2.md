# 6.035 L9: Program Analysis 2

Previous lecture: how to do analyses within a basic block. Now we extend them to cover the entire control flow graph of a procedure.

## Reaching definitions
Useful for constant propagation: if variable ia constant, we track all uses of this variable reached by this constant definition, and replace with the constant.

A definition reaches a use if the value is written by definition *may* be read by the use. More precisely, there exists a path between that definition and use in which there are no intervening definitions. Note the same variable may have multiple definitions. 

### Computing
At each basic block, we define the following sets:
* IN: set of definitions that reach beginning of block (from other blocks)
* OUT: set of definitions that reach end of block.
* GEN: set of definitions generated in block, *that reach the end of the block*
* KILL: set of definitions *present at beginning* killed in block

We use previous lecture's tools to compute GEN and KILL for all basic blocks- they do not involve other basic blocks. 

Dataflow equations to compute IN and OUT:
* Intuitively, IN should be the union of all the OUT sets of its predecessors. 
* Similarly, OUT is the IN of the block, without the definitions that were KILLed, and including all GENerated definitions. How OUT is computed from IN is called the **transfer function**.

This generates a system of equations on the order of the number of basic blocks. We can solve using a worklist/ fixed-point algorithm. 
* Essentially, initialize all OUT to emptyset, IN of the entry point to emptyset, OUT of entry to be GEN. Initialize the worklist to all blocks except the entry.
* Then pop the first block off the worklist, and apply the transfer function. If it changes OUT, push all successors to the worklist since their OUT's need to be updated as well.
* Repeat until worklist is empty. 

The algorithm eventually halts because the transfer function is monotonic: increase IN means increase in OUT, and there are only a finite number of definitions. 

## Available Expressions
Can be used to do global (across basic blocks) CSE. Fundamentally, if an expression is available at use, no need to reevaluate it. 

We follow a very similar process in that we have IN, OUT, GEN, KILL sets for each basic block, where GEN and KILL are constant and computed within each basic block, and we have a transfer function and a worklist algorithm. 

Big difference comes in how to compute IN: This time, intuitively for an expression to be available at a certain point, it has to be available from *all* predecessors, not just any. This changes the IN definition to be the intersection rather than the union of all OUT of the predecessors. 

For the worklist, we initialize all OUT to the complete set (assuming all expressions are available). Then, the algorithm will halt, but because of the inverse reason: decrease in IN means decrease in OUT, and only finite number of definitions. 

## Liveness
Dead code elimination across basic blocks.

Differs from the others in that it starts from the exit and goes backwards in the CFG. Thus the transfer function in this case computes IN from OUT, and similar changes are propagated throughout. We now define sets:
* IN: set of variables live at start of block
* OUT: set of variables live at end
* USE: set of variables with upwards exposed uses in block
* DEF: set of variables defined in block

OUT is the union of all successor's IN.
Transfer function: IN is all USE combined with all OUT without all DEF. 

## Pessimistic vs Optimistic Analyses
Optimistic: cannot stop early and use current result, not guaranteed to produce correct program until worklist algorithm stops. 
* Available expressions

Pessimistic: opposite
* Liveness for DCE
