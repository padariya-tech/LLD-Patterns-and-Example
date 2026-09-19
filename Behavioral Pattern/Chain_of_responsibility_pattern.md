# Chain of Responsibility Pattern ⛓️

> **One-liner (say this in an interview):**
> "Chain of Responsibility passes a request along a **chain of
> handlers**, where each handler decides either to process it or
> pass it to the next one — decoupling the sender from knowing
> exactly which handler will deal with it."

**Category:** Behavioral
**Real-world analogy:** Customer support escalation. Your ticket
first hits a **Level 1** agent. If they can't resolve it, it goes to
**Level 2**. Still unresolved? It escalates to a **Manager**. You
(the sender) never decide who handles it — you just submit the
ticket, and it flows through the chain until someone handles it.

---

## 🔴 The Problem (why this pattern exists)

A request needs to be handled by **one of several possible
handlers**, but the sender shouldn't need to know which one in
advance. Naive fix: the sender checks conditions itself and calls
the right handler directly, or one giant method handles everything
with nested `if/elif`.

```python
class SupportSystem:
    def handle_ticket(self, ticket):
        # 😬 One method knows about EVERY level and EVERY condition
        if ticket.severity == "low":
            print("Level 1: Resolved basic issue")
        elif ticket.severity == "medium":
            print("Level 2: Resolved technical issue")
        elif ticket.severity == "high":
            print("Manager: Resolved critical issue")
        else:
            print("Unhandled ticket!")
```

**Pain points:**
- ❌ **One class/method knows about every handler and condition** —
  tightly coupled, hard to maintain
- ❌ Adding a new handling level (e.g. "Director" for severity
  `"critical"`) means **editing this same method again**
- ❌ Violates **Open/Closed Principle**
- ❌ Can't reorder, insert, or remove handlers dynamically — the
  chain is hardcoded as one block of logic
- ❌ No clean way to have multiple handlers **optionally** look at
  the same request in sequence

---

## 🟢 The Fix — Chain of Responsibility Pattern

**Idea:** Each handler is its own class, holding a reference to the
**next** handler in the chain. A handler either processes the
request or passes it along. The sender just hands the request to the
**first** handler and doesn't care who ultimately deals with it.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common handler interface
class SupportHandler(ABC):
    def __init__(self):
        self._next_handler = None

    def set_next(self, handler):
        self._next_handler = handler
        return handler          # 🔗 allows chaining: h1.set_next(h2).set_next(h3)

    @abstractmethod
    def handle(self, ticket): ...

    def pass_to_next(self, ticket):
        if self._next_handler:
            self._next_handler.handle(ticket)
        else:
            print("No handler could resolve this ticket!")

# 2️⃣ Concrete handlers — each decides: handle it, or pass it on
class Level1Support(SupportHandler):
    def handle(self, ticket):
        if ticket["severity"] == "low":
            print("Level 1: Resolved basic issue")
        else:
            self.pass_to_next(ticket)          # not my job — pass along

class Level2Support(SupportHandler):
    def handle(self, ticket):
        if ticket["severity"] == "medium":
            print("Level 2: Resolved technical issue")
        else:
            self.pass_to_next(ticket)

class ManagerSupport(SupportHandler):
    def handle(self, ticket):
        if ticket["severity"] == "high":
            print("Manager: Resolved critical issue")
        else:
            self.pass_to_next(ticket)


# --- Usage ---
level1 = Level1Support()
level2 = Level2Support()
manager = ManagerSupport()
level1.set_next(level2).set_next(manager)      # build the chain

level1.handle({"severity": "low"})       # Level 1: Resolved basic issue
level1.handle({"severity": "medium"})    # passed to Level 2 -> Resolved
level1.handle({"severity": "high"})      # passed all the way to Manager

# ✅ New level? Add ONE class, insert into the chain. Nothing else changes.
class DirectorSupport(SupportHandler):
    def handle(self, ticket):
        if ticket["severity"] == "critical":
            print("Director: Resolved emergency issue")
        else:
            self.pass_to_next(ticket)

manager.set_next(DirectorSupport())
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Handler selection | One method with all conditions | Each handler decides for itself |
| Add new handler | Edit the shared method | Add a class, insert into chain |
| Sender's knowledge | Must effectively know all rules | Just hands off to the first handler |
| Chain order | Fixed, hardcoded | Configurable at runtime (`set_next`) |
| SOLID | Breaks Open/Closed | Follows Open/Closed |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Decouples request senders from
  specific handlers by letting the request travel through a chain
  until something (or nothing) handles it.
- **Key mechanism:** each handler holds a reference to the "next"
  handler; it either fully handles the request or forwards it —
  the sender only ever talks to the *first* link in the chain.
- **Handling can be exclusive OR cumulative:**
  - *Exclusive* — first handler that can process it, does, and stops
    the chain (support ticket example above)
  - *Cumulative* — every handler in the chain gets a chance to act
    (e.g. logging middleware, where each layer adds something before
    passing it on)
- **Design principle it embodies:** Decoupling sender from receiver —
  sender doesn't need to know which object (if any) will end up
  handling the request.
- **Similar-sounding pattern to not confuse it with:**
  - **Decorator** — every wrapper **always** processes the request
    (adds behavior); Chain of Responsibility handlers can **choose
    to skip** and pass along untouched.
  - **Observer** — broadcasts to **all** observers simultaneously,
    no forwarding order dependency; Chain passes **sequentially**,
    one at a time, and can stop early.
  - **Mediator** — centralizes communication **between many peer
    objects**; Chain is a **linear pipeline** for one request.
- **Real examples:** Middleware in web frameworks (Django/Express —
  each middleware can handle or pass the request on), event bubbling
  in the DOM, exception handling (`try/except` chains bubbling up
  call stack), logging frameworks (DEBUG → INFO → WARNING → ERROR
  handlers), approval workflows (manager → director → VP).

---

## ✅ Use it when
- More than one object might handle a request, and the exact handler
  isn't known in advance
- You want to issue a request without specifying the receiver explicitly
- The set of handlers and their order should be configurable/extendable
  at runtime

## ❌ Skip it when
- Only one handler will ever process the request — direct method
  call is simpler and clearer
- Debuggability matters a lot in a hot path — long chains can make it
  harder to trace exactly where/why a request was (or wasn't) handled

---

**TL;DR:** Instead of one method/class knowing every condition for
handling a request, build a chain of small handler classes — each
decides to handle the request or forward it to the next link — so
senders don't need to know who (if anyone) will end up handling it.