# Singleton Pattern 🔒

> **One-liner (say this in an interview):**
> "Singleton ensures a class has **only one instance** across the
> entire application, and gives a **global point of access** to it."

**Category:** Creational
**Real-world analogy:** A country's president. There's exactly one
active at a time, everyone refers to "the president" (not "a new
president each time"), and there's one well-known way to reach that
office regardless of who currently holds it.

---

## 🔴 The Problem (why this pattern exists)

Some things should logically exist **only once** — a database
connection pool, a config manager, a logger, a cache. Naive fix:
just create a new object wherever you need it.

```python
class ConfigManager:
    def __init__(self):
        self.settings = {"debug": True}   # loaded from a file, say

# 😬 Every part of the code creates its own instance
config1 = ConfigManager()
config2 = ConfigManager()

config1.settings["debug"] = False
print(config2.settings["debug"])   # True — inconsistent state!
```

**Pain points:**
- ❌ Multiple instances → **inconsistent state** across the app
- ❌ **Wasted resources** — re-reading config files, re-opening
  DB connections unnecessarily
- ❌ No single, guaranteed access point — every file imports/creates
  its own copy
- ❌ Hard to coordinate — one part's changes don't reflect elsewhere

---

## 🟢 The Fix — Singleton Pattern

**Idea:** Control instantiation so that no matter how many times the
class is "created," everyone gets **the same object**.

There are several ways to implement this in Python — here are the
main ones:

### Method 1: Override `__new__` (most common, classic OOP way)

```python
class ConfigManager:
    _instance = None

    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.settings = {"debug": True}
        return cls._instance

config1 = ConfigManager()
config2 = ConfigManager()
config1.settings["debug"] = False
print(config2.settings["debug"])   # False — same object ✅
print(config1 is config2)          # True
```

### Method 2: Decorator

```python
def singleton(cls):
    instances = {}
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    return get_instance

@singleton
class ConfigManager:
    def __init__(self):
        self.settings = {"debug": True}

config1 = ConfigManager()
config2 = ConfigManager()
print(config1 is config2)   # True
```

### Method 3: Metaclass (cleanest for large codebases / many singletons)

```python
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class ConfigManager(metaclass=SingletonMeta):
    def __init__(self):
        self.settings = {"debug": True}

config1 = ConfigManager()
config2 = ConfigManager()
print(config1 is config2)   # True
```

### Method 4: Module-level singleton (the "Pythonic" way 🐍)

```python
# config_manager.py
class _ConfigManager:
    def __init__(self):
        self.settings = {"debug": True}

config_manager = _ConfigManager()   # created once, on import

# anywhere_else.py
from config_manager import config_manager
config_manager.settings["debug"] = False
# every importer gets the SAME object — modules are cached by Python
```

### Method 5: `functools` — thread-safe lazy singleton via decorator

```python
import functools

@functools.lru_cache(maxsize=1)
def get_config_manager():
    return ConfigManagerPlainClass()   # instantiated only once, cached
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Instances created | Many, uncoordinated | Exactly one, guaranteed |
| State consistency | Diverges across instances | Always in sync |
| Access | Ad-hoc `ClassName()` everywhere | Same object every time |
| Resource use | Wasteful (repeated setup) | Setup happens once |

---

## 🧠 Pros & Cons

### ✅ Pros
- **Guaranteed single instance** — no accidental duplication of
  costly resources (DB pools, caches, config)
- **Global access point** — no need to pass the instance around
  everywhere (dependency-free convenience)
- **Lazy initialization possible** — instance created only when
  first needed, saving resources at startup
- **Controlled access** to shared resources (e.g. a single logger
  writing to one file, avoiding write conflicts)

### ❌ Cons
- **Global state in disguise** — behaves like a global variable,
  making code harder to reason about (hidden dependencies)
- **Hurts testability** — hard to mock/replace in unit tests since
  it's not injected; tests can leak state into each other via the
  shared instance
- **Hides dependencies** — a class secretly depending on a Singleton
  doesn't show that dependency in its constructor, unlike Dependency
  Injection
- **Concurrency issues** — in multithreaded code, naive
  implementations can create two instances if two threads check
  `if instance is None` at the same time (needs locks: `threading.Lock`)
- **Violates Single Responsibility Principle** — the class manages
  both its own business logic AND its own instantiation control
- **Tight coupling** — code calling `ConfigManager()` directly is
  coupled to that concrete class, not an abstraction

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Guarantees exactly one instance of
  a class exists, with a single global access point to it.
- **Most common Python implementations:** `__new__` override,
  metaclass, decorator, or simply a module (modules are singletons
  by nature since Python caches imports).
- **Thread-safety gotcha:** naive lazy singletons are **not**
  thread-safe — need a lock around the creation check (often called
  "double-checked locking").
- **Similar-sounding pattern to not confuse it with:**
  - **Factory** — about *which class* to instantiate; Singleton is
    about *how many* instances exist (usually just one).
  - **Monostate (Borg pattern)** — alternative approach: allows
    *multiple instances*, but they all **share the same state**
    (via shared `__dict__`) instead of being the literal same object.
- **Common criticism in interviews:** Singleton is often called an
  "anti-pattern" because it introduces global state and hurts
  testability — be ready to discuss **Dependency Injection** as the
  usual alternative (pass the single instance in explicitly, rather
  than having the class enforce its own uniqueness).
- **Real examples:** Database connection pools, logging modules,
  application configuration, thread pools, caches.

---

## ✅ Use it when
- Exactly one instance is truly required system-wide (e.g. hardware
  interface access, single config source)
- You need lazy, controlled initialization of an expensive resource
- Shared state genuinely needs central coordination

## ❌ Skip it when
- You just want "convenient global access" — that's a smell for
  passing dependencies explicitly (Dependency Injection) instead
- You need to unit test in isolation — Singleton's shared global
  state makes tests unpredictable across runs
- You might need multiple instances later (e.g. multi-tenant apps) —
  Singleton locks you into "exactly one," forever

---

**TL;DR:** Singleton restricts a class to one instance and gives a
single access point to it, implemented via `__new__`, metaclass,
decorator, or a plain module — powerful for shared resources, but
often criticized for introducing global state and hurting
testability.