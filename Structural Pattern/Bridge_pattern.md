# Bridge Pattern 🌉

> **One-liner (say this in an interview):**
> "Bridge splits a class into two independent hierarchies —
> **abstraction** and **implementation** — connected by composition,
> so either side can change/extend without touching the other,
> avoiding a combinatorial explosion of subclasses."

**Category:** Structural
**Real-world analogy:** A TV remote and the TV itself. Different
remotes (basic, smart, voice-controlled) can control different TV
brands (Sony, Samsung, LG) — and you don't need a separate remote
*class* for every remote-type × brand combination. The remote
("abstraction") just holds a reference to a TV ("implementation")
and delegates to it.

---

## 🔴 The Problem (why this pattern exists)

You have two independent "dimensions" that both vary — e.g. shape
**and** color, or device type **and** brand — and you try to model
every combination via **inheritance**. Naive fix: subclass for
every combination.

```python
class RedCircle:
    def draw(self): print("Drawing a Red Circle")

class BlueCircle:
    def draw(self): print("Drawing a Blue Circle")

class RedSquare:
    def draw(self): print("Drawing a Red Square")

class BlueSquare:
    def draw(self): print("Drawing a Blue Square")

# 😬 2 shapes × 2 colors = 4 classes.
# Add "Green" → 6 classes. Add "Triangle" → 9 classes.
# Every NEW shape or NEW color multiplies the class count.
```

**Pain points:**
- ❌ **Combinatorial class explosion** — M shapes × N colors = M×N
  classes, growing multiplicatively with every new variant
- ❌ Adding one new color means creating a new subclass **for every
  existing shape** (and vice versa)
- ❌ The two concerns (*what it is* vs *how it's rendered/implemented*)
  are **tangled together** in a single inheritance chain
- ❌ Violates **Open/Closed Principle** — can't extend one dimension
  without touching the other

---

## 🟢 The Fix — Bridge Pattern

**Idea:** Split the two varying dimensions into **two separate class
hierarchies**. The "abstraction" (Shape) holds a **reference** to an
"implementation" (Color) via composition, instead of inheriting a
combined identity. Now each hierarchy grows independently — no
multiplication.

```python
from abc import ABC, abstractmethod

# 1️⃣ Implementation hierarchy — the "how" (color)
class Color(ABC):
    @abstractmethod
    def fill(self) -> str: ...

class Red(Color):
    def fill(self): return "Red"

class Blue(Color):
    def fill(self): return "Blue"

# 2️⃣ Abstraction hierarchy — the "what" (shape), holds a Color via composition
class Shape(ABC):
    def __init__(self, color: Color):
        self._color = color              # 🌉 the bridge — composition, not inheritance

    @abstractmethod
    def draw(self) -> None: ...

class Circle(Shape):
    def draw(self):
        print(f"Drawing a {self._color.fill()} Circle")

class Square(Shape):
    def draw(self):
        print(f"Drawing a {self._color.fill()} Square")


# --- Usage ---
red_circle = Circle(Red())
blue_square = Square(Blue())

red_circle.draw()    # Drawing a Red Circle
blue_square.draw()   # Drawing a Blue Square

# ✅ Add a new color? ONE new class — every shape gets it for free.
class Green(Color):
    def fill(self): return "Green"

Circle(Green()).draw()   # Drawing a Green Circle — no Shape class touched!

# ✅ Add a new shape? ONE new class — every color works with it automatically.
class Triangle(Shape):
    def draw(self):
        print(f"Drawing a {self._color.fill()} Triangle")

Triangle(Red()).draw()   # Drawing a Red Triangle
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Shapes × Colors | One class per combination (M×N) | M shape classes + N color classes (M+N) |
| Add a new color | New subclass for every existing shape | One new `Color` class — works everywhere |
| Add a new shape | New subclass for every existing color | One new `Shape` class — works everywhere |
| Coupling | Shape & color logic fused via inheritance | Decoupled via composition ("the bridge") |
| SOLID | Breaks Open/Closed | Follows Open/Closed |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Prevents class explosion when two
  independent dimensions of variation exist, by decoupling them into
  separate hierarchies connected via composition rather than a single
  fused inheritance tree.
- **Key mechanism:** the "Abstraction" class holds a reference to an
  "Implementor" interface — that reference **is** the bridge.
- **Design principle it embodies:** "Favor composition over
  inheritance," decoupling *what something is* from *how it's
  implemented*, so both can vary/extend independently.
- **Similar-sounding pattern to not confuse it with:**
  - **Adapter** — fixes an interface mismatch **after the fact**,
    usually for existing incompatible classes; Bridge is designed
    **up front** to let two hierarchies evolve independently.
  - **Strategy** — swaps **one algorithm** dynamically, usually just
    one axis of variation; Bridge structurally splits **two
    hierarchies** (though the code shape is very similar — Bridge is
    really a structural, "designed-in" version of composing behavior).
- **Real examples:** JDBC drivers (same `Connection`/`Statement` API
  works across MySQL, PostgreSQL, Oracle...), cross-platform GUI
  toolkits (same `Window` abstraction, different OS-specific
  rendering implementations), device drivers, Python's ORM backends
  (same query API, different DB engine implementations underneath).

---

## ✅ Use it when
- You have two (or more) independent dimensions that both vary, and
  inheritance alone would multiply subclasses
- You want to swap out the "implementation" side at runtime without
  touching the "abstraction" side (e.g. switch rendering engines)
- You're designing a system **up front** where both hierarchies are
  expected to grow over time

## ❌ Skip it when
- Only one dimension actually varies — plain inheritance or Strategy
  is enough, Bridge adds unnecessary indirection
- The number of combinations is small and unlikely to grow — the
  upfront design cost isn't worth it

---

**TL;DR:** Instead of one inheritance tree trying to cover every
combination of two varying dimensions, split them into two separate
hierarchies connected by composition ("the bridge") → extend either
side independently, no class explosion.