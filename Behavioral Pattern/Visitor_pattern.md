# Visitor Pattern 🧑‍⚕️

> **One-liner (say this in an interview):**
> "Visitor lets you add **new operations** to a set of classes
> without modifying them — by moving the operation's logic into a
> separate Visitor object that each class accepts and 'visits.'"

**Category:** Behavioral
**Real-world analogy:** A traveling tax auditor (Visitor) who visits
different kinds of businesses (Restaurant, Retail Store, Factory).
Each business doesn't know how to audit itself — it just lets the
auditor in and cooperates (`accept(auditor)`). The auditor carries
all the audit *logic*; the businesses carry none of it. Next year, a
new type of auditor (health inspector) can "visit" the same
businesses without changing the businesses at all.

---

## 🔴 The Problem (why this pattern exists)

You have a fixed set of classes (a Composite tree, an AST, a
document's elements) and need to keep adding **new, unrelated
operations** over them (export, print, calculate cost, validate...).
Naive fix: add a new method to every class for every new operation.

```python
class Circle:
    def area(self): return 3.14 * 5 * 5
    def export_to_json(self): return '{"type": "circle"}'
    def export_to_xml(self): return "<circle/>"
    # 😬 every new export format = a new method on EVERY shape class

class Square:
    def area(self): return 5 * 5
    def export_to_json(self): return '{"type": "square"}'
    def export_to_xml(self): return "<square/>"
```

**Pain points:**
- ❌ Every new operation (export to PDF, calculate tax, validate...)
  means **editing every single class** in the hierarchy
- ❌ Classes become **bloated** with unrelated responsibilities —
  a `Circle` shouldn't need to know about XML export
- ❌ Violates **Single Responsibility Principle** — geometry logic
  and export logic get tangled together in one class
- ❌ If you don't own the classes (third-party/library code), you
  often **can't add methods to them at all**

---

## 🟢 The Fix — Visitor Pattern

**Idea:** Move each new operation into its own **Visitor** class.
Every element class gets **one small, permanent** `accept(visitor)`
method that just calls back into the visitor
(`visitor.visit_circle(self)`). New operations = new Visitor
classes; existing element classes never change again.

```python
from abc import ABC, abstractmethod

# 1️⃣ Visitor interface — one method per element type it can visit
class ShapeVisitor(ABC):
    @abstractmethod
    def visit_circle(self, circle): ...
    @abstractmethod
    def visit_square(self, square): ...

# 2️⃣ Element interface — just needs ONE permanent accept() method
class Shape(ABC):
    @abstractmethod
    def accept(self, visitor: ShapeVisitor): ...

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius
    def accept(self, visitor):
        visitor.visit_circle(self)      # 🔁 "double dispatch" — calls back into visitor

class Square(Shape):
    def __init__(self, side):
        self.side = side
    def accept(self, visitor):
        visitor.visit_square(self)

# 3️⃣ Concrete visitors — each is a NEW operation, no shape class touched
class JSONExportVisitor(ShapeVisitor):
    def visit_circle(self, circle):
        print(f'{{"type": "circle", "radius": {circle.radius}}}')
    def visit_square(self, square):
        print(f'{{"type": "square", "side": {square.side}}}')

class AreaCalculatorVisitor(ShapeVisitor):
    def visit_circle(self, circle):
        print(f"Circle area: {3.14 * circle.radius ** 2}")
    def visit_square(self, square):
        print(f"Square area: {square.side ** 2}")


# --- Usage ---
shapes = [Circle(5), Square(4)]

json_exporter = JSONExportVisitor()
for shape in shapes:
    shape.accept(json_exporter)
# {"type": "circle", "radius": 5}
# {"type": "square", "side": 4}

area_calculator = AreaCalculatorVisitor()
for shape in shapes:
    shape.accept(area_calculator)
# Circle area: 78.5
# Square area: 16

# ✅ New operation (e.g. XML export)? Add ONE new Visitor class.
# Circle and Square are NEVER touched again.
class XMLExportVisitor(ShapeVisitor):
    def visit_circle(self, circle):
        print(f'<circle radius="{circle.radius}"/>')
    def visit_square(self, square):
        print(f'<square side="{square.side}"/>')
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Add new operation | New method on every element class | One new Visitor class |
| Element classes | Keep growing, bloated with unrelated logic | Stay small — just one permanent `accept()` |
| Third-party/uneditable classes | Can't add new operations at all | Visitor works without touching them |
| SOLID | Breaks Single Responsibility & Open/Closed | Follows both |
| Trade-off | — | Adding a **new element type** now means editing every Visitor (see below) |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Lets you add new operations to a
  class hierarchy without modifying the classes — by externalizing
  the operation into a Visitor.
- **Key mechanism — "Double Dispatch":** `accept(visitor)` calls
  `visitor.visit_circle(self)`, which picks the right method based on
  **both** the element's type AND the visitor's type. Regular method
  calls only dispatch on one type (the object you call the method
  on); Visitor effectively achieves dispatch on two.
- **The classic trade-off (important — interviewers love this):**
  Visitor makes it **easy to add new operations** but **hard to add
  new element types** (every Visitor subclass must implement a
  method for it). This is the exact **opposite trade-off** of just
  adding methods directly to element classes — know this contrast!
- **Design principle it embodies:** Open/Closed for operations
  (closed for element modification, open for new operations) —
  Single Responsibility (logic for "how to export" lives in the
  exporter, not the shape).
- **Similar-sounding pattern to not confuse it with:**
  - **Strategy** — swaps **one algorithm** for a single operation;
    Visitor adds **operations across an entire object structure/tree**.
  - **Iterator** — focuses on **traversal**; Visitor focuses on the
    **operation performed** at each stop (often combined: iterate
    through a Composite, visiting each node).
  - **Composite** — Visitor is very commonly applied **on top of** a
    Composite tree to add new tree-wide operations cleanly.
- **Real examples:** Compilers/ASTs (type-checking, code generation,
  optimization passes as separate visitors over the same syntax
  tree), document object models (rendering, exporting, spell-checking
  as separate visitors), `ast.NodeVisitor` in Python's standard
  library (used to analyze/transform Python source code itself).

---

## ✅ Use it when
- You need to perform many **unrelated operations** across a stable
  set of classes, and don't want to keep bloating those classes
- The class hierarchy is **relatively fixed** (new element types are
  rare), but new operations are added often
- You want to keep operation-specific logic (export, calculate,
  validate) out of the core domain classes

## ❌ Skip it when
- New element types are added often but operations are stable — this
  flips the trade-off against you; just add methods directly instead
- The class hierarchy is small/simple and operations are unlikely to
  grow — the extra machinery (accept/visit boilerplate) isn't worth it

---

**TL;DR:** Instead of adding a new method to every class each time
you need a new operation, give each class one permanent
`accept(visitor)` method and move each operation into its own
Visitor class — new operations become new Visitors, and the element
classes never need to change again (at the cost of new element types
being harder to add).