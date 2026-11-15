# ASSIGNMENT 5 REPORT: RELIABLE SOFTWARE WITH RUNTIME POLYMORPHISM AND EXCEPTIONS

**Course:** Object Oriented Programming with C++ (DSA01 - CO5)  
**Assignment Title:** Build Robust C++ Applications by Applying Virtual Functions, Runtime Polymorphism and Exception Handling Mechanisms  
**Student Name:** Nirmal S  
**Register Number:** 192511249  
**Nature of Assignment:** Individual  
**Total Marks:** 50  
**GitHub Repository Link:** https://github.com/nirmal2308/DSA01-192511249-Assignment5  
**Bloom’s Level Assessed:** K5 – Evaluate  
**Compiler Used:** `g++ (Ubuntu 15.2.0-16ubuntu1) 15.2.0`  
**Standard & Compilation Flags:** `g++ -std=c++17 -Wall -Wextra -O2 -fsanitize=address`  

---

## EXECUTIVE SUMMARY

Assignment 5 evaluates the architecture and performance of reliable, fault-resilient healthcare enterprise software utilizing C++ runtime polymorphism and structured exception handling. The investigation evaluates:
1. Polymorphic bill abstraction across multiple patient care models.
2. The virtual table (`vtable`) dispatch mechanism and the necessity of virtual destructors.
3. Separation of billing liabilities across diverse payer schemes (Self-Pay, Private Insurance, Government Schemes, Corporate Tie-Ups).
4. Exception engineering, typed domain error hierarchies, and stack unwinding guarantees under RAII.
5. Large-scale empirical performance benchmarking comparing virtual dispatch, procedural `switch` branching, and exception penalties from 1,000 to 10,000,000 transactions.
6. Architectural extension and production deployment defense for a hospital network processing 200,000 transactions daily.

---

## PART A – DESIGN (25 MARKS)

### TASK A1 – DESIGN THE POLYMORPHIC BILL HIERARCHY (7 MARKS)

#### A1.1 Common Interface Identification
Every hospital bill shares essential invariant behaviors:
- `total()`: Returns final payable amount in rupees.
- `type()`: Returns string representation of care category.
- `print()`: Emits standardized receipt line.
- Protected attribute: `patient` (patient name / identification).

#### A1.2 UML Class Diagram
```
                       +-------------------------+
                       |          Bill           |  <<Abstract>>
                       +-------------------------+
                       | # patient: string       |
                       +-------------------------+
                       | + virtual ~Bill()       |
                       | + virtual total(): dbl=0|
                       | + virtual type(): str=0 |
                       | + print(): void         |
                       +-------------------------+
                                    ^
       +----------------------------+----------------------------+
       |                            |                            |
+---------------------+    +---------------------+    +---------------------+
|     OutPatient      |    |      InPatient      |    |   DayCareSurgery    |
+---------------------+    +---------------------+    +---------------------+
| - consultation: dbl |    | - days: int         |    | - packageFee: dbl   |
| - tests: dbl        |    | - perDay: dbl       |    | - anesthesiaFee: dbl|
|                     |    | - surgery: dbl      |    |                     |
+---------------------+    +---------------------+    +---------------------+
| + total(): double   |    | + total(): double   |    | + total(): double   |
| + type(): string    |    | + type(): string    |    | + type(): string    |
+---------------------+    +---------------------+    +---------------------+
```

#### A1.3 Pseudocode for Daily Total Calculation
```text
ALGORITHM CalculateDailyTotal(billsArray, count):
    INPUT: Array of Bill pointers 'billsArray', Integer 'count'
    OUTPUT: Double 'grandTotal'
    
    SET grandTotal = 0.0
    FOR index FROM 0 TO count - 1 DO
        CALL billsArray[index]->print()
        SET grandTotal = grandTotal + billsArray[index]->total()
    END FOR
    RETURN grandTotal
END ALGORITHM
```

---

### TASK A2 – UNDERSTAND RUNTIME BINDING (5 MARKS)

#### A2.1 Early vs Late Binding
- **Early Binding (Compile-Time):** Direct function calls and non-virtual member functions are resolved by the compiler at compile time based solely on the static type of the pointer or expression.
- **Late Binding (Run-Time):** Declaring a member function `virtual` instructs the compiler to resolve invocations dynamically at run-time through an indirect pointer table (`vtable`) based on the dynamic type of the object.

