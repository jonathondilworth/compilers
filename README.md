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

<UPDATE THIS>

#References
1. Rizos Sakellariou (2015), Compilers Lecture Slides, University of Manchester.
2. James Power (2002), Parsing Lecture Notes, National University of Ireland, Maynooth.