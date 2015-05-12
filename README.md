<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

# Compilers : COMP36512

Going to be making my notes in this markdown file, plus also maybe throwing some basic implementations written in python for lexing, parsing, etc (if I'm feeling adventurous).

## Index

1. Introduction
2. General Structure of a Compiler
3. Introduction to Lexical Analysis
4. Lecture Four: From REs to DFAs

## Lecture One: Introduction

A compiler takes some source code, and produces an output in another language, while the meaning of the source.

#### The compiler must:
 * Generate correct code.
 * Recognise errors.
 * Analyses and Synthesise.

#### The properties of a good compiler:
 * Generates correct code.
 * Generates fast code (optimisations).
 * Conforms to specifications of input language (standardisations).
 * Should be able to cope with arbitrary sized input (e.g: large number of variables, so forth).
 * Compilation time should scale linearly to the size of the source program.
 * Good diagnostics.
 * Consistent optimisations => similar logic in all instances.
 * Works with debugger.

#### Other issues:
 * Speed (of compiled code).
 * Space (we don't want our executable to be larger than the input).
 * Feedback.

Which issues do we prioritise? It is use case dependant.

## Lecture Two: General Structure of a Compiler

Two Major Phases: 
 * Front End: Analysis.
 * Back End: Synthesis.

Source -> (analysis) -> Compiler -> (Synthesis) -> Target.

The components of a compiler are outlined below:

#### Front End:

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

#### Back End:

 5. IR Optimisation: Optimise the intermediate code.
 6. Code Generation: Map AST into a linear list of target machine instructions, in a symbolic form:
 	a. Instruction Selection: Pattern Matching.
 	b. Register Allocation: NP-Complete Problem.
 	c. Instruction Scheduling: NP-Complete Problem.
 7. Target Code Optimisation: Machine code information required by the OS is generated.
 8. Target Code Generation: Machine code and associated information required by the Operating System are generated.

## Lecture Three: Introduction to Lexical Analysis

Natural languages have 'high degrees of freedom', as humans we're able to interpret words (symbols) in numerous ways, they're potentially ambiguous and based on their contextual confines (this is also arbitrary, because every person has their own internal dictionary and languages are not always set in stone, so to speak).

However, in a formalised language, such as a programming language, a 'high degree of freedom' isn't usually a good thing (although there is a trade off and you have some languages that are loosely typed, where as others are strongly typed - the degree of freedom which you design your language to have may affect the complexity of the implementation of the compiler / interpreter).

#### Formal Language Lingo:

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

#### Regular Expressions

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

## Lecture Four: From REs to DFAs

* REs can describe regular languages
* Every RE can be converted into an NFA (Thompson's Construction).
* Every NFA can be converted into a DFA (Hopcroft's algorithm).
* DFAs can automate the construction of lexical analysis.

Non deterministic finite automaton (NFA) are possible if one or more transitions exist from one state to another state under the same transition criteria (match).

Deterministic finite automaton will describe a transition for every possible input, and will always terminate. Every NFA can be described using a DFA by isolating those states that creates the non deterministic property from the transitions table and constructing a deterministic variant.

#### Key ideas for Thompson's Algorithm are outlined in an image in this Repo.

#### NFA to DFA, Two Key Functions:

1. Moves(STATE ID, INPUT), returns a list of the states possible to reach from the STATE ID, using the INPUT to traverse the automaton.
2. Epsilon_closeure(STATE ID), returns a list of the states possible to reach from the STATE ID only using empty transitions, essentially it's a recursive function:
'''
	Epsilon_closeure(STATE ID):
		Collection of reachable states += Moves(STATE ID, EMPTY TRANSITION)
		For each reachable state in reachable states:
			Epsilon_closeure(reachable state)

		Return reachable states
'''

As a side note that I thought was kind of interesting while coming up with this recursive algorithm, if we were to replace the collection of reachable states with a list, we could potentially end up with a infinite loop in our program (well actually, it would eventually terminate when we had a stack overflow, or ran out of memory or whatever, but you see what I', saying) theoretically it **could** never end since the NFA could be a cyclic graph, so we had better use a collection.

*IDEA: Make an implementation in python that takes a regular expression from the user, constructs an NFA from this expression, converts this NFA into a DFA and then map this out using networkx and matplotlib python libraries - hopefully I'll have time to do this, because I think this could be pretty fucking cool! Why do I have to revise when I could be spending my time doing cool things? :( Wait a minute... wouldn't that actually be the implementation of a lexical analyser? Will come back to this point soon..*

