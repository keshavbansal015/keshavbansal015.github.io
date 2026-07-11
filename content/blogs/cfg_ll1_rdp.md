---
title: "CFG, LL(1) and RDP"
date: 2026-07-10
draft: false
tags: ["compiler", "CFG", "LL(1)", "Recursive Descent Parsing"]
summary: "Demystifying CFG, LL(1) parsing, and Recursive Descent Parser (RDP) mechanics, key concepts in compiler engineering."
---

## CFG, LL(1), and Recursive Descent Parsing

In this post, we explore the theoretical core of compiler engineering: Context-Free Grammars (CFG), LL(1) parsing constraints, and Recursive Descent Parser (RDP) mechanics. 

---

## 1. Context-Free Grammars (CFG)

Formally, a Context-Free Grammar is defined as a 4-tuple:
$$G = (V, \Sigma, R, S)$$
where:
* **$V$**: A finite set of **non-terminal** variables (syntactic categories like `Statement`).
* **$\Sigma$**: A finite set of **terminal** symbols (tokens like `NUMBER` or `;`), disjoint from $V$.
* **$R$**: A relation mapping $V \rightarrow (V \cup \Sigma)^*$, representing **production rules**.
* **$S$**: The **start symbol** ($S \in V$).

### Derivations: Leftmost vs. Rightmost
* **Leftmost Derivation**: At each step, the leftmost non-terminal is replaced first. This corresponds to **Top-Down parsing** (like LL).
* **Rightmost Derivation**: The rightmost non-terminal is replaced first. This corresponds to **Bottom-Up parsing** (like LR).

---

## 2. What is LL(1) Parsing?

An **LL(1)** parser parses the input from **L**eft-to-right, performing a **L**eftmost derivation with **1** token of lookahead.

### Question: What makes a grammar LL(1)?
For a grammar to be LL(1), it must satisfy the following conditions for any non-terminal $A$ with production rules $A \rightarrow \alpha \mid \beta$:
1. **First-Set Disjointness**: $First(\alpha) \cap First(\beta) = \emptyset$. (The parser must know which rule to choose by checking the next token).
2. **Nullable Constraint**: If $\beta \Rightarrow^* \epsilon$, then $First(\alpha) \cap Follow(A) = \emptyset$.

### Mathematical Definitions of First and Follow
* **$First(\alpha)$**: The set of all terminal symbols that can appear as the first symbol of a string derived from $\alpha$.
* **$Follow(A)$**: The set of all terminal symbols that can appear immediately to the right of $A$ in any sentential form derived from the start symbol $S$.

### Question: Explain the "Dangling Else" problem.
* **Answer**: The Dangling Else problem is a classic grammatical ambiguity:
  ```
  Statement -> if (Expr) Statement
             | if (Expr) Statement else Statement
  ```
  For the input `if (a) if (b) s1 else s2`, it is unclear if the `else` belongs to the first `if` or the second `if`. An LL(1) parser resolves this by restructuring the grammar or matching the `else` with the nearest open `if` statement using explicit precedence rules.

---

## 3. Recursive Descent Parsing (RDP)

A Recursive Descent Parser is a top-down parsing technique where each non-terminal in the grammar is represented by a function in the code.

```python
    def Statement(self):
        # Lookahead token acts as the switch
        if self.lookahead["type"] == '{':
            return self.BlockStatement()
        return self.ExpressionStatement()
```

### Question: What are the Time and Space Complexities of RDP?
* **Time Complexity**: For an LL(1) grammar, parsing runs in **$O(N)$** linear time, where $N$ is the number of tokens. Every token is processed in constant time using the lookahead pointer.
* **Space Complexity**: **$O(D)$** space where $D$ is the maximum depth of the call stack (which is proportional to the nesting level of the program structures).

### Question: What if a Grammar is not LL(1), but instead LL(k)?
If a grammar requires more than 1 token of lookahead to decide (e.g. distinguishing between `x = 10;` and `x();` where both start with identifier `x`), it is not LL(1). We can resolve this using:
1. **Left Factoring**: Rewriting the grammar to delay decisions ($A \rightarrow xy \mid xz \implies A \rightarrow x A'$, $A' \rightarrow y \mid z$).
2. **Backtracking**: Trying one rule, and rolling back the tokenizer state if parsing fails (increases time complexity to exponential in the worst case).
3. **LL(k) or LR(k) Parsing**: Using parser generators (like Yacc or Bison) that look ahead $k$ tokens or construct a bottom-up shift-reduce state table.

In the [next](/blogs/operator_precedence) post, we will look at how to resolve left-recursion and parse complex operator precedence levels.
