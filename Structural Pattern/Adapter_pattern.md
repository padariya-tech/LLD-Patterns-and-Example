# Adapter Pattern 🔌

> **One-liner (say this in an interview):**
> "Adapter converts the interface of one class into another
> interface that the client expects — letting two otherwise
> incompatible things work together, without modifying either one."

**Category:** Structural
**Real-world analogy:** A travel power plug adapter. Your laptop
charger has a US plug, the wall socket in the UK is a different
shape. You don't rewire your charger or the wall — you plug in an
adapter that sits between them, translating one shape into the other.

---

## 🔴 The Problem (why this pattern exists)

You have existing code (client) that expects one interface, and a
class you want to use (often third-party or legacy) that exposes a
**different, incompatible interface**. Naive fix: rewrite one of
them to match the other — which you often **can't** do (it's a
third-party library) or **shouldn't** (it breaks other callers).

```python
# Your app expects every payment processor to have .pay(amount)
class StripePayment:
    def pay(self, amount):
        print(f"Paying ${amount} via Stripe")

# 😬 Third-party library — you don't own this code, can't edit it,
# and it has a totally different method name/signature
class LegacyPayPalSDK:
    def send_payment(self, amount_in_cents, currency="USD"):
        print(f"Sending {amount_in_cents} cents via PayPal ({currency})")

def checkout(payment_processor, amount):
    payment_processor.pay(amount)     # expects .pay(amount) — PayPal breaks!

checkout(StripePayment(), 50)         # ✅ works
checkout(LegacyPayPalSDK(), 50)       # 💥 AttributeError: no .pay() method
```

**Pain points:**
- ❌ **Incompatible interfaces** — can't swap in the third-party
  class without errors
- ❌ Can't modify the third-party/legacy class (no source access, or
  it'd break other code depending on its original interface)
- ❌ Rewriting client code to special-case every different interface
  defeats the whole point of having a common abstraction
- ❌ Tight coupling to *specific* implementations creeps back in

---

## 🟢 The Fix — Adapter Pattern

**Idea:** Wrap the incompatible class in an **Adapter** that
implements the interface your client code expects, and internally
translates calls into whatever the wrapped object actually needs.

```python
from abc import ABC, abstractmethod

# 1️⃣ The interface your client code expects
class PaymentProcessor(ABC):
    @abstractmethod
    def pay(self, amount: float) -> None: ...

# 2️⃣ Already-compatible class
class StripePayment(PaymentProcessor):
    def pay(self, amount):
        print(f"Paying ${amount} via Stripe")

# 3️⃣ Incompatible third-party class (can't touch its code)
class LegacyPayPalSDK:
    def send_payment(self, amount_in_cents, currency="USD"):
        print(f"Sending {amount_in_cents} cents via PayPal ({currency})")

# 4️⃣ Adapter — translates PaymentProcessor.pay() → send_payment()
class PayPalAdapter(PaymentProcessor):
    def __init__(self, paypal_sdk: LegacyPayPalSDK):
        self._paypal_sdk = paypal_sdk

    def pay(self, amount):
        cents = int(amount * 100)          # 🔄 do the translation here
        self._paypal_sdk.send_payment(cents, currency="USD")


# --- Usage ---
def checkout(payment_processor: PaymentProcessor, amount):
    payment_processor.pay(amount)

checkout(StripePayment(), 50)                          # works natively
checkout(PayPalAdapter(LegacyPayPalSDK()), 50)         # ✅ now works too!
# Sending 5000 cents via PayPal (USD)

# ✅ checkout() never changed. LegacyPayPalSDK never changed.
# The Adapter is the only new code needed.
```

---

## ⚡ Before vs After (quick recall table)

| | Before | After |
|---|---|---|
| Incompatible class | Breaks client code directly | Wrapped by an Adapter matching client's expected interface |
| Third-party/legacy code | Would need modification (often impossible) | Left completely untouched |
| Client code | Needs special-casing per implementation | Stays generic — talks to one interface only |
| Adding another incompatible library | Repeat special-casing | Write one more small Adapter class |

---

## 🧠 Interview Cheat Sheet

- **What problem does it solve?** Bridges an interface mismatch
  between existing client code and a class it needs to use, without
  modifying either side.
- **Two flavors:**
  - **Object Adapter** (shown above) — wraps the adaptee via
    **composition** (holds a reference to it); more flexible, works
    with subclasses too. Most common in Python.
  - **Class Adapter** — inherits from both the target interface and
    the adaptee (multiple inheritance); less common in Python since
    composition is generally preferred.
- **Design principle it embodies:** "Favor composition over
  inheritance" — the Object Adapter variant wraps rather than
  extends.
- **Similar-sounding pattern to not confuse it with:**
  - **Proxy** — keeps the **same** interface as the real object, just
    controls access to it; Adapter deliberately presents a
    **different** interface than what it wraps.
  - **Decorator** — adds **new behavior** while keeping the same
    interface; Adapter's whole job is **translating** a mismatched
    interface, not adding features.
  - **Facade** — simplifies access to a **whole subsystem** of many
    classes; Adapter typically wraps **one** class to fix an
    interface mismatch.
- **Real examples:** Python's `sqlite3` module implementing the
  DB-API 2.0 interface so different DB drivers are interchangeable,
  ORMs adapting different database engines to one query interface,
  wrapping an old XML-based API to look like a modern JSON API,
  `zipfile`/`tarfile` sharing a common archive-like usage pattern.

---

## ✅ Use it when
- You need to use an existing/third-party class but its interface
  doesn't match what your code expects
- You can't (or shouldn't) modify the original class's source
- You want your client code to stay decoupled from the specifics of
  whichever underlying implementation is plugged in

## ❌ Skip it when
- You control both pieces of code — just make the interfaces match
  directly, no adapter needed
- Only used once and the mismatch is trivial (e.g. renaming a single
  method) — sometimes a small wrapper function is simpler than a
  whole Adapter class

---

**TL;DR:** Instead of rewriting incompatible code or special-casing
client logic per implementation, wrap the mismatched class in an
Adapter that speaks the interface your code expects — translating
calls underneath, with zero changes to either original class.