#### A2.2 The Virtual Table (`vtable`)
Each class containing virtual functions has a compiler-generated `vtable` containing function pointers:
```
OutPatient Instance [b]          Bill VTable                       OutPatient VTable
+-------------------+            +---------------------+           +------------------------+
| __vptr            | ---------> | ~Bill()             |           | ~OutPatient()          |
| patient: "Ravi"   |            | total() = pure (0)  |           | OutPatient::total()    |
| consultation: 500 |            | type()  = "Bill"    |           | OutPatient::type()="OP"|
| tests: 1200       |            +---------------------+           +------------------------+
+-------------------+
```
When `b->total()` executes, the program dereferences `b->__vptr` to index slot 1 of `OutPatient VTable`, dynamically reaching `OutPatient::total()`.

#### A2.3 Virtual Destructors
If a base class pointer `Bill* b = new OutPatient(...)` is destroyed with `delete b;`:
- If `~Bill()` is non-virtual, the compiler performs static binding and executes only `~Bill()`. Any resources allocated in `OutPatient` leak!
- Making `~Bill()` virtual ensures that `delete b` dynamically routes through the vtable to `~OutPatient()`, which cleans derived resources before delegating to `~Bill()`.

#### A2.4 Task A2 Output Prediction
```cpp
Bill* b = new OPBill;
b->show();
delete b;
```
- **Predicted Output:** `500 ~OPBill ~Bill `
- **Actual Output:** `500 ~OPBill ~Bill `
- **Explanation:** Non-virtual `show()` invokes pure virtual `total()`, which resolves through late binding to `OPBill::total()` (returning 500). `delete b` invokes virtual `~OPBill()`, followed by base `~Bill()`.

---

### TASK A3 – DESIGN WITH ABSTRACT CLASSES (5 MARKS)

#### A3.1 Payer Abstract Interface
Declared with pure virtual function `virtual double amountCovered(const Bill& b) const = 0;`:
1. `SelfPay`: Patient assumes 100% liability (`return b.total()`).
2. `Insurance`: Calculates coverage factoring co-payment percentage and caps at `coverLimit`.
3. `GovernmentScheme`: Enforces fixed statutory rate cap (`fixedRateCap`).

#### A3.2 Extensibility (Open-Closed Principle)
Introducing a new third-party corporate agreement (`CorporateTieUp`) requires zero modifications to existing classes. The new class implements `Payer`, overrides `amountCovered()`, and seamlessly interacts with all existing bills.

---

### TASK A4 – DESIGN THE EXCEPTION STRATEGY (4 MARKS)

#### A4.1 Exception Class Hierarchy
```
               +-----------------------+
               |    std::exception     |
               +-----------------------+
                           ^
               +-----------+-----------+
               |  std::runtime_error   |
               +-----------------------+
                           ^
             +-------------+-------------+
             |                           |
   +--------------------+      +--------------------+
   |    BillingError    |      |     ClaimError     |
   +--------------------+      +--------------------+
   | - NegativeCharge   |      | - ExceedsCover     |
   | - MissingRecord    |      | - InvalidPolicy    |
   +--------------------+      +--------------------+
```

#### A4.2 Function Exception Specifications & `noexcept`
- **Destructors (`noexcept`):** Destructors must **never throw** exceptions. If an exception is thrown while another exception is unwinding the stack, the C++ runtime immediately invokes `std::terminate()`, aborting the process.
- **Accessors (`noexcept`):** Getters (`total()`, `getPatient()`) are marked `noexcept` as they perform non-failing primitive queries.
- **Mutators / Parsers (May Throw):** Input processors and insurance evaluators throw `ClaimError` or `std::invalid_argument`.

---

### TASK A5 – DESIGN RECOVERY AND RETHROWING (4 MARKS)

#### A5.1 Where to Catch and Rethrow
- **Middle Layer (`settle()`):** Catches `ClaimError`, logs an audit entry (`"audit: exceeds cover"`), and rethrows the exact same exception via `throw;` to allow the top-level handler to reject the transaction.
- **Top Layer (`main()` UI loop):** Catches `std::exception&`, notifies the user, and continues processing subsequent bills without crashing the system.

#### A5.2 Task A5 Output Prediction (Stack Unwinding)
```cpp
void f() { Guard g("f"); throw 42; }
int main() { try { Guard m("main"); f(); ... } catch(int x) { ... } }
```
- **Predicted Output:** `~f ~main caught 42`
- **Actual Output:** `~f ~main caught 42`
- **Explanation:** When `throw 42` occurs, the runtime unwinds activation records. Local `Guard g` in `f` is destroyed first (`~f `). Then `Guard m` in `main` is destroyed (`~main `). Then the matching `catch(int x)` block executes.

