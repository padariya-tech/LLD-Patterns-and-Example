# Design Patterns — Master Revision Sheet 🎯

The 3 categories, in one line each:

| Category | What it's about |
|---|---|
| 🏗️ **Creational** | How objects get **created** — hide/control the instantiation process |
| 🧩 **Structural** | How objects/classes are **composed** into larger structures |
| 🎭 **Behavioral** | How objects **communicate & assign responsibility** to each other |

---

## ⚡ Ultra-Quick Lookup Table

| Pattern | Category | One-line purpose |
|---|---|---|
| Singleton | Creational | Only ONE instance, globally accessible |
| Factory | Creational | Centralize "which class to instantiate" logic |
| Builder | Creational | Build a complex object step-by-step (no telescoping constructor) |
| Prototype | Creational | Create new objects by cloning an existing one |
| Adapter | Structural | Make an incompatible interface compatible |
| Bridge | Structural | Split 2 varying dimensions into 2 hierarchies (avoid class explosion) |
| Composite | Structural | Treat single objects & groups (trees) uniformly |
| Decorator | Structural | Add behavior dynamically by wrapping, not subclassing |
| Facade | Structural | One simple interface over a complex subsystem |
| Flyweight | Structural | Share common data across many objects to save memory |
| Proxy | Structural | Stand-in object controlling access to the real one |
| Strategy | Behavioral | Swap interchangeable algorithms at runtime |
| Observer | Behavioral | One-to-many: notify dependents automatically on change |
| Command | Behavioral | Turn a request into an object (undo, queue, log) |
| State | Behavioral | Object changes behavior as its internal state object changes |
| Template Method | Behavioral | Fixed algorithm skeleton, subclasses override specific steps |
| Chain of Responsibility | Behavioral | Pass a request along a chain until someone handles it |
| Interpreter | Behavioral | Represent grammar rules as classes, evaluate via a tree |
| Iterator | Behavioral | Traverse a collection without exposing its internal structure |
| Visitor | Behavioral | Add new operations to classes without modifying them |
| Mediator | Behavioral | Centralize many-to-many communication into one object |
| Memento | Behavioral | Snapshot & restore state without breaking encapsulation |

---

# 🏗️ Creational Patterns
*Hide/control **how** objects are created.*

### Singleton 🔒
- **Need:** Ensure a class has only one instance + global access point (config, logger, DB pool).
- **Problem without it:** Multiple instances → inconsistent state, wasted resources.
- **Fix:** Guard instantiation — via `__new__` override, metaclass, decorator, module-level object, or static `get_instance()` + "private" constructor.
```python
class Singleton:
    _instance = None
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```
- **Pros:** guaranteed single instance, controlled/lazy init, global access.
- **Cons:** global state in disguise, hurts testability, hides dependencies, thread-safety issues if naive.
- **Don't confuse with:** Monostate/Borg (many instances, shared state, not the same object).

### Factory 🏭
- **Need:** Decouple *which class* to instantiate from the calling code.
- **Problem without it:** Duplicated `if/elif` creation logic scattered everywhere.
- **Fix:** One factory method/class decides which concrete class to build.
```python
class AnimalFactory:
    @staticmethod
    def create(pet_type):
        return {"dog": Dog, "cat": Cat}[pet_type]()
```
- **Pros:** decoupled from concrete classes, single source of truth for creation.
- **Cons:** adds indirection; unnecessary if only one type ever exists.
- **Don't confuse with:** Builder (step-by-step complex object), Abstract Factory (families of related objects).

### Builder 🧱
- **Need:** Construct a complex object with many optional parts, step by step.
- **Problem without it:** "Telescoping constructor" — huge param list, unreadable, error-prone.
- **Fix:** Chainable setter methods + a final `build()`.
```python
pizza = (PizzaBuilder().set_size("L").add_topping("cheese").build())
```
- **Pros:** readable, flexible, validates before final object is built.
- **Cons:** overkill for objects with 2-3 simple fields (use `@dataclass` instead).
- **Don't confuse with:** Factory (one-shot creation), Prototype (clone instead of build).

### Prototype 🧬
- **Need:** Create new objects by copying an existing, already-configured one.
- **Problem without it:** Re-running expensive setup (DB calls, config loads) for every similar object.
- **Fix:** `clone()` method using `copy.deepcopy()` / `copy.copy()`.
```python
new_doc = existing_doc.clone(title="Copy")
```
- **Pros:** avoids expensive re-creation, works even if exact class is unknown at runtime.
- **Cons:** shallow vs deep copy gotchas; unclonable resources (sockets, file handles) are messy.
- **Don't confuse with:** Factory (builds from scratch), Singleton (opposite goal — many copies vs one instance).

---

# 🧩 Structural Patterns
*Compose classes/objects into larger structures while keeping them flexible.*

