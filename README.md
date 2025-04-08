# Interpreter

> An interpreter written in Go for a js-like programming language



## Overview

This interpreter is an inplementation of the Monkey programming language.
It follows core steps to turn the input code into something that can be run and evaluated.
We start with the lexer which turns the input code into tokens.
These tokens then are parsed through the parser to generate an Abstract Syntax Tree (AST).
After this the evaluator walks through and executes the AST to produce the desired result.
These steps are handled by the REPL (Read-Evaluate-Print-Loop) component which also provides an interactive environment.

This interpreter supports a variety of programming language features:
- Variable bindings
- Integers, Booleans, Null, Strings
- Arithmetic expressions



## Installation

To install and run you will need Go installed on your system. Clone this repository:
```
git clone https://github.com/SauravC99/Interpreter.git
```
Start the interactive REPL with this command:
```
cd Interpreter/
go run main.go
```


### Usage


## Lexing