---

## PART B – IMPLEMENTATION & EXPERIMENTAL INVESTIGATION (25 MARKS)

### TASK B1 – IMPLEMENT AND VALIDATE POLYMORPHIC BILLING (7 MARKS)

Executed in `Task01/task01.cpp`.
```text
OP Ravi: 1700
IP Meena: 47500
Total 49200
```
- `OutPatient` bill: $500 \text{ consultation} + 1200 \text{ tests} = \text{Rs } 1700$.
- `InPatient` bill: $3 \text{ days} \times 2500 + 40000 \text{ surgery} = \text{Rs } 47500$.
- Grand total: $\text{Rs } 49200$. Hand calculation verified.
- AddressSanitizer report: 0 leaks, clean termination.

---

### TASK B2 – ENGINEER ERROR CONDITIONS (5 MARKS)

Executed in `Task02/task02.cpp`.
```text
approved 40000
audit: exceeds cover
claim rejected: exceeds cover
error: negative amount
```
- Case 1 (40000 / 50000): Approved in full.
- Case 2 (60000 / 50000): Exceeds coverage limit. Caught in `settle()`, logged to audit, rethrown via `throw;`, caught at top level, and rejected.
- Case 3 (-1 / 50000): Negative amount rejected with `invalid_argument`.
- **`throw;` vs `throw e;`:**
  - `throw;` rethrows the original polymorphic exception object preserving derived type information.
  - `throw e;` creates a copy and slices derived exception attributes to the static type caught by the catch parameter.

---

### TASK B3 – VIRTUAL FUNCTIONS vs SWITCH vs EXCEPTIONS: THE COST (8 MARKS)

Empirical benchmarks conducted in `Task03/task03.cpp` compiled with `g++ -std=c++17 -O2`:

#### B3.1 & B3.2 Scaling Comparison (Virtual Functions vs Switch)
| Data Size ($N$ Bills) | Virtual Dispatch Time | Switch Dispatch Time |
| :--- | :--- | :--- |
| **$1,000$ Bills** | `0 ms` | `0 ms` |
| **$1,000,000$ Bills** | `4 ms` | `1 ms` |
| **$10,000,000$ Bills** | `53 ms` | `19 ms` |

*Interpretation:* The vtable pointer dereference incurs a modest ~34 ms penalty per 10 million transactions (3.4 nanoseconds per bill). For any realistic transaction load, this difference is completely unnoticeable.

#### B3.3 Cost of Exceptions Benchmark (1,000,000 Bills)
| Error Rate (%) | Return Codes Execution Time | Exceptions Execution Time |
| :--- | :--- | :--- |
| **0.0% (Happy Path)** | `0 ms` | `2 ms` |
| **1.0% (10,000 Errors)** | `0 ms` | `19 ms` |
| **50.0% (500,000 Errors)**| `0 ms` | `761 ms` |

*Interpretation:*
- On normal valid paths, modern zero-cost exceptions (table-driven DWARF unwinding) execute at native hardware speed with almost zero CPU overhead.
- When exceptions are thrown, stack frame allocation and table lookup take measurable time (~1.5 microseconds per throw). Hence, exceptions should be reserved for truly exceptional error conditions, not used as flow-control loops.

---

### TASK B4 – FINAL ENGINEERING DECISION (5 MARKS)

Executed in `Task04/task04.cpp`.
- Extended system with `DayCareSurgery` and `CorporateTieUp`.
- Processed mixed batch across all 4 payers.
- Threw and caught invalid bill exceptions under AddressSanitizer.

#### Production Decision Evaluation
- **Transaction Volume:** 200,000 bills/day across 15 hospitals.
- **Measured CPU Demand:** At 4 ms per 1,000,000 bills, processing 200,000 daily bills requires **under 1 millisecond of CPU time per day**.
- **Exception Overhead:** Assuming a realistic 0.5% rejection rate (1,000 errors/day), exception handling takes under 2 milliseconds of total CPU time per day.
- **Architectural Scalability:** Adding new bill types (e.g. Tele-Consultation) or insurance carriers requires zero modifications to existing hospital classes.
- **Verdict:** The polymorphic architecture with structured exceptions is **formally approved for production deployment**.

---

## CONCLUSION

Assignment 5 validates the power and resilience of modern C++ runtime polymorphism and exception safety. All objectives across Part A (Design) and Part B (Implementation) have been proven through code, theory, and empirical benchmarking.
