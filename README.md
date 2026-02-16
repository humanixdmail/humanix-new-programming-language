***NOTE: IF YOU ARE FACING ANY ISSUES, PLEASE CONTACT - humanix.dmail.company@gmail.com***



# Humanix — Small Python-implemented Systems Language

Status: Working prototype (lexer → parser → interpreter)  
Current date: 2026-02-05

Our website: humanix-v1.netlify.app

This document summarizes the Humanix project as it stands: what the repository contains, how the toolchain works, supported syntax and features, how to run and debug, known limitations, and suggested next steps.

---

## Project overview

Humanix is a small, Python-implemented language prototype with a Pythonic syntax and a minimal runtime. The implementation is structured as a classic compiler/interpreter pipeline:

- Lexer — tokenizes source text into tokens
- Parser — produces an AST from tokens
- AST — node definitions used by the interpreter
- Interpreter — executes the AST in an environment with functions, structs, and builtins
- Runner / CLI / REPL — user entry points for running files or interactive code

The emphasis so far is correctness and clarity (easy-to-follow pipeline) rather than full language completeness.

---

## Repository contents (high-level)

- `main.py` — top-level CLI wrapper (runs file or REPL)
- `humanix/`
  - `lexer.py` — lexical scanner (tokens, comments, shebangs, f-strings, numbers, strings)
  - `tokens.py` — `TokenType` enum and `Token` class (canonical token names)
  - `parser.py` — recursive-descent parser producing AST nodes
  - `ast.py` — dataclasses for AST nodes (Stmt, Expr, Program, etc.) and visitor interfaces
  - `interpreter.py` — tree-walking interpreter implementing runtime semantics
  - `errors.py` — error classes and pretty-print helpers
  - `runner.py` — file runner + REPL with debug support
  - (sample programs you saved during testing)
- (optional test helpers / scripts you may add)

---

## How it works (pipeline)

1. `Lexer` reads source string and produces a list of `Token` objects (one final `EOF` token).
   - Supports: identifiers, keywords, numbers (integers, floats, scientific notation), strings, f-strings (no interpolation yet), single- and multi-character operators, comments (`//`), shebang (`#!`), whitespace handling, line counting.

2. `Parser` (recursive-descent) consumes the token stream and builds an AST:
   - Top-level declarations: `let`, `fn`, `struct`
   - Statements: `if`, `return`, block statements, expression statements
   - Expressions: binary/unary ops, grouping, variables, literals, calls, member get/set, assignments, precedence handling
   - Type annotations are parsed (advisory): `int`, `f64`, `string`, `bool`, and `None`.

3. `AST` defines `Stmt` and `Expr` dataclasses and `Visitor` / `ExprVisitor` interfaces used by the interpreter.

4. `Interpreter` implements the visitor interfaces:
   - `Environment` holds variables/function bindings, supports nested scopes
   - `HumanixCallable` wraps function declarations with closure environment
   - `StructInstance` represents struct objects with named fields
   - Builtins: currently `println(interpreter, args)`
   - Type metadata: `HumanixValue` has `.value` and `.hx_type` (simple string type tags)
   - `ReturnValue` is a control-flow exception used internally to return from functions

5. `Runner` and `main.py` provide CLI and REPL:
   - `python main.py <file.hx>` runs a file
   - `python main.py` starts the REPL
   - `--debug` flag prints tokens and parsed-statement counts

---

## Files and responsibilities (brief)

- `lexer.py`
  - Tokenization logic, keyword map, f-string recognition (as HX_FSTRING literal), whitespace/comments, numeric/string literal decoding.
- `tokens.py`
  - `TokenType` enum (keywords, punctuation, operators, types) and `Token` class.
- `parser.py`
  - Recursive-descent parser implementing language grammar and precedence; returns `Program` AST.
- `ast.py`
  - Dataclasses for program, statements, expressions and the visitor interfaces used by the interpreter.
- `interpreter.py`
  - Interpreter visitor that executes statements and evaluates expressions; runtime environment and builtins.
- `errors.py`
  - Error types (LexerError, ParserError, HumanixRuntimeError) and `print_error` for pretty output.
  - `ReturnValue` is a non-error control-flow exception.
- `runner.py`
  - Runner for files and REPL, debug flag support, top-level error handling.
- `main.py`
  - CLI entrypoint, argument parsing, printing banner.

---

## Supported language features (current)

- Declarations:
  - `let <name>[: type] = <expr>;` or `let <name>[: type];` with optional initializer
  - `fn <name>(params) -> type { ... }` — functions
  - `struct <Name> { field: type, ... }`
- Statements:
  - Blocks `{ ... }`, `if` / `else`, `return`, expression statements (must end in `;`)
- Expressions:
  - Literals: integers, floats, strings, f-strings (no interpolation), `true`, `false`, `None`
  - Variables, grouping `(...)`, unary `!`, `-`, binary `+ - * / %`, comparisons `> >= < <=`, equality `== !=`, logical `&& ||`
  - Function calls: `f(a, b)` (arity-checked for user functions)
  - Member access and assignment: `obj.field` and `obj.field = value`
  - Assignments to variables: `x = expr` (target validation)
- Builtins:
  - `println(...)` — prints argument or newline if no args
- REPL:
  - Persistent environment across REPL lines, commands `exit`, `clear`, `help`
  - Multi-line entry is not implemented by default (REPL accepts single-line statements — multi-line functions require entering full braces on one line or pasting with care)

---

## Syntax examples

Save these examples as `.hx` files and run with `python main.py <file.hx>`.

- hello_arithmetic.hx
````markdown
```hx
fn main() -> None {
    println(5 + 7);
}

```





