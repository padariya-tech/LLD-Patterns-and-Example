# Strategy Pattern 🎯

> **One-liner (say this in an interview):**
> "Strategy lets you swap out an algorithm at runtime by putting each
> variant behind a common interface, instead of hardcoding it with
> if/elif chains."

**Category:** Behavioral
**Real-world analogy:** Google Maps. Same "get directions" request, but
you pick the strategy — Driving, Walking, Cycling, Transit. The map
doesn't change; the algorithm to reach the destination does.

---

## 🔴 The Problem (why this pattern exists)

You need one task done in **multiple interchangeable ways**
(discounts, sorting, payments, compression...). Naive fix: one class,
one giant `if/elif`.
Interface = what operations are required.
Composition = how one object uses another object.

```python
class DiscountCalculator:
    def calculate(self, customer_type, amount):
        if customer_type == "regular":
            return amount * 0.95
        elif customer_type == "vip":
            return amount * 0.80
        elif customer_type == "student":
            return amount * 0.90
        # 😬 every new discount = editing this method again
```

**Pain points:**
- ❌ New rule → must **edit existing, tested code** (risky)
- ❌ Violates **Open/Closed Principle** ("open for extension, closed for modification")
- ❌ Can't unit-test one rule without dragging the whole class in
- ❌ Class keeps growing forever → becomes a junk drawer

---

## 🟢 The Fix — Strategy Pattern

**Idea:** Pull each `if` branch out into its own class. All classes
implement the same interface. The main class just *calls* whichever
one it's holding — it doesn't care how the work gets done.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common interface all strategies must follow
class DiscountStrategy(ABC):
    @abstractmethod
    def apply(self, amount: float) -> float: ...

# 2️⃣ One class per algorithm
class RegularDiscount(DiscountStrategy):
    def apply(self, amount): return amount * 0.95

class VIPDiscount(DiscountStrategy):
    def apply(self, amount): return amount * 0.80

class StudentDiscount(DiscountStrategy):
    def apply(self, amount): return amount * 0.90

# 3️⃣ Context — holds a strategy, delegates the work
class DiscountCalculator:
    def __init__(self, strategy: DiscountStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: DiscountStrategy):
        self._strategy = strategy          # swap at runtime 🔁

    def calculate(self, amount):
        return self._strategy.apply(amount)


# --- Usage ---
calc = DiscountCalculator(RegularDiscount())
print(calc.calculate(100))          # 95.0

calc.set_strategy(VIPDiscount())
print(calc.calculate(100))          # 80.0

# ✅ New rule? Just add a class. Zero edits to existing code.
class EmployeeDiscount(DiscountStrategy):
    def apply(self, amount): return amount * 0.70
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Add new behavior | Edit existing method | Add a new class |
| Testability | Test whole class | Test strategy alone |
| Switch behavior | Not really possible | `set_strategy()` at runtime |
| SOLID | Breaks Open/Closed | Follows Open/Closed |
| Code smell | Long `if/elif` chain | Clean delegation |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Eliminates conditional logic for
  selecting an algorithm; makes algorithms swappable & independently
  testable.
- **Key players:** `Context` (holds a strategy) → `Strategy` interface
  → `ConcreteStrategy` classes.
- **Design principle it embodies:** *"Favor composition over
  inheritance"* + Open/Closed Principle.
- **Similar-sounding pattern to not confuse it with:**
  - **State pattern** — looks identical in code, but strategies are
    chosen *by the client*; states transition *themselves*.
  - **Template Method** — uses inheritance + fixed skeleton;
    Strategy uses composition + fully swappable behavior.
- **Real examples:** `sorted(key=...)` in Python, Comparator in Java,
  payment gateways, compression algorithms, route-planning apps.

---

## ✅ Use it when
- Multiple algorithms/behaviors do the same job differently
- You want to pick/change the algorithm at runtime
- You're fighting a giant `if/elif`/`switch` for behavior selection

## ❌ Skip it when
- There's only one algorithm and no sign that'll change
- The "variants" are trivial one-liners — a plain function param is enough

---

**TL;DR:** Conditional logic for choosing behavior → extract each
branch into its own class behind a common interface → context
delegates to whichever one it holds → swap freely, extend freely.