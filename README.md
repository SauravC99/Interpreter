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
After building, you will have an executable file named `interpreter` (or `interpreter.exe` on Windows).
Run it to start the interactive REPL:
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


### Tokens

Tokens are small, easily categorizable data structures.
This interpreter implements tokens as a data structure with fields that hold the type of token it is (int, minus, semicolon, L or R bracket), and the literal char(s) which make up that token.
```go
type Token struct {
	Type    TokenType // TokenType -> string
	Literal string
}
```

Here is an example of lexing a statement which contains multiple different symbols (formatted for readability):
```
>> let result = fn(x, y) {x + 99 / y == x * "three" - !true};

{Type:LET       Literal:let}
{Type:IDENT     Literal:result}
{Type:=         Literal:=}
{Type:FUNCTION  Literal:fn}
{Type:(         Literal:(}
{Type:IDENT     Literal:x}
{Type:,         Literal:,}
{Type:IDENT     Literal:y}
{Type:)         Literal:)}
{Type:{         Literal:{}
{Type:IDENT     Literal:x}
{Type:+         Literal:+}
{Type:INT       Literal:99}
{Type:/         Literal:/}
{Type:IDENT     Literal:y}
{Type:==        Literal:==}
{Type:IDENT     Literal:x}
{Type:*         Literal:*}
{Type:STRING    Literal:three}
{Type:-         Literal:-}
{Type:!         Literal:!}
{Type:TRUE      Literal:true}
{Type:}         Literal:}}
{Type:;         Literal:;}
```
All of these tokens have the type of token in the `Type` field, along with the original source code representation attached in the `Literal` field.
For example with the variable name `result` the token generated is of type identifier with the literal field being the variable name.
As you can see, whitespace and newline characters are not converted into tokens, as whitespace length is not significant in this programming language.
In other languages, like Python, length of whitespace is significant.



## Parsing

The Parser is the component which takes the sequence of tokens produced by the Lexer and builds a data structure, such as a tree or some other hierarchical representation.
It checks the structure against the grammar rules (most commonly in Backus-Naur Form) of the programming language to ensure the tokens form valid constructs.
It also checks for correct syntax in the process.
The data structure it generates for the internal representation of the source code is called a syntax tree.
So, parsers take source code as input and produce a data structure that represents the source code.
While building the data structure, parsers also analyze the input, checking that it fits to the expected structure.
Because of this, parsing is also called syntactic analysis.


### Abstract Syntax Tree

This Parser generates an Abstract Syntax Tree.
It is abstract since it omits certain details which are visible in the source code such as whitespace, newlines, semicolons, comments, braces, brackets, and parentheses.
These details are not represented in the Abstract Syntax Tree, but they help guide the Parser when constructing it.

Here is a simplified example of an Abstract Syntax Tree for the statement `let x = 10 + 5;`:
```
          --------
		  | root |
		  --------
		     |
           -------
		   | let |
		   -------
		   /     \
	   -----   --------------
	   | x |   | expression |
	   -----   --------------
	            /     |     \
			------  -----  -----	
			| 10 |  | + |  | 5 |
			------  -----  -----
```


### Recursive Descent Parsing (Pratt parsing)

This Parser is a recursive descent parser, more specifically a top down operator precedence parser.
It is also known as a Pratt parser, named after its creator Vaughan Pratt.
The main idea behind a Pratt parser is to associate parsing functions with token types.
When a certain token is found, the parser calls specific parsing functions to parse the expression and returns an Abstract Syntax Tree node which represents it.
This interpreter implements that idea with maps for prefix and infix functions inside the parser structure:
```go
prefixParseFns map[token.TokenType]prefixParseFn
infixParseFns  map[token.TokenType]infixParseFn
```

The maps associate `TokenType` which is the token identifier, with `prefixParseFn` or `infixParseFn` types. These types are defined as:
```go
type (
	prefixParseFn func() ast.Expression
	infixParseFn  func(ast.Expression) ast.Expression
)
```

`prefixParseFn` is any function that returns an Expression node from the Abstract Syntax Tree. This would be `-5` or `!true`.

`infixParseFn` is any function that takes an Expression node as an argument and returns an Expression node. This would be `1 + 2` or `x * y`.

A recursive descent parser uses the strategy of top down parsing, where it starts by constructing the root node of the Abstract Syntax Tree and then works its way down.
The operator precedence, also known as order of operations, describes which priority the different operators have.
As we can see in this example:
```
10 + 10 * 10
```
The answer should be `110` instead of `200` because the `*` operator has a higher precedence than the `+` operator.
The `*` operator is more important than the `+` operator so it gets evaluated before the other.
This interpreter implements operator precedence as follows:
```go
const (
	LOWEST
	EQUALS      // ==
	LESSGREATER // > or <
	SUM         // + or -
	PRODUCT     // * or /
	PREFIX      // -X or !X
	CALL        // myFunction(X)
	INDEX       // array[index]
)
```

The operators are in ascending order.
`LOWEST` is the lowest and default precedence with a value of `1` and they increase in precedence as we go down the list.
This allows us to correctly parse expressions which contain different operators:

```
>> let result = fn(x, y) {x + 99 / y == x * "three"};
let result = fn(x, y) ((x + (99 / y)) == (x * three));

>> let z = x * y / 3 - 5 * 8 + 9898;
let z = ((((x * y) / 3) - (5 * 8)) + 9898);

>> let x = 6 - 5 + 4 * 3 / 2 + 1 < 20;
let x = ((((6 - 5) + ((4 * 3) / 2)) + 1) < 20);
```
The `*` and `/` operators have a larger precedence than the `+` and `-` operators.
The recursive nature of this parser means those operators are nested deeper in the Abstract Syntax Tree and will be evaluated earlier, maintaining the correct order of operations.



## Evaluating

The Evaluator is the component that takes the Abstract Syntax Tree built by the Parser and interprets it.
It is implemented as a tree-walking interpreter, where it traverses the Abstract Syntax Tree, visits each node and evaluates them.
This interpreter does it with a recursive function called `Eval(node)`.
Here it a simplified pseudocode version:
```go
func Eval(node) {
	if node == integer {
		return node.intValue
	}
	else if node == boolean {
		return node.boolValue
	}
	else if node == infixExpression {
		leftSide = Eval(node.left)
		rightSide = Eval(node.right)

		if node.operator == + {
			return leftSide + rightSide
		}
		else if node.operator == - {
			return leftSide - rightSide
		}
	}
}
```

As the Evaluator traverses the Abstract Sytnax Tree, it will call `Eval(node)` and execute different actions based on the type of node it is on.
For example, if the node the Evaluator is currently on is an integer or bool, it will return an internal representation of that integer or bool.
If the current node is an infix expression, such as `2 + 3`, the Evaluator will recursively call `Eval(node)` on the left and right sides of that expression, resolve the numbers to their internal representation, and finally add them together.
The final call to `Eval(node)` will return the final result.
Since the function is recursive, it will also work for more complex expressions, such as `2 + 3 * 4 - 5`.
This is because the function recursively calls itself two times to evaluate the left and right sides, which can lead to the evaluation of another infix expression or integer or boolean or so on.
This will continue until the final topmost call to `Eval(node)` returns the result.
