# VIVA & CONCEPT GUIDE: RUNTIME POLYMORPHISM AND EXCEPTIONS
**Student:** Nirmal S (Reg No: 192511249)  
**Perspective:** Python Master to C++ Beginner Bridge  
**Goal:** Crack the viva and confidently explain/modify any part of Assignment 5 on the spot.

---

## 1. PYTHON VS C++ "CHEAT SHEET" (YOUR MENTAL BRIDGE)

| Concept | Python Mental Model | C++ Reality in Assignment 5 | Why Faculty Cares |
| :--- | :--- | :--- | :--- |
| **Polymorphism Mechanism** | Python inspects `__dict__` dynamically (Duck Typing). | C++ uses compiler-generated `vtable` and hidden `__vptr` per object. | Task A2 & B1: How `b->total()` reaches `OPBill::total()`. |
| **Pure Virtual Functions** | `@abstractmethod` decorator. | `virtual double total() const = 0;` makes class abstract. | Cannot instantiate abstract classes directly. |
| **Exception Hierarchy** | Inherit from `Exception`. | Inherit from `std::exception` or `std::runtime_error`; override `what() noexcept`. | Task A4: Typed domain error handling. |
| **Stack Unwinding & Cleanup**| `try...finally` or context managers (`with`). | Destructors run automatically during stack unwinding (RAII). No explicit finally needed! | Core of Task A5: Why `~f` and `~main` ran before `caught 42`. |
| **Rethrowing Syntax** | Bare `raise`. | Bare `throw;` rethrows original exception preserving dynamic polymorphic type. | Task B2: Preventing slicing when logging and rethrowing. |
| **Exception Performance** | Python exceptions are relatively lightweight. | C++ uses zero-cost tables on happy paths, but unwinding thrown exceptions takes ~1.5 microseconds. | Task B3: Why 50% exception rate took 761 ms vs 0 ms for return codes. |
| **Destructors & Exceptions** | `__del__` can theoretically raise exceptions. | Destructors MUST NEVER throw! If an exception escapes a destructor during unwinding, `std::terminate()` aborts immediately. | Task A4: Why destructors are implicitly `noexcept`. |

---

## 2. TASK-BY-TASK VIVA CHEAT SHEET

### TASK 01: Polymorphic Bill Hierarchy (`Task01/task01.cpp`)
- **What it does:** Implements abstract `Bill` with pure virtual `total()` and `type()`, derived `OutPatient` and `InPatient`. Verifies prediction `500 ~OPBill ~Bill `.
- **Viva Answer:** *"A pure virtual function (`= 0`) defines an interface without implementation, making the class abstract. When called through a base pointer, the call resolves at runtime via the virtual table (`vtable`)."*
- **Trap Question:** *"Why did `show()`, a non-virtual function in Bill, reach `OPBill::total()`?"*  
  $\rightarrow$ *"Because inside `show()`, `total()` is called on the implicit `this` pointer, which dynamically dispatches through the derived object's vtable to `OPBill::total()`."*

---

### TASK 02: Error Engineering & Rethrowing (`Task02/task02.cpp`)
- **What it does:** Validates claim amounts, audits claims above cover, and rethrows via `throw;`. Traces stack unwinding `~f ~main caught 42`.
- **Viva Answer:** *"When an exception is thrown, stack unwinding automatically calls the destructors of all local objects in reverse order of creation. We use bare `throw;` to rethrow the original exception without slicing derived attributes."*

---

### TASK 03: Virtual Calls vs Switch vs Exceptions Benchmark (`Task03/task03.cpp`)
- **What it does:** Benchmarks virtual dispatch vs switch (1K, 1M, 10M bills) and exception cost at 0%, 1%, 50% error rates.
- **Viva Answer:** *"Virtual dispatch takes only 53 ms for 10,000,000 bills (just 3.4 nanoseconds per bill). Modern C++ zero-cost exceptions have negligible overhead on valid data (2 ms for 1M bills), but throwing exceptions at high rates incurs unwinding penalties (761 ms at 50% error rate). Hence exceptions are for rare error conditions, not everyday control flow."*

---

### TASK 04: Final Engineering Decision & Extension (`Task04/task04.cpp`)
- **What it does:** Adds `DayCareSurgery` and `CorporateTieUp` without modifying existing classes; proves AddressSanitizer safety under exceptions.
- **Viva Answer:** *"Sir, for 200,000 daily bills across 15 hospitals, polymorphic dispatch requires less than 1 millisecond of total daily CPU time. The architecture satisfies the Open-Closed Principle, allowing new hospital departments and insurance providers to be added with zero risk to working code."*
