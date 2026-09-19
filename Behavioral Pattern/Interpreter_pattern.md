# Interpreter Pattern 🗣️

> **One-liner (say this in an interview):**
> "Interpreter defines a **grammar** for a simple language and
> represents each grammar rule as a class, so you can build an
> expression tree and evaluate sentences in that language by walking
> the tree recursively."

**Category:** Behavioral
**Real-world analogy:** A calculator parsing `"3 + 4 * 2"`. Instead
of one giant function full of string-parsing hacks, each *piece* of
the grammar (a number, an addition, a multiplication) is its own
small object. The whole expression becomes a **tree** of these
objects, and evaluating it means asking the top object to
`interpret()` itself — which recursively asks its children to do
the same.

---

## 🔴 The Problem (why this pattern exists)

You need to **evaluate expressions** in some simple custom language
(math expressions, search filters, rule engines, config DSLs).
Naive fix: one big function that parses and evaluates the string
directly, often with fragile string-splitting/regex logic.

```python
def evaluate(expression: str) -> int:
    # 😬 Manually splitting strings, extremely fragile, doesn't scale
    tokens = expression.split()
    result = int(tokens[0])
    i = 1
    while i < len(tokens):
        op = tokens[i]
        num = int(tokens[i + 1])
        if op == "+":
            result += num
        elif op == "-":
            result -= num
        # No operator precedence, no parentheses, no nesting support...
        i += 2
    return result

print(evaluate("3 + 4 - 2"))   # works, barely
print(evaluate("3 + 4 * 2"))   # 😬 wrong! no precedence handling
```

**Pain points:**
- ❌ **Fragile string manipulation** — breaks easily as the grammar
  grows (precedence, parentheses, nested expressions)
- ❌ Adding a new operator/rule means **editing the same tangled
  function** again
- ❌ No reusable structure — can't easily inspect, optimize, or
  re-evaluate parts of the expression independently
- ❌ Violates **Open/Closed Principle** and quickly becomes
  unmaintainable as grammar complexity grows

---

## 🟢 The Fix — Interpreter Pattern

**Idea:** Model each grammar rule as its own class implementing a
common `interpret()` method. Build an expression **tree** out of
these objects (terminal "leaf" expressions like numbers, and
non-terminal "composite" expressions like addition). Evaluating the
whole thing is just calling `interpret()` on the root — recursion
handles the rest.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common interface — every grammar rule implements this
class Expression(ABC):
    @abstractmethod
    def interpret(self) -> int: ...

# 2️⃣ Terminal expression — the simplest building block (a number)
class Number(Expression):
    def __init__(self, value):
        self.value = value
    def interpret(self):
        return self.value

# 3️⃣ Non-terminal expressions — combine other expressions
class Add(Expression):
    def __init__(self, left: Expression, right: Expression):
        self.left = left
        self.right = right
    def interpret(self):
        return self.left.interpret() + self.right.interpret()   # 🌳 recursion

class Subtract(Expression):
    def __init__(self, left: Expression, right: Expression):
        self.left = left
        self.right = right
    def interpret(self):
        return self.left.interpret() - self.right.interpret()

class Multiply(Expression):
    def __init__(self, left: Expression, right: Expression):
        self.left = left
        self.right = right
    def interpret(self):
        return self.left.interpret() * self.right.interpret()


# --- Usage: build the tree for "(3 + 4) * 2" ---
expression_tree = Multiply(
    Add(Number(3), Number(4)),
    Number(2),
)
print(expression_tree.interpret())   # 14 ✅

# ✅ New operator? Add ONE new class implementing Expression.
# Nothing else needs to change — the tree structure handles
# precedence naturally through HOW it's built.
class Divide(Expression):
    def __init__(self, left, right):
        self.left = left
        self.right = right
    def interpret(self):
        return self.left.interpret() / self.right.interpret()
```

> 💡 **Note:** Interpreter usually assumes you already have a parsed
> tree (or you write a small parser to build it from text). The
> pattern itself is about **evaluating** the tree, not about parsing
> text into one — in real-world tools, a separate parser/lexer
> typically builds this tree first.

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Grammar rules | Tangled inside one parsing function | Each rule is its own class |
| Add new operator | Edit the shared function | Add one new `Expression` class |
| Precedence/nesting | Hard to handle correctly with string splitting | Naturally handled by tree structure |
| Evaluating | Manual, imperative logic | Recursive `interpret()` calls |
| SOLID | Breaks Open/Closed | Follows Open/Closed |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Provides a structured,
  object-oriented way to represent and evaluate sentences in a
  simple language/grammar, instead of fragile ad-hoc string parsing.
- **Key players:**
  - **Terminal Expression** — a leaf node with no sub-expressions
    (e.g. a number, a variable) — the base case of the recursion
  - **Non-terminal Expression** — combines other expressions (e.g.
    Add, Subtract) — the recursive case
  - **Context** (optional) — holds global information needed during
    interpretation (e.g. variable values in `x + y`)
- **Design principle it embodies:** Composite structure — this
  pattern is essentially **Composite** applied specifically to
  grammar rules/expression trees (interpret() plays the role that
  Composite's shared operation plays).
- **Similar-sounding pattern to not confuse it with:**
  - **Composite** — Interpreter's tree structure literally *is* a
    Composite; the difference is **intent** — Composite is about
    part-whole hierarchies in general, Interpreter is specifically
    about representing and **evaluating** a grammar.
  - **Visitor** — often used *alongside* Interpreter to add new
    operations (e.g. "print", "optimize") onto the expression tree
    without modifying the Expression classes themselves.
- **Real examples:** Regex engines, SQL query parsers, expression
  evaluators (calculator apps), rule engines (business logic DSLs),
  configuration language interpreters, compilers'/interpreters'
  abstract syntax trees (ASTs).

---

## ✅ Use it when
- You have a simple, well-defined grammar that needs repeated
  evaluation (math expressions, filters, rules)
- The grammar is relatively simple — Interpreter doesn't scale well
  to genuinely complex languages
- You want an object-oriented, extensible way to add new grammar
  rules over time

## ❌ Skip it when
- The grammar is complex (like a real programming language) — use a
  proper parser generator/parsing library instead; Interpreter's
  one-class-per-rule approach becomes unwieldy at scale
- A simple existing tool (regex, an expression-eval library) already
  solves the problem — don't reinvent a mini-language unnecessarily

---

**TL;DR:** Instead of parsing and evaluating a mini-language with
fragile, tangled string logic, represent each grammar rule as its
own class implementing `interpret()` — build an expression tree out
of terminal and non-terminal expressions, and evaluate it by
recursively calling `interpret()` from the root down.