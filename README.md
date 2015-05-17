#Compilers : COMP36512

Going to be making my notes in this markdown file, plus also maybe throwing some basic implementations written in python for lexing, parsing, etc (if I'm feeling adventurous).

##Index

1. Introduction
2. General Structure of a Compiler
3. Introduction to Lexical Analysis
4. Lecture Four: From REs to DFAs
5. Lecture Five: DFA Minimisation
6. Lecture Six: Exercise Lecture
7. Lecture Seven: Introduction to Parsing
8. Lecture Eight: Top Down Syntax Analysis (needs updating)
9. Lecture Nine: Bottom Up Syntax Analysis (needs updating)
10. Lecture Ten: Exercise Lecture
11. Lecture Eleven: Context Sensitive Analysis
12. Lecture Twelve: Intermediate Representations & Symbols Tables
13. Lecture Thirteen: Procedure Abstraction, Run-Time Storage Organisation
14. Lecture Fourteen: Exercise Examples
15. Lecture Fifteen: Code Generation - Instruction Selection
16. Lecture Sixteen: Register Allocation
17. Lecture Seventeen: Register Allocation via Graph Colouring
18. Lecture Eighteen: Instruction Scheduling
19. Lecture Nineteen: Code Optimisation
20. Additional Notes for Exam

##Lecture One: Introduction

A compiler takes some source code, and produces an output in another language, while the retaining meaning of the source.

####The compiler must:
 * Generate correct code.
 * Recognise errors.
 * Analyses and Synthesise.

####The properties of a good compiler:
 * Generates correct code.
 * Generates fast code (optimisations).
 * Conforms to specifications of input language (standardisations).
 * Should be able to cope with arbitrary sized input (e.g: large number of variables, so forth).
 * Compilation time should scale linearly to the size of the source program.
 * Good diagnostics.
 * Consistent optimisations => similar logic in all instances.
 * Works with debugger.

####Other issues:
 * Speed (of compiled code).
 * Space (we don't want our executable to be larger than the input).
 * Feedback.

Which issues do we prioritise? It is use case dependant.

##Lecture Two: General Structure of a Compiler

####Two Major Phases: 
 * Front End: Analysis.
 * Back End: Synthesis.

Source -> (analysis) -> Compiler -> (Synthesis) -> Target.

The components of a compiler are outlined below:

####Front End:

 1. Lexical Analysis (scanning): reads characters from source and produces a set of tokens.
 	a. Tokens come in the form: <token_class, attribute>.
 	b. For example: a=b+c becomes <id,a> <=,> <id,b> <+,> <id,c>.
 	c. More info: 'man flex'.
 2. Syntax Analysis (parsing): analyses the produced set of tokens and produces an **Abstract Syntax Tree (AST)**.
 	a. AST is a hierarchical structure, expressed using recursive rules.
 	b. Context free grammars formalise these rules.
 	c. AST is not a parse tree, parse tree contains lot's of unneeded information.
 3. Semantic Analysis: annotates the AST.
 	a. Checks for semantic errors.
 	b. Annotates each node of the tree with results.
 	c. Examples: Type Checking, Flow-of-Control Checking, Uniqueness (or reserved) Checking.
 4. Intermediate Code Generation: Translates language specific constructs into more general constructs.

####Back End:

 5. IR Optimisation: Optimise the intermediate code.
 6. Code Generation: Map AST into a linear list of target machine instructions, in a symbolic form:
 	a. Instruction Selection: Pattern Matching.
 	b. Register Allocation: NP-Complete Problem.
 	c. Instruction Scheduling: NP-Complete Problem.
 7. Target Code Optimisation: Machine code information required by the OS is generated.
 8. Target Code Generation: Machine code and associated information required by the Operating System are generated.

##Lecture Three: Introduction to Lexical Analysis

Natural languages have 'high degrees of freedom', as humans we're able to interpret words (symbols) in numerous ways, they're potentially ambiguous and based on their contextual confines (this is also arbitrary, because every person has their own internal dictionary and languages are not always set in stone, so to speak).

However, in a formalised language, such as a programming language, a 'high degree of freedom' isn't usually a good thing (although there is a trade off and you have some languages that are loosely typed, where as others are strongly typed - the degree of freedom which you design your language to have may affect the complexity of the implementation of the compiler / interpreter).

####Formal Language Lingo:

 * Vocabulary: finite set of symbols.
 * String: finite sequence of symbols.
 * Language: any set of strings over a fixed vocabulary.
 * Grammar: finite way of describing a language.
 * Context Free Grammar: G=(S,N,T,P):
	* S - Starting Symbol.
	* N - set of non-terminal symbols.
	* T - set of terminal symbols.
	* P - set of production rules.

In order to construct a set of tokens during lexical analysis, we need to be able to recognise tokens (patterns) and we can identify an individual token using a CFG as above. Some tokens are easy to identify, e.g: white space (it can't really be much besides spaces and tabs, etc), but some tokens are going to be inherently more complex (e.g: floating point numbers?).

####Regular Expressions

A regular expression is a way of expressing a regular language, it is a formula that describes a possibly infinite set of strings. Methods of performing lexical analysis using regular expressions:
 * Ad Hoc: break down the input into numerous smaller problems and process them separately.
 	* Advantage: can be efficient.
 	* Disadvantage: requires a lot of work! (and we don't like that, do we?) + difficult to modify.
 * Automaton Analyser: Do it all automatically in the following manner.
 	* Data: List of tokens := {}, current token := (token.class, token.length) <- (null, 0), current regEx match length := 0
 	* for every regEx that belongs to the analysis of this language:
 		* if match length > current max match length:
 			* current token := (regEx.id, = match length)
 			* current max match length := match length
 	* List of tokens.append(current token)

Regular expressions can be expressed as Transitions tables, which lead to deterministic and non-deterministic Automatons.

##Lecture Four: From REs to DFAs

* REs can describe regular languages
* Every RE can be converted into an NFA (Thompson's Construction).
* Every NFA can be converted into a DFA (Hopcroft's algorithm).
* DFAs can automate the construction of lexical analysis.

Non deterministic finite automaton (NFA) are if one or more transitions exist from one state to another state under the same transition criteria (match). A regular expression is converted to an NFA using the rules described by Thompson's Construction.

####Key ideas for Thompson's Construction are outlined in an image in this Repo (ThompsonsPrincipals.png).

Deterministic finite automaton will describe a transition for every possible input, and will always terminate. Every NFA can be described using a DFA. This is done by isolating the states that create the non deterministic property from the transitions table and constructing a deterministic variant using the **subset construction algorithm**.

####NFA to DFA, Two Key Functions:

1. Moves(STATE ID, INPUT), returns a list of the states possible to reach from the STATE ID, using the INPUT to traverse the automaton, but *DOES NOT INCLUDE THE STARTING STATE*.

		Moves(STATE ID, INPUT):
			List of reachable states = {}
			For each transition from state STATE ID:
				if transition == INPUT:
					add transition.END STATE to reachable states

			Return reachable states

	This can also be modified to take a set of states (collection ideally) as the input:

		Moves_modifed({STATE IDS}, INPUT):
			List of reachable states = {}
			For each s in STATE IDS:
				For each transition from state s:
					if transition == INPUT:
						add transition.END STATE to reachable states

			Return reachable states

2. Epsilon_closeure(STATE ID), returns a list of the states possible to reach from the STATE ID only using empty transitions, essentially it's a recursive function.

		Epsilon_closeure(STATE ID):
			Collection of reachable states += Moves(STATE ID, EMPTY TRANSITION), this STATE ID
			For each reachable state in reachable states:
				Epsilon_closeure(reachable state)

			Return reachable states

	Again this could be modified to take a set as an argument:

		Epsilon-closeure_modifed({STATE IDS}):
			Collection of reachable states += {STATE IDS}
			For each s in STATE IDS:
				add Epsilon_closeure(s) to reachable states

			return reachable states


As a side note that I thought was kind of interesting while coming up with this recursive algorithm, if we were to replace the collection of reachable states with a list, we could potentially end up with a infinite loop in our program (well actually, it would eventually terminate when we had a stack overflow, or ran out of memory or whatever, but you see what I', saying) theoretically it **could** never end since the NFA could be a cyclic graph, so we had better use a collection.

*IDEA: Make an implementation in python that takes a regular expression from the user, constructs an NFA from this expression, converts this NFA into a DFA and then map this out using networkx and matplotlib python libraries - hopefully I'll have time to do this, because I think this could be pretty fucking cool! Why do I have to revise when I could be spending my time doing cool things? :( Wait a minute... wouldn't that actually be the implementation of a lexical analyser? Will come back to this point soon.. Would also like to point out that these can easily be modelled as directed graphs, states are nodes and transitions are directed edges between nodes which are weighted with value, this value COULD be numerical, since it is possible to map each symbol from the alphabet that makes up the vocabulary of the language to a value (maybe use a hashmap for this), or if netwokx allows, we could simply weight the edges of the directed graph with the symbols themselves...?*

####The Subset Construction Algorithm

The functions 'Move' and 'e-closure' are foundational to the subset construction algorithm, as described below [2]:

1. Create the start state of the DFA by taking the e-closure of the start state of the NFA.
2. Perform the following for the new DFA state:
	* For each possible input symbol:
		1. Apply 'Move' to the newly-created state and the input symbol; this will return a set of states.
		2. Apply the e-closure to this set of states, possibly resulting in a new set (This set of NFA states will be a single state in the DFA).
3. Each time we generate a new DFA state, we must apply step 2 to it. The process is complete when applying step 2 does not yield any new states.
4. The finish states of the DFA are those which contain any of the finish states of the NFA.

*implementing this is defiantly do-able, but it might take a bit of time, I'm not sure if I've got the time to do this atm..*

##Lecture Five: DFA Minimisation

If a DFA has the property where there exists two or more states whose transitions are found to be equivalent, as long as both are either final or regular regular then the DFA can be reduced. The reduction is based on Hopcroft's Algorithm, as follows:

1. Divide the states of the DFA into two groups: final and regular.
2. While there are group changes:
	* For each group change:
		* If any two states of the group and a given input symbol, their transitions do not lead to the same group, these states must belong to different groups.

NEED TO PRACTICE THIS.

Once you have a transition table, you have a potential efficiency problem, because you have to linearly run through a table every time you want to check a symbol, or do you...? Make the transition table essentially a look-up table that reflects the code, this can be done automatically when generating the code.

To-do:
* Re-watch lecture five and make some more notes, do some more practice.
* Practice exam questions associated with Lexical Analysis.

##Lecture Six: Examples Lecture

##Lecture Seven: Introduction to Parsing (Syntax Analysis)

Okay, so we've taken some input-stream and have generated a set of tokens based on that input, now we need to check that these tokens are syntactically correct; that is, that they are in the correct order, so to speak.

Regular Expressions have limits, for example:
* We can't use REs to check matching brackets.
* Or the set of zeros followed by an equal number of ones.

In regular expressions, a non-terminal symbol cannot be used before it has been fully defined.

All we're doing with LEXICAL ANALYSIS is detecting patterns one after another, there is no way of specifying rules such as, "if there is an opening bracket here, we need to make sure there is a closing bracket there..".

*Regular Languages* are a subset of *Context-Free Languages*, which are a subset of *Context Sensitive Languages*, which are a subset of *Phrase Structured Languages*.

In a context free grammar, the only additional rule to regular languages (REs) is that **every LHS symbol should be a non-terminal symbol**.

Replacing symbols in a set of tokens using a context free grammar, in order to ensure it is syntactically correct is accomplished through the execution of **derivations**. We can derive a set of symbols in two such ways:
 1. Left most derivation: at each iteration, replace the left most non-terminal symbol.
 2. Right most derivation: the same, but replace the right most non-terminal.

A parse tree can be constructed to graphically represent these processes (derivations):

	Start with the starting symbol (root of the tree).
	For each sentential form:
		add child nodes to the node corresponding to the LHS symbol.

	The leaves of the tree constitute a sentential form.

####Ambiguity

Any grammar that has more than one left-most or right-most derivation for the same output is ambiguous, and if it is ambiguous a grammar is not good for a compiler, because it will construct two different parse trees and so the language will have no notion of precedence.

If a programming language is defined such that it may accept ambiguous terms (loosely typed), the analysis of these parts must be postponed until later in the compilation process (FORTRAN, python? ruby?).

####Parsing Techniques

Once we've got a grammar that is not ambiguous, there are two different ways to do syntax analysis:
 1. Top down: Useful if we have simple grammars and simple strings.
 2. Bottom up: Most compilers use bottom up because this type of parsing can be automated. From a compilers point of view, it needs to have some deterministic way of finding out what the next substitution should be, at every token.


##Lecture Eight: Top Down Syntax Analysis
*(This lecture needs updating..)*

* Construct the top node of the tree & rest with pre-order (depth-first).
* Pick a production rule and try and match the input, if you fail, backtrack.
* Essentially: try and find a left most derivative for the input string.
* Some grammars are back-track free (predictive parsing)

####Problems

* Not necessarily very efficient, due to possible failures.
* There is potential to enter into a infinite loop (remember compilers are machines and are algorithmic, trying to tell whether a program is infinite or not is an NP complete problem, I think?).

####Left Recursive Grammars

Grammar is left recursive if it has a non-terminal A, such that A->Aa, for some string a.

####LL1 Property

<UPDATE THIS BIT>

#####Conclusion

Top down parsing is good if we have a really simple grammar (as long as the grammar has simple properties: LL1, use top down parsing because it can be done without backtracking). However in the general case, real compilers use bottom up syntax analysis because not all grammars have the LL1 property.

##Lecture Nine: Bottom Up Syntax Analysis

<UPDATE THIS>

####Lecture Ten: Examples Exercise

##Lecture Eleven: Context Sensitive Analysis

In terms of syntax analysis, we can have code that is syntactically correct but in terms of context, it may be incorrect. For example, you could have a function with five arguments, but you could have a call to this function with six arguments. This is syntactically correct, but contextually incorrect.

* Property: Something is an error because of something defined somewhere else in the code.

####Type Checking

Programming languages that have a declare before use policy have a table which can be checked to see how two different types may interact under different operations. Components of a type system:
* Base or Built In types (integers, boolean, characters, etc).
* Rules to:
	* construct new types. 
	* determine if two types are equivalent (table).
	* infer the type of source language expressions.

This is not difficult for a language with a declare before use policy, but if the language does not have a declare before use policy, then it becomes more challenging.

####Attribute Grammar

For every symbol in the language, have an associated value that has some kind of semantic meaning, such that we are in some kind of situation to answer some semantic questions. Essentially labels.

Two types of attributes:
1. Synthesised Attributes: derive their value from constants and children (evaluated bottom up).
2. Inherited Attributes: derive their value from parent, constants and siblings.

Example: you could have a attribute called type, which holds a value which indicates the type of the token.

Example: reading a binary number (-101):

Sign-> +
    |  -
List->List1, Bit
    | Bit
Bit->  0
    |  1

Sign has an attribute neg or pos, bit has two attributes, pos and val. Val is dependant on position.

####Dependancy Graph

If one node depends on another, then the former will have to be evaluated before the latter.

* Nodes represents attributes; edges represent the flow of values.
* Graph is specific to the parse tree.
* Evaluation order:
  * Parse tree methods: cyclic graphs fail.
  * Rule based methods: the order is statically predetermined.
  * Oblivious methods: convenient approach independent of semantic rules.

How do we deal with cycles?
Complex dependencies?

These dependency graphs produce a topological order based on the dependencies.

**This area of research isn't very successful for compilers in general because of the complexity involved in the graphs, etc**

## Lecture Twelve: Intermediate Representations & Symbols Tables

Now the compiler has established that the input is correct, can represent it in a form that is convenient (AST) and then perform optimisations on this input.

AST allows us to represent what we already know and in a hierarchical form and it contains all the information we need.

AST is the form of representation that is as close as possible to what we've already seen, and it's easy to manipulate. It's close to the high level implementation.

Why do we need this?

The IR is suppose to be convenient for the compiler.

####Design Issues

* We need something lightweight.
* We need something that is easy to manipulate.
* The decisions we make here in terms of the implementation of the IR directly influences the performance of the back-end, so it's important.
* ease of generation.
* ease of manipulation.
* cost of manipulation.

Control flow graph?

####Abstract Syntax Tree

Can be directly derived from the parse tree, all we have to do is eliminate all the symbols introduced in syntax analysis.

Capturing the hierarchy of the 

####Directed Acylic Graphs

Graphs that do not have cylces, every edge points from one node to another.

Advantages:
* if we use one of these, we can save space, because we can put a cycle where we have a repeated statement.

However, search (traversing the tree) can be problematic (slower) because we have to go in cycles sometimes.

####Control Flow Graph

Models the way that the code transfers control between blocks in the procedure.

* Node: a single basic block (a maximal straight line of code)
* Edge: transfer of control between basic blocks.
* (Captures loops, if statements, case, goto).

####Data Dependence Graph

Encodes the flow of data.

* Node: program statement
* Edge: connects two nodes if one uses the result of the other
* Useful in examining the legality of program transformations

####Call Graph

Show dependencies between procedures. Useful for inter-procedural analysis.

####Three-Address Code

	if (x > y) then z = x - 2 * y

becomes

	t1 = load x
	t2 = load y
	t3 = t1 > t2
	if not(t3) goto L
	t4 = 2 * t2
	t5 = t1 - t4
	z = store t5
	
	L

####Two Address Code

	LOAD	r1, x
	LOAD	r2, y

	CMP		r1, r2
	BLEQ	skip

	LOADI	r3, #2 			// load literal
	MULT	r2, r3, r2
	SUB		r1, r1, r2
	STR		z, r1			// assuming z is a memory address

	skip

####One address code

Basically similar to the above, but we just use the stack to push and pop variables..

####Symbol Table

Defines how types interact?? Captures information about all the useful symbol in the programme:
* variables.
* program names.
* labels.
* function names / subroutine names.
* everything that requires some memory space, or it effects control.
* once a compiler finds that there is a certain variable, contain all information associated with this variable in a centrally generated table (same block of memory? i.e: objects in Java stored in same memory block - makes garbage collection and referencing easier, perhaps..)

####Information compiler needs to know about each item:
* name.
* data type.
* declaring procedure (constructors?)
* storage information (where this is being held in memory? in Java this might be reference - hashed memory location, in C it might be an actual memory location).
* depending on what it is: perhaps the parameters if it's a function, etc.

####How can we organise a symbol table??

This determines the efficiency of compilation. It would be nice to have some kind of look-up table??

As a linear list - > BAD. It's going to be too big to linearly run through the list every time we want to check a symbol, your compilation time is going to be insane.

Binary Tree - > better? Good for searching, but not so great for re-structuring, we don't know how people are going to declare their variables, or structure their program. It's going to have to be rebalanced ALOT, and rebalancing is going to be computationally expensive.

Hash Table - > If we have a hash function, then we can map every name (id) to a different element of a the table. Which would be pretty good, right? Make a hash function which provides a unique hash for every different name and just not allow the reuse of the same name or reserved words. E.g: take the ASCII of each letter and perform some kind of unique hash function, which would be... I can't think of one / remember one off the top of my head; google search...

What if we don't have a unique hashing function..? If we get a collision, we can could create a linear list associated with that particular position. But the hash function has to be good and the table has to be quite a large container.

Alternative: Re-hashing, every time there is a collision, you can just rehash, this is equivalent to choosing a unique hash function, I believe...? Can you get unique hash functions..? Pretty sure it's accomplished through re-hashing anyway.

####Conclusion

Symbol table is an important data structure, and hash table is the best way of doing this. But we need a good hash function and we need to be able to resolve collisions.

##Lecture Thirteen: Procedure Abstraction, Run-Time Storage Organisation

Source Code -> FRONT END -> IR -> MIDDLE END -> IR -> BACK-END -> OBJECT CODE

We're talking about all kinds of stuff today (not really about compilers only):
* Interface of compilers
* OS
* Linkers
* Libraries
* Hardware

How is the compiler going to interact with the OS, in order to produce code that is going to respect what the OS is doing??

####The Procedure

Notions:
* functions / procedure / subroutine, etc..

What is a procedure (or function, method, etc..):

We need some kind of functions / methods to be able to write code.. From a procedure, we get a well defined entry and exit. At the entry point we can pass some parameters or arguments. The clear exit point returns either a value or just returns without a value. It's all well defined.

A procedure generally has it's own name space, so we can declare variables that are only live within that procedure.

The implications of translating procedures:
* We need to have some kind of agreement between the compiler, architecture and OS, since:
 * There needs to be a way of deciding where the memory will be allocated, etc..
 * How these procedures are going to be called, etc..

The compiler can compile procedures separately, however the code that the compiler generates should be in a position to be called by other programs.

From the HARDWARES point of view, the procedure is just an artificial creation, the hardware doesn't support anything that would make procedures possible. All the hardware knows is how to execute sequences of instructions. We have to think about what characterises a procedure.. 

Think about well defined ENTRY and EXIT, this implies the compiler will have to jump from one part of the program to somewhere else. This is fine, because all that will happen at run-time is that control will be transferred from one programme to another, so to speak..

We need to ensure that all of these qualities are implemented in the compiler. The abstraction that is supported by the high level languages needs to be translated to the hardware level, otherwise it's a wasted effort. How do we ensure this very nice abstraction is going to be translated in a way that really makes sense?

For example: we want to have name-spaces where variables defined within a certain scope are only available in that scope. This is easy to understand at a high level, but when we're actually translated this to the hardware level, how do we do it...? We're going to need to do a lot of book-keeping..

What we would like to do is to get the compiler to provide space at run-time for the variables that only going to be needed within a specified scope. Then when we enter into a new procedure, etc - we want to allocate some more space, then free it up again when we exit that procedure.

We use a convention to do this... The **Linkage Convention**.

####The Linkage Convention

At the beginning and at the end of a procedure some book-keeping is done. Space can be dynamically allocated at run-time, this space can then be removed once the procedure is exited (garbage collection in Java or free() in C, etc).

If at run-time we are in a position to allocate space only as it is needed, in modern systems, this is happening through activation records (stack frame, or a frame - some space in memory that will be reserved only at a call to a procedure); then it will be removed.

This stack frame includes:
* Parameters ()
* Save Area (we're going to need to save everything - values of registers - push everything onto another stack)
* space for return value
* address for resume (Link Register)
* access link - for non-local access to variable declarations (link to parent, or callee?)
* pointer to activation record of the caller
* space for local variables, temporaries, anything defined within this procedure.

What we really want is a compiler that will allocate space when a call to a function happens, then when it exits, it has to free up the memory.

**Calling a procedure has an overhead.**

####Procedure Linkages:

Caller (pre-call):
* Allocate Activation Record
* Evaluate Storage for Parameters
* Store Return Address
* Store self AR pointer
* store AR pointer to child
* jump to the child

callee (prologue):
* save current value of registers.
* extend AR for local data.
* get static data area base address.
* initialise those variables.
* fall through code.

callee (epiloge):
* Store return value
* restore registers and state
* deallocate what was extended here for local data
* restore the parents activation pointer.
* jump to the return address. 

Caller (post-return):
* copy return values
* deallocate the activation record for callee
* restore parameters

Pretty straight forward... Done a lot of this when writing assembly language..

*although.. what the fuck are activation records???*

Activation records: stack frames are otherwise known as activation records.. apparently.. (wikipedia)

####Placing Run-Time Data Structures:

Code | Static & Global | Heap ->> | <<- Stack

Most things to do with activation records are stored, usually, in the stack. Heap and stack grow towards each other. From a compilers point of view, the space for everything can be predicted and mapped accordingly to somewhere specific.

####Activation Record Details

How does the compiler find the variables?
* They are offsets from the AR base pointer.
* Everything is calculated as an offset from the AR base pointer.

Where do activation records live?
* If it makes no calls, then AR can be allocated statically.
* Place in heap if we need access to these ARs / AR attributes after EXIT, I.E global variables?
* Otherwise, whack it in the stack (implication: life-time of AR matches life-time of the invocation).

If AR makes no calls to any other procedures, then only one is active at a time. But what if a call makes a call to another procedure..? 

Efficiency wise: Static, Stack, Heap.

(Most efficient -> Least Efficient)

####Run-Time Storage Organisation

We can use either:
* Access links, which, if I understand correctly, basically every time you make a call to a function, from within a function, it throws everything on to the heap for storage, or allocates more memory on the stack??? Basically keeps stacking up, so there is a fair amount of overhead.
* Alternatively, use global display, that calculates an offset from the base AR, but also keep an array somewhere in memory that holds pointers to ARPs for each level. So basically, you're going to allocate memory in multiple locations for multiple AR, but keep the base pointer for each AR in an array, and just move through this array as you enter into new and exit from procedures..? I think..

**not too sure if I'm correct with the access links, I'll check with others when I go in on Monday to try and solidify my understanding of this**

####Other storage issues

* 32 bit, or 64 bit? Why does this matter -> because of the offset, right? Are we using a 4 bit offset, are we using a x bit offset? Essentially, what is the word boundary? Everything is arbitrary.
* Cache Performance: if two variables are in near proximity in the code, then it is convenient to store them in near proximity in the cache. Complex stuff.
* Conventional Wisdom: tight on registers: use access links, lot's of registers: use global display.
* memory to memory VS register to register.
* managing the heap: first-fit allocation with several pools for common sizes (usually powers of 2 up to page size). 
* object-orientated languages have complex name spaces.

####Conclusion / Finally...

* The compiler needs to emit code for each call to a procedure to take into account (at run-time) procedure linkage.
* The compiler needs to emit code to provide (at run-time) addressability for variables of other procedures.
* Inlining: the compiler can avoid some of the problems related to procedures by substituting a procedure call with the actual code for the procedure. There are advantages from doing this, but it may not be always possible (can you see why?) and there are disadvantages too.

####Lecture Fourteen: Examples Exercise

##Lecture Fifteen: Code Generation - Instruction Selection

Now, we're moving to the back end!

Instruction Selection.
* How do we convert intermediate representation into the target programme?
* How does the compiler decide what will be stored in registers and what should be placed in main memory, how do we optimise what stays where??
* Given we have some instructions, what is the right order for these instructions to execute?

If two instructions do not depend on each other, then they can be executed in any order. Modern processors have the ability to execute things in parallel.

What should be executed first? We need to optimise.. Ideally we would like the address all of the above problems as an optimisation problem. People tend to break down these problems.

We're going to look at:
* Instruction Selection.
* Register Allocation.
* Instruction Scheduling.

####Instruction Selection as a Problem:

The approach of mapping an Intermediate Representation onto low level instructions, we're talking about a pattern matching problem, consider an Add instruction at the intermediate level and we're mapping to a low level function, we're going to be implementing it as an ADD instruction, simple: Add -> Add. We can detect things like loops in the AST, in the same way we would recognise an ADD, we can recognise a loop and convert it to it's machine code equivalent.

So, we can issue multiple cycles at a time. An Add would be a single cycle, but a loop will be a multi-cycled set of instructions. So in Instruction Scheduling, there is an instruction latency, this is the time the instructions was issued until the time of the results of this instruction are available.

####Code Generation for Arithmetic Expressions

Example: X + Y

	LOAD r1, X
	LOAD r2, Y
	ADD	 R1, R1, R2
	STR  R1

The load is usually done using a base register and an offset:

	LOAD r1, [base, offset]

####Issues with Arithmetic Expressions

If the value of x is already in the memory somewhere, there's no point loading it in again. So how do we determine if it's already in the memory? Can potentially do a number of passes to work it out.

Which subtree do you evaluate first in a complex expression? Evaluate the most complex sub tree first.

If there are free registers available, should we use them? If all our registers are used, how do we determine which register do we select to switch out? (LRU?)

Multiplications and Divisions: The cost of multiplication should be several cycles, shift operations take just one cycle, can potentially optimise.

####Trading Register Usage with Performance

Two possible approaches of: W = W * 2 * x * y * z

	LOAD r1, @w 		
	MOV  r2, 2 			
	MULT r1, r1, r2 	
	LOAD r2, @x
	MULT r1, r1, r2
	LOAD r1, @y
	MULT r2, r1, r2
	LOAD r1, @z
	MULT r2, r1, r2
	STR  r1 			

The problem with this is that the programme is going to halt and wait for the values to be loaded into the registers (it's going to halt) until the values are available (due to the latency).

Approach 2:

	LOAD r1, @w
	LOAD r2, @x
	LOAD r3, @y
	LOAD r4, @z
	MOV  r5, 2
	MULT r1, r1, r5
	MULT r1, r1, r2
	MULT r1, r1, r3
	MULT r1, r1, r4
	STR  r1

Here, the LOAD instructions can be overlapped (remember the pipeline for OS year two), so we can save on instructions.

**Principal, do the things which have a higher latency as early as possible.**

&& and || operators, compilers will only evaluate one part (X) of the code and if it's false for an if(X && Y), because it's just wasted cycles.

##Lecture Sixteen: Register Allocation

* Assume RISC-like type of code.
* Makes use of **virtual registers** (virtual memory) - even though there are a limited number of **physical registers**.
* What is trying to be accomplished:
  * Produce correct K register code.
  * Minimise number of loads and stores (spill code) and their space.
  * The allocator must be efficient (no backtracking).

####Background

* Basic Block: Maximal length segment of straight line code.
* Local register allocation: within a single block.
* Global register allocation: across an entire procedure (multiple blocks).
* Allocation: choose what to keep in registers.
* Assignment: choose specific registers for values.
* Complexity: we need good heuristics, GOOD heuristics that are GOOD ENOUGH.

####Liveness and Live Ranges

Efficiently choosing which registers to allocate values to. The liveness is between definition and the last use before it is redefined. Live range is from variable definition to last use.

over all instructions in the basic block:
MAXLIVE is the maximum number of values live at an instruction, K.
if MAXLIVE <= k, then allocate is trivial.
if MAXLIVE > k, then values must be spilled into memory.

Live range: definition until the last time it is used. Applies to assembly and all programmes in any language.

When we're doing register allocation, we may decide that one live range is optimal 

If we are in a position where one live range starts at the end of another live range, we can subtract one from the live range, since it is possible to share registers (see lecture slide Example / Exercise 13-Apr). But all that matters in the exam, to be consistent.

**LIVE RANGES IS AN IMPORTANT CONCEPT IN REGISTER ALLOCATION - ALWAYS POPS UP IN EXAM, MAKE SURE YOU CAN CALCULATE IT**

####Register Allocation Schemes

#####Top Down (Local) Register Allocation

if we have 10 values, and only 5 registers on the processor (number of values to be stored > numbers of registers), we use two registers to transfer to and from memory. Using three registers, get the values that are used most often (most commonly used) and for the remaining 7 values, load and store from and to the memory when we need them.

Problem: what if the most commonly used values are only just used in the first half of the basic block, or they are just used in the second half of the basic block? It's not very efficient. Or perhaps, they are only just more commonly used than the other values.

#####Bottom Up Allocation

Load the most commonly used value into a register and use the remaining registers as needed. We have a pool of registers, and in this pool of registers, we strictly go with the live ranges.

Instruction 1: Get one from the pool.
Instruction 2: Get one from the pool...
...
Instruction 6: A live range terminates, return this register to the pool..
Instruction 7: A live range terminates, return this register to the pool..

If at some point the pool is empty, then try to store to the main memory, the register that is going to be used the furtherest in the future, because we don't need this register for quite a while.

##Lecture 17: Register Allocation via Graph Colouring

####Recap on last lecture

We have code that needs to allocate 30 values, we have 10 registers. What we are going to do, is we are going to maintain a pool of registers and then we are going to get one only when it is necessarily. It is only necessarily when we are going to assign a value.

At the point where we don't need the register any more (it's live range ends), the is returned to the pool. If at some point the pool has no more registers to offer, we look for the value that is not used until the longest time in the future, we push the value from this register into main memory, which we can restore again when we need it.

^^ Bottom Up Allocation: many people have argued that this algorithm gives good results.

There is another idea... Which is often used in practice, is because it allows us to formulate the problem of register allocation, using a well studied and well understood graph theory problem and then we can try and solve the graph theory problem and use a solution based on this...

####Register Allocation via Graph Colouring

How many colours do we need in order to colour this graph, in such a way that two neighbouring nodes do not share the same colour?

* The colouring of a graph is an assignment of colours to nodes, such that nodes connected by an edge have different colours.
* A graph is k-colour-able if it has a colouring with k-colours.

In our problem, the colours co-respond to registers. K = number of physical registers.

Principal: We have to take our live ranges that overlap cannot share the same register, and therefore they should be represented using a connection.

The interference Graph is derived from these live ranges, each register whose live rage overlaps with another register, then it should be represented as a node with an edge connected between them.

**REMEMBER: If we make the assumption that when one live range ends and another begins, we minus one from the live range and this principal needs to be carried into the interference graph.** 

Once we have produced this interference graph, we need to find the least number of colours which can be mapped to this graph, using graph colouring. Then we assign this value as K, the number of physical registers that we need.

####Top-Down Colouring

* Rank the live ranges.
 * possible ways of ranking: number of neighbours, spill cost, etc.
* Following the ranking to assign colours.
* If a live range cannot be coloured, then spill (store after definition, then load before each use) or split the live range.

####Bottom-Up Colouring

What really changes here is how we are going to assign the colours. There are some advantages to using this approach in practical scenarios.

We have a simple graph with only five nodes. We are trying to colour this with three colours. (See lecture slides).

1. Simplifying the graph
	* Pick any node, such that the number of neighbouring nodes is less than k (degree of the node), and put it on the stack.
	* Remove that node and all edges incident to it.
2. If the graph is non-empty (i.e., all nodes have k or more neighbours), then:
	* Use some heuristic to spill a live range; remove corresponding node; if this causes some neighbours to have fewer than k nodes goto step 1, otherwise repeat step 2.
3. Successively pop nodes off the stack and colour them using the first colour not used by some neighbour.
	* If a node cannot be coloured, leave it uncoloured (will have to spill).

(Observation: A graph having a node n with degree < k is k-colour-able iff the graph with node n removed is k-colour-able)

####Global Register Allocation via Graph Colouring

The idea: Live ranges that do not interfere can share the same registers.

The algorithm:
1. Construct global live ranges.
2. Build interference graph.
3. (try to) construct a k-colouring of the graph:
	* If unsuccessful, choose values to spill (on the basis of some cost estimates in each case) and repeat from start (spill placement becomes a critical issue).
4. Map colours onto physical registers.


####Considering Live Ranges

**LIVEIN and LIVEOUT**

* LIVEIN: LIVEIN(b): a value is in LIVEIN if it is defined along some path through the control-flow graph that leads to b and it is either used directly in b or it is in LIVEOUT(b).
* LIVEOUT: LIVEOUT(b): a value is in LIVEOUT if it is used along some path leaving b before being redefined, and it is either defined in b or is in LIVEIN(b).

Constructing a control flow diagram:
* first BB and last BB LIVEIN and LIVEOUT respectively is the empty set.
* If basic block has no successors, LIVEOUT(b) = empty.
* For all other basic blocks: LIVEOUT(b)= LIVEIN(s) for all immediate successors, s, of b in the control flow graph.
* A value is in LIVEIN(b):
	* if it is used before it is defined (if it is defined) in basic block b; or
	* it is not used, nor defined, but it is in LIVEOUT(b).

####Building the Interference graph

	for each basic block b
		LIVENOW(b)<-LIVEOUT(b)
		for each operation in b (in reverse order) of the type op lr1,lr2,lr3:
			for each live_range in LIVENOW(b) except lr2 and lr3:
				add an edge between live_range and lr1
			remove lr1 from LIVENOW(b)
			add lr2 and lr3 to LIVENOW(b)


##Lecture Eighteen: Instruction Scheduling

* The problem:
	* Given a code fragment for some target machine and the latencies for each individual operation, reorder operations to minimise execution time.
	* Recall: modern processors may have multiple functional units.
* The task:
	* Produce correct code; minimise wasted cycles; avoid spilling registers; operate efficiently.

Using instruction scheduling, we're trying to separate independent code blokes and which improves performance on machines with instruction pipelines. We're trying to:
* Avoid pipeline stalls by rearranging the order of instructions.
* Avoid illegal or semantically ambiguous operations (typically involving subtle instruction pipeline timing issues or non-interlocked resources.)

####Background

* Some operations have delay latency for execution. For example, loads and stores.

**Overview of a solution:**

Move loads back at least delay slots from where they are needed, but this increases register pressure (more registers may be needed) - recall the example in lecture 14 (slide 7).

Ideally, we want to minimise both hardware stalls and added register pressure.

If we have instructions that have a high latency, prioritise them, so we can overlap them in the processor pipeline, this allows us to overlap cycles, so essentially we're saving cycles.

Example: **(a+b) + c**

	LOAD r1, @a
	LOAD r2, @b
	ADD  r1, r1, r2
	LOAD r3, @c
	ADD  r1, r1, r3

Assuming loads are 3 cycles and adds are one cycle, we're going to have 9 cycles for this code, because the first two loads overlap in the pipline, so by the time we're at the first ADD instruction, we have processed four cycles. Then ADD is one cycle, that's five cycles, then another LOAD is three more cycles, that's eight cycles, then we had to STALL at this point, so we waited until r3 was loaded, then performed the ADD which was one more cycle = **9 cycles**. WHERE AS, the exact same can be done by:

	LOAD r1, @a
	LOAD r2, @b
	LOAD r3, @c
	ADD  r1, r1, r2
	ADD  r1, r1, r3

Here, the first three loads are overlapped in the pipeline, so in total we have FOUR (I know this seems weird at first, but it's because r1 and r2 have both been loaded in 4 cycles, and since ADD r1, r1, r2 DOES NOT depend on r3, we don't have to wait for r3 to be loaded, so we can execute the instruction after 4 cycles) cycles before we can execute the first add instruction. Then by the fifth cycle, r3 has it's value loaded and it's possible to perform the final ADD instruction, so in total this code, that does the exact same thing, will only take = **6 cycles**.

It makes sense to execute the things with the longest latencies first.

####Algorithmic Method

1. Build a precedence (data dependence) graph. (easy enough)
2. Compute a priority function for the nodes of the graph - we decide this.
3. Use list scheduling to construct, one cycle at a time:
	1. Use a queue of operations that are ready:
	2. At each cycle:
		1. Choose a ready operation and schedule it.
		2. update the ready queue.

This is just a heuristic. But it gives good results.

**Priority Function**

Assign to each node a weight equal to the longest delay latency (total) to reach a leaf in the graph from this node (include latency of current node).

weight_ i = latencyi + max(weight_all successor nodes)

*Do examples to get to understand this*

The higher the weight, the higher the importance to start with *this* instruction.

**Remember to READ THE QUESTION - if it says it can do two instructions per cycle, then it can do two instructions per cycle!!!!**

**Local List Scheduling**

	Cycle=1; Ready=set of available operations; Active={}
	While (Ready U Active != {})
		if (Ready!={}) then
			remove an op from Ready (based on the weight)
			schedule(op)=cycle
			Active=Active U op
		cycle=cycle+1
		for each op in Active
			if (schedule(op)+delay(op)<=cycle) then
				remove op from Active
				for each immediate successor s of op
					if (all operand of s are available) then
						Ready=Ready U s

This is forward list scheduling: start with all available operations; work forward in time (Ready: all operands available).

It is possible to do list scheduling in a backwards fashion: start with leaves; work backward in time (Ready: latency covers uses).

These two paradigms don't necessarily give the same outcome, so probably best to try both and keep the best result.

**Variations on computing priority function: (READ UP ON THESE FOR EXAM - COULD BE IN THERE)**
* Maximum path length containing node (decreases register usage).
* Prioritise critical path.
* Number of immediate successors or total number of descendants.
* Increment weight if node contains a last use (shortens live ranges)
* Do not add latency to node’s weight.
* Maximum delay latency from first available node.

**Instruction Scheduling interferes with Register Usage**

The more number of registers we use, the easier it becomes for instruction scheduling because there are fewer dependencies. If we try and use as few registers as possible, then we create more dependencies.

####Multiple Functional Units

* Modern architectures can run operations in parallel.
* List scheduling needs to be modified so that it can schedule as many operations per cycle as functional units (assuming that there are instructions available).

**THIS IS IMPORTANT IS IS INCLUDED IN THE EXAM - SEE PAPER 2014 for list scheduling question, two instructions are executed simultaneously using multiple functional units, usually have more NOP instructions**

As we increase the number of functional units, we're probably going to get more NOPS, so is it worth it? Depends on program an the architecture.

**It's also important in exercises where there are more than one functional units that we cannot run two instructions at the beginning because there are usually dependencies - see last but one lecture slide**

Optimisation can be based on multiple different things:
* optimising for power consumption (mobile devices).
* optimising for execution (number of cycles).
* optimising for size of the code.

##Lecture Nineteen: Code Optimisation

* Goal: improve program performance within some constraints (small sized code, power consumption, etc). <- HOW DO WE DEFINE OUR OBJECTIVE FUNCTION!? Usually via efficiency.
* Issues:
	* Legality: must preserve the meaning of the program.
		* Externally observable meaning may be sufficient/may need flexibility.
	* Benefit: must improve performance on average or common cases.
		* Predicting program performance is often non-trivial.
	* Compile-time cost justified: list of possible optimisations is huge.
		* Inter-procedural optimisations (O4).

If you test GCC by enabling optimisations, then the compilation time is going to be longer, moderately sized programme, not small but not huge.

####Optimising Transformations

How do the compilers apply the optimisations? They have different transformations and they apply these in some kind of a sequence: T1->T2->T3->...->T_N.

Transformations may improve program at:
 * Source Level
 * IR
 * Target Code

Some typical transformations:
 * Discover and propagate some constant value.
 * Remove unreachable / redundant computations.
 * Encode a computation in some particular efficient form.

For example, we might implement a multiplication as a number of additions and shifts. But this is machine-specific, because 32 bit, 64 bit, etc.

####Classification

* By Scope:
	* Local: within a single basic block.
	* Peep-hole: on a window of instructions (usually local).
	* Loop-Level: on one or more loops or loop nests.
	* Global: for an entire procedure.
	* Inter-procedural: across multiple procedures or whole program.
* By machine information used:
	* Machine-independent versus machine-dependent.
* By effect on program structure:
	* Algebraic transformations (e.g., x + 0, x * 1, 3 * z * 4, ...)
	* Reordering transformations (change the order of 2 computations)
		* Loop transformations: loop-level reordering transformations.

On average 90% of program execution is spent inside loops.

####Optimisation Transformations:

* Common subexpression elimination: searches for instances of identical expressions (i.e., they all evaluate to the same value), and analyses whether it is worthwhile replacing them with a single variable holding the computed value.
* Copy propagation: from a programmers point of view, it might be a good idea to implement two different variables for two separate purposes, that contain the same value, because it is sensical for maintenance. From a compilers point of view, there is no point, its a waste, because variables take registers, take space in memory, etc. We can not both using both variables, we just use one and propagate it as far as it is un-changed in the programme.
* Constant propagation: *Don't really get this.. constants should never change anyway?*
* Constant folding: compiler can compute literals such as 5*3+8-12/2 at compilation time, rather than run-time.
* Dead-code elimination: code that is redundant or is never going to be executed:

	if(3>7)
	{
		// Execute this
	}

The above is NEVER going to be executed, so we just remove it.

* Reduction in strength: Changing multiplication from a standard multiplication to additions and shifts to save on cycles.

Sometimes we need one transformation for the compiler to realise that another transformation is possible. For example, applying constant folding in order to apply constant propagation, in order to realise that dead-code can be removed:

T1->T2->T3

####Loop Transformations

* Loop-invariant code-motion:
	* Detect statements inside a loop whose operands are constant or have all their definitions outside the loop - move out of the loop.
* Loop interchange:
	* Interchange the order of two loops in a loop nest (needs to check legality): useful to achieve unit stride access.
* Strip mining:
	* May improve cache usage when combined with loop interchange.


####Loop Unrolling

Loops are interesting high level structures, but if we think about the low level implementation, they create problems, because a simple:

	for(int i = 0; i < 5; i ++)
	{
		// EXECUTE CODE
	}

We're doing so much for just one line of code. Declaring a variable, making a comparison, branching, incrementing, etc.

But we could just create the same loop body 5 times in a row and would be more optimal. Large Basic Blocks.

Compiler point of view, interesting transformations, has benefits. Increases the size of the code, but if the size of the code can be increase to gain a benefit in terms of run-time execution, then this is beneficial.

##Twenty: Additional Notes for Exam

Questions I'm likely to answer:
 1. DFA / NFA Question.
 2. Question on live ranges and graph colouring.
 3. Humm .......... Not sure!

**Need to go over these particular topics in detail!**

#References
1. Rizos Sakellariou (2015), Compilers Lecture Slides, University of Manchester.
2. James Power (2002), Parsing Lecture Notes, National University of Ireland, Maynooth.
3. Wikipedia (2015), Instruction Scheduling, http://en.wikipedia.org/wiki/Instruction_scheduling
4. Wikipedia (2015), Common subexpression elimination, http://en.wikipedia.org/wiki/Common_subexpression_elimination