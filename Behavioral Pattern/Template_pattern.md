# Template Method Pattern 📋

> **One-liner (say this in an interview):**
> "Template Method defines the **skeleton of an algorithm** in a base
> class — fixing the overall steps and their order — while letting
> subclasses override **specific steps** without changing the
> algorithm's structure."

**Category:** Behavioral
**Real-world analogy:** A recipe template for "making a hot
beverage": boil water → brew → pour into cup → add condiments. Tea
and Coffee both follow this **exact same sequence**, but "brew" and
"add condiments" differ (steep tea leaves vs. brew coffee grounds;
add lemon vs. add milk+sugar). The overall recipe structure never
changes — only specific steps do.

---

## 🔴 The Problem (why this pattern exists)

You have several classes that follow the **same overall algorithm**,
but a few steps differ between them. Naive fix: each class
duplicates the entire algorithm, repeating the identical parts and
only slightly tweaking a couple of steps.

```python
class Tea:
    def prepare(self):
        print("Boiling water")
        print("Steeping tea leaves")      # differs
        print("Pouring into cup")
        print("Adding lemon")             # differs

class Coffee:
    def prepare(self):
        print("Boiling water")             # duplicated
        print("Brewing coffee grounds")    # differs
        print("Pouring into cup")          # duplicated
        print("Adding milk and sugar")     # differs

# 😬 "Boiling water" and "Pouring into cup" are copy-pasted in every
# new beverage class. Forget to update one when the process changes
# (e.g. new hygiene rule for boiling) → inconsistent bugs.
```

**Pain points:**
- ❌ **Duplicated algorithm structure** across every subclass — only
  2 of 4 steps actually differ, yet all 4 are rewritten each time
- ❌ Changing a **shared** step (e.g. how water is boiled) means
  editing it in **every single class** that repeats it
- ❌ No enforced structure — a subclass could accidentally reorder
  steps or skip one, breaking consistency
- ❌ Violates **DRY** (Don't Repeat Yourself)

---

## 🟢 The Fix — Template Method Pattern

**Idea:** Put the **fixed algorithm skeleton** in a base class method
(the "template method"), calling abstract/overridable "step" methods
along the way. Subclasses **only** override the steps that actually
differ — the shared steps and the overall order live in exactly one
place.

```python
from abc import ABC, abstractmethod

class HotBeverage(ABC):
    def prepare(self):              # 📋 the TEMPLATE METHOD — fixed algorithm skeleton
        self.boil_water()           # shared step
        self.brew()                 # varies — subclass fills this in
        self.pour_in_cup()          # shared step
        self.add_condiments()       # varies — subclass fills this in

    def boil_water(self):           # shared step, defined once
        print("Boiling water")

    def pour_in_cup(self):          # shared step, defined once
        print("Pouring into cup")

    @abstractmethod
    def brew(self): ...             # must be implemented by subclass

    @abstractmethod
    def add_condiments(self): ...   # must be implemented by subclass


class Tea(HotBeverage):
    def brew(self):
        print("Steeping tea leaves")
    def add_condiments(self):
        print("Adding lemon")

class Coffee(HotBeverage):
    def brew(self):
        print("Brewing coffee grounds")
    def add_condiments(self):
        print("Adding milk and sugar")


# --- Usage ---
tea = Tea()
tea.prepare()
# Boiling water
# Steeping tea leaves
# Pouring into cup
# Adding lemon

coffee = Coffee()
coffee.prepare()
# Boiling water
# Brewing coffee grounds
# Pouring into cup
# Adding milk and sugar

# ✅ "Boiling water" logic lives in ONE place. Change it once,
# every beverage benefits. New beverage type? Just implement the
# two abstract steps — the algorithm's structure is guaranteed.
```

### Bonus: optional "hook" methods (steps with a default, overriding optional)

```python
class HotBeverage(ABC):
    def prepare(self):
        self.boil_water()
        self.brew()
        self.pour_in_cup()
        if self.customer_wants_condiments():   # 🪝 hook — subclass MAY override
            self.add_condiments()

    def customer_wants_condiments(self):
        return True   # default behavior; subclasses can override to change it
    # ... rest same as before
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Shared steps (boil water, pour) | Duplicated in every subclass | Defined once, in the base class |
| Changing a shared step | Edit every subclass | Edit one place |
| Algorithm structure | Could drift/get reordered per subclass | Fixed and guaranteed by the template method |
| What subclasses do | Reimplement everything | Override only the steps that differ |
| DRY | Violated | Respected |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Eliminates duplicated algorithm
  structure across subclasses by fixing the overall steps/order in
  one base-class method, letting subclasses customize only specific
  steps.
- **Key mechanism:** **inheritance** — the base class calls
  abstract/hook methods that subclasses implement or override; this
  is the classic "don't call us, we'll call you" (Hollywood
  Principle) — the base class controls the flow, not the subclass.
- **Hook methods:** optional steps with a **default** implementation
  that subclasses *may* (but don't have to) override — gives
  flexibility without forcing every subclass to implement every step.
- **Design principle it embodies:** DRY + Hollywood Principle
  ("Don't call us, we'll call you") — inverted control flow where the
  base class, not the subclass, drives the algorithm.
- **Similar-sounding pattern to not confuse it with:**
  - **Strategy** — uses **composition**, swaps the **entire**
    algorithm at runtime via an injected object; Template Method uses
    **inheritance**, fixes the algorithm's structure and only lets
    subclasses vary individual **steps**.
  - **Factory Method** — often literally *is* one step inside a
    Template Method (e.g. "create the object" step); Factory Method
    is about *what* to create, Template Method is about the *overall
    process* it fits inside.
- **Real examples:** Django class-based views (`get()`/`post()`
  override points within a fixed request-handling flow), `unittest`'s
  `setUp()`/`tearDown()` around test methods, sorting algorithms with
  a fixed structure but a custom comparator step, data-processing
  pipelines (`load()` → `transform()` → `save()`, where `transform()`
  varies).

---

## ✅ Use it when
- Multiple classes share the **same overall algorithm** but differ in
  a few specific steps
- You want to **enforce** a consistent structure/order across
  subclasses (prevent them from reordering or skipping mandatory steps)
- You want to avoid duplicating the parts of the algorithm that never
  change

## ❌ Skip it when
- The algorithm needs to change **entirely** at runtime, not just a
  step or two — Strategy (composition) fits better than inheritance
- Only one implementation will ever exist — no need for the
  base/subclass split at all

---

**TL;DR:** Instead of each subclass duplicating an entire algorithm
with slight tweaks, define the fixed skeleton once in a base class
"template method" that calls overridable step methods — subclasses
only implement the steps that actually differ, and the overall
structure stays guaranteed and DRY.