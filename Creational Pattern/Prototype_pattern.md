# Prototype Pattern 🧬

> **One-liner (say this in an interview):**
> "Prototype lets you create new objects by **cloning an existing
> instance** instead of building them from scratch with `new`/a
> constructor — useful when creation is expensive or the object's
> exact configuration is what you want to copy."

**Category:** Creational
**Real-world analogy:** Cell division / cloning a sheep (Dolly 🐑).
Instead of building a new sheep gene-by-gene from a blueprint, you
copy an existing sheep's DNA. Faster, and you get an exact (or
tweakable) replica without redoing all the setup.

---

## 🔴 The Problem (why this pattern exists)

You need many objects that are **mostly identical** to one you
already have — but building each one from scratch is **expensive**
(DB calls, heavy computation, complex config) or you simply don't
know the exact class/parameters up front. Naive fix: call the
constructor again and re-do all that setup, every time.

```python
class Document:
    def __init__(self, title, content, styles, permissions):
        self.title = title
        self.content = content
        self.styles = styles              # e.g. loaded from a theme file
        self.permissions = permissions    # e.g. fetched from a DB call

# 😬 Every "similar" document redoes the SAME expensive setup
doc1 = Document(
    "Report", "...", load_styles_from_theme("corporate"),
    fetch_permissions_from_db("team_a"),
)
doc2 = Document(
    "Report Copy", "...", load_styles_from_theme("corporate"),   # same call again!
    fetch_permissions_from_db("team_a"),                          # same call again!
)
```

**Pain points:**
- ❌ **Repeated expensive setup** — re-loading styles, re-querying a
  DB just to get the same values as an object you already have
- ❌ Client code needs to **know all constructor parameters** in
  detail to recreate something similar
- ❌ Tightly coupled to **concrete classes** — can't create a copy
  of an object whose exact class you don't even know at runtime
  (e.g. it came from a plugin)
- ❌ Wasteful when the "new" object is 95% the same as an existing one

---

## 🟢 The Fix — Prototype Pattern

**Idea:** Give the object a `clone()` method. To make a new similar
object, **copy the existing one** (using Python's `copy` module)
instead of re-running the constructor and all its setup logic.

```python
import copy

class Document:
    def __init__(self, title, content, styles, permissions):
        self.title = title
        self.content = content
        self.styles = styles
        self.permissions = permissions

    def clone(self, **overrides):
        new_doc = copy.deepcopy(self)   # 🧬 copy everything, no re-setup
        for key, value in overrides.items():
            setattr(new_doc, key, value)
        return new_doc


# --- Usage ---
doc1 = Document(
    "Report", "...", load_styles_from_theme("corporate"),
    fetch_permissions_from_db("team_a"),
)

# ✅ No re-loading styles, no re-querying DB — just clone & tweak
doc2 = doc1.clone(title="Report Copy")
doc3 = doc1.clone(title="Report v2", content="updated content...")

print(doc2.styles is doc1.styles)   # False — deep copy, independent object
print(doc2.title)                   # "Report Copy"
```

### `copy.copy()` vs `copy.deepcopy()` — know this cold
```python
shallow = copy.copy(doc1)      # copies doc1, but nested objects (styles, 
                                # permissions) are SHARED references
deep = copy.deepcopy(doc1)     # copies doc1 AND recursively copies every
                                # nested object — fully independent
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Creating a similar object | Re-run constructor + all setup | `clone()` an existing instance |
| Expensive setup (DB/config load) | Repeated every time | Done once, reused via copy |
| Coupling | Must know concrete class + all params | Just call `.clone()` on any prototype |
| Unknown/dynamic classes | Can't recreate without knowing the class | Clone works regardless of class details |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Avoids re-running expensive/complex
  construction logic by copying an existing, already-configured
  object instead.
- **Key mechanism in Python:** the `copy` module — `copy.copy()`
  (shallow) and `copy.deepcopy()` (deep), or a custom `clone()`
  method wrapping one of these.
- **Shallow vs Deep — classic gotcha:** shallow copy shares nested
  mutable objects (mutate one, both change); deep copy fully
  duplicates the nested structure (fully independent).
- **Design principle it embodies:** Copy an instance instead of
  depending on its concrete class/constructor — reduces coupling to
  "which class do I need to `import` and call."
- **Similar-sounding pattern to not confuse it with:**
  - **Factory** — creates objects **from scratch** via constructors;
    Prototype creates by **copying an existing instance**.
  - **Builder** — constructs a complex object **step-by-step** from
    parts; Prototype **duplicates a finished object** in one shot.
  - **Singleton** — guarantees **only one** instance ever; Prototype
    is literally about making **many copies**.
- **Real examples:** JavaScript's prototypal inheritance (`Object.create()`),
  `Cell.clone()` in spreadsheet apps, cloning game entities (NPCs,
  bullets) in game engines instead of re-instantiating from scratch,
  Python's `copy.deepcopy()` used directly as a poor-man's prototype.

---

## ✅ Use it when
- Object creation is expensive (DB calls, network requests, heavy
  computation) and you need many similar objects
- You want to avoid a huge hierarchy of subclasses just to produce
  slightly different pre-configured objects
- The exact class of the object to duplicate isn't known until
  runtime (e.g. plugin systems)

## ❌ Skip it when
- Object creation is cheap and simple — a normal constructor call is
  simpler and clearer
- Objects contain **unclonable resources** (open file handles, live
  network sockets, threads) — cloning these gets messy or meaningless
  and needs careful custom handling

---

**TL;DR:** Instead of rebuilding a similar object from scratch every
time (redoing expensive setup), give it a `clone()` method that
copies an existing instance — using shallow or deep copy depending
on whether nested objects should be shared or fully independent.