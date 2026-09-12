# Factory Pattern 🏭

> **One-liner (say this in an interview):**
> "Factory pattern hides object creation logic behind a method, so
> the client asks for *what* it wants without knowing or caring
> *how* it's built or *which* concrete class gets instantiated."

**Category:** Creational
**Real-world analogy:** Ordering food via a delivery app. You just
say "I want a Pizza." You don't know (or care) which specific
kitchen, chef, or recipe made it — the app's "factory" figures that
out and hands you a ready pizza.

---

## 🔴 The Problem (why this pattern exists)

You need to create objects, but **which class to instantiate depends
on some condition**, and that decision is scattered everywhere the
object is needed. Naive fix: `if/elif` directly at every call site.

```python
class Dog:
    def speak(self): return "Woof!"

class Cat:
    def speak(self): return "Meow!"

# 😬 This creation logic gets copy-pasted everywhere an animal is needed
def get_pet(pet_type):
    if pet_type == "dog":
        return Dog()
    elif pet_type == "cat":
        return Cat()
    else:
        raise ValueError("Unknown pet type")

# main.py
pet = get_pet("dog")

# somewhere_else.py — duplicated logic!
if pet_type == "dog":
    pet = Dog()
elif pet_type == "cat":
    pet = Cat()
```

**Pain points:**
- ❌ Object-creation logic **duplicated** across the codebase
- ❌ Adding a new type (`Bird`) → hunt down and edit **every**
  `if/elif` block that creates pets
- ❌ Violates **Open/Closed Principle**
- ❌ Client code is **tightly coupled** to concrete classes
  (`Dog`, `Cat`) instead of an abstraction

---

## 🟢 The Fix — Factory Pattern

**Idea:** Centralize the "which class to instantiate" decision in
**one place** — a factory method/class. Client code only talks to
the common interface and the factory; it never references concrete
classes directly.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common interface
class Animal(ABC):
    @abstractmethod
    def speak(self) -> str: ...

# 2️⃣ Concrete products
class Dog(Animal):
    def speak(self): return "Woof!"

class Cat(Animal):
    def speak(self): return "Meow!"

# 3️⃣ Factory — the ONLY place that knows how to create animals
class AnimalFactory:
    @staticmethod
    def create_animal(pet_type: str) -> Animal:
        animals = {
            "dog": Dog,
            "cat": Cat,
        }
        if pet_type not in animals:
            raise ValueError(f"Unknown pet type: {pet_type}")
        return animals[pet_type]()


# --- Usage ---
pet = AnimalFactory.create_animal("dog")
print(pet.speak())     # Woof!

pet = AnimalFactory.create_animal("cat")
print(pet.speak())     # Meow!

# ✅ New type? Add ONE class + ONE dict entry. No hunting through
# the codebase for scattered if/elif blocks.
class Bird(Animal):
    def speak(self): return "Tweet!"

# AnimalFactory.animals["bird"] = Bird   (or add to the dict above)
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Add new type | Edit every `if/elif` site | Add class + one factory entry |
| Creation logic | Duplicated everywhere | Centralized in one factory |
| Client coupling | Tied to concrete classes | Tied only to the interface |
| SOLID | Breaks Open/Closed | Follows Open/Closed |
| Code smell | Repeated conditional creation | Single source of truth for creation |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Decouples object creation from
  object usage — client code depends on an abstraction, not on
  concrete classes, and creation logic lives in exactly one place.
- **Two flavors (know the difference!):**
  - **Factory Method** — a method (often overridden by subclasses)
    that creates one type of object; each subclass decides *what* to
    create.
  - **Abstract Factory** — a factory of factories; creates **families
    of related objects** (e.g. `WindowsButton` + `WindowsCheckbox` vs
    `MacButton` + `MacCheckbox`) without specifying concrete classes.
- **Design principle it embodies:** Dependency Inversion — depend on
  abstractions (`Animal`), not concretions (`Dog`, `Cat`).
- **Similar-sounding pattern to not confuse it with:**
  - **Builder** — for constructing one **complex** object step-by-step
    (many optional parts); Factory is about **choosing which class**
    to instantiate.
  - **Singleton** — ensures only **one instance** ever exists; Factory
    is about **how** instances get created, can create many.
  - **Prototype** — creates new objects by **cloning** an existing
    instance; Factory creates via **constructors/classes**.
- **Real examples:** `logging.getLogger()`, Django's model
  `objects.create()`, `document.createElement()` in the DOM, ORMs
  picking a DB driver based on config.

---

## ✅ Use it when
- The exact class to instantiate depends on input/config and may grow
- You want client code to depend on interfaces, not concrete classes
- Object creation involves some setup logic you don't want repeated

## ❌ Skip it when
- You only ever create one type of object — just call its constructor
- Object creation is trivial and unlikely to vary — a factory here is
  unnecessary indirection

---

**TL;DR:** Instead of scattering `if/elif` object-creation logic
everywhere, centralize it in one factory → client code asks for
*what* it needs via an interface, factory decides *which* concrete
class to build.