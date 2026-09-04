# Observer Pattern 👀

> **One-liner (say this in an interview):**
> "Observer lets one object (Subject) notify multiple dependent
> objects (Observers) automatically whenever its state changes —
> without the Subject knowing who or what those objects are."

**Category:** Behavioral
**Real-world analogy:** YouTube channel subscriptions. You (Observer)
subscribe to a channel (Subject). When it uploads a video, **every**
subscriber gets notified automatically — the channel doesn't need to
know each subscriber personally, it just broadcasts.

---

## 🔴 The Problem (why this pattern exists)

You have one object whose state changes, and **multiple other parts
of the system need to react** to that change (update UI, log it, send
an alert...). Naive fix: the object directly calls each dependent
one by one, hardcoded.

```python
class WeatherStation:
    def __init__(self):
        self.temperature = 0
        self.phone_display = PhoneDisplay()
        self.tv_display = TVDisplay()

    def set_temperature(self, temp):
        self.temperature = temp
        # 😬 manually calling every dependent — hardcoded & rigid
        self.phone_display.update(temp)
        self.tv_display.update(temp)
```

**Pain points:**
- ❌ `WeatherStation` is **tightly coupled** to every display it updates
- ❌ Adding a new listener (e.g. a website widget) → must **edit**
  `set_temperature()` again
- ❌ Violates **Open/Closed Principle**
- ❌ Can't add/remove listeners **at runtime**
- ❌ Hard to reuse `WeatherStation` in a context with different listeners

---

## 🟢 The Fix — Observer Pattern

**Idea:** Subject keeps a **list** of observers (not fixed references).
Observers implement a common interface (`update()`). Subject just
loops through the list and calls `update()` — it has no idea what
each observer actually does with that data.

```python
from abc import ABC, abstractmethod

# 1️⃣ Observer interface
class Observer(ABC):
    @abstractmethod
    def update(self, temperature: float) -> None: ...

# 2️⃣ Subject — holds a list of observers, notifies them
class WeatherStation:
    def __init__(self):
        self._observers: list[Observer] = []
        self._temperature = 0

    def subscribe(self, observer: Observer):
        self._observers.append(observer)          # register 🔔

    def unsubscribe(self, observer: Observer):
        self._observers.remove(observer)           # deregister 🔕

    def set_temperature(self, temp):
        self._temperature = temp
        self._notify_all()

    def _notify_all(self):
        for observer in self._observers:
            observer.update(self._temperature)

# 3️⃣ Concrete observers — each reacts however it wants
class PhoneDisplay(Observer):
    def update(self, temperature):
        print(f"📱 Phone: {temperature}°C")

class TVDisplay(Observer):
    def update(self, temperature):
        print(f"📺 TV: {temperature}°C")


# --- Usage ---
station = WeatherStation()
phone = PhoneDisplay()
tv = TVDisplay()

station.subscribe(phone)
station.subscribe(tv)

station.set_temperature(30)
# 📱 Phone: 30°C
# 📺 TV: 30°C

# ✅ New listener? Just implement Observer & subscribe. Zero edits above.
class WebsiteWidget(Observer):
    def update(self, temperature):
        print(f"🌐 Website: {temperature}°C")

station.subscribe(WebsiteWidget())
station.set_temperature(35)   # now notifies all 3
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Add new listener | Edit Subject's method | `subscribe()` — no edits |
| Coupling | Subject knows every listener's class | Subject only knows `Observer` interface |
| Runtime flexibility | Fixed listeners | Add/remove anytime |
| SOLID | Breaks Open/Closed | Follows Open/Closed |
| Code smell | Hardcoded chain of calls | Clean loop over a list |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** One-to-many dependency — when one
  object changes state, all dependents are notified automatically,
  without tight coupling.
- **Key players:** `Subject` (a.k.a. Observable/Publisher) holds a
  list of `Observer`s (a.k.a. Subscribers) and calls `update()` on
  each when state changes.
- **Design principle it embodies:** Loose coupling — Subject depends
  only on the `Observer` interface, not concrete classes.
- **Push vs Pull model:**
  - *Push* — Subject sends the data directly (`update(temperature)`)
  - *Pull* — Subject just says "something changed", Observer calls
    back `subject.get_state()` to fetch what it needs
- **Similar-sounding pattern to not confuse it with:**
  - **Pub-Sub** — conceptually similar but usually has a **broker/event
    bus** in between (Subject and Observer never know each other
    directly); Observer is normally *direct* subscription.
  - **Mediator** — centralizes *many-to-many* communication between
    objects; Observer is strictly *one-to-many*.
- **Real examples:** Event listeners in JS (`addEventListener`), Python's
  `Subject`/`Observable` in RxPy, model-view updates in MVC, Django
  signals, Kafka/message queues (conceptually).

---

## ✅ Use it when
- Multiple objects need to react to one object's state change
- You don't want the "changing" object tightly bound to its listeners
- Listeners should be addable/removable at runtime

## ❌ Skip it when
- Only one listener will ever exist — a direct method call is simpler
- Notifications need guaranteed ordering/transactions — Observer gives
  no such guarantee out of the box, and debugging notification chains
  can get messy ("who called what, when")

---

**TL;DR:** Instead of hardcoding calls to every dependent object,
Subject maintains a list of Observers and notifies them all through
one common interface → add/remove listeners freely, zero coupling.