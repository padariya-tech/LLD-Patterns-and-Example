# Facade Pattern 🏛️

> **One-liner (say this in an interview):**
> "Facade provides a **simple, unified interface** to a complex
> subsystem of many classes — hiding the internal complexity so
> client code only talks to one clean entry point."

**Category:** Structural
**Real-world analogy:** A car's ignition button. Pressing "Start"
triggers dozens of coordinated steps under the hood — fuel pump
priming, spark plug ignition, starter motor engaging, sensors
checking. You don't do any of that manually; the button is a facade
over all that complexity.

---

## 🔴 The Problem (why this pattern exists)

You have a **complex subsystem** made of many classes that must be
used together, in a specific order, with specific coordination.
Naive fix: make the client code call all of them directly, every
single time.

```python
class CPU:
    def freeze(self): print("CPU: freezing")
    def jump(self, position): print(f"CPU: jumping to {position}")
    def execute(self): print("CPU: executing")

class Memory:
    def load(self, position, data): print(f"Memory: loading {data} at {position}")

class HardDrive:
    def read(self, lba, size): return f"data@{lba}[{size}]"

# 😬 Client code needs to know the EXACT sequence of a dozen calls,
# across multiple subsystem classes, just to start the computer
cpu = CPU()
memory = Memory()
hard_drive = HardDrive()

cpu.freeze()
data = hard_drive.read(0, 1024)
memory.load(0, data)
cpu.jump(0)
cpu.execute()
# Repeat this ENTIRE sequence everywhere the computer needs starting.
```

**Pain points:**
- ❌ Client code needs **deep knowledge** of the subsystem's classes,
  order of operations, and internal coordination
- ❌ **Duplicated boilerplate** — the same multi-step sequence gets
  copy-pasted wherever it's needed
- ❌ **Tight coupling** — client code depends on many concrete
  classes at once, not just one
- ❌ Subsystem internals become hard to change — any refactor risks
  breaking client code scattered everywhere

---

## 🟢 The Fix — Facade Pattern

**Idea:** Create one class — the Facade — that internally
coordinates all the subsystem classes, and exposes just **one or two
simple methods** for the client to call. The subsystem classes still
exist and can be used directly by anyone who needs fine-grained
control, but most callers just need the Facade.

```python
# Subsystem classes stay exactly the same — untouched
class CPU:
    def freeze(self): print("CPU: freezing")
    def jump(self, position): print(f"CPU: jumping to {position}")
    def execute(self): print("CPU: executing")

class Memory:
    def load(self, position, data): print(f"Memory: loading {data} at {position}")

class HardDrive:
    def read(self, lba, size): return f"data@{lba}[{size}]"

# 🏛️ Facade — one simple entry point hiding all the coordination
class ComputerFacade:
    def __init__(self):
        self.cpu = CPU()
        self.memory = Memory()
        self.hard_drive = HardDrive()

    def start(self):
        self.cpu.freeze()
        data = self.hard_drive.read(0, 1024)
        self.memory.load(0, data)
        self.cpu.jump(0)
        self.cpu.execute()


# --- Usage ---
computer = ComputerFacade()
computer.start()
# CPU: freezing
# Memory: loading data@0[1024] at 0
# CPU: jumping to 0
# CPU: executing

# ✅ Client code has ZERO knowledge of CPU/Memory/HardDrive internals.
# One method call. Subsystem can be refactored internally without
# breaking any client code that just calls .start().
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Starting the "computer" | Client calls 5 methods across 3 classes, in exact order | Client calls `.start()` — one line |
| Knowledge required | Client must understand subsystem internals | Client knows nothing about internals |
| Duplication | Same sequence repeated everywhere it's needed | Centralized once, inside the Facade |
| Refactoring subsystem | Risks breaking scattered client code | Only the Facade needs updating |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Simplifies interaction with a
  complex subsystem by providing a single, higher-level interface —
  reducing coupling between client code and subsystem internals.
- **Important nuance:** Facade doesn't **replace** the subsystem
  classes or hide them completely — advanced users can still access
  them directly. It just gives most callers a simpler default path.
- **Design principle it embodies:** "Law of Demeter" / minimizing
  coupling — client talks to one thing instead of many.
- **Similar-sounding pattern to not confuse it with:**
  - **Adapter** — makes **one** mismatched interface compatible with
    what's expected; Facade simplifies access to a **whole
    subsystem** of multiple classes (no interface mismatch involved).
  - **Proxy** — controls access to **one specific object** (same
    interface as that object); Facade simplifies **many different**
    objects/interfaces into one new, simpler interface.
  - **Mediator** — coordinates communication **between** peer objects
    that talk to each other; Facade coordinates a **one-directional**
    call from client → subsystem (subsystem classes typically don't
    know the Facade exists).
- **Real examples:** `requests.get(url)` (hides socket handling,
  redirects, encoding, connection pooling), a `Database` class
  wrapping connection/cursor/transaction management, jQuery hiding
  raw DOM API complexity, ORMs hiding raw SQL.

---

## ✅ Use it when
- A subsystem is complex, with many classes/steps that must be
  coordinated in a specific way
- Most client code only needs a simple, common use case — not
  fine-grained control over every subsystem detail
- You want to decouple client code from subsystem internals so the
  subsystem can evolve independently

## ❌ Skip it when
- The subsystem is already simple — wrapping one or two classes in a
  Facade adds a pointless extra layer
- Client code genuinely needs fine-grained, direct control over
  subsystem internals for every call — a Facade would get in the way

---

**TL;DR:** Instead of client code juggling many subsystem classes in
a specific, error-prone order, wrap that coordination inside one
Facade class exposing a simple method → client code stays clean,
subsystem stays flexible to change internally.