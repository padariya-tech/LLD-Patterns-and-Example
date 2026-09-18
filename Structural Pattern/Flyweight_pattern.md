# Flyweight Pattern 🪶

> **One-liner (say this in an interview):**
> "Flyweight minimizes memory usage by **sharing** the parts of an
> object's state that are common across many objects (intrinsic
> state), instead of storing that data redundantly in every single
> instance."

**Category:** Structural
**Real-world analogy:** A print shop's font glyphs. Whether you're
printing 1 page or 10,000 pages of text, the shape/design of the
letter "A" is stored **once**. Each printed "A" on the page just
references that one shared glyph, plus its own position on the page
— you don't create 10,000 separate copies of "what an A looks like."

---

## 🔴 The Problem (why this pattern exists)

You need to create a **huge number of similar objects**, and each
one stores its own full copy of data that's actually **identical**
across most instances — leading to massive memory waste.

```python
class Tree:
    def __init__(self, x, y, name, color, texture):
        self.x = x                # unique per tree
        self.y = y                # unique per tree
        self.name = name          # shared, e.g. "Oak"
        self.color = color        # shared, e.g. "Green"
        self.texture = texture    # shared — could be a LARGE image/mesh object!

# 😬 Rendering a forest with 1,000,000 trees...
forest = []
for i in range(1_000_000):
    forest.append(Tree(x=i, y=i, name="Oak", color="Green",
                        texture=load_heavy_texture("oak.png")))
# Each Tree loads/stores its OWN full copy of "oak.png" texture data.
# 1,000,000 trees × (texture size) = catastrophic memory usage,
# even though all Oak trees look visually IDENTICAL.
```

**Pain points:**
- ❌ **Massive memory waste** — identical data (name, color, texture)
  duplicated across every single instance
- ❌ Doesn't scale — systems with thousands/millions of similar
  objects (particles, characters, tiles, glyphs) become unusable
- ❌ Redundant loading — the same heavy resource (texture, font,
  image) is loaded/stored again and again for no reason
- ❌ No separation between "data that's the same for a whole
  category" vs. "data that's unique per instance"

---

## 🟢 The Fix — Flyweight Pattern

**Idea:** Split object state into two parts:
- **Intrinsic state** — shared, context-independent data (name,
  color, texture) → stored **once** in a shared Flyweight object
- **Extrinsic state** — unique, context-dependent data (x, y
  position) → kept **outside** the flyweight, passed in when needed

A **Factory** ensures flyweights are reused instead of recreated.

```python
class TreeType:                        # 🪶 the Flyweight — shared, intrinsic state
    def __init__(self, name, color, texture):
        self.name = name
        self.color = color
        self.texture = texture         # heavy resource — loaded ONCE per type

    def draw(self, x, y):              # extrinsic state passed in at call time
        print(f"Drawing {self.color} {self.name} at ({x}, {y})")


class TreeTypeFactory:                 # ensures reuse — never duplicates a TreeType
    _tree_types = {}

    @classmethod
    def get_tree_type(cls, name, color, texture):
        key = (name, color, texture)
        if key not in cls._tree_types:
            print(f"Creating NEW TreeType: {name}/{color}")   # only happens once per unique type
            cls._tree_types[key] = TreeType(name, color, texture)
        return cls._tree_types[key]


class Tree:                            # holds only extrinsic state + a reference to shared type
    def __init__(self, x, y, tree_type: TreeType):
        self.x = x
        self.y = y
        self.tree_type = tree_type     # 🔗 shared reference, not a copy!

    def draw(self):
        self.tree_type.draw(self.x, self.y)


# --- Usage ---
forest = []
for i in range(1_000_000):
    oak_type = TreeTypeFactory.get_tree_type("Oak", "Green", "oak.png")
    forest.append(Tree(x=i, y=i, tree_type=oak_type))

# Creating NEW TreeType: Oak/Green   <-- printed only ONCE, not 1,000,000 times!
forest[0].draw()      # Drawing Green Oak at (0, 0)
forest[999].draw()    # Drawing Green Oak at (999, 999)

print(forest[0].tree_type is forest[999].tree_type)   # True — same shared object ✅
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Shared data (name, color, texture) | Duplicated in every instance | Stored once, shared via reference |
| Unique data (x, y) | Stored per instance (necessary) | Still stored per instance (unavoidable) |
| Memory usage at scale | Grows linearly with object count × full data | Grows with unique combos, not total count |
| Creating a "new" type | Always allocates fresh memory | Factory reuses existing flyweight if it exists |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Reduces memory footprint when
  many objects share large amounts of identical data, by separating
  shared ("intrinsic") state from per-instance ("extrinsic") state.
- **Key vocabulary (say these exact terms):**
  - **Intrinsic state** — shared, immutable, stored inside the
    flyweight (e.g. texture, font glyph shape)
  - **Extrinsic state** — unique per object, passed in from outside
    at the time it's needed (e.g. x/y position)
- **Key mechanism:** a **Factory** that caches and returns existing
  flyweights instead of creating duplicates — this caching step is
  what makes Flyweight actually work.
- **Design principle it embodies:** Memory optimization through
  sharing — trades a bit of extra complexity (managing two kinds of
  state) for large memory savings at scale.
- **Similar-sounding pattern to not confuse it with:**
  - **Singleton** — guarantees exactly **one** instance total;
    Flyweight can have **many** shared instances (one per unique
    intrinsic-state combination, not globally one).
  - **Object Pool** — reuses **mutable** objects to avoid
    allocation/GC cost (e.g. connections, threads); Flyweight shares
    **immutable** state to save memory, and shared objects are used
    *concurrently* by many owners, not checked in/out one at a time.
- **Real examples:** Character glyphs in text editors/word
  processors, particle systems in games (bullets, raindrops sharing
  sprite/texture data), string interning in Python (`sys.intern`),
  tile-based game maps (many tiles referencing few tile-type objects).

---

## ✅ Use it when
- You need a **huge number** of objects and memory usage is a real
  concern
- Most of each object's data is identical across many instances
- The shared data can cleanly be separated from the per-instance data

## ❌ Skip it when
- Object count is small — the added complexity isn't worth it for a
  handful of objects
- Objects don't actually share meaningful data — forcing intrinsic/
  extrinsic separation onto genuinely unique objects adds needless
  complexity for no memory benefit

---

**TL;DR:** Instead of every object storing its own full copy of data
that's actually identical across thousands of instances, split state
into shared ("intrinsic") and unique ("extrinsic") parts — store the
shared part once via a caching Factory, and pass in the unique part
whenever it's needed.