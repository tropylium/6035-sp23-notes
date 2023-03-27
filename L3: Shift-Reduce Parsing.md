# 6.035 L3: Shift-Reduce Parsing

## Pushdown Automata
The recognition analogy to CFA's (context-free grammars). Worked example in slides.

Pushdown automata take the program as a string and try to work their way through it, recognizing the productions of grammar rules along the way. 

Pushdown automata contain a symbol stack that contains terminals and nonterminals to store the symbols waiting to be reduced (i.e. simplified by the rules of the grammar). This stack is necessary because most useful grammars have rules w/ >1 symbol in the RHS. Ex. `Expr -> Expr Op Expr` requires storing at least 3 symbols. 

At each step, perform one of three actions depending on state and input:
  * Shift: push next input symbol onto stack
  * Reduce: If some number of symbols from the top of the stack match a grammar production rule `LHS -> RHS`, pop these symbols from the RHS off the stack, push LHS onto the stack. Set this LHS to be parent of the symbols in the RHS. In this way, we see pushdown automata construct a parse tree as they go.
  * Accept: when no more symbols and starting nonterminal sequence is valid

## Constructing a Parser
To simplify, ignore lookahead/ backtracking for now. Assume no conflicts (see next section). 

Basic Idea: to convert grammar to pushdown automaton, build a DFA to control shift and reduce actions. 
* DFA states should encode information about how much of a production the parser has scanned through so far. 
* DFA transitions occur when the parser encounters a symbol, and then based on that symbol goes to another state. 

To construct states, useful to think about position of parser within a production. Ex. in rule `X -> *(X)`, there are 4 positions the parser could be in, represented by `*` AKA an **item**: 
* `X -> *(X)`
* `X -> (*X)`
* `X -> (X*)`
* `X -> (X)*`

Suppose we have example grammar, starting with `S`:
* `S -> X`
* `X -> (X)`
* `X -> 2`

Language of this grammar is clearly `2` nested in some amount of parentheses ex. `2`, `(2)`, `((((2))))`, etc. 

We want to make a start state, which corresponds to the item `S -> *X`. So, make a transition for symbol `X` to the state for the item `S -> X*`. But note `X` is a nonterminal, and so it is never actually seen in the string. Clearly, the first terminal is always `2` or `(`, which can be deduced by following the productions of the symbol `X` directly after the item. Thus, this start state should incorporate items `X -> *2` and `X -> *(X)` which transition to `X -> 2*` and `X -> (*X)` respectively.

### Closures
More generally, define the idea of a **Closure**, which finds all the items that should be in the same state and rigorously defines the "following productions" idea. Then, for a set of items $I$ of the form $X \to \alpha^* A\beta$ where $\alpha$, $\beta$ represents a some sequence of grammar symbols, and $A$ is a single symbol: 
* $I \subseteq \text{Closure}(I)$
* If $I': A \to ^*\gamma$ is an item for some sequence $\gamma$, then $I' \in \text{Closure}(I)$

### Gotos
Similarly, **Goto** defines the transitions and codifies the idea of "moving the dot" over a symbol in the rule. For a set of items $I$ of the form $X \to \alpha^* A\beta$ and a symbol $A$, 
$$
\text{Goto}(I,A) = \text{Closure}(\{X \to \alpha A^*\beta \;|\; X \to \alpha^* A\beta\})
$$

i.e. the Closure of the set of items obtained by moving the dot one over. 

### Reductions
We add a state stack in addition to the symbol stack, because it is necessary to store the history of the parse for nested expressions- the idea being once the nested expression has been parsed, the parsing of the outer expression can resume in the same state. 

Ex. consider the same example grammar from above with the string `2`. The parse tree is `S -> X -> 2`. Parsing will start with the state for item `X -> *2` (call this state 1), and proceed to the state for item `X -> 2*` (state 2). Clearly, the parser should reduce `2` here to `X`. Then, the parser should add `X` to the symbol stack, backtrack to state 1, and proceed to the state for item `S -> *X` and continue from there according to Goto.

This indicates some defining rules for the state stack:
* State stack initially contains the initial state.
* Whenever parser shifts a symbol and moves to a new state, add that state to the stack.
* Whenever reduces a production of some length of symbols, pop that many states off of the stack, and fall back to the state that is now back on top.

This maintains the invariant that there is always one more or equal numbers in the state stack than there are symbols in the symbol stack. 

### Parse Table
We encode the actions for each state and input in a parse table (worked example in slides). 

At each step, look up the action using the state on top of the stack and the current symbol, and take it. Actions may include:
* Shift to [state]: shift next symbol and [state]. 
* Reduce [rule]: Pop RHS of [rule] and pop state stack as many time. Push LHS of [rule]. New State is now the top unpopped state. 
* Goto [state]: push [state].
* Accept

Some states may throw an error for illegal strings, because illegal strings include characters at positions that the parser did not expect for those states. 

## Conflicts
Worked example in slides

* *Reduce-Reduce* Conflict: if the top of the stack matches the RHS of multiple productions. This implies grammar is ambiguous.
* *Shift-Reduce* Conflict: if the stack matches RHS of production rule, but may not be right match; may need to first shift an input then later find a different reduction. Ex. Grammar `X -> a, X -> a b`. At the state for item `X -> *a`, should reduce or shift `b`?

One solution is using **lookahead**, which aims to prevent reducing too soon, by splitting up individual states to take different actions based on the next (some number of) symbols. 
* One possibility is to reduce only when the next symbol can follow the nonterminal being produced in some derivation. In the example above, `b` will always be reduced according to `X -> a b`, so it should always be shifted!

There are many different classes of parsing techniques for lookahead. In general, may have `LR(k)` parsers, where `k` is the number of symbols of lookahead. 
* The first letter `L` stands for left-to-right parsing.
* The second letter `R` stands for rightmost derivation- in the parse tree for a valid parse, rightmost nonterminal is always expanded (most apparent for ambiguous grammars). 
