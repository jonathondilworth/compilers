<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>

# Compilers : COMP36512

Going to be making my notes in this markdown file, plus also maybe throwing some basic implementations written in python for lexing, parsing, etc (if I'm feeling adventurous).

## Index

1. Introduction
2. General Structure of a Compiler
3. Introduction to Lexical Analysis

## Lecture One: Introduction

A compiler takes some source code, and produces an output in another language, while the meaning of the source.

The compiler must:
 * Generate correct code.
 * Recognise errors.
 * Analyses and Synthesise.

The properties of a good compiler:
 * Generates correct code.
 * Generates fast code (optimisations).
 * Conforms to specifications of input language (standardisations).
 * Should be able to cope with arbitrary sized input (e.g: large number of variables, so forth).
 * Compilation time should scale linearly to the size of the source program.
 * Good diagnostics.
 * Consistent optimisations => similar logic in all instances.
 * Works with debugger.

Other issues:
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

### Front End:

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

### Back End:

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

### Formal Language Lingo:
 * Vocabulary: finite set of symbols.
 * String: finite sequence of symbols.
 * Language: any set of strings over a fixed vocabulary.
 * Grammar: finite way of describing a language.
 * Context Free Grammar: G=(S,N,T,P):
	*. S - Starting Symbol.
	*. N - set of non-terminal symbols.
	*. T - set of terminal symbols.
	*. P - set of production rules.

In order to construct a set of tokens during lexical analysis, we need to be able to recognise tokens (patterns) and we can identify an individual token using a CFG as above. Some tokens are easy to identify, e.g: white space (it can't really be much besides spaces and tabs, etc), but some tokens are going to be inherently more complex (e.g: floating point numbers?).

### Regular Expressions