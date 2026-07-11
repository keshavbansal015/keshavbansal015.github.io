---
title: "Building a Compiler: Tokenizer and Parser"
date: 2026-07-10
draft: false
tags: ["compiler", "tokenizer", "parser", "ast"]
summary: "Building a Regex Tokenizer and Parser from scratch in Python, transforming raw text into an Abstract Syntax Tree (AST)."
---

## From Numbers to Blocks — Building a Regex Tokenizer & Parser

In this post, we build a basic lexical scanner (Tokenizer) and a parser from scratch in Python, transforming raw text into an Abstract Syntax Tree (AST). We will start with numeric literals and progress to scoped code blocks.

---

## 1. Compiler Theory: Lexical vs. Syntactic Analysis

Compilers partition code translation into distinct phases. The first two are:
1. **Lexical Analysis (Tokenization)**: Converting a stream of characters into a stream of tokens.
2. **Syntactic Analysis (Parsing)**: Converting a stream of tokens into a structured syntax tree (AST) matching a language grammar.

### Question: Why can't we use Regex to parse an entire programming language?
* **Answer**: According to the **Chomsky Hierarchy**, regular expressions define **Regular Languages** (which are recognized by Finite Automata). Regular languages cannot keep track of arbitrary nesting (like nested parentheses `(((...)))` or nested blocks `{ { { ... } } }`) because finite automata have a fixed, finite state memory. Parsing nested structures requires a **Context-Free Language (CFL)**, which is recognized by a Pushdown Automaton (an automaton with a stack memory).

---

## 2. The Tokenizer (Lexical Analysis)

A tokenizer reads source code character-by-character and groups them into typed **tokens**. We map regular expressions to token types using Python's `re` module.

### Token Specification
Rules are evaluated in order. Whitespace and comments are mapped to `None` so they are discarded.

```python
import re

class Tokenizer:
    spec = [
        (r'^\s+', None),                  # Skip whitespace
        (r'^\/\/.+', None),               # Skip single-line comments
        (r'^\d+', 'NUMBER'),              # Numeric literals
        (r'^"[^"]*"', 'STRING'),          # String literals
        (r'^\{', '{'),                    # Block start
        (r'^\}', '}'),                    # Block end
        (r'^;', ';'),                     # Semi-colon
    ]
```

### The Scanning Loop
The scanner advances a cursor through the source string, matching rules at the current index.

```python
class Tokenizer:
    def initialize(self, string):
        self.string = string
        self.cursor = 0

    def getNextToken(self):
        if self.cursor >= len(self.string):
            return None
        
        slice_str = self.string[self.cursor:]
        for regex, token_type in self.spec:
            match = re.match(regex, slice_str)
            if match:
                value = match.group()
                self.cursor += len(value)
                if token_type is None:
                    return self.getNextToken()
                return {"type": token_type, "value": value}
        
        raise SyntaxError(f"Unexpected token: {slice_str[0]}")
```

### Question: What is the time complexity of tokenization?
* **Answer**: If implemented efficiently (e.g. using a single DFA), tokenization runs in **$O(N)$** time, where $N$ is the length of the input source string.

---

## 3. The Parser (Syntactic Analysis)

The parser reads the token stream and builds an **Abstract Syntax Tree (AST)**, which is a tree representation of the abstract syntactic structure of source code.

### Defining AST Nodes
AST nodes represent code constructs as JSON-like dictionary objects:

```python
def program_node(body):
    return {"type": "Program", "body": body}

def numeric_literal(value):
    return {"type": "NumericLiteral", "value": int(value)}

def string_literal(value):
    return {"type": "StringLiteral", "value": value[1:-1]}

def block_statement(body):
    return {"type": "BlockStatement", "body": body}
```

### Parsing Literals and Blocks
A recursive descent parser processes rules using recursive function calls.

```python
class Parser:
    def parse(self, string):
        self.tokenizer = Tokenizer()
        self.tokenizer.initialize(string)
        self.lookahead = self.tokenizer.getNextToken()
        return self.Program()

    def Program(self):
        return program_node(self.StatementList())

    def StatementList(self, stop_lookahead=None):
        statement_list = [self.Statement()]
        while self.lookahead and self.lookahead["type"] != stop_lookahead:
            statement_list.append(self.Statement())
        return statement_list

    def Statement(self):
        if self.lookahead["type"] == '{':
            return self.BlockStatement()
        return self.ExpressionStatement()

    def BlockStatement(self):
        self._eat('{')
        body = []
        if self.lookahead["type"] != '}':
            body = self.StatementList(stop_lookahead='}')
        self._eat('}')
        return block_statement(body)

    def ExpressionStatement(self):
        expression = self.Literal()
        self._eat(';')
        return {"type": "ExpressionStatement", "expression": expression}

    def Literal(self):
        token = self.lookahead
        if token["type"] == 'NUMBER':
            self._eat('NUMBER')
            return numeric_literal(token["value"])
        if token["type"] == 'STRING':
            self._eat('STRING')
            return string_literal(token["value"])
        raise SyntaxError(f"Unexpected literal: {token['value']}")

    def _eat(self, expected_type):
        token = self.lookahead
        if token is None:
            raise SyntaxError(f"Unexpected end of input, expected: {expected_type}")
        if token["type"] != expected_type:
            raise SyntaxError(f"Expected {expected_type}, got {token['type']}")
        self.lookahead = self.tokenizer.getNextToken()
        return token
```

### Example Walkthrough
Given the input:
```javascript
{
  42;
  "hello";
}
```

The output AST is:
```json
{
  "type": "Program",
  "body": [
    {
      "type": "BlockStatement",
      "body": [
        {
          "type": "ExpressionStatement",
          "expression": { "type": "NumericLiteral", "value": 42 }
        },
        {
          "type": "ExpressionStatement",
          "expression": { "type": "StringLiteral", "value": "hello" }
        }
      ]
    }
  ]
}
```
In the [next](/blogs/cfg_ll1_rdp) post, we will explore the mathematical foundations of parser theory: Context-Free Grammars (CFG), LL(1) constraints, and First/Follow sets.
