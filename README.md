# Compilers : COMP36512

Going to be making my notes in this markdown file, plus also maybe throwing some basic implementations written in python for lexing, parsing, etc (if I'm feeling adventurous).

## Index

1. Introduction
2. General Structure of a Compiler

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

Front End:

 1. Lexical Analysis (scanning): produces a set of tokens.
 2. Syntax Analysis (parsing): analyses the produced set of tokens and produces an **Abstract Syntax Tree (AST)**.
 3. Semantic Analysis: annotates the AST.
 4. Intermediate Code Generation: 

Back End:

 5. IR Optimisation: 
 6. Code Generation: 
 7. Target Code Optimisation: 
 8. Target Code Generation:

## Lecture Three: Lexing

Lexing: reads characters and provides a sequence of tolkens.