### Adapter 🔌
- **Need:** Make two incompatible interfaces work together.
- **Problem without it:** Client code breaks calling a 3rd-party/legacy class with a different method signature.
- **Fix:** Wrap the incompatible class; translate calls internally.
```python
class PayPalAdapter(PaymentProcessor):
    def pay(self, amount): self._sdk.send_payment(int(amount*100))
```
- **Pros:** no changes to either original class; reusable across libraries.
- **Cons:** adds an extra layer for what might be a trivial mismatch.
- **Don't confuse with:** Proxy (same interface, controls access), Decorator (same interface, adds behavior).

### Bridge 🌉
- **Need:** Decouple two independently varying dimensions (e.g. shape × color).
- **Problem without it:** M×N subclass explosion (every shape/color combo = new class).
- **Fix:** Split into 2 hierarchies connected via composition, not inheritance.
```python
class Circle(Shape):
    def draw(self): print(f"{self._color.fill()} Circle")
```
- **Pros:** extend either side independently, no combinatorial explosion.
- **Cons:** upfront design complexity if only one dimension varies.
- **Don't confuse with:** Adapter (retrofit fix vs. designed upfront), Strategy (one axis, not two hierarchies).

### Composite 🌳
- **Need:** Treat individual objects and groups (trees) the same way.
- **Problem without it:** `isinstance()`/`if-elif` branching everywhere to distinguish leaf vs. container.
- **Fix:** Common interface; composite delegates the operation recursively to children.
```python
class Folder(FileSystemItem):
    def get_size(self): return sum(c.get_size() for c in self.children)
```
- **Pros:** uniform treatment, recursion "just works" for any depth.
- **Cons:** forcing shared interface onto genuinely different leaf/container needs gets awkward.
- **Don't confuse with:** Decorator (wraps ONE object), Iterator (often used to walk a Composite tree).

### Decorator 🎁
- **Need:** Add behavior to an object dynamically, in any combination.
- **Problem without it:** Subclass explosion trying to cover every feature combination.
- **Fix:** Wrap the object in decorator(s) sharing its interface; stack freely.
```python
coffee = WhippedCreamDecorator(SugarDecorator(MilkDecorator(SimpleCoffee())))
```
- **Pros:** composable, no class explosion, runtime flexibility.
- **Cons:** many stacked layers can complicate debugging.
- **Don't confuse with:** Proxy (controls access, doesn't add features), Adapter (fixes interface, not behavior).

### Facade 🏛️
- **Need:** Simplify access to a complex subsystem of many classes.
- **Problem without it:** Client must know & coordinate many subsystem classes in exact order.
- **Fix:** One class internally coordinates the subsystem; exposes 1-2 simple methods.
```python
class ComputerFacade:
    def start(self): self.cpu.freeze(); ...; self.cpu.execute()
```
- **Pros:** decouples client from subsystem internals, easy refactoring.
- **Cons:** unnecessary if subsystem is already simple.
- **Don't confuse with:** Adapter (fixes mismatch), Mediator (coordinates PEERS, not client→subsystem).

### Flyweight 🪶
- **Need:** Reduce memory when many objects share the same data.
- **Problem without it:** Each of millions of objects stores its own full copy of identical data.
- **Fix:** Split intrinsic (shared) vs extrinsic (unique) state; cache shared parts via a factory.
```python
tree_type = TreeTypeFactory.get_tree_type("Oak", "Green", texture)  # reused, not recreated
```
- **Pros:** massive memory savings at scale.
- **Cons:** added complexity not worth it for small object counts.
- **Don't confuse with:** Singleton (ONE instance total), Object Pool (reuses mutable objects, not shared immutable state).

### Proxy 🛡️
- **Need:** Control access to an object (lazy load, permissions, logging).
- **Problem without it:** Expensive objects created immediately/always; no centralized access control.
- **Fix:** Same-interface stand-in that decides when/whether to forward to the real object.
```python
class LazyImageProxy(Image):
    def display(self):
        if self._real is None: self._real = HighResImage(self.filename)
        self._real.display()
```
- **Pros:** lazy loading, access control, caching/logging — transparently.
- **Cons:** unnecessary indirection if no real need for control exists.
- **Don't confuse with:** Decorator (adds features, same interface), Adapter (different interface).

---

# 🎭 Behavioral Patterns
*Define **how objects interact** and how responsibility is distributed.*

### Strategy 🎯
- **Need:** Swap interchangeable algorithms at runtime.
- **Problem without it:** Giant `if/elif` selecting behavior; violates Open/Closed.
- **Fix:** Each algorithm = its own class behind a common interface; context delegates.
```python
calculator.set_strategy(VIPDiscount())
```
- **Confuse with:** State (same code shape, different intent — algorithm choice vs. lifecycle stage).

