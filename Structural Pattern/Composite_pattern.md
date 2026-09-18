# Composite Pattern 🌳

> **One-liner (say this in an interview):**
> "Composite lets you treat individual objects and groups of objects
> **uniformly**, through the same interface — so client code doesn't
> need to know or care whether it's dealing with a single item or a
> whole tree of them."

**Category:** Structural
**Real-world analogy:** A computer's file system. A `File` and a
`Folder` are treated the same way when you ask for "size" or
"delete" — a folder just delegates the request to everything inside
it (which may themselves be folders). You never need special-case
code like "if it's a folder, loop through it differently."

---

## 🔴 The Problem (why this pattern exists)

You have a **tree-like structure** — files/folders, UI widgets,
org charts, menus — where some things are single items ("leaves")
and others are groups of items ("containers"). Naive fix: treat
them as fundamentally different types, and special-case every
operation.

```python
class File:
    def __init__(self, name, size):
        self.name = name
        self.size = size

class Folder:
    def __init__(self, name):
        self.name = name
        self.children = []       # Files AND Folders mixed together

    def add(self, item):
        self.children.append(item)

# 😬 Client code has to know the difference and branch manually
def total_size(item):
    if isinstance(item, File):
        return item.size
    elif isinstance(item, Folder):
        total = 0
        for child in item.children:
            total += total_size(child)     # recursive if/elif again
        return total
```

**Pain points:**
- ❌ Client code is littered with `isinstance()` checks / `if/elif`
  for "is this a leaf or a container?"
- ❌ Every new operation (size, print, search...) needs the **same
  branching logic repeated**
- ❌ Adding a new leaf/container type → hunt down and update every
  place that branches on type
- ❌ Doesn't scale to deeply nested trees cleanly — the branching
  logic multiplies with tree depth

---

## 🟢 The Fix — Composite Pattern

**Idea:** Define a **common interface** for both leaves and
containers. A container simply loops over its children and calls
the *same* method on each — whether that child is a leaf or another
container, it doesn't care. This works recursively for free.

```python
from abc import ABC, abstractmethod

# 1️⃣ Common interface — leaves AND composites implement this
class FileSystemItem(ABC):
    @abstractmethod
    def get_size(self) -> int: ...

# 2️⃣ Leaf — the simplest building block, no children
class File(FileSystemItem):
    def __init__(self, name, size):
        self.name = name
        self.size = size

    def get_size(self):
        return self.size

# 3️⃣ Composite — holds children (Files or Folders), delegates to them
class Folder(FileSystemItem):
    def __init__(self, name):
        self.name = name
        self.children: list[FileSystemItem] = []

    def add(self, item: FileSystemItem):
        self.children.append(item)

    def get_size(self):
        # 🌳 no isinstance checks — just call get_size() on each child,
        # whether it's a File or another Folder
        return sum(child.get_size() for child in self.children)


# --- Usage ---
root = Folder("root")
root.add(File("resume.pdf", 500))
root.add(File("photo.png", 2000))

pictures = Folder("pictures")
pictures.add(File("cat.jpg", 1500))
pictures.add(File("dog.jpg", 1800))
root.add(pictures)          # a Folder inside a Folder — works seamlessly

print(root.get_size())      # 500 + 2000 + 1500 + 1800 = 5800
# ✅ Client code never had to know which items were files vs folders.
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Handling leaves vs containers | `isinstance()`/`if-elif` branching everywhere | Same interface, no branching |
| Adding a new operation | Repeat the branching logic again | Just implement the method once per class |
| Nested trees | Branching logic multiplies with depth | Recursion "just works" via delegation |
| Client code | Must know the tree's internal structure | Treats leaf and tree identically |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Lets client code treat individual
  objects ("leaf") and groups of objects ("composite") through one
  uniform interface — enabling recursive tree structures without
  special-casing.
- **Key players:** `Component` (common interface) → `Leaf`
  (no children, does the real work) → `Composite` (has children,
  delegates work to them recursively).
- **Design principle it embodies:** Uniformity — client code depends
  on one abstraction regardless of tree depth or shape.
- **Similar-sounding pattern to not confuse it with:**
  - **Decorator** — wraps **one** object to add behavior, no tree
    structure; Composite manages a **whole tree** of objects.
  - **Iterator** — traverses a collection; Composite defines the
    tree's *structure* (Iterator is often used *on top of* a
    Composite to walk through it).
- **Real examples:** DOM tree (HTML elements containing other
  elements), file systems, UI component trees (a `Panel` containing
  `Button`s and other `Panel`s), org charts (`Employee` vs `Manager`
  who "is" also an Employee with reports), JSON/XML nested structures.

---

## ✅ Use it when
- You're modeling a **part-whole hierarchy** (tree structure) —
  files/folders, UI widgets, org charts, menus
- You want client code to apply the same operation uniformly
  regardless of whether it's a single item or a whole subtree
- The set of operations needs to work recursively without manual
  tree-walking logic at every call site

## ❌ Skip it when
- Your data isn't actually tree-shaped — forcing Composite onto flat
  data adds pointless indirection
- Leaves and containers need genuinely **different interfaces** —
  forcing a shared interface (e.g. `add()` on a `File` that can't
  have children) leads to awkward no-op/error methods

---

**TL;DR:** Instead of branching on "is this a leaf or a group?"
everywhere, give leaves and containers the same interface — a
container just delegates each operation to its children, recursively
— so client code treats a single object and an entire tree
identically.