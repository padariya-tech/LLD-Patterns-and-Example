# Memento Pattern 💾

> **One-liner (say this in an interview):**
> "Memento captures and externalizes an object's internal state so it
> can be **restored later** (undo/rollback), without violating
> encapsulation — the object's private details never get exposed to
> whoever's storing the snapshot."

**Category:** Behavioral
**Real-world analogy:** A video game save file. When you save, the
game writes a snapshot of your progress to a file — but you (the
player) can't read or edit that file's internal format, you can only
hand it back to the game to **restore** exactly that state later.
The save system doesn't need to expose *how* the game stores its
internal data.

---

## 🔴 The Problem (why this pattern exists)

You need **undo/rollback** functionality — the ability to save an
object's state and restore it later. Naive fix: expose all internal
fields publicly so external code can copy and later reassign them.

```python
class TextEditor:
    def __init__(self):
        self.content = ""
        self.cursor_position = 0
        self.font_size = 12

# 😬 To "save" state, external code must reach into EVERY private field
editor = TextEditor()
editor.content = "Hello"

# manual, error-prone snapshot — and it breaks encapsulation
saved_content = editor.content
saved_cursor = editor.cursor_position
saved_font = editor.font_size

editor.content = "Hello World"     # edit happens...

# manual, error-prone restore
editor.content = saved_content
editor.cursor_position = saved_cursor
editor.font_size = saved_font
```

**Pain points:**
- ❌ **Breaks encapsulation** — external code needs direct access to
  every internal field to save/restore state
- ❌ Adding a new field to `TextEditor` means **updating every place**
  that manually saves/restores state
- ❌ **Error-prone** — easy to forget a field, or restore fields in
  the wrong order/combination
- ❌ No natural way to keep a **history** of multiple states (undo
  stack) — you're just juggling loose variables

---

## 🟢 The Fix — Memento Pattern

**Idea:** The object itself (**Originator**) knows how to create a
**Memento** — an opaque snapshot of its own state — and how to
restore itself from one. External code (**Caretaker**) stores
Mementos but **never looks inside them** — it just hands them back
to the Originator when a restore is needed.

```python
import copy

# 1️⃣ Memento — an opaque snapshot; caretaker can't peek inside meaningfully
class EditorMemento:
    def __init__(self, content, cursor_position, font_size):
        self._content = content
        self._cursor_position = cursor_position
        self._font_size = font_size
    # note: no public getters exposed to the outside world — only
    # the Originator (TextEditor) knows how to use this snapshot

# 2️⃣ Originator — knows how to save/restore its OWN state
class TextEditor:
    def __init__(self):
        self.content = ""
        self.cursor_position = 0
        self.font_size = 12

    def save(self) -> EditorMemento:
        return EditorMemento(self.content, self.cursor_position, self.font_size)

    def restore(self, memento: EditorMemento):
        self.content = memento._content
        self.cursor_position = memento._cursor_position
        self.font_size = memento._font_size

# 3️⃣ Caretaker — stores mementos, never inspects their contents
class History:
    def __init__(self):
        self._snapshots = []

    def push(self, memento: EditorMemento):
        self._snapshots.append(memento)

    def pop(self) -> EditorMemento:
        return self._snapshots.pop()


# --- Usage ---
editor = TextEditor()
history = History()

editor.content = "Hello"
history.push(editor.save())         # 💾 snapshot before the risky edit

editor.content = "Hello World! Oops typo"
print(editor.content)                # Hello World! Oops typo

editor.restore(history.pop())        # ⏪ undo!
print(editor.content)                # Hello

# ✅ TextEditor's internal fields never leaked to History.
# Adding a new field to TextEditor? Only EditorMemento & TextEditor
# change — History (the caretaker) stays untouched.
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Saving state | External code copies every field manually | Originator creates its own Memento |
| Restoring state | External code reassigns every field manually | Originator restores itself from a Memento |
| Encapsulation | Broken — internals exposed to callers | Preserved — Memento is opaque to the caretaker |
| Adding a new field | Update every save/restore call site | Update only the Originator + Memento |
| Undo history | Manual juggling of loose variables | Clean stack of Memento objects |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Enables saving/restoring an
  object's state for undo/rollback, without exposing that object's
  internal structure to the code managing the history.
- **Key players:**
  - **Originator** — the object whose state is being saved; creates
    and restores from Mementos
  - **Memento** — an immutable snapshot of the Originator's state;
    intentionally opaque to everyone except the Originator
  - **Caretaker** — stores Mementos (e.g. in a stack for undo
    history) but never reads or modifies their contents
- **How encapsulation is preserved (Python nuance):** Python has no
  true `private` access control, so this is usually done by
  **convention** — Memento's fields are prefixed `_` and only the
  Originator class is expected to touch them (or, in stricter
  languages like Java/C++, Memento can use actual access modifiers +
  nested/friend classes to enforce it at compile time).
- **Design principle it embodies:** Encapsulation — state capture and
  restoration logic stays inside the object that owns that state.
- **Similar-sounding pattern to not confuse it with:**
  - **Command** — often used **alongside** Memento: a Command
    represents the *action* taken, and can store a Memento internally
    to know how to undo that specific action (more powerful than a
    simple inverse operation).
  - **Prototype** — clones an object to make a **new, usable copy**
    for continued use; Memento captures a snapshot **purely for
    restoration later**, not meant to be used as a live object on
    its own.
- **Real examples:** Undo/redo in text editors and IDEs, game save
  states/checkpoints, database transaction rollback (savepoints),
  form "revert to draft" features, version control snapshots
  (conceptually similar to `git commit`/`git checkout`).

---

## ✅ Use it when
- You need undo/rollback functionality for an object's state
- You want to snapshot state **without breaking encapsulation** —
  the object's internal representation should stay hidden from
  whoever manages the history
- You need to maintain a **history** of multiple past states

## ❌ Skip it when
- The object's state is simple/small — just copying a couple of
  public fields directly might be simpler than the full pattern
- Storing many large snapshots would be memory-expensive — consider
  storing **diffs/deltas** instead of full state copies for
  large objects, or combine with Command's inverse-operation approach

---

**TL;DR:** Instead of external code reaching into an object's
internals to manually save/restore state, let the object create its
own opaque Memento snapshot of itself — a Caretaker stores these
snapshots for undo history, without ever needing to know what's
inside them.