### Observer 👀
- **Need:** Notify multiple dependents automatically when one object's state changes.
- **Problem without it:** Hardcoded calls to every dependent; tight coupling.
- **Fix:** Subject keeps a list of Observers, loops & calls `update()` on each.
```python
station.subscribe(phone); station.set_temperature(30)
```
- **Confuse with:** Pub-Sub (has a broker in between), Mediator (many-to-many, not one-to-many).

### Command 🎮
- **Need:** Turn a request into an object — enables undo, queuing, logging.
- **Problem without it:** Hardcoded trigger→action coupling; no undo/history support.
- **Fix:** `Command` objects with `execute()`/`undo()`; invoker just calls `execute()`.
```python
button = Button(TurnOnCommand(light))
```
- **Confuse with:** Strategy (algorithm, not action+undo), Chain of Responsibility (no chain here).

### State 🚦
- **Need:** Object changes behavior as its internal state changes.
- **Problem without it:** Repeated `if/elif` on a state flag, in every method.
- **Fix:** Each state = its own class; state decides its own transitions.
```python
light.state.next(light)   # state swaps itself
```
- **Confuse with:** Strategy (client picks it, fixed for lifetime vs. self-transitioning).

### Template Method 📋
- **Need:** Share an algorithm's structure across subclasses, vary only specific steps.
- **Problem without it:** Entire algorithm duplicated per subclass for minor tweaks.
- **Fix:** Base class fixes the skeleton; calls overridable step methods.
```python
def prepare(self): self.boil_water(); self.brew(); self.pour_in_cup(); self.add_condiments()
```
- **Confuse with:** Strategy (composition, swaps WHOLE algorithm vs. inheritance, varies STEPS).

### Chain of Responsibility ⛓️
- **Need:** Let a request be handled by one of several possible handlers, sender doesn't choose which.
- **Problem without it:** One method/class knows about every handler & condition.
- **Fix:** Handlers linked in a chain; each handles or passes it on.
```python
level1.set_next(level2).set_next(manager)
```
- **Confuse with:** Decorator (always processes, doesn't skip), Observer (broadcasts to ALL at once).

### Interpreter 🗣️
- **Need:** Evaluate sentences in a simple custom grammar/language.
- **Problem without it:** Fragile string-parsing logic, no precedence/nesting support.
- **Fix:** Each grammar rule = a class with `interpret()`; build & walk an expression tree.
```python
Multiply(Add(Number(3), Number(4)), Number(2)).interpret()   # 14
```
- **Confuse with:** Composite (Interpreter IS a Composite, applied specifically to grammars).

### Iterator 🔁
- **Need:** Traverse a collection without exposing its internal structure.
- **Problem without it:** Client reaches directly into internals (`._books`) to loop.
- **Fix:** Separate Iterator object with `__iter__`/`__next__` (built into Python).
```python
for book in collection: print(book.title)
```
- **Confuse with:** Visitor (traversal vs. operation performed at each stop — often combined).

### Visitor 🧑‍⚕️
- **Need:** Add new operations to a class hierarchy without modifying the classes.
- **Problem without it:** New method added to EVERY class for every new operation.
- **Fix:** One permanent `accept(visitor)`; new operations = new Visitor classes.
```python
shape.accept(json_exporter)   # double dispatch
```
- **Key trade-off:** easy to add operations, HARD to add new element types (opposite of naive approach).
- **Confuse with:** Strategy (single algorithm, not tree-wide operations).

### Mediator 🎛️
- **Need:** Centralize many-to-many communication between objects.
- **Problem without it:** Tangled web — every object holds references to every other object.
- **Fix:** Objects only talk to a central Mediator; it owns all coordination logic.
```python
self.mediator.notify(self, "text_changed")
```
- **Risk:** can become a bloated "God object."
- **Confuse with:** Observer (one-to-many, one-directional), Facade (client→subsystem, not peer↔peer).

### Memento 💾
- **Need:** Save/restore an object's state (undo) without breaking encapsulation.
- **Problem without it:** External code manually copies every private field to save/restore.
- **Fix:** Originator creates its own opaque Memento; Caretaker stores it, never reads it.
```python
history.push(editor.save()); editor.restore(history.pop())
```
- **Confuse with:** Command (often paired — Command can hold a Memento for undo), Prototype (clone for reuse, not just for restoring).

---

## 🧠 Final Interview Reflexes

- **Asked "which pattern fits this scenario?"** → first ask yourself: is this about **creating**, **structuring**, or **behaving/communicating**? That narrows it to ~7 candidates immediately.
- **Two patterns look like the same code?** → the difference is almost always **intent**, not structure (Strategy vs. State, Decorator vs. Proxy, Composite vs. Interpreter).
- **"Isn't X an anti-pattern?"** → be ready for this on **Singleton** (global state) and **Mediator** (God object risk) — both are legitimate but commonly criticized.
- **SOLID connection:** most patterns exist to satisfy **Open/Closed** (add without editing) or **Single Responsibility** (separate concerns) — naming the principle a pattern embodies is a strong interview answer.