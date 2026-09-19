# Command Pattern 🎮

> **One-liner (say this in an interview):**
> "Command turns a **request into a standalone object**, so you can
> parameterize methods with it, queue it, log it, or **undo** it —
> decoupling the object that *invokes* an action from the object
> that actually *performs* it."

**Category:** Behavioral
**Real-world analogy:** A restaurant order slip. The waiter
(Invoker) doesn't cook — they just write down the order (Command)
and hand it to the kitchen (Receiver). The slip itself can be
queued, handed to any chef, or even used to "undo" an order
(cancel it) — the waiter never needs to know *how* the dish is made.

---

## 🔴 The Problem (why this pattern exists)

You have UI elements (buttons, menu items) or callers that need to
**trigger actions**, but hardcoding "this button calls that method"
directly makes the code rigid — no queuing, no undo, no logging, and
tight coupling between the trigger and the action.

```python
class Light:
    def turn_on(self):
        print("Light is ON")
    def turn_off(self):
        print("Light is OFF")

class Button:
    def __init__(self, light):
        self.light = light           # 😬 tightly coupled to Light specifically

    def press(self):
        self.light.turn_on()         # hardcoded action — can't reconfigure,
                                      # can't undo, can't queue, can't log
```

**Pain points:**
- ❌ **Tight coupling** — the button directly knows about `Light`
  and calls a specific method; reusing the button for a different
  action means rewriting it
- ❌ **No undo/redo support** — there's no record of what action was
  performed, so reversing it is impossible without custom logic
  bolted on separately
- ❌ **No queuing/scheduling** — can't stack up actions to run later,
  in order, or on a background thread
- ❌ **No logging/history** — no clean way to track what actions were
  triggered, when, for audit or replay purposes

---

## 🟢 The Fix — Command Pattern

**Idea:** Wrap every action in its own **Command object**
implementing a common `execute()` (and optionally `undo()`) method.
The invoker (button) just calls `execute()` on whatever command it's
holding — it doesn't know or care what that command actually does.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common Command interface
class Command(ABC):
    @abstractmethod
    def execute(self): ...
    @abstractmethod
    def undo(self): ...

# 2️⃣ Receiver — the object that actually does the work
class Light:
    def turn_on(self):
        print("Light is ON")
    def turn_off(self):
        print("Light is OFF")

# 3️⃣ Concrete commands — wrap a receiver + the action to perform
class TurnOnCommand(Command):
    def __init__(self, light: Light):
        self.light = light
    def execute(self):
        self.light.turn_on()
    def undo(self):
        self.light.turn_off()

class TurnOffCommand(Command):
    def __init__(self, light: Light):
        self.light = light
    def execute(self):
        self.light.turn_off()
    def undo(self):
        self.light.turn_on()

# 4️⃣ Invoker — holds a Command, has NO idea what it does
class Button:
    def __init__(self, command: Command):
        self.command = command
    def press(self):
        self.command.execute()


# --- Usage ---
light = Light()
on_button = Button(TurnOnCommand(light))
off_button = Button(TurnOffCommand(light))

on_button.press()    # Light is ON
off_button.press()   # Light is OFF

# ✅ Undo support, for free — because the command remembers how to reverse itself
on_command = TurnOnCommand(light)
on_command.execute()   # Light is ON
on_command.undo()      # Light is OFF

# ✅ Command history — enables undo stacks, macro commands, logging, replay
history = []
cmd = TurnOnCommand(light)
cmd.execute()
history.append(cmd)
history[-1].undo()     # undo the last action, generically, no special-casing
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Invoker ↔ action coupling | Hardcoded direct method call | Invoker just calls `execute()` on any Command |
| Undo support | Not possible without ad-hoc logic | Built in via `undo()` |
| Queuing/logging actions | No clean way | Store Command objects in a list/queue |
| Reusing invoker for a different action | Rewrite the invoker | Just plug in a different Command object |
| SOLID | Tightly coupled | Follows Open/Closed & Single Responsibility |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Decouples the object that triggers
  an action (Invoker) from the object that performs it (Receiver),
  by wrapping the request itself as an object — enabling undo,
  queuing, logging, and reuse.
- **Key players:** `Command` interface → `ConcreteCommand` (wraps a
  Receiver + the action) → `Invoker` (holds and triggers a Command)
  → `Receiver` (the object that does the actual work).
- **Undo mechanism:** each command remembers enough state (or the
  inverse action) to reverse itself — usually via its own `undo()`
  method; a **history stack** of executed commands enables full
  undo/redo functionality.
- **Macro Commands:** a `Command` that holds a **list of other
  Commands** and executes them all in sequence — lets you compose
  complex operations out of simple ones (this is basically Command +
  Composite together).
- **Design principle it embodies:** Encapsulate a request as an
  object — decoupling sender from receiver (similar spirit to
  Strategy, but for *actions/requests* rather than *algorithms*).
- **Similar-sounding pattern to not confuse it with:**
  - **Strategy** — swaps **how** something is done (an algorithm);
    Command wraps **a request/action to be performed**, often with
    undo/queuing/logging concerns attached.
  - **Chain of Responsibility** — passes a request **along a chain**
    until someone handles it; Command wraps a request as an object
    but doesn't imply any chain — it's handed to one specific invoker.
  - **Memento** — often used **alongside** Command to snapshot state
    before an action, enabling more complex undo scenarios than a
    simple inverse action can handle.
- **Real examples:** GUI buttons/menu items, undo/redo stacks in text
  editors, task queues/job schedulers, transaction logs in databases,
  remote controls (literally the GoF book's example), `Celery` tasks
  as serialized commands, macro recording in software.

---

## ✅ Use it when
- You need undo/redo functionality
- You want to queue, schedule, log, or replay actions
- You want to decouple UI/trigger code from the logic that actually
  performs an action, so either can change independently

## ❌ Skip it when
- The action is simple and permanent (no undo needed) and there's
  only one caller — direct method calls are simpler
- Adding a whole `Command` class per action is overkill for a tiny,
  static set of one-off operations

---

**TL;DR:** Instead of hardcoding "this trigger calls that method,"
wrap each action in its own Command object with `execute()` (and
`undo()`) — the invoker just calls `execute()` without knowing what
it does, enabling undo, queuing, logging, and full decoupling
between trigger and action.