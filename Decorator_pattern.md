# Decorator Pattern 🎁

> **One-liner (say this in an interview):**
> "Decorator lets you attach new behavior to an object dynamically by
> wrapping it in another object with the same interface — instead of
> creating a new subclass for every possible combination of features."

**Category:** Structural
**Real-world analogy:** Ordering coffee. You start with a plain
`Coffee`. Then you *wrap* it — add Milk, wrap again — add Sugar, wrap
again — add Whipped Cream. Each wrapper adds cost/description without
touching the original `Coffee` class, and you can stack any
combination you like.

---

## 🔴 The Problem (why this pattern exists)

You have a base object, and you want to add optional
features/behaviors to it — in **any combination**. Naive fix:
create a subclass for every combination.

```python
class Coffee:
    def cost(self):
        return 50

class CoffeeWithMilk(Coffee):
    def cost(self):
        return super().cost() + 10

class CoffeeWithSugar(Coffee):
    def cost(self):
        return super().cost() + 5

class CoffeeWithMilkAndSugar(Coffee):
    def cost(self):
        return super().cost() + 10 + 5

# 😬 Now add "Whipped Cream" as another option...
# CoffeeWithWhippedCream, CoffeeWithMilkAndWhippedCream,
# CoffeeWithSugarAndWhippedCream, CoffeeWithMilkAndSugarAndWhippedCream...
```

**Pain points:**
- ❌ **Class explosion** — N optional features → up to 2ⁿ subclasses
- ❌ Violates **Open/Closed Principle** — new feature = editing the
  hierarchy, adding many new classes
- ❌ Combinations are **fixed at compile time**, can't add features
  to an object that already exists at runtime
- ❌ Massive code duplication across subclasses

---

## 🟢 The Fix — Decorator Pattern

**Idea:** Wrap the object instead of subclassing it. Both the base
object and the wrappers ("decorators") implement the **same
interface**, so they're interchangeable — and decorators can be
**stacked** in any order, at runtime.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common interface for base object AND decorators
class Coffee(ABC):
    @abstractmethod
    def cost(self) -> float: ...
    @abstractmethod
    def description(self) -> str: ...

# 2️⃣ Concrete base object
class SimpleCoffee(Coffee):
    def cost(self): return 50
    def description(self): return "Coffee"

# 3️⃣ Base Decorator — wraps a Coffee, still IS a Coffee
class CoffeeDecorator(Coffee):
    def __init__(self, coffee: Coffee):
        self._coffee = coffee
    def cost(self):
        return self._coffee.cost()
    def description(self):
        return self._coffee.description()

# 4️⃣ Concrete decorators — each adds ONE behavior
class MilkDecorator(CoffeeDecorator):
    def cost(self): return self._coffee.cost() + 10
    def description(self): return self._coffee.description() + " + Milk"

class SugarDecorator(CoffeeDecorator):
    def cost(self): return self._coffee.cost() + 5
    def description(self): return self._coffee.description() + " + Sugar"

class WhippedCreamDecorator(CoffeeDecorator):
    def cost(self): return self._coffee.cost() + 15
    def description(self): return self._coffee.description() + " + Cream"


# --- Usage ---
order = SimpleCoffee()
order = MilkDecorator(order)          # wrap 🎁
order = SugarDecorator(order)         # wrap again 🎁
order = WhippedCreamDecorator(order)  # wrap again 🎁

print(order.description())   # Coffee + Milk + Sugar + Cream
print(order.cost())          # 80

# ✅ Any combination, chosen at RUNTIME, zero new classes needed
# for "MilkAndSugarAndCream" — just stack existing decorators.
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Add new feature | New subclass(es), possible combinatorial explosion | One new decorator class |
| Combine features | Needs a subclass per combo | Stack decorators freely |
| When features are chosen | Compile time (fixed hierarchy) | Runtime (wrap dynamically) |
| SOLID | Breaks Open/Closed | Follows Open/Closed |
| Code smell | Class explosion / duplication | Composable, minimal classes |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Adding responsibilities to an
  object dynamically without altering its class or triggering a
  subclass explosion for every combination.
- **Key idea:** Decorator **implements the same interface** as the
  object it wraps, so it's a drop-in replacement — this is what lets
  you stack them (each decorator wraps another decorator or the base).
- **Design principle it embodies:** "Favor composition over
  inheritance" + Open/Closed Principle.
- **Similar-sounding pattern to not confuse it with:**
  - **Adapter** — changes an object's *interface* to make it
    compatible; Decorator keeps the *same* interface and adds behavior.
  - **Proxy** — controls *access* to an object (lazy load, permissions);
    Decorator *adds* new behavior/responsibility.
  - **Strategy** — swaps *one* algorithm for another; Decorator *stacks*
    multiple added behaviors on top of each other.
- **Real examples:** Python's `@decorator` syntax (functionally similar
  idea), Java I/O streams (`BufferedReader(FileReader(...))`), Django
  middleware, UI toolkits adding scrollbars/borders to widgets.

---

## ✅ Use it when
- You need to add responsibilities to individual objects, not the
  whole class, and want to do it dynamically
- You'd otherwise need a huge number of subclasses to cover every
  feature combination
- You want to add/remove behavior at runtime without touching existing code

## ❌ Skip it when
- The set of features is small and fixed — plain subclassing is simpler
- Too many small decorator layers can make debugging/stack traces
  confusing — don't over-decorate

---

**TL;DR:** Instead of subclassing for every feature combination, wrap
the object in decorator classes that share its interface → stack any
combination of behaviors at runtime, no class explosion.