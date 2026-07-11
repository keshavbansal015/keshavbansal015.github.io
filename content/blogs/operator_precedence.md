---
title: "Operator Precedence and Associativity"
date: 2026-07-10
draft: false
tags: ["compiler", "operator precedence", "parser", "ast"]
summary: "Parsing algebraic math expressions with operator precedence and associativity."
---

# Resolving Operator Precedence & Associativity

In this post, we solve the challenge of parsing algebraic math expressions (like `2 + 3 * 4`) and assignment statements by addressing **Precedence** and **Associativity** in a recursive descent parser.

---

## 1. Precedence and Associativity

When parsing expressions, two main concepts determine the shape of the AST:
1. **Precedence**: Determines which operator executes first in a mixed expression. (e.g. `*` has higher precedence than `+`, so `2 + 3 * 4` parses as `2 + (3 * 4)`).
2. **Associativity**: Determines grouping order for operators of the *same* precedence level.
   * **Left-Associative**: Groups left-to-right. (e.g. Subtraction: `x - y - z` evaluates as `(x - y) - z`).
   * **Right-Associative**: Groups right-to-left. (e.g. Exponentiation or Assignment: `x = y = 5` evaluates as `x = (y = 5)`).

---

## 2. Removing Left-Recursion Mathematically

In formal grammar, left-associative binary expressions are written using **left-recursive** rules:
$$A \rightarrow A\,\alpha \mid \beta$$
However, top-down parsers (like LL) will loop infinitely on left-recursion. To fix this, we eliminate left-recursion by rewriting the rule into right-recursive form:
$$A \rightarrow \beta\,A'$$
$$A' \rightarrow \alpha\,A' \mid \epsilon$$

In an iterative parser implementation, this translates directly to a `while` loop structure:
```python
# Before (Left-Recursive): AdditiveExpression -> AdditiveExpression + MultiplicativeExpression
# After (Iterative): AdditiveExpression -> MultiplicativeExpression (+ MultiplicativeExpression)*
```

### Implementing Left-Associativity (Iterative Loop)
For left-associative operators like `+` and `-`, we use a `while` loop:

```python
    def AdditiveExpression(self):
        left = self.MultiplicativeExpression()

        while self.lookahead and self.lookahead["type"] in ('ADDITIVE_OPERATOR',):
            operator = self._eat('ADDITIVE_OPERATOR')["value"]
            right = self.MultiplicativeExpression()
            
            # Left-associative: nest the previous sub-tree on the left
            left = {
                "type": "BinaryExpression",
                "operator": operator,
                "left": left,
                "right": right
            }
        return left
```

### Implementing Right-Associativity (Right Recursion)
For right-associative operators like assignment `=`, we recurse on the right-hand side rather than looping:

```python
    def AssignmentExpression(self):
        left = self.LogicalORExpression()

        if self.lookahead and self.lookahead["type"] == 'ASSIGNMENT_OPERATOR':
            operator = self._eat('ASSIGNMENT_OPERATOR')["value"]
            # Recurse on AssignmentExpression to group right-to-left
            right = self.AssignmentExpression()
            
            return {
                "type": "AssignmentExpression",
                "operator": operator,
                "left": left,
                "right": right
            }
        return left
```

---

## 3. Comparing Parser Architectures

| Parser Type | Direction | Parsing Strategy | Handles Left-Recursion? | Key Features |
| :--- | :--- | :--- | :--- | :--- |
| **Recursive Descent (RDP)** | Top-Down | LL(1) | No (requires rewriting grammar) | Simple to hand-write, excellent errors, easy to step-debug. |
| **Pratt Parser** | Top-Down | TDOP (Top-Down Operator Precedence) |  Yes | Associatively maps binding power integers directly to tokens. Highly dynamic. |
| **Shift-Reduce (LR/LALR)** | Bottom-Up | Shift, Reduce, Accept, Error |  Yes | Uses parsing tables (DFA + stack), handles a wider range of grammars, but harder to write by hand (requires parser generators). |

### Question: What is a Shift-Reduce conflict?
* **Answer**: In bottom-up parsing, a shift-reduce conflict occurs when the parser's state transition table cannot determine whether it should **shift** the next incoming token onto the stack, or **reduce** the current top of the stack into a non-terminal. This usually indicates an ambiguous grammar (like the Dangling Else).
