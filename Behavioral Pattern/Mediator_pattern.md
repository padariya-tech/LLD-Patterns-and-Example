# Mediator Pattern 🎛️

> **One-liner (say this in an interview):**
> "Mediator centralizes communication between a set of objects into
> one object — so those objects talk **to the mediator**, not
> directly to each other, reducing tangled many-to-many dependencies
> into clean one-to-many ones."

**Category:** Behavioral
**Real-world analogy:** An airport's air traffic control tower.
Planes don't communicate directly with each other to coordinate
landings/takeoffs (imagine the chaos) — every plane talks **only to
the tower**, and the tower coordinates everything. Removing tangled
plane-to-plane communication is exactly the point.

---

## 🔴 The Problem (why this pattern exists)

You have several objects that need to **react to each other** — a
change in one affects several others. Naive fix: each object holds
direct references to every other object it needs to notify or query.

```python
class TextField:
    def __init__(self):
        self.text = ""
        self.submit_button = None   # 😬 needs a reference to Button
        self.checkbox = None        # 😬 AND to Checkbox

    def on_change(self, text):
        self.text = text
        # directly reaching into other components
        self.submit_button.enabled = len(text) > 0 and self.checkbox.checked

class Checkbox:
    def __init__(self):
        self.checked = False
        self.text_field = None      # 😬 needs a reference back to TextField
        self.submit_button = None   # 😬 AND to Button

    def on_toggle(self, checked):
        self.checked = checked
        self.submit_button.enabled = len(self.text_field.text) > 0 and checked

# Every component holds references to every OTHER component it
# affects. Adding a new field means updating multiple existing classes.
```

**Pain points:**
- ❌ **Many-to-many web of dependencies** — every object references
  every other object it needs to coordinate with
- ❌ Adding a new component (e.g. a dropdown) means **editing
  multiple existing classes** to wire up the new references
- ❌ Components are **tightly coupled** to each other — can't reuse
  `TextField` elsewhere without dragging `Checkbox` and `Button`
  along with it
- ❌ Coordination logic is **scattered** across many classes instead
  of living in one place
- ❌ Hard to understand overall behavior — you have to read every
  class to piece together the full interaction flow

---

## 🟢 The Fix — Mediator Pattern

**Idea:** Introduce a **Mediator** object that all components talk
to. Components no longer reference each other directly — they only
notify the mediator ("I changed"), and the mediator decides what
happens next across the whole group.

```python
from abc import ABC, abstractmethod

# 1️⃣ Mediator interface
class DialogMediator(ABC):
    @abstractmethod
    def notify(self, sender, event): ...

# 2️⃣ Components — only know about the mediator, NOT each other
class TextField:
    def __init__(self, mediator: DialogMediator):
        self.text = ""
        self.mediator = mediator
    def on_change(self, text):
        self.text = text
        self.mediator.notify(self, "text_changed")   # 🔔 just notify — no idea who's listening

class Checkbox:
    def __init__(self, mediator: DialogMediator):
        self.checked = False
        self.mediator = mediator
    def on_toggle(self, checked):
        self.checked = checked
        self.mediator.notify(self, "checkbox_toggled")

class SubmitButton:
    def __init__(self):
        self.enabled = False

# 3️⃣ Concrete mediator — owns ALL the coordination logic in one place
class FormMediator(DialogMediator):
    def __init__(self, text_field, checkbox, submit_button):
        self.text_field = text_field
        self.checkbox = checkbox
        self.submit_button = submit_button

    def notify(self, sender, event):
        # 🎛️ single source of truth for how components affect each other
        self.submit_button.enabled = (
            len(self.text_field.text) > 0 and self.checkbox.checked
        )
        print(f"Submit button enabled: {self.submit_button.enabled}")


# --- Usage ---
submit_button = SubmitButton()
mediator = FormMediator(None, None, submit_button)
text_field = TextField(mediator)
checkbox = Checkbox(mediator)
mediator.text_field = text_field
mediator.checkbox = checkbox

text_field.on_change("hello")     # Submit button enabled: False (checkbox unchecked)
checkbox.on_toggle(True)          # Submit button enabled: True

# ✅ Adding a dropdown that also affects submit_button? Update ONLY
# FormMediator.notify() — TextField, Checkbox, SubmitButton never change.
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Communication | Many-to-many, direct references everywhere | Many-to-one — everyone talks to the mediator |
| Coordination logic | Scattered across every component | Centralized in one Mediator class |
| Add new component | Edit several existing classes to wire it up | Update the mediator only |
| Reusability | Components tightly coupled to each other | Components reusable independently |
| Understanding behavior | Must read every class | Read one mediator class |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Reduces chaotic many-to-many
  coupling between objects by routing all their interactions through
  one central Mediator object.
- **Key players:** `Mediator` interface → `ConcreteMediator` (owns
  the coordination logic) → `Colleague`/`Component` objects (only
  know about the mediator, notify it of events, never reference each
  other directly).
- **Trade-off to mention:** Mediator can become a **"God object"** if
  it accumulates too much logic — centralizing coordination is good,
  but an overloaded mediator becomes its own maintenance headache.
  Watch for this in a large system.
- **Design principle it embodies:** Loose coupling — components
  depend only on the mediator abstraction, not on each other.
- **Similar-sounding pattern to not confuse it with:**
  - **Observer** — a Subject notifies **all** subscribed Observers of
    its own state changes (one-directional, one-to-many); Mediator
    coordinates **two-way** interactions **between many peer objects**
    that are otherwise unrelated to each other.
  - **Facade** — simplifies access **into** a subsystem from outside
    (one-directional: client → subsystem); Mediator coordinates
    communication **among** objects that are peers of each other
    (subsystem objects usually don't know Facade exists either, but
    Mediator's whole job is inter-object coordination).
  - **Chain of Responsibility** — passes a request **linearly** along
    a chain; Mediator coordinates **non-linear**, many-directional
    interactions from one central point.
- **Real examples:** Air traffic control, GUI dialog boxes
  coordinating widget interactions, chat room servers (relaying
  messages between users, users never talk to each other directly),
  `pubsub`/event bus systems where a central dispatcher coordinates
  loosely-coupled modules.

---

## ✅ Use it when
- Many objects interact with each other in complex, tangled ways
- You want to reuse individual components independently, without
  dragging along a web of other objects they were wired to
- Coordination logic is scattered and hard to follow across many
  classes

## ❌ Skip it when
- Only two or three objects interact, simply — direct references are
  clearer than adding a mediator layer
- The mediator would end up as a bloated "God object" handling too
  much unrelated coordination logic — consider splitting it or using
  Observer/events for simpler one-directional notification cases

---

**TL;DR:** Instead of objects holding direct references to every
other object they need to coordinate with (many-to-many chaos),
route all interactions through a single Mediator — components only
talk to the mediator, and all coordination logic lives in one place.