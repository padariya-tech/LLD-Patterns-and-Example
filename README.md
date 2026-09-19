# Design Patterns Repo 📚

A personal reference/revision repo covering the classic Gang of Four
(GoF) design patterns, organized by category, with Python examples.

---

## 📁 Structure

```
.
├── README.md                 # you are here
├── revision.md                # 🎯 master cheat sheet — all patterns, quick lookup table
│
├── Creational/                # patterns about HOW objects are created
│   ├── singleton.md
│   ├── factory.md
│   ├── builder.md
│   └── prototype.md
│
├── Structural/                 # patterns about HOW objects/classes are composed
│   ├── adapter.md
│   ├── bridge.md
│   ├── composite.md
│   ├── decorator.md
│   ├── facade.md
│   ├── flyweight.md
│   └── proxy.md
│
└── Behavioral/                 # patterns about HOW objects communicate
    ├── strategy.md
    ├── observer.md
    ├── command.md
    ├── state.md
    ├── template_method.md
    ├── chain_of_responsibility.md
    ├── interpreter.md
    ├── iterator.md
    ├── visitor.md
    ├── mediator.md
    └── memento.md
```

---

## 📖 What's Inside

| Location | Purpose |
|---|---|
| **`Master_Revision.md`** | Single condensed cheat sheet — every pattern in one place, with a quick-lookup table, 1-2 line category primers, and short need/problem/fix summaries. Best for a fast pre-interview scan. |
| **`Creational/*.md`** | One file per creational pattern — the need for it, the problem without it, and full before/after Python code. |
| **`Structural/*.md`** | One file per structural pattern — same format as above. |
| **`Behavioral/*.md`** | One file per behavioral pattern — same format as above. |

Each individual pattern file follows a consistent template:
1. **One-liner** — interview-ready summary
2. **Real-world analogy**
3. **The Problem** — pain points + "before" code
4. **The Fix** — the pattern applied + "after" code
5. **Pros & Cons** / trade-offs
6. **Interview Cheat Sheet** — key vocabulary, similar patterns it's often confused with, real-world examples
7. **When to use / when to skip**

---

## 🗺️ Category Quick Reference

| Category | Concerned with |
|---|---|
| 🏗️ **Creational** | Controlling and abstracting the object-creation process |
| 🧩 **Structural** | Composing classes/objects into larger, flexible structures |
| 🎭 **Behavioral** | Assigning responsibility and communication between objects |

---

## 🧭 How to Use This Repo

- **Quick revision before an interview?** → open `Master_Revision.md`
- **Need the full explanation + code for one specific pattern?** →
  go to its category folder and open that pattern's `.md` file