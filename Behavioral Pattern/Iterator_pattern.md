# Iterator Pattern 🔁

> **One-liner (say this in an interview):**
> "Iterator provides a way to access elements of a collection
> **sequentially**, without exposing its underlying structure
> (array, tree, linked list...) — so client code can traverse
> different collections through one uniform interface."

**Category:** Behavioral
**Real-world analogy:** A TV remote's "channel up" button. You don't
need to know how channels are internally stored or wired at the
broadcast station — you just press "next" repeatedly, and the remote
(iterator) handles moving to the next channel for you.

---

## 🔴 The Problem (why this pattern exists)

You have collections with **different internal structures** (list,
tree, linked list, custom data structure), and client code needs to
traverse them — but exposing the internal structure directly leaks
implementation details and forces every caller to know how to walk
it.

```python
class BookCollection:
    def __init__(self):
        self._books = []          # 😬 internal structure exposed directly

    def add_book(self, book):
        self._books.append(book)

# 😬 Client code must know it's a list, and index into it manually
collection = BookCollection()
collection.add_book("Dune")
collection.add_book("1984")

i = 0
while i < len(collection._books):     # reaching into private internals!
    print(collection._books[i])
    i += 1
# If BookCollection later switches to a tree or a set internally,
# ALL client code that does this breaks.
```

**Pain points:**
- ❌ Client code **reaches into internal data structures** directly
  (`._books`), violating encapsulation
- ❌ Traversal logic is **duplicated** everywhere the collection is
  looped over
- ❌ If the internal structure changes (list → tree → database
  cursor), **every caller breaks**
- ❌ No standard way to support **multiple simultaneous traversals**
  of the same collection (e.g. two independent "current position"
  pointers)
- ❌ Different collection types need different, inconsistent
  traversal code at every call site

---

## 🟢 The Fix — Iterator Pattern

**Idea:** Give the collection a method that returns a separate
**Iterator object** responsible for traversal. The iterator exposes
a simple, uniform interface (`has_next()` / `next()`, or Python's
`__iter__`/`__next__`) — client code never touches the collection's
internals directly.

```python
class Book:
    def __init__(self, title):
        self.title = title

# 1️⃣ Collection — exposes an iterator, hides its internal storage
class BookCollection:
    def __init__(self):
        self._books = []          # internals can now change freely

    def add_book(self, book):
        self._books.append(book)

    def __iter__(self):           # 🔁 returns an Iterator — Python's built-in hook
        return BookIterator(self._books)

# 2️⃣ Iterator — knows HOW to traverse, hides that from the client
class BookIterator:
    def __init__(self, books):
        self._books = books
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self._index >= len(self._books):
            raise StopIteration
        book = self._books[self._index]
        self._index += 1
        return book


# --- Usage ---
collection = BookCollection()
collection.add_book(Book("Dune"))
collection.add_book(Book("1984"))

for book in collection:              # ✅ client uses standard for-loop syntax
    print(book.title)
# Dune
# 1984

# ✅ Internal storage can now change (list -> dict -> tree) without
# breaking ANY client code — as long as __iter__/__next__ still work.
```

### Note: Python already builds this in — you rarely write it by hand

```python
# Any object implementing __iter__ (and __next__, or being a generator)
# automatically works with for-loops, list(), sum(), etc.
class BookCollectionSimple:
    def __init__(self):
        self._books = []
    def add_book(self, book):
        self._books.append(book)
    def __iter__(self):
        yield from self._books      # generator — Python handles the Iterator protocol for you

for book in BookCollectionSimple():
    print(book)
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Traversal logic | Duplicated at every call site | Centralized in one Iterator class |
| Access to internals | Client reaches into `._books` directly | Client only calls `next()`/uses `for` loop |
| Changing internal structure | Breaks all client code | Client code untouched |
| Multiple simultaneous traversals | Awkward, needs manual index tracking | Each iterator has its own independent state |
| Consistency across collection types | Every collection traversed differently | One uniform interface for any collection |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Decouples traversal logic from the
  collection's internal structure, giving a uniform way to iterate
  over different kinds of collections without exposing internals.
- **Key players:** `Iterable`/`Collection` (exposes a way to get an
  iterator, e.g. `__iter__`) → `Iterator` (tracks traversal state,
  e.g. `__next__`/`has_next()`).
- **Python-specific note:** the Iterator pattern is **built into the
  language** via the iterator protocol (`__iter__` + `__next__`, plus
  `StopIteration`) — `for` loops, generators, and comprehensions all
  use this under the hood. In Python you rarely hand-roll an Iterator
  class; a **generator function** (`yield`) usually does the job.
- **Design principle it embodies:** Single Responsibility — the
  collection manages storage, the iterator manages traversal state,
  cleanly separated.
- **Similar-sounding pattern to not confuse it with:**
  - **Composite** — defines a **tree structure**; Iterator is often
    used **on top of** a Composite to walk through its tree uniformly.
  - **Visitor** — performs an **operation** on each element as you
    traverse; Iterator focuses purely on **how to move through**
    elements, not what to do with them (though they're often combined).
- **Real examples:** Python's `for` loop / `iter()` / `next()`,
  database cursors (fetch rows one at a time without loading
  everything into memory), file readers (`for line in file`), any
  `yield`-based generator, Java's `Iterator`/`Iterable` interfaces.

---

## ✅ Use it when
- You need to traverse a collection without exposing its internal
  representation
- You want to support multiple simultaneous, independent traversals
  of the same collection
- You want one consistent way to iterate across different collection
  types

## ❌ Skip it when
- The language/library already gives you built-in iteration (as
  Python does) — no need to hand-roll a custom Iterator class for
  simple cases; a generator function is usually enough
- The collection is trivial and traversal logic is genuinely used in
  exactly one place — the extra class adds no real benefit

---

**TL;DR:** Instead of client code reaching directly into a
collection's internals to loop over it, expose a separate Iterator
object that knows how to traverse — collection internals can then
change freely without breaking any client code, and Python gives you
this for free via `__iter__`/`__next__` or generators.