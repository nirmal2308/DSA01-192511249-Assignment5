# DSA01 - Assignment 5: Reliable Software with Runtime Polymorphism and Exceptions

**Course:** Object Oriented Programming with C++ / Data Structures and Algorithms (CO5)  
**Student Name:** Nirmal S  
**Register Number:** 192511249  
**Nature of Assignment:** Individual  
**Total Marks:** 50 (Bloom's Level: K5 – Evaluate)  

---

## 1. System and Compiler Information

- **Operating System:** Linux (Ubuntu 24.04 LTS x86_64 via WSL)
- **Compiler:** `g++` (Ubuntu 15.2.0-16ubuntu1) 15.2.0
- **C++ Standard:** C++17 (`-std=c++17`)
- **Sanitizers & Optimization:** `-std=c++17 -Wall -Wextra -O2 -fsanitize=address`

---

## 2. Directory Structure

```text
DSA01-192511249-Assignment5/
├── .gitignore
├── README.md
├── REPORT.md
├── VIVA_NOTES.md
├── Task01/
│   ├── task01.cpp
│   └── output.txt
├── Task02/
│   ├── task02.cpp
│   └── output.txt
├── Task03/
│   ├── task03.cpp
│   └── output.txt
└── Task04/
    ├── task04.cpp
    └── output.txt
```

---

## 3. Compilation and Execution Commands

Run the following commands from repository root:

### Task 01: Polymorphic Bill Hierarchy (Task A2 & B1)
```bash
g++ -std=c++17 -fsanitize=address -Wall -Wextra Task01/task01.cpp -o Task01/task01
./Task01/task01
```

### Task 02: Engineer Error Conditions & Stack Unwinding (Task A5 & B2)
```bash
g++ -std=c++17 -fsanitize=address -Wall -Wextra Task02/task02.cpp -o Task02/task02
./Task02/task02
```

### Task 03: Virtual Functions vs switch vs Exceptions Benchmark (Task B3)
```bash
g++ -std=c++17 -O2 -Wall -Wextra Task03/task03.cpp -o Task03/task03
./Task03/task03
```

### Task 04: Final Engineering Decision & Extension (Task A3 & B4)
```bash
g++ -std=c++17 -fsanitize=address -Wall -Wextra Task04/task04.cpp -o Task04/task04
./Task04/task04
```

---

## 4. Summary of Tasks and Key Findings

| Task | Topic | Key Findings & Evaluation |
| :--- | :--- | :--- |
| **Task 01** | Polymorphic Billing Hierarchy | Verified runtime binding via `vtable` and virtual destructor lifecycle (`500 ~OPBill ~Bill `); completed `OutPatient` and `InPatient` billing totals. |
| **Task 02** | Error Engineering & Rethrowing | Traced stack unwinding under RAII (`~f ~main caught 42`); implemented partial handling and exact exception rethrow (`throw;`) in insurance claim audits. |
| **Task 03** | Performance Cost Benchmarking | Compared virtual dispatch vs `switch` across 1,000 to 10,000,000 bills (virtual calls process 10M items in 53 ms). Measured exception overhead across 0%, 1%, and 50% failure rates. |
| **Task 04** | Practical Extension & Production Decision | Extended system with `DayCareSurgery` and `CorporateTieUp` without modifying existing classes; verified leak-free exception handling under AddressSanitizer; approved polymorphic design for hospital group. |
