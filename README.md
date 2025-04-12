# Interpreter

> An interpreter for a js-like programming language written in Go 



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
- Functions, Calls
- Conditionals
- Prefix and Infix expressions
- Array data structure
- Hash data structure
- Built in functions



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
You can also use the build command and build the project after cloning:
```
cd Interpreter/
go build -o interpreter
```
After building, you will have an executable file named `interpreter` (or `interpreter.exe` on Windows). Run it to start the interactive REPL:
```
./interpreter
```


### Usage

Once the interpreter is running, you can interact with it using the REPL (Read-Evaluate-Print-Loop).
Here is an example of creating and calling a function and some arithmetic:
```
>> let sum = fn(x, y, z) {x + y + z};
>> print(sum(3, 5 * 4, 8 - 6));
25
null

>> print(3 + 2 * 4 - 5)
6
null
```
The syntax is similar to C and JavaScript, making it easy to pick up and start using.



## Lexing

The Lexer, also known as scanner or tokenizer, is the component that performs lexical analysis to turn source code into tokens.
Lexing is the first step in the process of interpreting code.
While plain text is easy to work with in an editor, it becomes tedious really fast when trying to interpret it as a programming language.
This means we need to represent the source code in another form that is easier to work with, such as tokens.
The lexer also removes irrelevant details in the source code such as comments or whitespace, depending on the programing language specification, further increasing ease of working with it.
Later these tokens will be fed into the parser, which transforms them again and turns the tokens into an Abstract Syntax Tree.
After that, the evaluator will traverse the Abstract Syntax Tree and interpret it to evaluate the result.
```
---------------        ----------        -------
| Source Code | -----> | Tokens | -----> | AST |
---------------        ----------        -------
                Lexer             Parser
```
This interpreter implements the lexer data structure as follows:
```go
type Lexer struct {
	input        string // source code
	position     int    // current position in input (current char pos)
	readPosition int    // current reading position (after current)
	ch           byte   // current char looking at
}
```
The `input` field holds the source code to be lexed, the `position` and `readPosition` fields are pointers to positions in the input, and the `ch` field holds the actual character from the input.


