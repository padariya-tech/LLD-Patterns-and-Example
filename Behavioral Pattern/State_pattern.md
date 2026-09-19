# State Pattern 🚦

> **One-liner (say this in an interview):**
> "State lets an object **change its behavior at runtime** by
> delegating to a separate State object representing its current
> state — so the object appears to change its class, without
> giant conditional logic tracking 'what mode am I in.'"

**Category:** Behavioral
**Real-world analogy:** A traffic light. Its behavior — what it does
next — depends entirely on its **current state** (Red, Yellow,
Green). Each state knows what comes next (Red → Green → Yellow →
Red). The traffic light itself doesn't contain one giant rulebook
checking "if red, do X; if yellow, do Y" — each state handles its
own transition.

---

## 🔴 The Problem (why this pattern exists)

An object's behavior needs to **change based on its internal
state**, and that state can change over time. Naive fix: store the
state as a flag/enum, and litter every method with `if/elif`
checking that flag.

```python
class TrafficLight:
    def __init__(self):
        self.state = "red"

    def next(self):
        if self.state == "red":
            print("Red -> Green")
            self.state = "green"
        elif self.state == "green":
            print("Green -> Yellow")
            self.state = "yellow"
        elif self.state == "yellow":
            print("Yellow -> Red")
            self.state = "red"

    def can_cars_go(self):
        # 😬 SAME if/elif chain repeated for every behavior that
        # depends on state
        if self.state == "red":
            return False
        elif self.state == "green":
            return True
        elif self.state == "yellow":
            return False
```

**Pain points:**
- ❌ **Every method** that depends on state repeats the **same**
  `if/elif` chain — massive duplication
- ❌ Adding a new state (e.g. "flashing") means editing **every
  single method** in the class to add another branch
- ❌ Violates **Open/Closed Principle**
- ❌ State-transition logic is scattered across many methods instead
  of living with the state it belongs to
- ❌ Class balloons as more states/behaviors are added — a classic
  "God object" in the making

---

## 🟢 The Fix — State Pattern

**Idea:** Represent each state as its **own class** implementing a
common interface. The main object (`Context`) holds a reference to
its *current* State object and **delegates** all state-dependent
behavior to it. Transitioning states = just swapping which State
object is held.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common State interface
class TrafficLightState(ABC):
    @abstractmethod
    def next(self, light): ...
    @abstractmethod
    def can_cars_go(self) -> bool: ...

# 2️⃣ Concrete states — each knows its OWN behavior and transition
class RedState(TrafficLightState):
    def next(self, light):
        print("Red -> Green")
        light.state = GreenState()      # 🔁 state decides the next state
    def can_cars_go(self):
        return False

class GreenState(TrafficLightState):
    def next(self, light):
        print("Green -> Yellow")
        light.state = YellowState()
    def can_cars_go(self):
        return True

class YellowState(TrafficLightState):
    def next(self, light):
        print("Yellow -> Red")
        light.state = RedState()
    def can_cars_go(self):
        return False

# 3️⃣ Context — delegates everything to its current state
class TrafficLight:
    def __init__(self):
        self.state: TrafficLightState = RedState()

    def next(self):
        self.state.next(self)           # delegate — no if/elif here!

    def can_cars_go(self):
        return self.state.can_cars_go() # delegate again


# --- Usage ---
light = TrafficLight()
print(light.can_cars_go())   # False (red)
light.next()                  # Red -> Green
print(light.can_cars_go())   # True (green)
light.next()                  # Green -> Yellow
light.next()                  # Yellow -> Red

# ✅ New state (e.g. "flashing yellow")? Add ONE new class.
# TrafficLight and every existing state class stay untouched.
class FlashingYellowState(TrafficLightState):
    def next(self, light):
        light.state = RedState()
    def can_cars_go(self):
        return False
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Behavior per state | `if/elif` repeated in every method | Delegated to a dedicated State class |
| Add new state | Edit every method across the class | Add one new State class |
| Transition logic | Scattered, tangled with other logic | Lives inside each state, clean |
| SOLID | Breaks Open/Closed | Follows Open/Closed |
| Code smell | Duplicated conditional chains | Clean delegation, no branching |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Removes repeated conditional logic
  for state-dependent behavior by giving each state its own class —
  the object's behavior changes as its internal state object changes.
- **Key players:** `Context` (holds current state, delegates calls)
  → `State` interface → `ConcreteState` classes (each knows its own
  behavior + how to transition to the next state).
- **Who decides the next state?** Usually the **state itself**
  (as shown above) — this keeps all transition logic co-located with
  the state it belongs to, rather than back in the Context.
- **Design principle it embodies:** "Favor composition over
  conditionals" — same principle as Strategy, applied to *state*
  instead of *algorithm choice*.
- **Similar-sounding pattern to not confuse it with (classic interview
  trap — the code looks almost identical!):**
  - **Strategy** — the *client* chooses which strategy to inject, and
    it typically stays fixed for the object's lifetime, chosen for
    *how* to do something.
  - **State** — the states **transition among themselves**
    automatically as part of the object's natural lifecycle, chosen
    for *what mode* the object is currently in.
  - Bottom line: same structure (composition + shared interface),
    different **intent** — Strategy = pick an algorithm; State =
    represent a lifecycle.
- **Real examples:** TCP connection states (Listening, Established,
  Closed), media player states (Playing, Paused, Stopped), order
  processing (Pending, Shipped, Delivered, Cancelled), game character
  states (Idle, Running, Jumping, Attacking), vending machines.

---

## ✅ Use it when
- An object's behavior depends heavily on its current state, and
  that state changes over time
- You're seeing the same `if/elif` on a state flag repeated across
  multiple methods
- You want state transition rules to be self-contained and easy to
  extend

## ❌ Skip it when
- There are only 2 simple states with trivial behavior — a boolean
  flag and a couple of `if`s is simpler and clearer
- States rarely/never change once set — a fixed conditional or even
  a Strategy might fit better than a full state machine

---

**TL;DR:** Instead of one class riddled with repeated `if/elif`
checks on a state flag, give each state its own class implementing a
shared interface — the Context delegates all behavior to its current
state object, and transitioning states is as simple as swapping that
reference.