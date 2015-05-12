<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

#Compilers : COMP36512

Going to be making my notes in this markdown file, plus also maybe throwing some basic implementations written in python for lexing, parsing, etc (if I'm feeling adventurous).

##Index

1. Introduction
2. General Structure of a Compiler
3. Introduction to Lexical Analysis
4. Lecture Four: From REs to DFAs

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

####Key ideas for Thompson's Construction are outlined in an image in this Repo.

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

###The Subset Construction Algorithm

The functions 'Move' and 'e-closure' are foundational to the subset construction algorithm, as described below:

1. Create the start state of the DFA by taking the e-closure of the start state of the NFA.
2. Perform the following for the new DFA state:
	* For each possible input symbol:
		1. Apply 'Move' to the newly-created state and the input symbol; this will return a set of states.
		2. Apply the e-closure to this set of states, possibly resulting in a new set (This set of NFA states will be a single state in the DFA).
3. Each time we generate a new DFA state, we must apply step 2 to it. The process is complete when applying step 2 does not yield any new states.
4. The finish states of the DFA are those which contain any of the finish states of the NFA.





	// initialise the algorithm, by defining the initial state of the NFA that we're converting to DFA.
	Initial_NFA_State := state 0
	Set_of_DFA_States := {}

	// The first DFA state is always the result of the e-closure function on the first NFA state
	Set_of_DFA_States.add( e-closure(Initial_NFA_state) ) 

	// perform construction of the remaining states: