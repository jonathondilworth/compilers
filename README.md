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


#References
1. Rizos Sakellariou (2015), Compilers Lecture Slides, University of Manchester.
2. James Power (2002), Parsing Lecture Notes, National University of Ireland, Maynooth.