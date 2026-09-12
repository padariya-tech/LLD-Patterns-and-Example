# Builder Pattern 🧱

> **One-liner (say this in an interview):**
> "Builder lets you construct a complex object **step by step**,
> separating the construction process from its final representation
> — so the same construction steps can produce different variations,
> without a constructor explosion."

**Category:** Creational
**Real-world analogy:** Ordering a custom burger at Subway. You don't
call one constructor with 15 parameters (`bread, cheese, sauce1,
sauce2, veggie1...`). You build it step-by-step — pick bread, add
protein, add veggies, add sauce — and only at the end do you get
the finished burger.

---

## 🔴 The Problem (why this pattern exists)

You have an object with **many optional parameters/configuration
combinations**. Naive fix: one giant constructor that tries to cover
every combination — or worse, a pile of overloaded constructors.

```python
class Pizza:
    def __init__(self, size, cheese=True, pepperoni=False,
                 mushrooms=False, olives=False, extra_cheese=False,
                 stuffed_crust=False, gluten_free=False):
        self.size = size
        self.cheese = cheese
        self.pepperoni = pepperoni
        self.mushrooms = mushrooms
        self.olives = olives
        self.extra_cheese = extra_cheese
        self.stuffed_crust = stuffed_crust
        self.gluten_free = gluten_free

# 😬 "Telescoping constructor" — which positional arg is which??
pizza = Pizza("large", True, False, True, False, True, False, True)
# Impossible to read at a glance. Miss one flag? Silent bug.
```

**Pain points:**
- ❌ **Telescoping constructor** — too many parameters, easy to mix
  up order or forget one
- ❌ Poor readability — `Pizza("large", True, False, True, False, ...)`
  tells you nothing without checking the signature
- ❌ Adding a new option means **editing the constructor signature**
  everywhere it's called
- ❌ No way to build the object **incrementally** — must know
  everything up front, in one call
- ❌ Hard to enforce required vs optional fields cleanly

---

## 🟢 The Fix — Builder Pattern

**Idea:** Separate the object's **construction** (step-by-step,
readable, one setting at a time) from its **representation** (the
final `Pizza` object). A `Builder` class exposes chainable methods
for each optional part, and a `build()` method returns the finished
object.

```python
class Pizza:
    def __init__(self):
        self.size = None
        self.toppings = []
        self.stuffed_crust = False
        self.gluten_free = False

    def __repr__(self):
        return f"Pizza({self.size}, toppings={self.toppings}, " \
               f"stuffed_crust={self.stuffed_crust}, gluten_free={self.gluten_free})"


class PizzaBuilder:
    def __init__(self):
        self._pizza = Pizza()

    def set_size(self, size):
        self._pizza.size = size
        return self          # 🔗 return self → enables method chaining

    def add_topping(self, topping):
        self._pizza.toppings.append(topping)
        return self

    def set_stuffed_crust(self, value=True):
        self._pizza.stuffed_crust = value
        return self

    def set_gluten_free(self, value=True):
        self._pizza.gluten_free = value
        return self

    def build(self):
        return self._pizza


# --- Usage ---
pizza = (
    PizzaBuilder()
    .set_size("large")
    .add_topping("pepperoni")
    .add_topping("mushrooms")
    .set_stuffed_crust()
    .build()
)
print(pizza)
# Pizza(large, toppings=['pepperoni', 'mushrooms'], stuffed_crust=True, gluten_free=False)

# ✅ Fully readable — every setting is explicit, order doesn't matter,
# and you only set what you actually need.
```

### Bonus: Python's built-in shortcut — `@dataclass` (for simpler cases)

```python
from dataclasses import dataclass, field

@dataclass
class Pizza:
    size: str
    toppings: list = field(default_factory=list)
    stuffed_crust: bool = False
    gluten_free: bool = False

pizza = Pizza(size="large", toppings=["pepperoni"], stuffed_crust=True)
# Great for simple cases with named/default args — but for genuinely
# complex, multi-step, or conditional construction, a real Builder
# with validation logic in build() is still the stronger choice.
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Setting options | Positional args in one huge constructor call | Chained, named method calls |
| Readability | `Pizza("large", True, False, True, ...)` — unclear | `.set_size("large").add_topping(...)` — self-documenting |
| Adding new option | Edit constructor signature everywhere | Add one new builder method |
| Partial/incremental build | Not possible | Natural — build step by step |
| Validation before creation | Scattered / awkward | Centralized in `build()` |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Eliminates telescoping
  constructors and separates the *process* of building a complex
  object from its *final representation*.
- **Key mechanism:** builder methods return `self` (method
  chaining / "fluent interface"), and a final `build()` method
  assembles + validates the object.
- **Director (optional, from the GoF book):** a separate class that
  knows the *order* of building steps for common configurations
  (e.g. `build_margherita_pizza()`), while the `Builder` just knows
  *how* to set individual parts. Often skipped in modern code —
  method chaining alone is usually enough.
- **Design principle it embodies:** Separation of concerns —
  construction logic lives outside the object being built.
- **Similar-sounding pattern to not confuse it with:**
  - **Factory** — creates an object in **one shot** based on a type;
    Builder assembles a **complex object incrementally**, part by part.
  - **Prototype** — creates new objects by **cloning** an existing
    one; Builder creates from **scratch**, step by step.
- **Real examples:** `StringBuilder` in Java, SQLAlchemy's query
  builder (`.filter().order_by().limit()`), Django's `QuerySet`
  chaining, HTML builder libraries, `requests.Session()` config chains.

---

## ✅ Use it when
- An object has many optional parameters/configurations and a plain
  constructor becomes unreadable or error-prone
- You want to construct an object over multiple steps, possibly
  conditionally
- You want to validate the final object's consistency in one place
  (`build()`) before it's used

## ❌ Skip it when
- The object has few parameters (2-3) with sensible defaults — a
  plain constructor or `@dataclass` is simpler
- There's no real risk of "telescoping constructor" confusion — don't
  add ceremony where it's not needed

---

**TL;DR:** Instead of one overloaded constructor trying to handle
every combination of options, use a Builder with chainable methods
for each part and a `build()` step at the end → readable, flexible,
step-by-step object construction.