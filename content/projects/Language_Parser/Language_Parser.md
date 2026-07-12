---
title: "Language Parser"
date: 2026-07-10
summary: "A custom LL(1) recursive descent parser and tokenizer built in Python."
draft: false
tags: ["Parser", "Python", "Compiler"]
image: "/projects/Language_Parser/parser.jpeg"
---

# Language Parser
[Repository](https://github.com/keshavbansal015/Language_Parser)

A custom LL(1) recursive descent parser and tokenizer built in Python. It parses a JavaScript-like syntax (including classes, functions, variables, control flow structures, and operations) and produces an AST (Abstract Syntax Tree) representation.

## Features

- **Class Declarations**: Supports `class` declarations, `extends` syntax, constructor definitions, member assignments, and `super()` calls.
- **Functions & Control Flow**: Supports function definitions (`def` syntax), `if-else` conditionals, `while` and `do-while` loops, and loop control statements (`break`, `continue`).
- **Variables & Scopes**: Parsed variables (`let`, `const`, `var`), assignments, and references.
- **Expressions & Operators**: Parses member access expressions (`this`, `super`, `object.property`), logical operators, relational comparisons, arithmetic equations, and update operators (`++`, `--`).
- **AST Generation**: Translates code segments into standard AST structures (similar to ESTree format).

---

## Directory Structure

```
├── src/
│   ├── Tokenizer.py   # Regex-based scanner for input source code
│   └── Parser.py      # LL(1) parser converting token streams into AST nodes
├── tests/
│   └── test_parser.py # Comprehensive unit test suite (80 test cases)
├── cli.py             # Command line utility to run the parser
├── example.txt        # Example code snippet for testing classes
└── README.md
```

---

## Command Line Interface (CLI)

You can use the CLI to parse source code from a file or directly from an expression.

### CLI Usage

```bash
python3 cli.py -f <path_to_file> [-o {json,str,raw}]
python3 cli.py -e <expression> [-o {json,str,raw}]
```

### CLI Arguments

- `-f`, `--file`: Path to the source file to parse.
- `-e`, `--eval`: Code expression string to parse directly.
- `-o`, `--format`: Output formatting options:
  - `json` (default): Pretty-printed JSON AST.
  - `str`: String representation.
  - `raw`: Raw Python dictionary representation.

### Examples

**1. Parse a file:**
```bash
python3 cli.py -f example.txt
```

**2. Evaluate an expression directly and get the raw representation:**
```bash
python3 cli.py -e "let x = 10;" -o raw
```

Output:
```python
{
    'type': 'Program',
    'body': [
        {
            'type': 'VariableStatement',
            'declarations': [
                {
                    'type': 'VariableDeclaration',
                    'id': {'type': 'Identifier', 'name': 'x'},
                    'init': {'type': 'NumericLiteral', 'value': 10}
                }
            ]
        }
    ]
}
```

---

## Running Tests

Run the complete test suite containing 80 test cases:

```bash
python3 -m unittest discover tests
```


** This project is based on the course by [dmitrysoshnikov](https://dmitrysoshnikov.com)