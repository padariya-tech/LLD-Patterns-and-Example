# Proxy Pattern 🛡️

> **One-liner (say this in an interview):**
> "Proxy provides a stand-in/placeholder object that controls access
> to a real object — implementing the *same interface* so the client
> can't tell the difference, while adding checks, caching, lazy
> loading, or logging behind the scenes."

**Category:** Structural
**Real-world analogy:** A credit card is a proxy for your bank
account. You hand over the card, not stacks of cash — the card
controls access to your money (checks balance, applies fraud checks,
logs the transaction) while behaving *as if* it were the money
itself.

---

## 🔴 The Problem (why this pattern exists)

You have an object that's **expensive to create**, or **needs access
control**, or lives **remotely** — but you want client code to use
it exactly like the real thing, without scattering that extra logic
everywhere it's used.

```python
class HighResImage:
    def __init__(self, filename):
        self.filename = filename
        self._load_from_disk()          # 😬 expensive — happens immediately

    def _load_from_disk(self):
        print(f"Loading {self.filename} from disk... (slow!)")

    def display(self):
        print(f"Displaying {self.filename}")


# Client code
gallery = [HighResImage(f"photo{i}.png") for i in range(10)]
# 😬 ALL 10 images loaded from disk immediately, even if the user
# never scrolls down to see most of them. No access control either —
# anyone can call .display() regardless of permissions.
```

**Pain points:**
- ❌ **No lazy loading** — expensive setup happens even if the
  object is never actually used
- ❌ **No access control** — anything with a reference can call any
  method, no permission checks
- ❌ Cross-cutting concerns (logging, caching, security) get
  **duplicated** at every call site instead of centralized
- ❌ Client code is tightly bound to the real object's lifecycle —
  can't easily swap in a "fake until needed" placeholder

---

## 🟢 The Fix — Proxy Pattern

**Idea:** Create a Proxy class that implements the **same
interface** as the real object. Client code talks to the Proxy,
which decides *when* and *whether* to forward the call to the real
object — adding lazy loading, checks, caching, or logging in between.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common interface
class Image(ABC):
    @abstractmethod
    def display(self): ...

# 2️⃣ Real object — expensive to create
class HighResImage(Image):
    def __init__(self, filename):
        self.filename = filename
        self._load_from_disk()

    def _load_from_disk(self):
        print(f"Loading {self.filename} from disk... (slow!)")

    def display(self):
        print(f"Displaying {self.filename}")

# 3️⃣ Proxy — same interface, controls access/timing
class LazyImageProxy(Image):
    def __init__(self, filename):
        self.filename = filename
        self._real_image = None          # not created yet!

    def display(self):
        if self._real_image is None:     # ⏳ created only when needed
            self._real_image = HighResImage(self.filename)
        self._real_image.display()


# --- Usage ---
gallery = [LazyImageProxy(f"photo{i}.png") for i in range(10)]
# ✅ Nothing loaded yet — all 10 are just lightweight proxies

gallery[3].display()
# Loading photo3.png from disk... (slow!)
# Displaying photo3.png
# Only photo3 was ever actually loaded — the other 9 stay untouched.
```

### Bonus: Access-control proxy (a different flavor of the same idea)

```python
class ProtectedImageProxy(Image):
    def __init__(self, filename, user_role):
        self._real_image = HighResImage(filename)
        self._user_role = user_role

    def display(self):
        if self._user_role != "admin":
            print("🚫 Access denied — admin role required")
            return
        self._real_image.display()
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Expensive object creation | Happens immediately, always | Deferred until actually needed (lazy) |
| Access control | None / scattered checks | Centralized in the proxy |
| Client code | Talks directly to real object | Talks to proxy — unaware of the difference |
| Cross-cutting logic (logging, caching) | Duplicated everywhere | Centralized in the proxy |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Controls access to an object —
  deferring costly work, adding checks, or adding logging/caching —
  without the client knowing it's not talking to the real thing.
- **Common proxy types (know these by name):**
  - **Virtual Proxy** — delays expensive object creation until
    actually needed (the lazy-loading example above)
  - **Protection Proxy** — adds access control/permission checks
  - **Remote Proxy** — represents an object living in a different
    address space/machine (e.g. gRPC/RPC stubs)
  - **Caching Proxy** — stores results of expensive operations and
    returns cached data on repeat calls
  - **Logging/Smart Reference Proxy** — adds logging, reference
    counting, etc. around access
- **Design principle it embodies:** Same interface as the real
  subject → transparent substitution (client code doesn't change).
- **Similar-sounding pattern to not confuse it with:**
  - **Decorator** — *adds new behavior/responsibilities*; Proxy
    *controls access* to existing behavior (the intent differs even
    though the code structure looks nearly identical).
  - **Adapter** — changes an incompatible **interface** to match
    what's expected; Proxy keeps the **same** interface throughout.
  - **Facade** — simplifies access to a **complex subsystem** of
    multiple classes; Proxy controls access to **one** object.
- **Real examples:** Django ORM's lazy `QuerySet` evaluation, Python's
  `unittest.mock.Mock`/`MagicMock`, ORMs' lazy-loaded relationships,
  gRPC client stubs, `weakref.proxy`, CDN as a caching proxy for origin
  servers.

---

## ✅ Use it when
- Object creation is expensive and might not always be needed (lazy
  loading)
- You need to add access control/permission checks without changing
  the real object's code
- You want to add logging, caching, or monitoring transparently
- The real object lives remotely and you need a local stand-in

## ❌ Skip it when
- There's no real need for access control, laziness, or added
  behavior — a direct reference is simpler and avoids an extra
  indirection layer
- The added proxy layer would meaningfully hurt performance in a
  hot path where directness matters more than the extra control

---

**TL;DR:** Instead of client code talking directly to an expensive or
sensitive object, put a Proxy in front of it — same interface, but
now controlling *when* and *whether* the real work actually happens
(lazy loading, access control, caching, logging).