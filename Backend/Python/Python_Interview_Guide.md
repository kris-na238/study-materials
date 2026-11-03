# Python — Senior Software Engineer Interview Guide

Each section explains **what the concept is, why it matters, key concepts with code, and gotchas**, followed by **frequently asked interview questions with senior-level answers** (trade-offs, internals and failure modes, not just definitions).

> Python versions matter. Notes reflect Python up to 3.14 (2025–2026). Version-specific features are tagged, e.g. *(3.12+)*. Exact behaviour can vary slightly between versions; check the docs before quoting it in an interview.

**How to revise with this document**
- First pass: read **What / Why / Key concepts**, run the code in a REPL.
- Revision passes: read only the **Interview Questions**. Try answering out loud before reading the answer.
- Night before: [Appendix A (Predict the Output)](#appendix-a-predict-the-output) + [Appendix B (Cheat Sheet)](#appendix-b-revision-cheat-sheet).
- ⭐ = very frequently asked.

---

## Table of Contents

**[The Big Picture](#the-big-picture)**: how all Python topics connect

**Part 1 — Language Core**
[1. What Python Is & How It Runs](#1-what-python-is--how-it-runs) · [2. Syntax Essentials & Style](#2-syntax-essentials--style) · [3. Objects, Names & Variables](#3-objects-names--variables) · [4. Scope & Namespaces](#4-scope--namespaces) · [5. Built-in Types & Mutability](#5-built-in-types--mutability) · [6. Operators](#6-operators) · [7. Control Flow](#7-control-flow) · [8. Strings](#8-strings)

**Part 2 — Data Structures**
[9. Lists, Tuples, Dicts & Sets](#9-lists-tuples-dicts--sets) · [10. Hashing & Hashability](#10-hashing--hashability) · [11. Comprehensions & Generator Expressions](#11-comprehensions--generator-expressions) · [12. Unpacking, Slicing & Sorting](#12-unpacking-slicing--sorting) · [13. Collections Module, heapq & bisect](#13-collections-module-heapq--bisect) · [14. Copying: Assignment vs Shallow vs Deep](#14-copying-assignment-vs-shallow-vs-deep)

**Part 3 — Functions**
[15. Function Fundamentals & Argument Passing](#15-function-fundamentals--argument-passing) · [16. First-Class Functions, Lambdas & Higher-Order Functions](#16-first-class-functions-lambdas--higher-order-functions) · [17. Closures & Late Binding](#17-closures--late-binding) · [18. Decorators](#18-decorators) · [19. Functools & Itertools](#19-functools--itertools) · [20. Recursion](#20-recursion)

**Part 4 — OOP & the Data Model**
[21. Classes & Objects](#21-classes--objects) · [22. Inheritance, MRO & Super](#22-inheritance-mro--super) · [23. Polymorphism, Duck Typing, ABCs & Protocols](#23-polymorphism-duck-typing-abcs--protocols) · [24. The Data Model & Dunder Methods](#24-the-data-model--dunder-methods) · [25. Properties & Descriptors](#25-properties--descriptors) · [26. Dataclasses, NamedTuple, Slots & Enum](#26-dataclasses-namedtuple-slots--enum) · [27. Metaclasses & Class Creation Hooks](#27-metaclasses--class-creation-hooks)

**Part 5 — Iteration, Errors, I/O, Modules & Typing**
[28. Iterators & Generators](#28-iterators--generators) · [29. Context Managers](#29-context-managers) · [30. Exceptions & Error Handling](#30-exceptions--error-handling) · [31. File IO & Serialization](#31-file-io--serialization) · [32. Modules, Packages & Imports](#32-modules-packages--imports) · [33. Type Hints & Static Typing](#33-type-hints--static-typing)

**Part 6 — Internals, Concurrency & Performance**
[34. CPython Execution Model](#34-cpython-execution-model) · [35. Memory Management & Garbage Collection](#35-memory-management--garbage-collection) · [36. The GIL & Free-Threading](#36-the-gil--free-threading) · [37. Threading & Synchronization](#37-threading--synchronization) · [38. Multiprocessing](#38-multiprocessing) · [39. Asyncio](#39-asyncio) · [40. Choosing a Concurrency Model](#40-choosing-a-concurrency-model) · [41. Performance & Profiling](#41-performance--profiling)

**Part 7 — Tooling & Ecosystem**
[42. Packaging, Environments & uv](#42-packaging-environments--uv) · [43. Code Quality: Linting, Formatting & Type Checking](#43-code-quality-linting-formatting--type-checking) · [44. Testing with Pytest](#44-testing-with-pytest) · [45. Logging & Configuration](#45-logging--configuration) · [46. Web & Backend Python](#46-web--backend-python) · [47. Data, ML & GenAI Ecosystem](#47-data-ml--genai-ecosystem)

**Part 8 — Architecture & Production**
[48. Design Patterns in Python](#48-design-patterns-in-python) · [49. Clean Architecture, SOLID & Dependency Injection](#49-clean-architecture-solid--dependency-injection) · [50. Security Pitfalls](#50-security-pitfalls) · [51. Running Python Services in Production](#51-running-python-services-in-production)

**Appendices**
[A. Predict the Output](#appendix-a-predict-the-output) · [B. Revision Cheat Sheet](#appendix-b-revision-cheat-sheet)

---

## The Big Picture

Python topics stack in layers; each layer builds on the ones below it.

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│ ARCHITECTURE (Part 8)   patterns · SOLID · DI · security · production services    │
├──────────────────────────────────────────────────────────────────────────────────┤
│ ECOSYSTEM (Part 7)      Web/Backend │ Data │ ML │ GenAI/Agents │ Automation        │
│ TOOLING   (Part 7)      uv · Ruff · mypy/Pyright/ty · pytest · pyproject.toml     │
├──────────────────────────────────────────────────────────────────────────────────┤
│ RUNTIME (Part 6)        CPython · bytecode · ref counting · GC · GIL              │
│                         threading · multiprocessing · asyncio · profiling         │
├──────────────────────────────────────────────────────────────────────────────────┤
│ STRUCTURE (Part 5)      iterators/generators · context managers · exceptions      │
│                         I/O · modules/imports · type hints                        │
├──────────────────────────────────────────────────────────────────────────────────┤
│ ABSTRACTION (Parts 3–4) functions → closures → decorators │ classes → MRO →       │
│                         dunder methods → descriptors → metaclasses                │
├──────────────────────────────────────────────────────────────────────────────────┤
│ DATA (Parts 1–2)        objects & names · types · mutability · hashing ·          │
│                         list/dict/set internals · collections                     │
└──────────────────────────────────────────────────────────────────────────────────┘
```

**The single idea that ties everything together:** *everything in Python is an object, and behaviour comes from protocols (dunder methods).* `for` calls `__iter__`, `with` calls `__enter__/__exit__`, `len()` calls `__len__`, `a + b` calls `__add__`, `await` calls `__await__`.

**Threads that cross layers (interviewers love these):**

1. **Objects → dunder methods → everything.** Everything is an object (§3) → behaviour via dunder methods (§24) → iteration (§28), context managers (§29), operator overloading in NumPy/pandas (§47).
2. **Functions → closures → decorators → frameworks.** First-class functions (§16) → closures capture enclosing scope via LEGB (§4, §17) → decorators (§18) → `@app.get`, `@pytest.fixture`, `@lru_cache` (§44, §46).
3. **Iterators → generators → coroutines → asyncio.** The iterator protocol (§28) → generators pause with `yield` → `send()` makes them coroutines → `async/await` (§39).
4. **Mutability → hashing → data structure choice → performance.** Only immutables are hashable (§10) → dict/set are O(1) hash tables (§9) → the right structure is the first performance win (§41).
5. **Ref counting → GIL → concurrency choice.** Ref counting isn't thread-safe (§35) → GIL (§36) → threads for I/O, processes for CPU (§40) → free-threading changes this (3.14+).
6. **C API → native libraries → "slow Python still wins".** NumPy, PyTorch, Pydantic-core and Polars run native code (§34, §47); Python is the steering wheel.
7. **Type hints → Pydantic → FastAPI → LLM structured output.** Hints are metadata (§33) → Pydantic validates at runtime → FastAPI request models (§46) → agent tool schemas (§47).

---

# PART 1 — LANGUAGE CORE

---

## 1. What Python Is & How It Runs

**What it is:** A high-level, general-purpose, **dynamically and strongly typed**, garbage-collected, multi-paradigm (procedural, OOP, functional) language. Created by **Guido van Rossum**, started December 1989 at CWI (Netherlands), first released **1991**. Named after *Monty Python's Flying Circus*, not the snake. Milestones: Python 2.0 (2000), Python 3.0 (2008, a deliberately backward-incompatible cleanup), Python 2 end-of-life in 2020.

**Why it exists:** Guido had worked on **ABC**, which was easy to learn but not extensible. Python aimed to combine ABC's readability with the practicality of C for systems work, sitting between shell scripts (easy but fragile) and C (powerful but verbose, with manual memory management).

**How it solved that (design choices):**
- **Readability through indentation**: blocks are defined by indentation, not braces, so code looks consistent everywhere.
- **Dynamic typing + automatic memory management**: no type declarations, no `malloc/free`.
- **"Batteries included" standard library**: files, JSON, HTTP, regex, SQLite, threading, testing and more ship with the language.
- **Interpreted with a REPL**: no separate compile step, so you iterate quickly.
- **Multi-paradigm**: choose procedural, OOP or functional style per problem.
- **A simple C API**: hot paths can be written in C/C++/Rust and called from Python.

**Key concepts**

*Compiled or interpreted? Both.* CPython compiles source to **bytecode** (cached as `.pyc` in `__pycache__/`), then the **Python Virtual Machine (PVM)**, a loop in C, interprets that bytecode.

```
app.py ──(compile: tokenize → parse → AST → bytecode)──► .pyc ──► PVM (eval loop in C) ──► result
```

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
#  LOAD_FAST   a
#  LOAD_FAST   b
#  BINARY_OP   0 (+)       # exact opcode names vary by version
#  RETURN_VALUE
```

*Implementations:*

| Implementation | Notes |
|---|---|
| **CPython** | The reference implementation, written in C; what "Python" usually means |
| **PyPy** | Has a JIT compiler; often much faster for long-running pure-Python code |
| **MicroPython** | Runs on microcontrollers |
| Jython / IronPython | JVM / .NET versions; mostly legacy |

*"Batteries included":* the standard library covers files (`pathlib`, `shutil`), data formats (`json`, `csv`, `sqlite3`), networking (`urllib`, `http`), concurrency (`threading`, `asyncio`, `concurrent.futures`), collections (`collections`, `heapq`), testing (`unittest`), CLIs (`argparse`) and security (`hashlib`, `secrets`). The trade-off: some modules are dated (`urllib` vs `requests`/`httpx`), and PEP 594 removed "dead batteries" in 3.13.

*Recent Python versions to know:*

| Version | Headline features |
|---|---|
| 3.8 | Walrus `:=`, positional-only `/`, `f"{x=}"` |
| 3.9 | `dict` merge `\|`, built-in generics `list[int]`, `removeprefix` |
| 3.10 | `match/case`, `X \| Y` union types, dataclass `slots=True` |
| 3.11 | 10–60% faster (adaptive specializing interpreter), `ExceptionGroup`/`except*`, `TaskGroup`, `tomllib` |
| 3.12 | Type parameter syntax `def f[T]`, `type` alias statement, more flexible f-strings |
| 3.13 | Experimental free-threaded build (no GIL) and experimental JIT, new REPL |
| 3.14 | Free-threading officially supported, template strings `t"..."`, lazy (deferred) annotations, `concurrent.interpreters` |

**Use cases & when to choose Python:** web backends and APIs (Django, FastAPI), data engineering and science, ML/AI, automation and DevOps tooling, testing, scripting and glue code. Choose it when **developer productivity and time-to-market** matter more than raw CPU speed, or when you need the data/AI ecosystem. Avoid it for hard real-time systems, mobile apps, or CPU-bound hot paths that can't be pushed into native code (use Go, Rust, C++ or Java there).

**Advantages vs limitations**

| Advantages | Limitations |
|---|---|
| Readable, fast to write and onboard | Slower than compiled languages for pure-Python CPU work |
| Huge standard library + PyPI ecosystem | GIL limits CPU parallelism with threads (changing with free-threading) |
| Dominant in data and AI | Higher memory per object |
| Easy C/C++/Rust interop | Type errors found at runtime unless you use type checkers |
| Cross-platform | Weak for mobile/frontend |

### Interview Questions

**Q1. Is Python compiled or interpreted?** ⭐
Both. CPython first **compiles** source to platform-independent bytecode (cached in `__pycache__/*.pyc` for imported modules), then the **PVM interprets** that bytecode in a C eval loop. It's not compiled to machine code ahead of time like C. PyPy adds a JIT that compiles hot bytecode to machine code at runtime, and CPython 3.13+ ships an experimental JIT. So the precise answer is "compiled to bytecode, then interpreted", with JIT options.

**Q2. Python was originally built for data science and AI, right?**
No. It started as a general-purpose scripting and glue language; its first use was system utilities for the Amoeba OS. Early AI used **Lisp/Prolog**, statistics used **R/MATLAB/SAS/SPSS**, NLP used **Perl/Java**, and big data used **Java/Scala** (Hadoop, Spark). Python won data and AI later through its ecosystem: NumPy (2006), scikit-learn (2007), pandas (2008), Jupyter (2014), TensorFlow (2015), PyTorch (2016) and Hugging Face.

**Q3. Why is Python preferred for data science and AI over Java or JavaScript?** ⭐
1. **Ecosystem**: NumPy, pandas, scikit-learn, PyTorch, Hugging Face; new research ships in Python first.
2. **Experimentation**: REPL and Jupyter suit the explore → plot → tweak loop; Java is too verbose for it.
3. **Audience**: scientists and researchers can read and write it.
4. **Slowness doesn't matter**: heavy maths runs in C/Fortran/CUDA; Python only orchestrates.
5. **Easy native interop** through the C API, Cython and pybind11.

Java is used around Python for big-data infrastructure (Spark, Kafka) and enterprise serving; JavaScript for in-browser inference (TensorFlow.js, Transformers.js).

**Q4. What does "batteries included" mean, and what's the trade-off?**
The standard library covers most common tasks without `pip install`: files, JSON, HTTP, SQLite, threading, testing and so on. Benefits: fewer dependencies, smaller supply-chain risk, consistent documentation. Costs: some modules are dated (`urllib` vs `requests`/`httpx`, `datetime` quirks), and once a module is in the stdlib it can only evolve at the language's release pace.

**Q5. What are Python's main limitations, and how do teams work around them?**
- **CPU speed**: vectorize with NumPy, use native extensions (Cython, PyO3/Rust, numba), or move hot loops to another service.
- **GIL**: multiprocessing for CPU work, asyncio/threads for I/O, free-threaded builds (3.14+).
- **Runtime type errors**: type hints with mypy/Pyright in CI, Pydantic at boundaries.
- **Memory**: `__slots__`, generators, Polars/Arrow instead of object-heavy structures.

---

## 2. Syntax Essentials & Style

**What it is:** The grammar rules (indentation, keywords, identifiers, comments) plus the community style guide **PEP 8** and the philosophy **PEP 20, the Zen of Python** (`import this`).

**Why it matters:** Interviewers judge "Pythonic" code. Knowing the conventions (`snake_case`, `_internal`, `__mangled`, dunders) signals experience.

**Key concepts**

- **Indentation is syntax.** Blocks start after `:` and are defined by indentation (4 spaces by convention). Wrong indentation raises `IndentationError`.
- **Keywords (35 hard keywords)**: `False None True and as assert async await break class continue def del elif else except finally for from global if import in is lambda nonlocal not or pass raise return try while with yield`.
- **Soft keywords** (`match`, `case`, `type`, `_`) are only special in context, so `match = 5` is still valid.
- **Identifiers**: start with a letter or `_`, then letters, digits or `_`; case-sensitive; Unicode allowed.
- **Syntax vs semantics**: a syntax error is caught at parse time, before any line runs; a semantic error is valid code that fails at runtime or gives wrong results.

```python
import keyword
keyword.iskeyword("class")    # True
keyword.softkwlist            # ['_', 'case', 'match', 'type']

# Syntax error — caught at parse time, nothing runs:
# if x > 5          # SyntaxError: expected ':'

# Semantically wrong — fails only at runtime:
def average(nums):
    return sum(nums) / len(nums)
average([])                   # ZeroDivisionError
```

**Naming conventions (PEP 8)**

| Kind | Convention | Example |
|---|---|---|
| Variable, function, method, module | `snake_case` | `total_price`, `calculate_tax()` |
| Class, exception | `PascalCase` | `UserAccount`, `PaymentError` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RETRIES = 5` |
| Internal (by convention) | leading `_` | `_helper()`, `self._cache` |
| Name-mangled class attribute | leading `__` | `self.__pin` → `_UserAccount__pin` |
| Avoid clashing with a keyword | trailing `_` | `class_`, `type_` |
| Special / magic methods | `__dunder__` | `__init__`, `__len__` |

**Comments & docstrings**

```python
# Explain WHY, not WHAT: retry because the payment API times out under load
retries = 3

def area(r: float) -> float:
    """Return the area of a circle with radius r."""   # docstring -> area.__doc__, help(area)
    return 3.14159 * r ** 2
```

Python has no multi-line comment syntax. A bare triple-quoted string is just an unused string expression; it's a *docstring* only when it's the first statement of a module, class or function.

**The Zen of Python, most-quoted lines:** "Explicit is better than implicit." "Simple is better than complex." "Readability counts." "Errors should never pass silently." "There should be one — and preferably only one — obvious way to do it."

### Interview Questions

**Q1. What is PEP 8, and how do you enforce it in a team?**
PEP 8 is the official style guide: naming, 4-space indentation, line length, import ordering, whitespace. Teams don't enforce it by hand; they run **Ruff** (lint + format) or Black, plus isort-style import sorting, in **pre-commit hooks and CI**. That way review time goes to design, not formatting.

**Q2. What's the difference between `_name`, `__name` and `__name__`?** ⭐
- `_name`: convention for "internal, don't touch". Not enforced, but `from module import *` skips it.
- `__name` (inside a class): **name mangling**. Python rewrites it to `_ClassName__name` to avoid accidental clashes in subclasses. It is not security; you can still access `obj._ClassName__name`.
- `__name__`: dunder names reserved for the language (`__init__`, `__len__`, the module attribute `__name__`). Never invent your own.

**Q3. What is a docstring, and how is it different from a comment?**
A docstring is a string literal that's the first statement in a module, class or function. It's stored on the object (`__doc__`) and used by `help()`, IDEs and doc generators. Comments (`#`) are discarded by the parser. Docstrings describe the *contract* (what and how to use it); comments explain *why* the implementation is the way it is.

---

## 3. Objects, Names & Variables

**What it is:** In Python, **everything is an object** (ints, functions, classes, modules). Every object has an **identity** (`id()`), a **type** (`type()`) and a **value**. Variables are **names bound to objects**, like labels on boxes, not the boxes themselves.

**Why it matters:** This one model explains pass-by-object-reference, mutable default arguments, shallow-copy bugs, `is` vs `==`, and why `a = b = []` shares one list. It's the root of most Python "gotcha" questions.

**Key concepts**

- **Assignment never copies**; it binds a name to an existing object.
- **No declaration step**: a variable exists from its first assignment. Type hints (`x: int`) alone create nothing.
- **Dynamic typing**: a name can be rebound to an object of any type. **Strong typing**: objects don't get silently coerced (`"5" + 3` raises `TypeError`).
- **`is` vs `==`**: `is` compares identity (same object); `==` compares value (calls `__eq__`).
- **Caching / interning**: CPython caches small ints **-5 to 256** and interns many identifier-like strings, so `is` can *appear* to work for values. That's an implementation detail; never rely on it.
- **Namespaces are dicts**: `globals()` and `locals()`; name lookup is a dictionary lookup.

```python
a = [1, 2, 3]
b = a                  # second name for the SAME list
b.append(4)
a                      # [1, 2, 3, 4]
a is b                 # True — same identity
id(a) == id(b)         # True

x = y = []             # gotcha: ONE list, two names
x.append(1); y         # [1]

s = "hi"; t = s
s += "!"               # str is immutable -> s is rebound to a NEW object
t                      # 'hi'

256 is 256             # True (small-int cache) — never write code like this
x: int                 # annotation only; no variable created
# print(x)             # NameError

"5" + 3                # TypeError — strong typing
1 + True               # 2 — not coercion: bool is a subclass of int
```

*Constants:* Python has no enforced constants. Use the `UPPER_CASE` convention, `typing.Final` (checked by mypy, not at runtime), `Enum` members, or frozen dataclasses for real immutability.

```python
from typing import Final
TIMEOUT: Final = 30
TIMEOUT = 60           # runs! mypy reports "Cannot assign to final name"
```

### Interview Questions

**Q1. What does "everything is an object" mean in practice?** ⭐
Ints, strings, functions, classes, modules and even `None` are heap objects with an identity, a type and a reference count. Consequences: functions can be passed around and decorated; classes can be created at runtime (they're instances of `type`); and variables are just references, so assignment and argument passing never copy data.

**Q2. What's the difference between `is` and `==`?** ⭐
`==` checks **value equality** via `__eq__`; `is` checks **identity**, meaning the same object in memory. Use `is` only for singletons: `x is None`, `x is True`, sentinel objects. `a is b` on ints or strings may look like it works because of small-int caching and string interning, but `1000 is 1000`-style results vary between REPL, script and Python version. That's a bug waiting to happen.

**Q3. Is Python statically or dynamically typed? Strongly or weakly?** ⭐
**Dynamically** typed: types belong to objects, not names, and are checked at runtime. **Strongly** typed: no implicit unsafe coercion, so `"5" + 3` raises `TypeError`, whereas JavaScript gives `"53"`. Type hints add optional *static* checking through tools like mypy, but the runtime ignores them.

**Q4. Why does `x = y = []` cause bugs, while `x = y = 0` doesn't?**
Both bind two names to **one object**. With an int, nothing can mutate the object; `x += 1` rebinds `x` to a new int. With a list, `x.append(1)` mutates the shared object, so `y` sees it. Use `x, y = [], []` for separate lists.

**Q5. How are variables stored internally?**
A name is a key in a namespace dict (module `globals()`, class `__dict__`, instance `__dict__`) whose value is a pointer to a heap object. Function locals are an optimization: the compiler assigns them to fixed slots in an array, which is why local lookups are faster than global ones. Each object carries a reference count; when it reaches zero the object is freed (see §35).

---

## 4. Scope & Namespaces

**What it is:** Scope decides where a name is visible. Python resolves names with the **LEGB rule**: **L**ocal → **E**nclosing function → **G**lobal (module) → **B**uilt-in.

**Why it matters:** It explains closures, decorators, `UnboundLocalError`, and why loop variables survive the loop.

**Key concepts**

- Only **modules, functions, classes and comprehensions** create scopes. `if`, `for`, `while` and `with` do **not**, so there's no block scope.
- **Assigning** to a name anywhere in a function makes it local for the *whole* function. That's the cause of `UnboundLocalError`.
- `global name` rebinds a module-level name; `nonlocal name` rebinds a name in the nearest enclosing function.
- **Class bodies are not enclosing scopes** for their methods; methods must use `self.x` or `ClassName.x`.
- **Lifetime ≠ scope**: an object lives as long as something references it, even after its scope ends (that's how closures work).

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)          # local      (L)
    inner()
    print(x)              # enclosing  (E)

outer()
print(x)                  # global     (G)
print(len)                # built-in   (B)

for i in range(3):
    pass
print(i)                  # 2 — no block scope; the loop variable survives

counter = 0
def broken():
    counter += 1          # UnboundLocalError: assignment makes counter local
def fixed():
    global counter
    counter += 1

def make_counter():
    n = 0
    def inc():
        nonlocal n        # rebind the enclosing n
        n += 1
        return n
    return inc
c = make_counter(); c(); c()   # 2

class Config:
    retries = 3
    def show(self):
        # print(retries)        # NameError — class body is not an enclosing scope
        return self.retries     # OK

# Shadowing a built-in — a classic bug
list = [1, 2]
# list("abc")             # TypeError: 'list' object is not callable
del list                  # restores access to the built-in
```

### Interview Questions

**Q1. Explain the LEGB rule.** ⭐
Python looks up a name in Local, then Enclosing function scopes (innermost first), then the module's Global namespace, then Built-ins, and raises `NameError` if all fail. The compiler decides which scope a name belongs to *at compile time*: if a function assigns to the name, it's local throughout that function unless declared `global` or `nonlocal`.

**Q2. Why does this raise `UnboundLocalError`?** ⭐
```python
x = 10
def f():
    print(x)
    x = 20
```
Because `x` is assigned in `f`, the compiler marks it local for the *entire* function. At `print(x)` the local `x` has no value yet. Fix it with `global x`, or (better) pass `x` as a parameter and return the new value.

**Q3. Does Python have block scope?**
No. Variables created inside `if`, `for`, `while`, `with` or `try` blocks belong to the enclosing function or module. Comprehensions *do* have their own scope in Python 3, so `[i for i in range(3)]` doesn't leak `i`.

**Q4. What's the difference between `global` and `nonlocal`? When should you use them?**
`global` rebinds a module-level name; `nonlocal` rebinds a variable in the nearest enclosing function (needed for closures that keep counters or state). Both are code smells in application code: global mutable state is hard to test and isn't thread-safe. Prefer return values, objects or closures with explicit state.

---

## 5. Built-in Types & Mutability

**What it is:** Python has no true "primitives"; every value is an object. Built-in types fall into **immutable** (`int`, `float`, `complex`, `bool`, `str`, `bytes`, `tuple`, `frozenset`, `NoneType`) and **mutable** (`list`, `dict`, `set`, `bytearray`, most user-defined objects).

**Why it matters:** Mutability decides hashability (dict keys), copying behaviour, default-argument bugs, thread-safety and API design (`list.sort()` vs `sorted()`).

**Key concepts**

| Type | Mutable | Ordered | Hashable | Notes |
|---|---|---|---|---|
| `int` | ✗ | — | ✓ | Arbitrary precision, never overflows |
| `float` | ✗ | — | ✓ | IEEE-754 double: `0.1 + 0.2 != 0.3` |
| `bool` | ✗ | — | ✓ | Subclass of `int` (`True + True == 2`) |
| `str` | ✗ | ✓ | ✓ | Unicode code points; no `char` type |
| `bytes` / `bytearray` | ✗ / ✓ | ✓ | ✓ / ✗ | Raw bytes |
| `tuple` | ✗ | ✓ | ✓ only if all items are hashable | |
| `list` | ✓ | ✓ | ✗ | Dynamic array |
| `dict` | ✓ | insertion order (3.7+) | ✗ | Hash table |
| `set` / `frozenset` | ✓ / ✗ | ✗ | ✗ / ✓ | Hash table |
| `NoneType` | ✗ | — | ✓ | Singleton `None` |

*Numbers:*

```python
2 ** 100                     # 1267650600228229401496703205376 — no overflow
0.1 + 0.2                    # 0.30000000000000004
from decimal import Decimal
Decimal("0.1") + Decimal("0.2")   # Decimal('0.3') — use for money
round(2.5), round(3.5)       # (2, 4) — banker's rounding (round half to even)
import math
math.isclose(0.1 + 0.2, 0.3) # True — compare floats this way
```

*Truthiness:* falsy values are `False`, `None`, `0`, `0.0`, `0j`, `""`, `b""`, `[]`, `()`, `{}`, `set()`, `range(0)`, and objects whose `__bool__` returns `False` or `__len__` returns 0. Everything else is truthy.

```python
items = []
if not items:                # Pythonic emptiness check
    print("empty")

value = 0
if value is not None:        # use this when 0 / "" are valid values!
    print("has a value")
```

*Type checks & conversion:*

```python
type(True) is int            # False
isinstance(True, int)        # True — isinstance respects inheritance; prefer it
isinstance(5, (int, float))  # True — tuple = "any of these"

int("42"), float("3.5"), str(10), list("abc")   # explicit conversion
int("abc")                   # ValueError
True + 1.5                   # 2.5 — implicit widening only within the numeric tower
```

*Mutable vs immutable in action:*

```python
s = "abc"; old = id(s)
s += "d";  id(s) == old      # False — a new str object

lst = [1, 2]; old = id(lst)
lst.append(3); id(lst) == old  # True — mutated in place

t = ([1, 2], 3)
t[0].append(99)              # allowed! the tuple is immutable, the list inside is not
# hash(t)                    # TypeError: unhashable type: 'list'
```

### Interview Questions

**Q1. Which built-in types are mutable vs immutable, and why does it matter?** ⭐
Immutable: numbers, `bool`, `str`, `bytes`, `tuple`, `frozenset`, `None`. Mutable: `list`, `dict`, `set`, `bytearray`, most custom objects. It matters because:
1. Only immutable (hashable) objects can be dict keys or set members.
2. Mutable defaults and shared references cause state leaks.
3. Immutable objects are safe to share across threads and cache.
4. API design: mutating methods return `None` (`list.sort()`), while immutable types return new objects (`str.upper()`).

**Q2. Is a tuple always immutable? Can it be a dict key?**
The tuple's *structure* is immutable: you can't add, remove or rebind its elements. But elements that are mutable (a list inside) can still change, and then the tuple is **not hashable**, so it can't be a dict key. A tuple is hashable only if all its elements are.

**Q3. Why is `0.1 + 0.2 != 0.3`, and how do you handle money?**
Floats are binary IEEE-754 doubles, and 0.1 has no exact binary representation, so the rounding errors add up. Compare with `math.isclose`. For money use `Decimal` (constructed from strings) or store integer minor units (cents/paise).

**Q4. `type()` vs `isinstance()`?**
`type(x) is T` is an exact type match; `isinstance(x, T)` also accepts subclasses and ABCs and Protocols marked `@runtime_checkable`. Prefer `isinstance` because it respects polymorphism. Even better, avoid type checks and rely on duck typing or `functools.singledispatch`.

**Q5. What's the difference between `if x:` and `if x is not None:`?**
`if x:` tests truthiness, so `0`, `""` and `[]` all fail it. When those are legitimate values (quantity 0, empty string from the user), `if x:` silently drops them. Use `is not None` when you mean "was a value provided".

---

## 6. Operators

**What it is:** Arithmetic, comparison, logical, bitwise, identity (`is`), membership (`in`), assignment (including augmented assignment and the walrus `:=`) and the conditional expression.

**Why it matters:** Several operators behave differently from C, Java or JavaScript, and interviewers use them for "predict the output" questions.

**Key concepts**

```python
# Arithmetic
7 / 2        # 3.5  — true division ALWAYS returns float
7 // 2       # 3    — floor division
-7 // 2      # -4   — floors toward -infinity, not toward zero!
-7 % 3       # 2    — result takes the sign of the divisor (C gives -1)
2 ** 3 ** 2  # 512  — ** is right-associative: 2 ** 9
-2 ** 2      # -4   — ** binds tighter than unary minus
divmod(17, 5)  # (3, 2)

# Comparison: chaining
x = 5
1 < x < 10     # True — means (1 < x) and (x < 10), x evaluated once
# 3 < "3"      # TypeError — no ordering between unrelated types

# Logical operators return an OPERAND, not necessarily a bool
0 or "default"      # 'default'
"a" or "b"          # 'a'
[] and "x"          # []   (short-circuits)
name = user_input or "Guest"   # default-value idiom (beware: also replaces "" and 0)

# Identity & membership
x is None           # identity — the idiomatic None check
3 in [1, 2, 3]      # O(n) for list/tuple
3 in {1, 2, 3}      # O(1) average for set/dict
"a" in {"a": 1}     # checks dict KEYS

# Augmented assignment: in-place for mutables!
a = [1]; b = a
a += [2]            # list.__iadd__ mutates in place -> b is [1, 2] too
a = a + [3]         # creates a NEW list; b unchanged

# No ++ / --
n = 5
++n                 # 5 — it's +(+n), NOT an increment
n += 1              # the Pythonic increment

# Walrus (3.8+): assign inside an expression
if (n := len(data)) > 10:
    print(f"too long: {n}")
while (line := f.readline()):
    process(line)

# Conditional expression
status = "adult" if age >= 18 else "minor"

# Bitwise
5 & 3, 5 | 3, 5 ^ 3, ~5, 1 << 3   # (1, 7, 6, -6, 8);  ~x == -x - 1
```

**Precedence (high → low, simplified):** `()` → `**` → unary `+ - ~` → `* / // %` → `+ -` → shifts → `&` → `^` → `|` → comparisons (`in`, `is`, `<`, `==`) → `not` → `and` → `or` → `if-else` → `lambda` → `:=`. When it isn't obvious, add parentheses.

### Interview Questions

**Q1. What's the difference between `/` and `//`? What is `-7 // 2`?** ⭐
`/` is true division and always returns a float (`4 / 2 == 2.0`). `//` is floor division and rounds toward negative infinity, so `-7 // 2 == -4`, not -3 as in C or Java (which truncate toward zero). The matching identity is `a == (a // b) * b + a % b`, which is why `-7 % 3 == 2`.

**Q2. What do `and` / `or` return?** ⭐
They return one of the operands, not a bool: `a or b` returns `a` if it's truthy, otherwise `b`; `a and b` returns `a` if it's falsy, otherwise `b`. Evaluation short-circuits, which makes guards like `obj and obj.value` and defaults like `x or default` work. Pitfall: `x or default` also replaces legitimate falsy values like `0` and `""`.

**Q3. Does Python have `++`? What does `++x` do?**
No. `++x` parses as `+(+x)`, which is a no-op, and `x++` is a `SyntaxError`. Use `x += 1`. Guido left increment/decrement out to avoid the pre/post-increment ambiguity found in C.

**Q4. `a += b` vs `a = a + b` for lists?** ⭐
`a += b` calls `__iadd__`, which **mutates the list in place**, so every other name referencing it sees the change. `a = a + b` calls `__add__`, builds a **new** list and rebinds `a`. For immutables (int, str, tuple), both create new objects. A related trap: `t = ([1],); t[0] += [2]` raises `TypeError` *and* still mutates the list.

**Q5. When is the walrus operator useful?**
It avoids computing a value twice or splitting a statement, e.g. `if (m := pattern.search(s)):`, `while (chunk := f.read(8192)):`, or filtering on a computed value in a comprehension: `[y for x in data if (y := f(x)) > 0]`. Avoid it where it hurts readability.

---

## 7. Control Flow

**What it is:** `if/elif/else`, conditional expressions, `match/case` (3.10+), `for` and `while` loops, `break`/`continue`/`pass`, and loop `else` clauses.

**Why it matters:** The basics are easy, but `for...else`, `match` patterns, iteration helpers and the "don't mutate while iterating" rule are common interview topics.

**Key concepts**

```python
# if / elif / else — any object's truthiness is accepted
if score >= 90:
    grade = "A"
elif score >= 75:
    grade = "B"
else:
    grade = "C"

# Guard clauses instead of deep nesting
def process(user):
    if not user:
        return None
    if not user.is_active:
        return None
    return do_work(user)

# for iterates over ANY iterable (no C-style for)
for i in range(0, 10, 2): ...                  # 0, 2, 4, 6, 8
for i, name in enumerate(names, start=1): ...  # index + value
for a, b in zip(xs, ys): ...                   # parallel iteration (stops at shortest)
for a, b in zip(xs, ys, strict=True): ...      # 3.10+: raises if lengths differ
for k, v in d.items(): ...                     # dict key/value pairs
for x in reversed(seq): ...

# for...else / while...else: else runs ONLY if the loop did NOT break
for n in nums:
    if n < 0:
        print("found negative")
        break
else:
    print("no negatives")                      # search-without-flag idiom

# do-while idiom
while True:
    cmd = input("> ")
    if cmd == "quit":
        break

# Never change a collection's size while iterating over it
d = {"a": 1, "b": 2}
# for k in d: del d[k]                         # RuntimeError: dictionary changed size
for k in list(d): del d[k]                     # iterate over a snapshot
nums = [x for x in nums if x % 2]              # build a new list instead of removing items

# Dispatch table (pre-3.10 "switch")
handlers = {"start": start, "stop": stop}
handlers.get(cmd, unknown)()
```

*Structural pattern matching (3.10+)* matches the **shape** of data, not just values:

```python
def handle(event):
    match event:
        case {"type": "click", "pos": (x, y)}:        # mapping + sequence pattern
            return f"click at {x},{y}"
        case {"type": "key", "key": "q" | "Q"}:       # OR pattern
            return "quit"
        case [first, *rest]:                          # sequence with a capture
            return f"list starting {first}"
        case Point(x=0, y=0):                         # class pattern
            return "origin"
        case int(n) if n > 0:                         # type pattern + guard
            return "positive int"
        case _:                                       # wildcard (default)
            return "unknown"
```

Gotcha: a bare name in a `case` **captures** instead of comparing. `case RED:` matches anything and binds it to `RED`. To compare against a constant, use a dotted name (`case Color.RED:`).

### Interview Questions

**Q1. What does `else` on a `for` loop do?** ⭐
It runs when the loop finishes **without hitting `break`** (including when the iterable is empty). It's the idiom for "search, and handle not-found" without a flag variable. Many developers misread it as "runs if the loop didn't execute", so add a comment when you use it.

**Q2. Python has no switch. What do you use?**
Before 3.10: `if/elif` chains or a **dict dispatch table** (`handlers.get(key, default)()`), which is O(1) and extensible. From 3.10, `match/case` adds structural pattern matching: destructuring sequences and mappings, class patterns, OR patterns and guards. It's far more than a C-style switch, which makes it great for parsing JSON events, ASTs and commands.

**Q3. How do you remove items from a list while iterating over it?**
Don't mutate it during iteration: elements get skipped as indices shift (and dicts and sets raise `RuntimeError`). Instead, build a new list with a comprehension (`[x for x in lst if keep(x)]`), iterate over a copy (`for x in lst[:]`), or iterate backwards by index.

**Q4. Why prefer `enumerate` and `zip` over `range(len(...))`?**
They're more readable, work with any iterable (not just sequences that support indexing), avoid off-by-one errors, and are lazy. `zip(..., strict=True)` (3.10+) also catches length mismatches that would otherwise be silently truncated.

---

## 8. Strings

**What it is:** `str` is an **immutable sequence of Unicode code points**. There is no `char` type; a character is a string of length 1. `bytes` is a separate type for raw bytes.

**Why it matters:** String handling appears in nearly every coding round (palindromes, anagrams, parsing), and there are well-known gotchas around performance (`+=` in loops), `strip()`, `split()` and encoding.

**Key concepts**

```python
s = 'single'; s2 = "double"; s3 = """multi
line"""
raw = r"C:\new\table"          # raw string: backslashes kept (regex, Windows paths)
b = b"bytes"                   # bytes, NOT str
msg = ("adjacent literals "
       "are joined at compile time")

# Indexing & slicing (slices never raise IndexError)
s = "Python"
s[0], s[-1]                    # ('P', 'n')
s[1:4], s[::2], s[::-1]        # ('yth', 'Pto', 'nohtyP')
s[2:100]                       # 'thon' — clamped
# s[0] = "J"                   # TypeError — immutable

# Concatenation: use join for many pieces
parts = ["a", "b", "c"]
"".join(parts)                 # O(n) — idiomatic
# result += piece in a loop can degrade to O(n^2); don't rely on CPython's optimization
", ".join(map(str, [1, 2, 3])) # join needs str items

# Searching
"nan" in "banana"              # True
"banana".find("x")             # -1  (index() raises ValueError instead)
"banana".count("a")            # 3
"report.pdf".endswith((".pdf", ".doc"))   # tuple of options

# Splitting
"a  b\tc\n".split()            # ['a', 'b', 'c'] — any whitespace, empties dropped
"a,,b".split(",")              # ['a', '', 'b'] — explicit separator keeps empties
"k=v=w".split("=", 1)          # ['k', 'v=w']
"user@host".partition("@")     # ('user', '@', 'host')

# Trimming — the classic gotcha
"xxhixx".strip("x")            # 'hi'
"docs.csv".rstrip(".csv")      # 'do'  !!! the argument is a SET of characters
"docs.csv".removesuffix(".csv")  # 'docs' (3.9+) — correct

# Case
"Straße".casefold() == "STRASSE".casefold()   # True — use casefold for comparisons
"they're".title()              # "They'Re" — gotcha

# Replace / translate / regex
"a-b-c".replace("-", "_", 1)   # 'a_b-c'
"hello".translate(str.maketrans("", "", "aeiou"))   # 'hll'
import re
re.sub(r"\s+", " ", "too    many   spaces")          # 'too many spaces'
re.findall(r"\d+", "order 12, qty 30")              # ['12', '30']
```

*Formatting.* f-strings are the default; the format-spec mini-language is `{value:[fill][align][width][,][.precision][type]}`:

```python
name, pi, n = "Asha", 3.14159, 1234567.891
f"{name} owes {pi:.2f}"        # 'Asha owes 3.14'
f"{n:,.2f}"                    # '1,234,567.89'
f"{0.256:.1%}"                 # '25.6%'
f"{42:>8}", f"{'hi':*^8}"      # '      42', '***hi***'
f"{7:03d}", f"{255:x}"         # '007', 'ff'
f"{n * 2 = }"                  # 'n * 2 = 2469135.782' — debug specifier (3.8+)
f"{name!r}"                    # "'Asha'" — repr()

# Logging: pass args lazily; SQL: NEVER f-strings (injection)
logger.info("user %s logged in", name)
cursor.execute("SELECT * FROM users WHERE name = %s", (name,))
```

*Template strings (3.14+):* `t"Hello {name}"` returns a `Template` object instead of a `str`, so libraries can escape the interpolated values safely (for HTML or SQL) before rendering.

*Encoding:* `str` is text, `bytes` is binary. Encode text to bytes, decode bytes to text; UTF-8 is the default codec.

```python
s = "café"
b = s.encode("utf-8")          # b'caf\xc3\xa9'
len(s), len(b)                 # (4, 5) — é takes 2 bytes
b.decode("utf-8")              # 'café'
# "a" + b"b"                   # TypeError — never mix str and bytes
ord("A"), chr(65)              # (65, 'A')
open("f.txt", encoding="utf-8")  # always pass encoding; the default is OS-dependent
```

*Interning:* CPython interns identifier-like strings (and anything passed to `sys.intern()`), so `is` sometimes "works". Always compare strings with `==`.

### Interview Questions

**Q1. Why are strings immutable in Python?** ⭐
1. **Hashability**: strings are the most common dict keys, and a hash must never change.
2. **Safety**: they can be shared across threads and references with no defensive copies.
3. **Performance**: interning and caching of literals.

The cost is that every "modification" creates a new string, so heavy text building should use `"".join(list)` or `io.StringIO`.

**Q2. Why is `"".join(parts)` preferred over `+=` in a loop?** ⭐
Each `+=` on an immutable string may copy everything built so far, which is O(n²) overall. `join` computes the total length once and copies each piece once, which is O(n). CPython sometimes optimizes `+=` in place when the refcount is 1, but that's an implementation detail (PyPy doesn't do it), so don't rely on it.

**Q3. What does `"docs.csv".rstrip(".csv")` return?**
`'do'`. `strip`, `lstrip` and `rstrip` treat their argument as a **set of characters** to remove from the ends, not as a substring, so they also eat the `s` and `c` of "docs". Use `removesuffix` / `removeprefix` (3.9+) to remove an exact substring.

**Q4. What's the difference between `str` and `bytes`? What causes `UnicodeDecodeError`?**
`str` holds Unicode code points (text); `bytes` holds raw 0–255 values (what's actually on disk or on the wire). Python 3 never mixes them implicitly. `UnicodeDecodeError` means you decoded bytes with the wrong codec (e.g. UTF-8 bytes read as ASCII, or a Latin-1 file read as UTF-8). Fix it by knowing the source encoding and passing `encoding=` explicitly, or use `errors="replace"` when lossy decoding is acceptable.

**Q5. f-strings vs `.format()` vs `%`: which, and when?**
Use f-strings by default: they're the most readable and fastest, and support expressions plus `=` and `!r`. `.format()` suits templates defined elsewhere (e.g. config). Keep `%` style for **logging** calls (`logger.info("x=%s", x)`), because formatting only happens if the record is emitted. Never use any of them to build SQL or shell commands; use parameterized queries and argument lists.

**Q6. Write a function to check if two strings are anagrams.**
```python
from collections import Counter
def is_anagram(a: str, b: str) -> bool:
    return Counter(a.replace(" ", "").casefold()) == Counter(b.replace(" ", "").casefold())
# O(n) time; sorted(a) == sorted(b) also works but is O(n log n)
```

---

# PART 2 — DATA STRUCTURES

---

## 9. Lists, Tuples, Dicts & Sets

**What it is:** The four core built-in containers.
- `list`: ordered, mutable **dynamic array**
- `tuple`: ordered, immutable sequence
- `dict`: insertion-ordered **hash map**
- `set`: unordered **hash set** of unique items

**Why it matters:** Choosing the right container is the most common performance decision in Python, and internals and Big-O questions are interview staples.

**Key concepts — internals**

- **`list`** is a contiguous array of pointers to objects, not a linked list. It **over-allocates** on growth (roughly 1.125× plus a constant), so `append` is **amortized O(1)**. Indexing is O(1). `insert(0, x)` and `pop(0)` are **O(n)** because every element shifts.
- **`tuple`** is a fixed-size array of pointers. It's smaller and faster to create than a list, hashable if its items are, and its literals can be constant-folded by the compiler.
- **`dict`** is a hash table with **open addressing**. Since 3.6 it uses a *compact* layout: a sparse index table pointing into a dense array of `(hash, key, value)` entries. That made it ~20–25% smaller and gave insertion order for free, which became a **language guarantee in 3.7**. Lookup is O(1) on average, O(n) in the worst case under pathological collisions. String hashes are randomized per process (`PYTHONHASHSEED`) to prevent hash-flooding DoS.
- **`set`** is a hash table of keys only, with the same O(1) average membership. It has **no order guarantee**.

**Time complexity cheat sheet**

| Operation | list | dict | set |
|---|---|---|---|
| Index / key lookup | O(1) | O(1) avg | — |
| `x in c` | **O(n)** | O(1) avg | O(1) avg |
| Append / add | O(1) amortized | O(1) avg | O(1) avg |
| Insert / delete at front | **O(n)** | — | — |
| Delete by key / value | O(n) | O(1) avg | O(1) avg |
| `pop()` from end | O(1) | `popitem()` O(1) | O(1) |
| Sort | O(n log n) | — | — |
| Copy | O(n) | O(n) | O(n) |

```python
# list
nums = [3, 1, 2]
nums.append(4); nums.extend([5, 6]); nums.insert(0, 0)
nums.pop(); nums.remove(3)          # remove: first match, O(n)
nums.sort(); sorted(nums)           # in place (returns None) vs a new list

# tuple
point = (3, 4)
single = (1,)                       # the comma makes the tuple, not the parentheses
x, y = point

# dict
user = {"name": "Asha", "age": 30}
user.get("email")                   # None — no KeyError
user.get("email", "n/a")            # default
user.setdefault("tags", []).append("admin")
user | {"age": 31}                  # merge into a new dict (3.9+); right side wins
user |= {"city": "Pune"}            # in-place merge
for k, v in user.items(): ...
user.keys() & {"name", "zip"}       # dict views support set operations

# set
a, b = {1, 2, 3}, {2, 3, 4}
a | b, a & b, a - b, a ^ b          # union, intersection, difference, symmetric difference
seen = set()
seen.add(1); seen.discard(99)       # discard never raises; remove() raises KeyError
unique_in_order = list(dict.fromkeys([3, 1, 3, 2]))   # [3, 1, 2] — dedupe, keep order
```

**When to use which**

| Need | Use |
|---|---|
| Ordered, indexable, append-heavy | `list` |
| Fixed record / dict key / function return | `tuple` (or `NamedTuple`) |
| Key → value lookup, counting, caching, grouping | `dict` (`defaultdict`, `Counter`) |
| Membership tests, dedupe | `set` |
| Queue / both-ends operations | `collections.deque` |
| Priority queue / top-k | `heapq` |
| Sorted-order maintenance | `bisect` on a list (or `sortedcontainers`) |

### Interview Questions

**Q1. How is a Python list implemented? What's the complexity of `append` vs `insert(0)`?** ⭐
A list is a dynamic array of object pointers. When it's full, CPython reallocates with extra capacity (over-allocation), so occasional O(n) copies are spread over many appends: `append` is **amortized O(1)**. `insert(0, x)` and `pop(0)` shift every element, so they're **O(n)**. For a queue, use `collections.deque`, which has O(1) operations at both ends.

**Q2. How does a dict work internally? Why is it ordered?** ⭐
It's a hash table: `hash(key)` picks a slot, and collisions are resolved by **open addressing** (probing other slots with a perturbation scheme), not by chaining. Since 3.6 CPython stores entries in a dense array, in insertion order, with a separate sparse index table. Iterating the dense array gives insertion order naturally; 3.7 made it a language guarantee. Lookups are O(1) on average. Keys must be hashable, and equal keys must have equal hashes.

**Q3. Why is `x in my_set` faster than `x in my_list`?** ⭐
A set hashes `x` and jumps straight to the right slot: O(1) on average. A list compares element by element: O(n). On a million items that's the difference between microseconds and milliseconds per check. Converting a list to a set costs O(n) once, which pays off as soon as you do more than a few lookups.

**Q4. Tuple vs list: when do you use each?**
- **Tuple**: a fixed, heterogeneous record (`(lat, lng)`), returning multiple values, dict keys or set members, data that must not change. It also uses slightly less memory.
- **List**: a homogeneous, growing or changing collection.

A good rule of thumb is "tuples are records, lists are collections". For named fields, use `NamedTuple` or a dataclass.

**Q5. Are dicts and sets ordered?**
Dicts preserve **insertion order** (guaranteed since 3.7), but they are not *sorted*. Sets have **no ordering guarantee**; iteration order depends on hashes and can differ between runs for strings. If you need sorted output, use `sorted(...)`. If you need an ordered set, use `dict.fromkeys(items)`.

**Q6. What's the worst-case complexity of a dict lookup, and when does it happen?**
O(n), when many keys collide (identical or crafted hashes force long probe sequences). Attackers used this for hash-flooding DoS via POST parameters, which is why Python randomizes `str`/`bytes` hashing per process. It also happens with a badly written `__hash__` (e.g. returning a constant).

---

## 10. Hashing & Hashability

**What it is:** An object is **hashable** if it has a hash value that never changes during its lifetime (`__hash__`) and can be compared for equality (`__eq__`). Only hashable objects can be dict keys or set members.

**Why it matters:** It ties together immutability, dict/set performance and custom class design, and it's a frequent senior-level question ("what breaks if you override `__eq__` only?").

**Key concepts**

- **The hash contract**: if `a == b`, then `hash(a) == hash(b)`. The reverse isn't required (collisions are allowed).
- Built-in immutables are hashable; `list`, `dict` and `set` are **not** (`__hash__ = None`).
- User-defined classes are hashable by default, by **identity** (`id`-based hash, `is`-based equality).
- **If you define `__eq__` without `__hash__`, Python sets `__hash__ = None`**, so instances become unhashable. That protects you from breaking the contract.
- The hash of a key must not change while it's in a dict, which is why mutable objects make bad keys.

```python
hash("abc"), hash(42), hash((1, 2))     # fine
# hash([1, 2])                          # TypeError: unhashable type: 'list'

class Point:
    def __init__(self, x, y):
        self.x, self.y = x, y
    def __eq__(self, other):
        return isinstance(other, Point) and (self.x, self.y) == (other.x, other.y)
    def __hash__(self):
        return hash((self.x, self.y))   # hash the SAME fields used in __eq__

{Point(1, 2), Point(1, 2)}              # one element

# The danger of a mutable key
p = Point(1, 2)
d = {p: "here"}
p.x = 99                                # hash changes while p is inside the dict
# d[p] -> KeyError; d[Point(1, 2)] -> KeyError — the entry is "lost"

# Simplest correct way: a frozen dataclass generates both methods
from dataclasses import dataclass
@dataclass(frozen=True)
class Coord:
    x: int
    y: int
```

### Interview Questions

**Q1. What makes an object hashable? Why can't a list be a dict key?** ⭐
It needs a `__hash__` that stays constant for its lifetime and an `__eq__` consistent with it. A list is mutable: if its contents (and so its hash) changed while it was a dict key, the dict would look in the wrong slot and the entry would become unreachable. So `list.__hash__` is `None`. Use a tuple (or `frozenset`) of its contents as the key instead.

**Q2. What happens if you override `__eq__` but not `__hash__`?** ⭐
Python sets `__hash__` to `None`, so instances can no longer go into sets or be dict keys (`TypeError: unhashable type`). That's deliberate: inheriting the identity-based hash would break the rule that equal objects have equal hashes. Fix it by defining `__hash__` over the same immutable fields used in `__eq__`, or by using `@dataclass(frozen=True)`.

**Q3. Can two unequal objects have the same hash? Can two equal objects have different hashes?**
Unequal objects with the same hash is a **collision**: allowed and handled by the hash table, at some performance cost. Equal objects with different hashes is a **contract violation**: dicts and sets will treat them as different keys, producing silent duplicates and missed lookups.

**Q4. Why is `hash("abc")` different between Python runs?**
Hash randomization (on by default since 3.3) seeds `str` and `bytes` hashing per process to stop hash-flooding DoS attacks. Never persist `hash()` values or use them across processes. For stable hashes use `hashlib` (e.g. SHA-256) or set `PYTHONHASHSEED` for reproducible tests.

---

## 11. Comprehensions & Generator Expressions

**What it is:** Concise syntax for building collections from iterables. List `[...]`, dict `{k: v ...}` and set `{...}` comprehensions are **eager**; generator expressions `(...)` are **lazy**.

**Why it matters:** They're idiomatic, usually faster than `for` + `append`, and the eager-vs-lazy choice is a memory-design question.

**Key concepts**

```python
squares = [x * x for x in range(10)]                  # list
evens = {x for x in nums if x % 2 == 0}               # set
index = {u.id: u for u in users}                      # dict (id -> object lookup)
inverted = {v: k for k, v in mapping.items()}
labels = ["even" if x % 2 == 0 else "odd" for x in nums]   # if-else goes BEFORE for
pairs = [(i, j) for i in range(3) for j in range(3) if i != j]  # nested: outer loop first
flat = [x for row in matrix for x in row]             # flatten a 2D list

# Generator expression: lazy, O(1) memory
total = sum(x * x for x in range(10**8))              # never builds a list
any(line.startswith("ERROR") for line in open("app.log"))   # stops at the first match
```

- Comprehensions run in their own scope (the loop variable doesn't leak).
- They're faster than `append` loops because the loop runs as specialized bytecode with no repeated `.append` attribute lookup.
- **Use a generator expression** for single-pass pipelines feeding `sum`, `any`, `all`, `max`, `"".join` or another generator. **Use a list** when you need `len()`, indexing or several passes.
- Don't use comprehensions for side effects (`[print(x) for x in xs]`); use a plain loop.
- Past two levels of nesting or a complex condition, a regular loop is more readable.

### Interview Questions

**Q1. List comprehension vs generator expression: what's the difference?** ⭐
A list comprehension builds the whole list immediately (O(n) memory) and can be reused, indexed and measured with `len()`. A generator expression produces items on demand (O(1) memory), can be consumed only once, and does the work lazily. For `sum(x*x for x in range(10**8))` the generator uses kilobytes; the list version would allocate gigabytes.

**Q2. Why is a comprehension faster than a `for` loop with `append`?**
The comprehension compiles to tight bytecode using a dedicated `LIST_APPEND` instruction. The manual loop does an attribute lookup (`result.append`) and a method call on every iteration. The gain is typically 20–40%, but readability is the bigger reason to use them.

**Q3. Do comprehension variables leak into the enclosing scope?**
Not in Python 3: comprehensions have their own scope (Python 2 list comprehensions did leak). Since 3.12 (PEP 709) comprehensions are *inlined* for speed but keep the same isolation semantics. Note that a walrus inside a comprehension (`[y := f(x) for x in data]`) *does* bind `y` in the enclosing scope, by design.

**Q4. When would you not use a comprehension?**
When the logic has side effects, needs `try/except`, has multiple statements, or nests more than about two levels. Readability beats cleverness; interviewers grade that too.

---

## 12. Unpacking, Slicing & Sorting

**What it is:** Unpacking binds several names from an iterable at once; slicing extracts sub-sequences with `[start:stop:step]`; sorting uses **Timsort**, a stable O(n log n) hybrid of merge sort and insertion sort.

**Why it matters:** Used constantly in idiomatic code and coding rounds, e.g. swap, head/tail splitting, sort by multiple keys.

**Key concepts**

```python
# Unpacking
a, b = 1, 2
a, b = b, a                          # swap — no temp variable
first, *middle, last = [1, 2, 3, 4, 5]   # extended unpacking: middle = [2, 3, 4]
(x, y), z = (1, 2), 3                # nested
for i, (name, score) in enumerate(pairs): ...

def f(a, b, c): ...
f(*[1, 2, 3])                        # spread positional
f(**{"a": 1, "b": 2, "c": 3})        # spread keywords
merged = {**defaults, **overrides}   # later keys win
combined = [*list_a, *list_b]

# Slicing: start inclusive, stop exclusive, never raises IndexError
s = [0, 1, 2, 3, 4, 5]
s[1:4], s[:3], s[3:], s[-2:]         # [1,2,3] [0,1,2] [3,4,5] [4,5]
s[::2], s[::-1]                      # [0,2,4] [5,4,3,2,1,0]
copy_ = s[:]                         # shallow copy
s[1:3] = [10, 20, 30]                # slice assignment can change the length
del s[::2]                           # delete every 2nd element
window = slice(1, 4); s[window]      # named slice object

# Sorting
sorted(words)                        # returns a new list; works on any iterable
words.sort()                         # in place, returns None (a classic bug: x = x.sort())
sorted(words, key=len)               # by length
sorted(words, key=str.casefold)      # case-insensitive
sorted(users, key=lambda u: (-u.score, u.name))   # score desc, then name asc
from operator import attrgetter, itemgetter
sorted(users, key=attrgetter("age"))
sorted(rows, key=itemgetter(1, 0))
max(users, key=lambda u: u.score)    # min/max/heapq also accept key=
```

- **Stability**: equal elements keep their original order, so you can sort by a secondary key first and the primary key second.
- `key=` is called **once per element** (decorate–sort–undecorate), which is much faster than comparator functions. For legacy comparators use `functools.cmp_to_key`.

### Interview Questions

**Q1. `sorted()` vs `list.sort()`?** ⭐
`sorted(iterable)` works on any iterable and returns a **new** list. `list.sort()` sorts **in place** and returns `None`, so it saves memory, but `x = x.sort()` sets `x` to `None`. Both use Timsort, are stable, and accept `key` and `reverse`.

**Q2. What sorting algorithm does Python use? Is it stable?**
**Timsort**: it finds already-sorted "runs" in the data and merges them, using insertion sort on small runs. Worst case O(n log n); **O(n) on already-sorted or nearly-sorted data**, which is very common in practice. It's **stable**, which enables multi-pass sorting by secondary and then primary key.

**Q3. How do you sort by multiple keys, one descending?**
Return a tuple from `key`: `sorted(users, key=lambda u: (-u.score, u.name))`. For non-numeric descending keys, use stability instead: sort by the secondary key, then by the primary key with `reverse=True`.

**Q4. How does `first, *rest = items` work, and what happens with an empty list?**
Extended unpacking assigns the first element to `first` and collects the rest into a **list** `rest` (always a list, even from a tuple). With an empty iterable it raises `ValueError: not enough values to unpack`, because the starred name can be empty but the mandatory names can't.

---

## 13. Collections Module, heapq & bisect

**What it is:** Standard-library data structures that cover the gaps left by the built-ins: `Counter`, `defaultdict`, `deque`, `OrderedDict`, `ChainMap` and `namedtuple` (in `collections`), `heapq` (binary heap priority queue) and `bisect` (binary search on sorted lists).

**Why it matters:** They turn many interview problems into a few lines (top-k, frequency counts, sliding windows, LRU caches, BFS) and show stdlib fluency.

**Key concepts**

```python
from collections import Counter, defaultdict, deque, OrderedDict, ChainMap

# Counter — a frequency map
c = Counter("mississippi")
c.most_common(2)                 # [('i', 4), ('s', 4)]
c["z"]                           # 0 — missing keys count as 0, no KeyError
Counter(a) == Counter(b)         # anagram check
c1 + c2, c1 - c2, c1 & c2        # multiset arithmetic

# defaultdict — automatic default values
groups = defaultdict(list)
for word in words:
    groups[len(word)].append(word)   # no "if key not in" boilerplate
graph = defaultdict(set)

# deque — O(1) at both ends; maxlen gives a bounded window
q = deque([1, 2, 3])
q.appendleft(0); q.append(4); q.popleft(); q.pop()
q.rotate(1)
last_5 = deque(maxlen=5)         # automatically drops the oldest item

# BFS uses deque
def bfs(start, graph):
    seen, queue = {start}, deque([start])
    while queue:
        node = queue.popleft()
        for nxt in graph[node]:
            if nxt not in seen:
                seen.add(nxt); queue.append(nxt)
    return seen

# OrderedDict — order-aware operations useful for LRU caches
od = OrderedDict()
od.move_to_end(key); od.popitem(last=False)

# ChainMap — layered lookup (CLI args -> env -> defaults)
config = ChainMap(cli_args, env_vars, defaults)

import heapq
nums = [5, 1, 8, 3]
heapq.heapify(nums)              # O(n), a MIN-heap in place
heapq.heappush(nums, 2)          # O(log n)
heapq.heappop(nums)              # O(log n), smallest first
heapq.nlargest(3, nums)          # top-k in O(n log k)
heapq.heappush(pq, (priority, counter, task))   # tuple ordering; counter breaks ties
heapq.heappush(maxh, -value)     # max-heap trick: negate values

import bisect
scores = [10, 20, 30, 40]
bisect.bisect_left(scores, 25)   # 2 — insertion point, O(log n)
bisect.insort(scores, 25)        # insert keeping order (the insert itself is O(n))
```

**LRU cache with OrderedDict (a classic coding question)**

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity, self.data = capacity, OrderedDict()

    def get(self, key):
        if key not in self.data:
            return -1
        self.data.move_to_end(key)          # mark as most recently used
        return self.data[key]

    def put(self, key, value):
        self.data[key] = value
        self.data.move_to_end(key)
        if len(self.data) > self.capacity:
            self.data.popitem(last=False)   # evict the least recently used
```

### Interview Questions

**Q1. `dict` vs `defaultdict` vs `Counter`?**
`defaultdict(factory)` calls the factory for missing keys, which is ideal for grouping (`list`), adjacency sets (`set`) and counting (`int`). `Counter` is a dict subclass for counting, with `most_common`, arithmetic, and 0 for missing keys. Caveat: just *reading* a missing key from a `defaultdict` inserts it, so use `.get()` when you only want to check.

**Q2. Why use `deque` instead of a list as a queue?** ⭐
`list.pop(0)` is O(n); `deque.popleft()` is O(1). A deque is a doubly linked list of fixed-size blocks, so both ends are cheap. `maxlen` gives a ring buffer for sliding windows and "last N" logs. Trade-off: indexing into the middle of a deque is O(n).

**Q3. How do you find the top-k elements efficiently?** ⭐
`heapq.nlargest(k, items)` keeps a heap of size k: O(n log k) time and O(k) memory, better than a full O(n log n) sort when k ≪ n. For streaming data, keep a min-heap of size k and replace the root whenever a bigger item arrives. For k = 1, just use `max()`.

**Q4. `heapq` is a min-heap. How do you get a max-heap?**
Push negated values (`-x`), or for objects push `(-priority, counter, item)`. The `counter` breaks ties so Python never has to compare the `item` objects themselves (which may not support `<`).

**Q5. Implement an LRU cache.** ⭐
Use the `OrderedDict` pattern above (O(1) get and put), or a dict plus a doubly linked list if they want it built from scratch. In real code, `functools.lru_cache` memoizes functions. Mention its limits: it's per process, keys must be hashable, and it isn't a distributed cache (use Redis for that).

---

## 14. Copying: Assignment vs Shallow vs Deep

**What it is:** **Assignment** copies nothing (it's a new name for the same object). A **shallow copy** creates a new outer container that still references the **same inner objects**. A **deep copy** recursively copies everything.

**Why it matters:** Shared nested state is one of the most common real Python bugs: test fixtures leaking between tests, config dicts mutated by a callee, 2D grids where every row is the same list.

**Key concepts**

```python
import copy

original = {"users": [{"name": "Asha"}], "count": 1}

alias = original                    # assignment: the same object
shallow = copy.copy(original)       # also: dict(original), original.copy(), list[:], list(l)
deep = copy.deepcopy(original)

shallow["count"] = 2                # top level: independent
shallow["users"][0]["name"] = "Bo"  # nested: SHARED
original["users"][0]["name"]        # 'Bo'  <- original affected!

deep["users"][0]["name"] = "Cy"
original["users"][0]["name"]        # still 'Bo'

# The classic 2D-grid bug
grid = [[0] * 3] * 3                # 3 references to ONE row
grid[0][0] = 1
grid                                # [[1,0,0],[1,0,0],[1,0,0]]
grid = [[0] * 3 for _ in range(3)]  # correct: 3 separate rows
```

- `deepcopy` keeps a `memo` dict of already-copied objects, so it handles **circular references** and shared sub-objects correctly.
- Customize with `__copy__` / `__deepcopy__`, e.g. to share a DB connection instead of trying to copy it.
- If the nested items are immutable (ints, strings, tuples of immutables), shallow and deep copies behave the same, and shallow is cheaper.

### Interview Questions

**Q1. Explain shallow vs deep copy with an example.** ⭐
A shallow copy duplicates only the outer container; its elements are the same objects. Mutating a nested list through the copy changes the original. A deep copy recursively duplicates every nested object, giving a fully independent structure, at the cost of O(size of the whole object graph) time and memory. `list[:]`, `list(l)`, `dict(d)`, `.copy()` and `copy.copy` are all shallow.

**Q2. Why does `[[0]*3]*3` behave strangely?** ⭐
`*` repeats **references**: the outer list holds three pointers to the same inner list, so changing one "row" changes them all. Build the rows with a comprehension so each is a distinct list.

**Q3. When is a shallow copy the right choice?**
When the elements are immutable, or when you *want* to share them (e.g. a new list of the same user objects, re-sorted). Deep copying is expensive and can be wrong: copying objects that hold sockets, locks or DB connections fails or makes no sense.

**Q4. How does `deepcopy` handle circular references?**
It keeps a `memo` dict mapping `id(original)` → copy. Before copying an object it checks the memo. If the object was already copied (a cycle or a shared reference), it reuses that copy, which avoids infinite recursion and preserves the shape of the object graph.

---

# PART 3 — FUNCTIONS

---

## 15. Function Fundamentals & Argument Passing

**What it is:** Functions are defined with `def` (or `lambda`). They take parameters, return a value (`None` if there's no `return`), and are **objects** created when the `def` statement *executes*.

**Why it matters:** Parameter kinds, argument passing semantics and the mutable-default bug are among the most-asked Python questions at every level.

**Key concepts**

*Parameters vs arguments:* parameters are the names in the definition; arguments are the values passed at the call site.

*Parameter kinds, in order:*

```python
def f(pos_only, /, normal, *args, kw_only, **kwargs): ...
#     ^ before /     ^ positional or keyword
#                    ^ *args: extra positionals -> tuple
#                              ^ after * or *args: keyword-only
#                                       ^ **kwargs: extra keywords -> dict

def create_user(name, /, email, *, role="member", **extra):
    return {"name": name, "email": email, "role": role, **extra}

create_user("Asha", "a@x.com", role="admin", team="platform")
# create_user(name="Asha", ...)   # TypeError: name is positional-only
# create_user("Asha", "a@x.com", "admin")   # TypeError: role is keyword-only
```

- **Positional-only (`/`, 3.8+)** lets library authors rename parameters without breaking callers (many built-ins use it: `len(obj, /)`).
- **Keyword-only (`*`)** forces callers to write `retry(fn, attempts=3)`, which is self-documenting and stops boolean flags from being passed by position.

*Returning:* `return a, b` returns **one tuple**, which the caller unpacks. With no `return`, the function returns `None`.

*No overloading:* defining `def f` twice just rebinds the name. Use defaults, `*args`, `functools.singledispatch`, or `typing.overload` (for type checkers only).

*Pass by object reference ("call by sharing"):*

```python
def mutate(lst):
    lst.append(100)          # mutates the caller's object

def reassign(lst):
    lst = [9, 9]             # rebinds the LOCAL name only

nums = [1, 2]
mutate(nums);   nums         # [1, 2, 100]
reassign(nums); nums         # [1, 2, 100] — unchanged
```

*Mutable default arguments:* defaults are evaluated **once, at definition time**, and stored in `f.__defaults__`.

```python
def add_item(item, cart=[]):       # BUG: one list shared by every call
    cart.append(item)
    return cart

add_item("apple")    # ['apple']
add_item("banana")   # ['apple', 'banana']  <- state leaked between calls
add_item.__defaults__   # (['apple', 'banana'],)

def add_item(item, cart=None):     # FIX: a None sentinel
    if cart is None:
        cart = []
    cart.append(item)
    return cart

# When None is a legitimate value, use a private sentinel
_MISSING = object()
def get(key, default=_MISSING): ...
```

The same "evaluated once" rule means `def log(msg, ts=datetime.now())` freezes the timestamp at import time.

### Interview Questions

**Q1. Is Python pass-by-value or pass-by-reference?** ⭐
Neither: it's **pass-by-object-reference** (call by sharing). The function's parameter is bound to the *same object* the caller passed. **Mutations** through it are visible to the caller; **rebinding** the parameter (`x = ...`) only changes the local name. So whether the caller "sees changes" depends on mutability, not on the passing mechanism. One way to put it: "Python passes references by value."

**Q2. Explain the mutable default argument bug and its fix.** ⭐
Default values are evaluated once, when the `def` statement runs, and stored on the function object (`__defaults__`). A mutable default (`[]`, `{}`) is therefore shared across all calls that don't pass the argument, so state accumulates. Fix it with `None` (or a sentinel) and create the object inside the body. Dataclasses guard against this: `items: list = []` raises `ValueError`, forcing `field(default_factory=list)`.

**Q3. What are `*args` and `**kwargs`?** ⭐
`*args` collects extra positional arguments into a **tuple**; `**kwargs` collects extra keyword arguments into a **dict**. At call sites, `*` and `**` *unpack* iterables and mappings. Their main use is wrappers and decorators that forward any signature: `def wrapper(*args, **kwargs): return fn(*args, **kwargs)`. Avoid `**kwargs` in public APIs when you could list explicit parameters; it hurts IDE autocomplete and type checking.

**Q4. What are positional-only and keyword-only parameters, and why use them?**
Parameters before `/` must be passed by position, so you can rename them later without breaking callers. Parameters after `*` must be passed by keyword, which keeps call sites readable (`connect(host, timeout=5, retries=3)`) and prevents positional mistakes with boolean flags. Both make public APIs safer to evolve.

**Q5. Does Python support function overloading?**
Not by signature: a second `def` with the same name replaces the first. Options: default and variadic arguments; `functools.singledispatch` / `singledispatchmethod` for dispatch on the first argument's type; `typing.overload` to describe several signatures to type checkers (one real implementation underneath).

---

## 16. First-Class Functions, Lambdas & Higher-Order Functions

**What it is:** Functions are **first-class objects**: they can be assigned, stored in data structures, passed as arguments and returned. A **higher-order function** takes or returns functions. A **lambda** is an anonymous function limited to a single expression.

**Why it matters:** This is the foundation for callbacks, strategies, decorators, `sorted(key=...)` and the functional style.

**Key concepts**

```python
def shout(text): return text.upper()

speak = shout                         # assign
handlers = {"loud": shout, "quiet": str.lower}   # store in a dict
handlers["loud"]("hi")                # 'HI'

def apply_twice(fn, x):               # takes a function
    return fn(fn(x))
apply_twice(lambda x: x + 3, 10)      # 16

def multiplier(n):                    # returns a function
    return lambda x: x * n
double = multiplier(2); double(5)     # 10

# Functions are objects with attributes
shout.__name__, shout.__doc__, shout.__defaults__, shout.__code__.co_varnames

# map / filter / reduce
list(map(str.strip, ["  a ", "b  "]))        # ['a', 'b'] — lazy iterator in Py3
list(filter(None, [0, 1, "", "x"]))          # [1, 'x'] — filter(None) drops falsy items
from functools import reduce
reduce(lambda acc, x: acc * x, [1, 2, 3, 4], 1)   # 24

# Usually clearer as comprehensions or built-ins:
[s.strip() for s in raw]
sum(xs), max(xs, key=len), any(...), all(...)
from operator import mul, itemgetter, attrgetter, methodcaller
reduce(mul, [1, 2, 3, 4])
```

**Lambda limits:** a single expression only (no statements, no annotations), and a poor `__name__` (`<lambda>`) in tracebacks. PEP 8 says don't assign a lambda to a name; use `def`.

### Interview Questions

**Q1. What does "functions are first-class objects" mean?** ⭐
Functions are values like any other: they can be assigned to variables, stored in dicts and lists, passed as arguments, returned from other functions, and given attributes. That enables callbacks, dispatch tables, the strategy pattern without classes, closures and decorators.

**Q2. `map`/`filter` vs comprehensions: which do you prefer?**
Comprehensions are usually more readable, especially when a lambda would be needed. `map` is fine with an existing named function (`map(int, tokens)`) and is lazy in Python 3. Both are O(n); performance is similar, and `map` with a built-in C function can be slightly faster.

**Q3. What are the limitations of lambdas?**
A single expression only: no statements, `try`, assignments (except walrus) or annotations. They show up as `<lambda>` in tracebacks and are harder to test or reuse. Use them for short inline `key=` and callback arguments; otherwise use `def`. Also watch for late binding in loops (§17).

**Q4. What is a higher-order function? Give examples from the stdlib.**
A function that takes and/or returns functions. Examples: `sorted(key=)`, `map`, `filter`, `functools.reduce`, `functools.partial`, `functools.lru_cache`, and every decorator.

---

## 17. Closures & Late Binding

**What it is:** A **closure** is an inner function that remembers variables from its enclosing scope even after the outer function has returned. Python stores the captured variables in **cells** (`fn.__closure__`).

**Why it matters:** Closures are how decorators, factories and callbacks keep state, and the **late-binding pitfall** is a classic interview trick.

**Key concepts**

```python
def make_counter():
    count = 0
    def inc():
        nonlocal count          # needed to REBIND (reading alone doesn't need it)
        count += 1
        return count
    return inc

c = make_counter()
c(), c(), c()                    # (1, 2, 3)
c.__closure__[0].cell_contents   # 3 — the captured cell

# Late binding: closures capture VARIABLES (cells), not values
funcs = [lambda: i for i in range(3)]
[f() for f in funcs]             # [2, 2, 2]  — all see the final i

# Fix 1: a default argument binds the value at definition time
funcs = [lambda i=i: i for i in range(3)]
# Fix 2: a factory creates a new scope per value
def make(i): return lambda: i
funcs = [make(i) for i in range(3)]
# Fix 3: functools.partial
from functools import partial
funcs = [partial(lambda i: i, i) for i in range(3)]
[f() for f in funcs]             # [0, 1, 2]
```

- Closures vs classes: a closure is a lightweight object with one method. Use a class once you need several methods or inspectable state.
- Late binding also bites in `for` loops that register callbacks or `asyncio` tasks referencing the loop variable.

### Interview Questions

**Q1. What is a closure? How does Python implement it?** ⭐
It's a nested function that references variables from an enclosing function. When the inner function is created, Python stores references to those variables in **cell objects** (`__closure__`), so they outlive the outer call. That's what lets decorators hold the wrapped function, and factories hold configuration.

**Q2. Why does `[lambda: i for i in range(3)]` give `[2, 2, 2]`?** ⭐
Closures are **late-binding**: they look up `i` when they're *called*, not when they're created, and all three lambdas share the same variable, whose final value is 2. Fix it by binding the current value: a default argument (`lambda i=i: i`), a factory function, or `functools.partial`.

**Q3. When do you need `nonlocal`?**
Only when the inner function *rebinds* the captured name (`count += 1`). Reading it, or mutating a captured mutable object (`items.append(x)`), works without it. Without `nonlocal`, assignment makes the name local and you get `UnboundLocalError`.

---

## 18. Decorators

**What it is:** A decorator is a callable that takes a function (or class) and returns a replacement, usually a wrapper that adds behaviour. `@decorator` above `def f` is syntactic sugar for `f = decorator(f)`, and it runs **at definition (import) time**.

**Why it matters:** Decorators are everywhere: `@app.get`, `@pytest.fixture`, `@lru_cache`, `@login_required`, `@retry`, `@property`. "Write a decorator" is one of the most common Python coding questions.

**Key concepts**

```python
import functools, time, logging

# 1) Basic decorator
def timer(fn):
    @functools.wraps(fn)                 # keep __name__, __doc__, __wrapped__, signature
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return fn(*args, **kwargs)
        finally:
            logging.info("%s took %.3fs", fn.__name__, time.perf_counter() - start)
    return wrapper

@timer
def load_users(): ...

# 2) Decorator with arguments = a decorator FACTORY (three levels deep)
def retry(times=3, exceptions=(Exception,), delay=0.5):
    def decorator(fn):
        @functools.wraps(fn)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return fn(*args, **kwargs)
                except exceptions:
                    if attempt == times:
                        raise
                    time.sleep(delay * 2 ** (attempt - 1))   # exponential backoff
        return wrapper
    return decorator

@retry(times=3, exceptions=(ConnectionError,))
def fetch(): ...
# equivalent to: fetch = retry(times=3, exceptions=(ConnectionError,))(fetch)

# 3) Stacking: applied bottom-up, executed top-down
@timer          # 2nd applied -> outermost wrapper
@retry()        # 1st applied
def job(): ...  # job = timer(retry()(job))

# 4) Stateful / class-based decorator
class CountCalls:
    def __init__(self, fn):
        functools.update_wrapper(self, fn)
        self.fn, self.calls = fn, 0
    def __call__(self, *args, **kwargs):
        self.calls += 1
        return self.fn(*args, **kwargs)

# 5) Decorator that works for BOTH sync and async functions
import asyncio, inspect
def log_calls(fn):
    if inspect.iscoroutinefunction(fn):
        @functools.wraps(fn)
        async def async_wrapper(*args, **kwargs):
            logging.info("calling %s", fn.__name__)
            return await fn(*args, **kwargs)
        return async_wrapper
    @functools.wraps(fn)
    def sync_wrapper(*args, **kwargs):
        logging.info("calling %s", fn.__name__)
        return fn(*args, **kwargs)
    return sync_wrapper

# 6) Registration decorator (returns fn unchanged)
ROUTES = {}
def route(path):
    def register(fn):
        ROUTES[path] = fn
        return fn
    return register
```

**Built-in decorators to know:** `@property`, `@classmethod`, `@staticmethod`, `@functools.lru_cache` / `@cache`, `@functools.cached_property`, `@functools.singledispatch`, `@functools.wraps`, `@dataclass`, `@contextlib.contextmanager`, `@abc.abstractmethod`, `@typing.override` (3.12+).

### Interview Questions

**Q1. What is a decorator, and when does it run?** ⭐
A callable that takes a function and returns a (usually wrapped) function: `@d` above `def f` means `f = d(f)`. The decorator itself runs **once, at definition time** (usually at import); the wrapper runs on every call. That's why registration decorators like Flask's `@app.route` can populate a route table at import.

**Q2. Write a decorator that takes arguments.** ⭐
It needs three levels: an outer factory takes the arguments and returns the real decorator, which takes `fn` and returns the wrapper. See `retry` above. `@retry(times=3)` first *calls* `retry(times=3)`, then applies the result to the function.

**Q3. Why use `functools.wraps`?** ⭐
Without it, the wrapped function's `__name__`, `__doc__`, `__module__`, `__qualname__` and signature become the wrapper's. That breaks logging, debugging, `help()`, pickling, and frameworks that inspect signatures (FastAPI dependency injection, pytest fixtures, Click). `wraps` copies the metadata and sets `__wrapped__`, so `inspect.signature` follows through to the original.

**Q4. In what order are stacked decorators applied?**
Bottom-up when **applied** (the one closest to `def` wraps first) and top-down when **executed** (the outermost wrapper runs first). `@a @b def f` → `f = a(b(f))`. Order matters, e.g. put `@cache` outside `@timer` if you don't want cache hits timed.

**Q5. How would you write a decorator that works for both sync and async functions?**
Check `inspect.iscoroutinefunction(fn)` and return an `async def` wrapper that `await`s the function; otherwise return a normal wrapper. A single sync wrapper around a coroutine function would just return an un-awaited coroutine, so logic like timing or try/except would run around coroutine *creation*, not its execution.

**Q6. Decorator vs context manager: when do you use each?**
A decorator wraps a **whole function**: cross-cutting concerns applied at every call (auth, caching, retries, metrics). A context manager wraps a **block of code** inside a function: resources with a clear scope (locks, transactions, files, timers). `contextlib.ContextDecorator` lets one object serve both roles.

---

## 19. Functools & Itertools

**What it is:** `functools` holds higher-order tools for functions (caching, partial application, dispatch); `itertools` holds fast, lazy building blocks for iteration, implemented in C.

**Why it matters:** They replace hand-written loops with tested, efficient primitives, and memoization questions (`lru_cache`) come up constantly.

**Key concepts — functools**

```python
from functools import lru_cache, cache, partial, reduce, wraps, singledispatch, \
    total_ordering, cached_property, cmp_to_key

@lru_cache(maxsize=1024)            # memoize; args must be hashable
def fib(n): return n if n < 2 else fib(n - 1) + fib(n - 2)
fib.cache_info(); fib.cache_clear()

@cache                              # 3.9+: unbounded lru_cache(maxsize=None)
def get_settings(): ...

int_from_hex = partial(int, base=16)   # freeze some arguments
int_from_hex("ff")                     # 255

@singledispatch                     # dispatch on the type of the first argument
def to_json(obj): raise TypeError(obj)
@to_json.register
def _(obj: datetime): return obj.isoformat()

@total_ordering                     # define __eq__ + one of __lt__/__le__/... and get the rest
class Version:
    def __init__(self, major, minor): self.t = (major, minor)
    def __eq__(self, o): return self.t == o.t
    def __lt__(self, o): return self.t < o.t

class Report:
    @cached_property                # computed once per instance, then stored in __dict__
    def stats(self): return expensive()
```

**Key concepts — itertools**

```python
from itertools import (count, cycle, repeat, chain, islice, groupby, accumulate,
                       product, permutations, combinations, combinations_with_replacement,
                       zip_longest, takewhile, dropwhile, starmap, tee, pairwise, batched)

list(chain([1, 2], [3], [4, 5]))              # [1, 2, 3, 4, 5]
list(chain.from_iterable([[1, 2], [3]]))      # flatten one level
list(islice(count(10), 3))                    # [10, 11, 12] — slice any iterator
list(accumulate([1, 2, 3, 4]))                # [1, 3, 6, 10] — running sums
list(pairwise([1, 2, 3]))                     # [(1, 2), (2, 3)] (3.10+)
list(batched(range(7), 3))                    # [(0,1,2), (3,4,5), (6,)] (3.12+)
list(product("ab", repeat=2))                 # aa ab ba bb
list(permutations([1, 2, 3], 2))              # ordered pairs: 6 results
list(combinations([1, 2, 3], 2))              # unordered pairs: 3 results
list(zip_longest("ab", "xyz", fillvalue="-"))

# groupby groups CONSECUTIVE equal keys — sort first!
data = sorted(orders, key=lambda o: o.customer)
for customer, group in groupby(data, key=lambda o: o.customer):
    total = sum(o.amount for o in group)
```

### Interview Questions

**Q1. How does `lru_cache` work? What are its pitfalls?** ⭐
It wraps a function with a dict keyed on the call's arguments, plus a doubly linked list tracking recency, so lookups and evictions are O(1). Pitfalls:
1. Arguments must be hashable.
2. It's per process and not shared across workers.
3. On methods it caches `self` too, so instances are never garbage collected (a memory leak). Use `cached_property` or a per-instance cache instead.
4. Unbounded `@cache` can grow forever.
5. Returned mutable objects are shared between callers.
6. Only cache **pure** functions.

**Q2. Why must data be sorted before `itertools.groupby`?**
`groupby` only groups **consecutive** items with equal keys: it starts a new group whenever the key changes. Unsorted input gives repeated, fragmented groups. Sort by the same key first, or use a `defaultdict(list)` when order doesn't matter.

**Q3. What does `functools.partial` do, and where is it useful?**
It returns a new callable with some arguments pre-filled: `partial(int, base=2)`. Useful for callbacks that must take a fixed signature (`executor.submit`, `map`, button handlers), for fixing the late-binding problem, and for configuring a function once and reusing it.

**Q4. permutations vs combinations vs product?**
`permutations(it, r)` gives ordered arrangements without repetition (n!/(n−r)!). `combinations(it, r)` gives unordered selections without repetition (n choose r). `product(it, repeat=r)` gives the Cartesian product with repetition (nʳ). All are lazy, so huge result spaces don't allocate memory up front.

---

## 20. Recursion

**What it is:** A function calling itself, with a **base case** that stops the recursion and a **recursive case** that moves toward it.

**Why it matters:** It shows up in tree, graph and backtracking coding rounds. Python has specific limits worth knowing.

**Key concepts**

- Default recursion limit is about **1000** frames (`sys.getrecursionlimit()`); exceeding it raises `RecursionError`.
- **No tail-call optimization**: Guido rejected it to keep complete tracebacks. Deep recursion has to be rewritten iteratively with an explicit stack.
- `sys.setrecursionlimit()` can raise the limit, but the real C stack can still overflow and crash the interpreter.
- Memoize overlapping subproblems with `@cache`.

```python
from functools import cache

def factorial(n: int) -> int:
    if n <= 1:                       # base case
        return 1
    return n * factorial(n - 1)      # recursive case

@cache
def fib(n: int) -> int:              # O(2^n) -> O(n) with memoization
    return n if n < 2 else fib(n - 1) + fib(n - 2)

# Recursive DFS vs iterative DFS with an explicit stack (no depth limit)
def dfs_iter(root):
    stack, out = [root], []
    while stack:
        node = stack.pop()
        out.append(node.val)
        stack.extend(reversed(node.children))
    return out
```

### Interview Questions

**Q1. Does Python optimize tail recursion?** ⭐
No. CPython has no tail-call elimination: every call pushes a frame, and the default limit is about 1000. Guido rejected TCO because it destroys traceback information and Python favours explicit iteration. Convert deep recursion to a loop with an explicit stack or queue.

**Q2. What happens when you exceed the recursion limit? Should you just raise it?**
You get `RecursionError`. Raising the limit with `sys.setrecursionlimit` is risky because each Python frame also uses C stack, and too much can segfault the process. Raising it is acceptable for known, bounded depths (e.g. parsing a tree of depth 5,000 in a script); for production code on unbounded input, go iterative.

**Q3. How do you speed up a recursive function with overlapping subproblems?**
Memoize it with `@functools.cache` / `lru_cache`, which turns exponential Fibonacci-style recursion into O(n). Or convert to bottom-up dynamic programming, which avoids both the recursion depth and the cache overhead.

---

# PART 4 — OOP & THE DATA MODEL

---

## 21. Classes & Objects

**What it is:** A **class** is a blueprint (itself an object, an instance of `type`); an **object** is an instance of it. Attributes live in dicts: the instance's `__dict__` and the class's `__dict__`.

**Why it matters:** Class vs instance attributes, `self`, the three method types and `__new__` vs `__init__` are core OOP interview questions in every Python round.

**Key concepts**

```python
class Account:
    bank = "HDFC"                    # CLASS attribute — shared by all instances
    _count = 0

    def __init__(self, owner: str, balance: float = 0):
        self.owner = owner           # INSTANCE attributes — per object
        self._balance = balance      # "internal" by convention
        self.__pin = 1234            # name-mangled to _Account__pin
        Account._count += 1

    def deposit(self, amount):       # instance method: receives self
        self._balance += amount

    @classmethod
    def from_csv(cls, line: str):    # receives cls: alternative constructors, works with subclasses
        owner, bal = line.split(",")
        return cls(owner, float(bal))

    @staticmethod
    def validate(amount) -> bool:    # no self/cls: a namespaced utility function
        return amount > 0

    @property
    def balance(self):               # read-only computed attribute (see §25)
        return self._balance

    def __repr__(self):
        return f"Account(owner={self.owner!r}, balance={self._balance})"

acc = Account.from_csv("Asha,500")
acc.deposit(100)                     # really: Account.deposit(acc, 100)
acc.balance                          # 600
acc._Account__pin                    # 1234 — mangling is not security
```

*Attribute lookup order (simplified):* data descriptors on the class (e.g. `property`) → instance `__dict__` → class and parent attributes along the MRO (including methods) → `__getattr__` if defined → `AttributeError`.

*The shared mutable class attribute bug:*

```python
class Team:
    members = []                     # BUG: one list shared by ALL teams
    def add(self, m): self.members.append(m)

a, b = Team(), Team()
a.add("x"); b.members                # ['x']  <- leaked

class Team:
    def __init__(self):
        self.members = []            # FIX: create it per instance in __init__

# Assignment through an instance creates a shadowing instance attribute
class Dog: species = "canine"
d = Dog(); d.species = "wolf"        # Dog.species is still 'canine'
```

*`__new__` vs `__init__`:* `__new__(cls, ...)` **creates** and returns the instance (a static method on the class); `__init__(self, ...)` **initializes** it and must return `None`. `__init__` is only called if `__new__` returns an instance of `cls`.

```python
class UpperStr(str):                 # immutable types must be customized in __new__
    def __new__(cls, value):
        return super().__new__(cls, value.upper())
UpperStr("hi")                       # 'HI'
```

*Encapsulation in Python:* there are no `private`/`protected` keywords. The convention is "we're all consenting adults": `_x` means internal and `__x` means mangled. Real invariants are protected with properties and validation.

### Interview Questions

**Q1. Class attributes vs instance attributes?** ⭐
Class attributes are defined in the class body and shared by all instances: constants, counters, defaults. Instance attributes are set on `self` (usually in `__init__`) and belong to each object. Reading `obj.x` falls back to the class if the instance doesn't have `x`; **assigning** `obj.x = ...` creates an instance attribute that shadows the class one. The classic bug is a mutable class attribute (`members = []`) being shared by every instance.

**Q2. `@staticmethod` vs `@classmethod` vs instance method?** ⭐
- **Instance method**: gets `self`; works with instance state.
- **`@classmethod`**: gets `cls`; used for alternative constructors (`from_json`, `dict.fromkeys`) and factories that must respect subclasses (`cls(...)` builds the subclass).
- **`@staticmethod`**: gets neither; a utility function namespaced in the class. If it doesn't really belong to the class, use a module-level function instead.

**Q3. `__new__` vs `__init__`?** ⭐
`__new__` is the constructor: it allocates and returns the object. `__init__` is the initializer: it sets up the already-created object. You override `__new__` when subclassing immutable types (`str`, `int`, `tuple`), for singletons or instance caching, or in metaclasses. If `__new__` returns something that isn't an instance of the class, `__init__` is skipped.

**Q4. Does Python have private members?**
No enforced privacy. `_name` signals "internal, not part of the public API". `__name` triggers name mangling (`_Class__name`) to prevent accidental clashes with subclasses, but it's still accessible. To protect invariants, expose behaviour through methods and properties, validate in setters, and document the public API.

**Q5. What is `self`? Why is it explicit?**
`self` is the instance, passed automatically as the first argument when you call `obj.method()`, which is really `Class.method(obj)`. Being explicit ("explicit is better than implicit") makes it clear which attributes are instance state versus locals, and it's what makes methods plain functions stored on the class, which the descriptor protocol binds to instances (§25).

---

## 22. Inheritance, MRO & Super

**What it is:** A subclass inherits and can override its parents' behaviour. Python supports **multiple inheritance**. Method lookup follows the **MRO (Method Resolution Order)**, computed with **C3 linearization**.

**Why it matters:** MRO, `super()` and the diamond problem are senior favourites. Mixins power Django class-based views, DRF and much of the testing stack.

**Key concepts**

```python
class Base:
    def greet(self): print("Base")

class A(Base):
    def greet(self): print("A"); super().greet()

class B(Base):
    def greet(self): print("B"); super().greet()

class C(A, B):
    def greet(self): print("C"); super().greet()

C().greet()                 # C, A, B, Base — each class exactly once
C.__mro__                   # (C, A, B, Base, object)
```

- **`super()` means "the next class in the MRO of the instance's type"**, not "my parent". In the example, `A`'s `super()` goes to `B`, a class `A` knows nothing about. That's **cooperative multiple inheritance**.
- **C3 linearization rules:** a child comes before its parents; parents keep the order they're listed in; the order is consistent across the hierarchy. If no consistent order exists, class creation fails with `TypeError: Cannot create a consistent method resolution order`.
- Cooperative `__init__` must pass `**kwargs` along so that every class in the chain gets its arguments:

```python
class LoggingMixin:
    def __init__(self, *, log_level="INFO", **kwargs):
        super().__init__(**kwargs)           # keep the chain going
        self.log_level = log_level

class TimestampMixin:
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.created_at = time.time()

class Service(LoggingMixin, TimestampMixin):
    def __init__(self, name, **kwargs):
        super().__init__(**kwargs)
        self.name = name
```

- **Mixin**: a small class that adds one capability (`JSONSerializableMixin`, `LoginRequiredMixin`). It isn't meant to be instantiated alone and goes to the **left** of the main base class.
- **Composition over inheritance**: prefer "has-a" (holding a collaborator object) to deep "is-a" trees. Inheritance couples you to the parent's internals (the fragile base class problem).

```python
class Car:                                   # composition
    def __init__(self, engine: Engine):
        self.engine = engine
    def start(self): return self.engine.start()
```

### Interview Questions

**Q1. What is the MRO, and how is it computed?** ⭐
The MRO is the ordered list of classes Python searches for attributes and methods (`Class.__mro__` or `Class.mro()`). It's computed once at class creation with **C3 linearization**, which guarantees that subclasses come before base classes, the listed order of bases is preserved, and the result is monotonic. Conflicting orders raise `TypeError` at class definition.

**Q2. What does `super()` actually do?** ⭐
It returns a proxy that resolves attributes starting from the class **after** the current one in the instance's MRO. In a diamond (`C(A, B)`, both inheriting `Base`), `A`'s `super()` calls `B`, not `Base`, so each `__init__` runs exactly once. Calling `Base.__init__(self)` directly breaks this chain and can run a base initializer twice or skip a sibling entirely.

**Q3. How does Python solve the diamond problem?**
With C3 MRO plus cooperative `super()`: the shared base appears once, at the end of the linearization, and each class calls `super()` and forwards `**kwargs`. Every class in the diamond runs once, in a predictable order.

**Q4. What is a mixin? Mixins vs multiple inheritance vs composition?**
A mixin is a narrowly scoped class that adds reusable behaviour through multiple inheritance (e.g. Django's `LoginRequiredMixin`). It should be stateless or lightly stateful and cooperate through `super()`. Composition is preferred when behaviour can be delegated to a collaborator object: looser coupling and easier testing. Mixins fit when frameworks are designed around them.

**Q5. Composition vs inheritance: how do you choose?**
Use inheritance for a true **is-a** relationship where the subclass can stand in for the parent (Liskov substitution) and the hierarchy is shallow. Use composition for **has-a** or **uses-a**, to swap behaviour at runtime, or to avoid inheriting a large API you don't want. Python's duck typing and Protocols make composition especially easy: no shared base class is needed.

---

## 23. Polymorphism, Duck Typing, ABCs & Protocols

**What it is:**
- **Polymorphism**: the same interface, different behaviour.
- **Duck typing**: "if it walks like a duck…"; objects are used for what they *can do*, not what they *are*.
- **ABCs** (`abc.ABC`): *nominal* interfaces enforced by inheritance.
- **Protocols** (`typing.Protocol`): *structural* interfaces checked statically, with no inheritance needed.

**Why it matters:** These are Python's answer to Java-style interfaces, and a common senior design discussion: "How do you define an interface in Python?"

**Key concepts**

```python
# Duck typing — anything with .read() works
def load(source):
    return source.read()            # file, io.StringIO, HTTP response, ...

# ABC — nominal, enforced at instantiation time
from abc import ABC, abstractmethod
class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount: int) -> str: ...
    def refund(self, txn_id: str):  # concrete shared behaviour is allowed
        raise NotImplementedError

class Stripe(PaymentGateway):
    def charge(self, amount): return "txn_123"

# PaymentGateway()                 # TypeError: Can't instantiate abstract class
# A subclass missing charge() also fails at instantiation

# Protocol — structural (static duck typing)
from typing import Protocol, runtime_checkable
@runtime_checkable
class SupportsClose(Protocol):
    def close(self) -> None: ...

def shutdown(resource: SupportsClose) -> None:
    resource.close()                # any object with close() type-checks

isinstance(open("f.txt"), SupportsClose)   # True (runtime_checkable checks method presence only)

# Method overriding = runtime polymorphism
class Shape:
    def area(self) -> float: raise NotImplementedError
class Circle(Shape):
    def __init__(self, r): self.r = r
    def area(self): return 3.14159 * self.r ** 2
total = sum(s.area() for s in shapes)
```

`collections.abc` provides ready-made ABCs (`Iterable`, `Mapping`, `Sequence`, `Callable`, …). Implementing the abstract methods of `Mapping` gives you mixin methods like `keys`, `items` and `get` for free.

**EAFP vs LBYL** (see §30): duck typing pairs with *"easier to ask forgiveness than permission"*. Just call the method and catch `AttributeError`/`TypeError`, rather than checking types first.

### Interview Questions

**Q1. What is duck typing?** ⭐
Using an object based on the methods and attributes it supports, not its class. If it has `.read()`, treat it as a readable source. It makes code flexible: tests can pass fakes, and `StringIO` works where a file is expected. The risk is runtime errors when an object doesn't quite fit; type hints with Protocols catch that statically.

**Q2. How do you define an interface in Python? ABC vs Protocol?** ⭐
- **ABC**: nominal. Implementers must inherit and are checked at instantiation (abstract methods must be overridden). It can also provide shared concrete methods. Good for a plugin hierarchy you own.
- **Protocol**: structural. Any class with matching methods satisfies it, with no inheritance, checked by mypy/Pyright. It's best for dependency injection and third-party types you can't modify. `@runtime_checkable` allows `isinstance`, but it only checks that methods *exist*, not their signatures.

**Q3. Does Python support method overloading and overriding?**
Overriding: yes. A subclass redefines a method, resolved at runtime via the MRO, and can call the parent with `super()`. Overloading by signature: no (see §15); use defaults, `*args` or `singledispatch`. Operator overloading: yes, via dunder methods (§24).

**Q4. What's the difference between abstraction and encapsulation in Python?**
Abstraction exposes only *what* an object does (ABCs, Protocols, clean public methods). Encapsulation hides *how* it stores state (the `_internal` convention, properties guarding invariants). Python enforces neither strictly; it relies on conventions, ABC instantiation checks and type checkers.

---

## 24. The Data Model & Dunder Methods

**What it is:** The **Python data model** is the set of special ("dunder") methods that let objects plug into language syntax and built-ins: `len(x)` → `x.__len__()`, `x[k]` → `__getitem__`, `a + b` → `__add__`, `for` → `__iter__`, `with` → `__enter__/__exit__`.

**Why it matters:** It's what makes custom classes feel built-in, how NumPy, pandas and ORMs overload operators, and a strong senior-level signal.

**Key concepts**

| Category | Methods | Triggered by |
|---|---|---|
| Construction | `__new__`, `__init__`, `__del__` | `Cls()`, garbage collection |
| Representation | `__repr__`, `__str__`, `__format__` | `repr()`, `print()`/`str()`, f-strings |
| Comparison | `__eq__`, `__ne__`, `__lt__`, `__le__`, `__gt__`, `__ge__` | `==`, `<`, … |
| Hashing / truth | `__hash__`, `__bool__` | `hash()`, dict keys, `if obj:` |
| Arithmetic | `__add__`, `__sub__`, `__mul__`, `__matmul__`, `__truediv__`, … | `+ - * @ /` |
| Reflected / in-place | `__radd__`, `__iadd__`, … | `1 + obj`, `obj += x` |
| Containers | `__len__`, `__getitem__`, `__setitem__`, `__delitem__`, `__contains__`, `__iter__`, `__reversed__` | `len`, `[]`, `in`, `for` |
| Callable | `__call__` | `obj()` |
| Attribute access | `__getattr__`, `__getattribute__`, `__setattr__`, `__delattr__`, `__dir__` | `obj.x` |
| Context managers | `__enter__`, `__exit__` (`__aenter__`, `__aexit__`) | `with` / `async with` |
| Descriptors | `__get__`, `__set__`, `__delete__`, `__set_name__` | attribute access (§25) |
| Class hooks | `__init_subclass__`, `__class_getitem__` | subclassing, `Cls[int]` |
| Async | `__await__`, `__aiter__`, `__anext__` | `await`, `async for` |

```python
from functools import total_ordering

@total_ordering
class Money:
    __slots__ = ("amount", "currency")

    def __init__(self, amount: int, currency: str = "INR"):
        self.amount, self.currency = amount, currency

    def __repr__(self):                          # unambiguous, for developers
        return f"Money({self.amount!r}, {self.currency!r})"

    def __str__(self):                           # readable, for users
        return f"{self.currency} {self.amount / 100:.2f}"

    def __eq__(self, other):
        if not isinstance(other, Money):
            return NotImplemented                # let Python try the other side
        return (self.amount, self.currency) == (other.amount, other.currency)

    def __hash__(self):
        return hash((self.amount, self.currency))

    def __lt__(self, other):
        if not isinstance(other, Money):
            return NotImplemented
        return self.amount < other.amount

    def __add__(self, other):
        if not isinstance(other, Money) or other.currency != self.currency:
            return NotImplemented
        return Money(self.amount + other.amount, self.currency)

    def __radd__(self, other):                   # makes sum([...]) work (start value is 0)
        return self if other == 0 else self.__add__(other)

    def __bool__(self):
        return self.amount != 0

sum([Money(100), Money(250)])                    # Money(350, 'INR')

# A sequence-like container: __len__ + __getitem__ is enough for iteration, `in` and slicing
class Deck:
    def __init__(self): self._cards = [f"{r}{s}" for s in "SHDC" for r in "A23456789TJQK"]
    def __len__(self): return len(self._cards)
    def __getitem__(self, i): return self._cards[i]

# __getattr__ (a fallback) vs __getattribute__ (every access)
class Proxy:
    def __init__(self, target): self._target = target
    def __getattr__(self, name):                 # called only when normal lookup FAILS
        return getattr(self._target, name)
```

### Interview Questions

**Q1. What are dunder methods? Why do they matter?** ⭐
Special methods Python calls implicitly to implement syntax and built-ins (`len`, `+`, `[]`, `in`, `for`, `with`, `==`, `hash`). Implementing them makes custom types behave like built-ins: a `Money` that adds with `+`, a `Deck` that works with `len()`, slicing and `random.choice`. It's how NumPy overloads `@` and pandas overloads `df[...]`.

**Q2. `__repr__` vs `__str__`?** ⭐
`__repr__` is the unambiguous, developer-facing representation, ideally valid Python (`Money(100, 'INR')`), used by the REPL, debuggers, logs and containers. `__str__` is the readable, user-facing form used by `print()` and `str()`. If only `__repr__` exists, `str()` falls back to it, so always implement `__repr__` first.

**Q3. Why return `NotImplemented` instead of raising, or returning `False`?**
`NotImplemented` tells Python "I don't know how to handle this type". Python then tries the reflected operation on the other operand (`other.__radd__`, `other.__eq__`) before raising `TypeError` (or, for `==`, falling back to identity). Returning `False` would wrongly short-circuit that negotiation. Note that `NotImplemented` (a value) is different from `NotImplementedError` (an exception for abstract methods).

**Q4. `__getattr__` vs `__getattribute__`?** ⭐
`__getattribute__` runs on **every** attribute access; overriding it is powerful but dangerous, since accessing `self.x` inside it recurses infinitely unless you go through `object.__getattribute__`. `__getattr__` runs only **after normal lookup fails**, which makes it the safe hook for proxies, lazy attributes and delegation.

**Q5. How do you make a custom class iterable? Work with `len()`? Callable?**
Iterable: implement `__iter__` (ideally as a generator function), or `__getitem__` with integer indices as a legacy fallback. `len()`: implement `__len__`, which must return an int ≥ 0. Callable: implement `__call__`, useful for stateful functions, class-based decorators and strategy objects.

**Q6. What is `__call__` used for?**
It makes instances callable like functions while keeping state between calls: class-based decorators, configurable strategies (`Validator(min=3)(value)`), ML models (`model(x)` in PyTorch calls `forward` via `__call__`), and memoizers.

---

## 25. Properties & Descriptors

**What it is:** A **descriptor** is any object defining `__get__`, `__set__` or `__delete__` that lives as a **class attribute**; it intercepts attribute access on instances. `property`, `classmethod`, `staticmethod`, functions (which become bound methods), `cached_property`, `__slots__` and ORM fields are all descriptors.

**Why it matters:** Properties are everyday Python. Descriptors explain *how* methods bind to `self` and how Django and SQLAlchemy fields work, which is a top-tier senior question.

**Key concepts**

```python
# property: computed and validated attributes with attribute syntax
class Temperature:
    def __init__(self, celsius: float):
        self.celsius = celsius               # goes through the setter

    @property
    def celsius(self) -> float:
        return self._celsius

    @celsius.setter
    def celsius(self, value: float):
        if value < -273.15:
            raise ValueError("below absolute zero")
        self._celsius = value

    @property
    def fahrenheit(self) -> float:           # read-only, computed
        return self._celsius * 9 / 5 + 32

t = Temperature(25); t.fahrenheit            # 77.0
# t.fahrenheit = 0                           # AttributeError (no setter)

# A custom reusable descriptor (how ORM fields work)
class Positive:
    def __set_name__(self, owner, name):     # 3.6+: learns its attribute name
        self.private = f"_{name}"
    def __get__(self, obj, objtype=None):
        if obj is None:                      # accessed on the class itself
            return self
        return getattr(obj, self.private)
    def __set__(self, obj, value):
        if value <= 0:
            raise ValueError(f"{self.private[1:]} must be positive")
        setattr(obj, self.private, value)

class Order:
    quantity = Positive()                    # one descriptor, reused across fields and classes
    price = Positive()
    def __init__(self, quantity, price):
        self.quantity, self.price = quantity, price
```

**Lookup precedence:**
1. **Data descriptors** (define `__set__` or `__delete__`, e.g. `property`)
2. Instance `__dict__`
3. **Non-data descriptors** (only `__get__`, e.g. functions, `cached_property`) and other class attributes

That's why you can't shadow a property by assigning to the instance, but `cached_property` *does* store its value in the instance dict, so later lookups skip the descriptor entirely.

**How methods bind:** a function is a non-data descriptor. `obj.method` calls `function.__get__(obj, type(obj))`, which returns a **bound method** with `self` pre-filled.

### Interview Questions

**Q1. Why use `@property` instead of getters and setters?** ⭐
Start with plain public attributes. If you later need validation, computation or lazy loading, switch to a property **without changing the public API**: callers still write `obj.x`. Java-style `get_x()` / `set_x()` is unidiomatic in Python. Properties suit cheap operations; expensive work should be an explicit method (or `cached_property`) so callers aren't surprised.

**Q2. What is a descriptor? Give real examples.** ⭐
An object with `__get__`/`__set__`/`__delete__` stored as a class attribute that controls access to that attribute on instances. Examples: `property`, `classmethod`, `staticmethod`, plain functions (bound methods), `functools.cached_property`, `__slots__` members, Django `models.CharField`, SQLAlchemy `Column`/`mapped_column`, Pydantic fields. Custom descriptors give reusable validation across many attributes, which is DRY-er than repeating properties.

**Q3. Data vs non-data descriptors: why does it matter?**
Data descriptors (with `__set__`/`__delete__`) take priority over the instance `__dict__`; non-data descriptors (only `__get__`) are overridden by it. So properties can't be shadowed by instance assignment, while methods and `cached_property` can. `cached_property` relies on this: the first access computes the value and stores it in `__dict__`, and later accesses are plain dict hits.

**Q4. How does `self` get passed to methods?**
Functions implement `__get__`. Accessing `obj.method` triggers `Class.__dict__['method'].__get__(obj, Class)`, which returns a bound method that inserts `obj` as the first argument when called. `classmethod` does the same with the class, and `staticmethod` returns the function unchanged.

---

## 26. Dataclasses, NamedTuple, Slots & Enum

**What it is:** Tools that remove boilerplate for data-holding classes:
- `@dataclass` (3.7+) generates `__init__`, `__repr__`, `__eq__` and optionally ordering, hashing and slots.
- `NamedTuple` is an immutable tuple with named fields.
- `__slots__` declares fixed attributes with no per-instance `__dict__`.
- `Enum` provides named constants.

**Why it matters:** These are the idiomatic choice for DTOs, value objects and config; you're expected to know the trade-offs versus regular classes and Pydantic.

**Key concepts**

```python
from dataclasses import dataclass, field, asdict, replace
from typing import NamedTuple
from enum import Enum, IntEnum, StrEnum, auto

@dataclass
class Order:
    id: str
    items: list[str] = field(default_factory=list)   # NEVER items: list = [] (ValueError)
    total: float = 0.0
    _cache: dict = field(default_factory=dict, repr=False, compare=False)

    def __post_init__(self):                          # validation or derived fields
        if self.total < 0:
            raise ValueError("total must be >= 0")

@dataclass(frozen=True, slots=True, order=True)       # immutable + hashable + compact + sortable
class Version:
    major: int
    minor: int = 0

Version(1, 2) < Version(1, 3)        # True (compares fields as a tuple)
v2 = replace(Version(1, 2), minor=5) # copy with changes (frozen-friendly)
asdict(Order("o1"))                  # {'id': 'o1', 'items': [], 'total': 0.0, '_cache': {}}

@dataclass(kw_only=True)             # 3.10+: all fields keyword-only
class Config:
    host: str
    port: int = 8080

class Point(NamedTuple):
    x: float
    y: float = 0.0
p = Point(1, 2); x, y = p; p._replace(x=5); p._asdict()

# __slots__ on a regular class
class Pixel:
    __slots__ = ("x", "y", "color")  # no __dict__: less memory, faster attribute access
    def __init__(self, x, y, color): self.x, self.y, self.color = x, y, color
# Pixel(1, 2, "red").z = 3           # AttributeError

class Status(StrEnum):               # 3.11+; members are real str values
    PENDING = auto()                 # 'pending'
    PAID = auto()
Status.PAID == "paid"                # True
Status("pending")                    # lookup by value
[s.name for s in Status]
```

**Choosing**

| Need | Use |
|---|---|
| Mutable record with methods and defaults | `@dataclass` |
| Immutable, hashable value object | `@dataclass(frozen=True)` |
| Millions of instances | `slots=True` / `__slots__` |
| Tuple behaviour (unpacking, indexing), lightweight | `NamedTuple` |
| Validation, parsing and coercion of external data (API, config, LLM output) | **Pydantic** `BaseModel` |
| Fixed set of named constants | `Enum` / `StrEnum` / `IntEnum` |
| Fully custom behaviour | Regular class |

### Interview Questions

**Q1. What does `@dataclass` generate? `frozen`, `slots`, `order`?** ⭐
By default it generates `__init__`, `__repr__` and `__eq__` from the annotated fields. `order=True` adds `<`, `<=`, `>`, `>=` (fields compared as a tuple). `frozen=True` blocks assignment (`FrozenInstanceError`) and, combined with `eq`, generates `__hash__`, so instances can be dict keys. `slots=True` (3.10+) generates `__slots__`. `kw_only=True` makes every field keyword-only. Type hints are **not validated** at runtime.

**Q2. Why `field(default_factory=list)`?**
Same root cause as mutable default arguments: a literal `[]` default would be shared by all instances. Dataclasses refuse unhashable defaults like `list`, `dict` and `set` with a `ValueError` and require `default_factory`, which creates a fresh object per instance.

**Q3. dataclass vs NamedTuple vs Pydantic?** ⭐
- **dataclass**: a general-purpose internal data structure; mutable by default; no validation; fast.
- **NamedTuple**: an immutable tuple with all tuple behaviour (indexing, unpacking, equal to plain tuples, which can surprise you); lightweight.
- **Pydantic**: runtime **validation, coercion and serialization** of untrusted input (requests, env config, LLM output) with JSON Schema generation; a bit more overhead.

Rule of thumb: Pydantic at the boundaries, dataclasses inside the domain.

**Q4. What does `__slots__` do? What are the trade-offs?** ⭐
It replaces the per-instance `__dict__` with fixed slots (implemented as descriptors). That cuts memory per object, often by 40–60% for small objects, speeds up attribute access a little, and blocks typo-created attributes. Trade-offs: no dynamic attributes; no `__weakref__` unless you list it; every class in an inheritance chain needs its own `__slots__` for the savings to hold; and pickling or multiple inheritance with slots needs care. Use it for classes with millions of instances.

**Q5. Why use `Enum` instead of string or int constants?**
Type safety (a closed set of valid values), readable names in logs and debuggers, iteration over members, identity comparison (`status is Status.PAID`), and type checkers can verify exhaustive `match` statements. `StrEnum`/`IntEnum` keep interop with APIs and databases that expect raw strings or ints.

---

## 27. Metaclasses & Class Creation Hooks

**What it is:** A **metaclass** is "the class of a class". Classes are instances of `type` by default. A `class` statement effectively calls `type(name, bases, namespace)`; a custom metaclass (a subclass of `type`) hooks into **class creation**. Lighter alternatives are `__init_subclass__` (3.6+) and class decorators.

**Why it matters:** It's asked to test depth. The senior answer is knowing *when not* to use metaclasses.

**Key concepts**

```python
# Classes are objects created by type
Dynamic = type("Dynamic", (object,), {"greet": lambda self: "hi"})
Dynamic().greet()                    # 'hi'
type(Dynamic), type(type)            # (<class 'type'>, <class 'type'>)

# A metaclass: validate or register classes at definition time
class PluginMeta(type):
    registry: dict[str, type] = {}
    def __new__(mcls, name, bases, ns):
        cls = super().__new__(mcls, name, bases, ns)
        if bases:                                  # skip the abstract base itself
            if "run" not in ns:
                raise TypeError(f"{name} must define run()")
            mcls.registry[name.lower()] = cls
        return cls

class Plugin(metaclass=PluginMeta): ...
class Exporter(Plugin):
    def run(self): ...
PluginMeta.registry                  # {'exporter': <class 'Exporter'>}

# Usually enough: __init_subclass__
class Plugin:
    registry: dict[str, type] = {}
    def __init_subclass__(cls, *, key: str, **kwargs):
        super().__init_subclass__(**kwargs)
        Plugin.registry[key] = cls

class CsvExporter(Plugin, key="csv"): ...

# Singleton via a metaclass (__call__ controls instantiation)
class SingletonMeta(type):
    _instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]
```

**Class creation order:** the metaclass's `__prepare__` (namespace) → the class body executes → metaclass `__new__` / `__init__` → `__set_name__` on descriptors → parent's `__init_subclass__` → class decorators.

### Interview Questions

**Q1. What is a metaclass?** ⭐
The class of a class, which controls how classes are created. `type` is the default metaclass. A custom metaclass can modify the class namespace, validate definitions, auto-register subclasses or change instantiation (`__call__`). ORMs (Django `Model`, SQLAlchemy declarative), ABCs (`ABCMeta`) and `Enum` (`EnumMeta`) use metaclasses.

**Q2. When should you use a metaclass?**
Rarely. Reach for, in order: a class decorator, `__init_subclass__`, descriptors with `__set_name__`, and only then a metaclass. Metaclasses are hard to read, and **metaclass conflicts** arise when combining bases with different metaclasses. They're justified for framework-level magic (custom namespaces via `__prepare__`, controlling `isinstance` through `__instancecheck__`), not application code. Saying this explicitly signals seniority.

**Q3. What is `__init_subclass__`?**
A classmethod hook (3.6+) on a base class that runs whenever a subclass is defined, and it can take keyword arguments from the class statement (`class X(Base, key="x")`). It covers most metaclass use cases (registration, validation, configuration) with ordinary inheritance and no metaclass conflicts.

---

# PART 5 — ITERATION, ERRORS, I/O, MODULES & TYPING

---

## 28. Iterators & Generators

**What it is:**
- An **iterable** is anything you can loop over: it has `__iter__`, which returns an iterator.
- An **iterator** produces values one at a time: `__next__` returns the next value or raises `StopIteration`, and `__iter__` returns `self`.
- A **generator** is the easy way to write an iterator: a function using `yield`, whose execution pauses and resumes while keeping its local state.

**Why it matters:** Generators enable streaming over huge data with O(1) memory, and they're the historical foundation of `async/await`. Top-5 interview topic.

**Key concepts**

```python
# How a for loop really works
it = iter([10, 20])        # calls __iter__
next(it), next(it)         # (10, 20) — calls __next__
# next(it)                 # StopIteration -> the for loop ends quietly

# Iterable vs iterator: an iterable can be looped over many times; an iterator is single-use
nums = [1, 2, 3]           # iterable (iter() returns a NEW iterator each time)
gen = (x for x in nums)    # iterator
list(gen), list(gen)       # ([1, 2, 3], []) — exhausted!

# Class-based iterator
class Countdown:
    def __init__(self, start): self.current = start
    def __iter__(self): return self
    def __next__(self):
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

# The same thing as a generator function (much simpler)
def countdown(start):
    while start > 0:
        yield start        # pause here, hand back a value, resume on the next next()
        start -= 1

# A re-iterable container: make __iter__ a generator
class Batch:
    def __init__(self, items): self.items = items
    def __iter__(self):
        yield from self.items          # each loop gets a fresh generator

# Streaming pipeline: constant memory even for a 50 GB file
def read_lines(path):
    with open(path, encoding="utf-8") as f:
        yield from f                   # file objects are themselves line iterators

def errors(lines):
    return (ln for ln in lines if "ERROR" in ln)

def parse(lines):
    for ln in lines:
        ts, level, msg = ln.split(" ", 2)
        yield {"ts": ts, "msg": msg.strip()}

for record in parse(errors(read_lines("app.log"))):
    ...

# Infinite and lazy
from itertools import count, islice
evens = (n for n in count() if n % 2 == 0)
list(islice(evens, 5))             # [0, 2, 4, 6, 8]

# Paginated API as a generator
def fetch_all(client):
    page = 1
    while True:
        data = client.get("/items", params={"page": page}).json()
        if not data["items"]:
            return                     # return inside a generator = StopIteration
        yield from data["items"]
        page += 1
```

*Generator methods (coroutine features):*

```python
def running_avg():
    total = count = 0
    avg = None
    while True:
        value = yield avg          # receives the value passed to send()
        total += value; count += 1
        avg = total / count

g = running_avg(); next(g)         # "prime" it: advance to the first yield
g.send(10), g.send(20)             # (10.0, 15.0)
g.close()                          # raises GeneratorExit inside the generator
```

- `yield from sub` delegates to a sub-generator, forwarding `send`, `throw` and `close`, and evaluates to the sub-generator's `return` value.
- PEP 479: a `StopIteration` raised *inside* a generator body becomes `RuntimeError` (so bugs aren't silently swallowed).
- Generators are lazy: errors appear at iteration time, not at creation time.

### Interview Questions

**Q1. Iterable vs iterator vs generator?** ⭐
- **Iterable**: has `__iter__`, which returns an iterator; can be iterated many times (list, dict, file path…).
- **Iterator**: has `__next__` and `__iter__` returning itself; single pass and stateful.
- **Generator**: an iterator created by a `yield` function or a generator expression, which keeps its frame (local variables and instruction pointer) suspended between `next()` calls.

Every generator is an iterator; every iterator is iterable; not every iterable is an iterator.

**Q2. How do generators save memory? What's the trade-off?** ⭐
They produce one value at a time on demand, so memory stays O(1) regardless of how long the sequence is. That's how you process 50 GB files, infinite streams and paginated APIs. Trade-offs: single pass only, no `len()` or indexing, slightly slower per item than a list, and errors surface lazily during iteration.

**Q3. What happens when you call a generator function?**
Nothing in its body runs. It returns a generator object. The body runs up to the first `yield` on the first `next()`, then pauses. Each later `next()` resumes from that point. When the function returns, `StopIteration` is raised (carrying the return value).

**Q4. What do `yield from`, `send()` and `close()` do?**
`yield from it` delegates iteration to another iterable or generator, transparently forwarding `send`/`throw`/`close`, and returns the sub-generator's return value. `send(v)` resumes the generator and makes the paused `yield` expression evaluate to `v` (prime it with `next()` first). `close()` raises `GeneratorExit` at the paused point so `finally` blocks can clean up. These features made generators into coroutines, the basis of pre-3.5 asyncio.

**Q5. Why does iterating a generator twice return nothing the second time?**
A generator is an iterator, and iterators are exhausted after one pass. If you need several passes, materialize it into a list, recreate the generator, or expose a class whose `__iter__` returns a fresh generator each time. `itertools.tee` gives multiple independent iterators, but buffers data internally.

**Q6. How would you process a file too large to fit in memory?**
Iterate it line by line (`for line in f` is lazy), or read fixed-size chunks with `f.read(1 << 20)` for binary data. Chain generator stages (read → filter → parse → aggregate) so only one record is in memory at a time. For columnar or tabular data, use streaming or lazy engines (Polars `scan_csv`, DuckDB, `pyarrow.dataset`).

---

## 29. Context Managers

**What it is:** An object that defines setup and teardown around a block via `__enter__` / `__exit__`, used with the `with` statement. Teardown runs **even if the block raises**. It replaces `try/finally` boilerplate.

**Why it matters:** It's how resources are managed: files, locks, DB transactions, connections, temporary state, timing. "Write a context manager" is a common question.

**Key concepts**

```python
# The with statement, roughly:
#   mgr = EXPR; value = mgr.__enter__()
#   try: BODY
#   except: if not mgr.__exit__(*sys.exc_info()): raise
#   else: mgr.__exit__(None, None, None)

# Class-based: full control
class Transaction:
    def __init__(self, conn): self.conn = conn
    def __enter__(self):
        self.conn.begin()
        return self.conn                   # bound to the "as" target
    def __exit__(self, exc_type, exc, tb):
        if exc_type is None:
            self.conn.commit()
        else:
            self.conn.rollback()
        return False                       # False/None = re-raise; True = SUPPRESS the exception

# Generator-based with contextlib (preferred for simple cases)
from contextlib import contextmanager
import time

@contextmanager
def timer(label):
    start = time.perf_counter()
    try:
        yield                              # the with-block runs here
    finally:                               # needed so cleanup runs on exceptions
        print(f"{label}: {time.perf_counter() - start:.3f}s")

with timer("query"):
    run_query()

# Several at once, with parentheses (3.10+)
with (
    open("in.txt") as src,
    open("out.txt", "w") as dst,
):
    dst.write(src.read())

# Useful contextlib tools
from contextlib import suppress, ExitStack, nullcontext, redirect_stdout, chdir
with suppress(FileNotFoundError):          # a clean "ignore this specific error"
    os.remove("tmp.txt")

with ExitStack() as stack:                 # a dynamic number of resources
    files = [stack.enter_context(open(p)) for p in paths]

cm = lock if threaded else nullcontext()   # optional context manager
with cm: ...

# Async context managers: __aenter__ / __aexit__
from contextlib import asynccontextmanager
@asynccontextmanager
async def db_session():
    session = await pool.acquire()
    try:
        yield session
    finally:
        await pool.release(session)

async with db_session() as s: ...
```

### Interview Questions

**Q1. What is a context manager, and why use one?** ⭐
An object implementing `__enter__` and `__exit__` so `with` can guarantee cleanup, even on exceptions or early returns. It makes resource handling correct by construction: files closed, locks released, transactions committed or rolled back, and temporary state restored. It's cleaner and less error-prone than a hand-written `try/finally`.

**Q2. Write a context manager two ways.** ⭐
A class with `__enter__`/`__exit__`, or a generator function decorated with `@contextlib.contextmanager` (code before `yield` is setup, after it is teardown). Wrap the `yield` in `try/finally` so teardown still runs when the block raises. See `Transaction` and `timer` above.

**Q3. What do `__exit__`'s arguments and return value mean?**
It receives `(exc_type, exc_value, traceback)`, all `None` if the block succeeded. Returning a truthy value **suppresses** the exception; returning `False`/`None` lets it propagate. Suppressing by accident (e.g. `return True` everywhere) hides real bugs, so only suppress specific, expected exceptions, or use `contextlib.suppress`.

**Q4. What is `ExitStack` for?**
Managing a variable or conditional number of context managers (open N files, acquire optional locks) and registering arbitrary cleanup callbacks (`stack.callback(fn)`). It unwinds everything in LIFO order, even if acquiring a later resource fails partway through.

**Q5. How do async context managers differ?**
They define `__aenter__`/`__aexit__` (coroutines) and are used with `async with`, so setup and teardown can `await` I/O such as acquiring a pooled connection or closing an HTTP session. Using a sync `with` on an async resource (or forgetting `async with`) leaks connections.

---

## 30. Exceptions & Error Handling

**What it is:** Python signals errors by raising exception objects that **propagate up the call stack** until caught. `try / except / else / finally` handles them. Every exception derives from `BaseException`; application errors should derive from `Exception`.

**Why it matters:** Production judgment: catch specific exceptions, don't swallow errors, preserve tracebacks, design exception hierarchies, and know EAFP vs LBYL.

**Key concepts**

```
BaseException
 ├── SystemExit, KeyboardInterrupt, GeneratorExit      ← don't catch these casually
 ├── BaseExceptionGroup → ExceptionGroup (3.11+)
 └── Exception
      ├── ArithmeticError → ZeroDivisionError, OverflowError
      ├── LookupError → KeyError, IndexError
      ├── OSError → FileNotFoundError, PermissionError, TimeoutError, ConnectionError
      ├── ValueError → UnicodeError
      ├── TypeError, AttributeError, NameError → UnboundLocalError
      ├── RuntimeError → RecursionError, NotImplementedError
      ├── ImportError → ModuleNotFoundError
      └── StopIteration, StopAsyncIteration
```

```python
def load_config(path):
    try:
        f = open(path, encoding="utf-8")
    except FileNotFoundError:                     # specific first
        return {}
    except (PermissionError, IsADirectoryError) as e:
        raise ConfigError(f"cannot read {path}") from e   # translate + chain
    else:                                         # runs only if try raised nothing
        with f:
            return json.load(f)
    finally:                                      # ALWAYS runs (cleanup, metrics)
        log.debug("load_config done")

# Custom exception hierarchy for a domain
class PaymentError(Exception):
    """Base class: callers can catch all payment failures."""

class InsufficientFunds(PaymentError):
    def __init__(self, account_id: str, needed: int):
        super().__init__(f"account {account_id} needs {needed}")
        self.account_id, self.needed = account_id, needed

class GatewayTimeout(PaymentError): ...

# Re-raise while keeping the traceback
try:
    charge()
except GatewayTimeout:
    log.exception("charge failed")               # logs the full traceback
    raise                                         # bare raise re-raises the same exception

# Chaining
raise ServiceError("x") from err                  # explicit cause: __cause__
raise ServiceError("x") from None                 # hide the context (use sparingly)

# Exception groups (3.11+): several errors at once, e.g. from TaskGroup
async def sync_all():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(sync_users())
            tg.create_task(sync_orders())
    except* ConnectionError as eg:            # handles only the ConnectionErrors in the group
        for e in eg.exceptions:
            log.warning("sync failed: %s", e)
    except* ValueError:
        log.error("bad data")

# Adding context (3.11+)
try:
    process(row)
except ValueError as e:
    e.add_note(f"while processing row {row_id}")
    raise

# EAFP vs LBYL
value = d[key] if key in d else default           # LBYL: look before you leap
try:                                              # EAFP: easier to ask forgiveness
    value = d[key]
except KeyError:
    value = default
value = d.get(key, default)                       # the best option here
```

**Best practices:**
- Never use a bare `except:` or `except BaseException:`. They catch `KeyboardInterrupt`/`SystemExit` and stop clean shutdown.
- Don't write `except Exception: pass`. Silent failures are the worst kind.
- Catch at the level that can actually **handle** the error (retry, fall back, translate), and let the rest propagate.
- Translate low-level errors into domain errors at boundaries (DB → `RepositoryError` → HTTP 503) and keep the cause with `from`.
- Use `logger.exception(...)` inside `except` blocks to capture the traceback.
- `assert` is for internal invariants, not input validation; it's stripped under `python -O`.
- Since 3.11, entering `try` costs nothing ("zero-cost exceptions"); raising still has a cost, so don't use exceptions for hot-path control flow.

### Interview Questions

**Q1. Explain `try / except / else / finally`.** ⭐
`try` holds code that may fail. `except` blocks handle specific exception types (the first match wins, so list specific before general). `else` runs only if no exception occurred, which keeps the "success path" out of the try block so its errors aren't caught by mistake. `finally` always runs (success, exception, `return`, `break`) for cleanup. A `return` inside `finally` overrides earlier returns and swallows exceptions; avoid it.

**Q2. Why is a bare `except:` dangerous?** ⭐
It catches everything derived from `BaseException`, including `KeyboardInterrupt`, `SystemExit` and `GeneratorExit`. That blocks Ctrl-C and graceful shutdown signals, and it hides genuine bugs (`NameError`, `TypeError`). Catch the specific exceptions you can handle, or at most `Exception` at a top-level boundary, and always log or re-raise.

**Q3. `raise` vs `raise e` vs `raise New(...) from e`?** ⭐
A bare `raise` re-raises the current exception with its original traceback. `raise e` re-raises it but adds the current line to the traceback. `raise New(...) from e` raises a new (usually domain) exception with `__cause__ = e`, so the traceback shows "The above exception was the direct cause of…". Without `from`, Python still records `__context__` ("During handling of the above exception, another exception occurred"). `from None` suppresses that context.

**Q4. How do you design custom exceptions for a service?**
Create one base exception per package or domain (`PaymentError(Exception)`), with specific subclasses for conditions callers handle differently (`InsufficientFunds`, `GatewayTimeout`). Attach structured data (IDs, retryable flags) as attributes. Map them to HTTP status codes or retries at the edge. Don't create an exception class per function; create one per distinct handling strategy.

**Q5. EAFP vs LBYL: which is Pythonic?** ⭐
EAFP ("try it, catch the failure") is idiomatic. It avoids race conditions: with LBYL, a file could be deleted between `os.path.exists` and `open` (TOCTOU). It's also fast when failures are rare. LBYL is better when failure is common and checking is cheap, or when the operation has side effects that can't be undone. Often a dedicated API beats both: `dict.get`, `getattr(obj, name, default)`.

**Q6. What are exception groups and `except*`?** *(3.11+)*
`ExceptionGroup` bundles several unrelated exceptions, as when concurrent tasks in a `TaskGroup` fail at the same time. `except* T` handles the matching subset of the group while letting the others propagate, so no failure is silently lost.

---

## 31. File IO & Serialization

**What it is:** Reading and writing files (`open`, `pathlib`), and converting objects to and from storable formats: `json`, `csv`, `pickle`, and in practice also YAML, TOML, Parquet and MessagePack.

**Why it matters:** Practical everyday skill, plus a security question (`pickle`) and encoding pitfalls.

**Key concepts**

```python
from pathlib import Path

# Modes: r (default), w (truncate!), a (append), x (create, fail if exists), + (read/write), b (binary)
with open("data.txt", "r", encoding="utf-8") as f:
    content = f.read()                      # whole file (beware of huge files)
with open("data.txt", encoding="utf-8") as f:
    for line in f:                          # streaming, line by line
        process(line.rstrip("\n"))
with open("img.png", "rb") as f:
    header = f.read(8)                      # binary returns bytes

# pathlib: object-oriented, cross-platform paths
base = Path(__file__).resolve().parent
cfg = base / "config" / "app.toml"          # / joins paths
cfg.exists(), cfg.suffix, cfg.stem, cfg.parent
text = cfg.read_text(encoding="utf-8")
Path("out").mkdir(parents=True, exist_ok=True)
for p in base.rglob("*.py"): ...

# Atomic write: never leave a half-written file behind
import os, tempfile
def atomic_write(path: Path, data: str):
    with tempfile.NamedTemporaryFile("w", dir=path.parent, delete=False, encoding="utf-8") as tmp:
        tmp.write(data)
        tmp.flush(); os.fsync(tmp.fileno())
    os.replace(tmp.name, path)              # atomic rename on POSIX and Windows

# JSON
import json
s = json.dumps({"id": 1, "tags": ["a"]}, indent=2)
obj = json.loads(s)
json.dumps({"when": datetime.now()}, default=str)   # handle non-JSON types
# Tuples become lists, dict keys become strings; no datetime/Decimal/set support by default

# CSV — handles quoting; never use split(",")
import csv
with open("users.csv", newline="", encoding="utf-8") as f:
    for row in csv.DictReader(f):
        print(row["email"])

# TOML (stdlib read-only, 3.11+)
import tomllib
with open("pyproject.toml", "rb") as f:
    project = tomllib.load(f)

# pickle — Python-only, handles arbitrary objects, UNSAFE on untrusted input
import pickle
blob = pickle.dumps(model)
model2 = pickle.loads(blob)                 # can execute arbitrary code during load!
```

| Format | Human-readable | Cross-language | Safe on untrusted input | Typical use |
|---|---|---|---|---|
| JSON | ✓ | ✓ | ✓ | APIs, config, messages |
| CSV | ✓ | ✓ | ✓ | Tabular exports |
| TOML / YAML | ✓ | ✓ | TOML ✓; YAML only with `safe_load` | Config files |
| pickle | ✗ | ✗ (Python only) | **✗** | Internal caches, multiprocessing IPC |
| Parquet / Arrow | ✗ | ✓ | ✓ | Analytics, columnar data |
| MessagePack / Protobuf | ✗ | ✓ | ✓ | Compact RPC and event payloads |

### Interview Questions

**Q1. Why use `with open(...)`?** ⭐
`open` returns a context manager that closes the file when the block exits, even on exceptions. That releases OS file handles, which are limited, and flushes buffers. Without it, a crash before `f.close()` leaks the handle and can leave data unwritten.

**Q2. Why is `pickle` dangerous?** ⭐
Unpickling can run arbitrary code: an object's `__reduce__` can tell the loader to call any function, e.g. `os.system`. Never unpickle data from untrusted sources (user uploads, network messages, shared caches without integrity checks). Use JSON, MessagePack or Protobuf for data crossing trust boundaries. Pickles are also fragile across code versions.

**Q3. How do you read a huge file efficiently?**
Iterate lazily (`for line in f`) or read in chunks, chain generators, and avoid `read()` / `readlines()`, which load everything. For tabular or columnar data, use streaming engines (Polars lazy scans, DuckDB, pyarrow). Pass `encoding=` explicitly and use buffered binary mode for raw bytes.

**Q4. What can't `json` serialize, and how do you handle it?**
`datetime`, `Decimal`, `set`, `bytes` and custom objects aren't supported, and tuples silently become lists. Options: `default=` (a function converting the unknown type), a custom `JSONEncoder` subclass, Pydantic's `model_dump_json()`, or `orjson` (faster, and handles dataclasses and datetimes natively).

**Q5. Why use `pathlib` over `os.path`?**
Path objects are typed and cross-platform, join with `/`, and come with methods (`read_text`, `glob`, `mkdir(parents=True)`, `suffix`, `stem`) instead of scattered string functions. Code is shorter and less error-prone, and most stdlib and third-party APIs accept `Path` objects (`os.PathLike`).

---

## 32. Modules, Packages & Imports

**What it is:** A **module** is a `.py` file. A **package** is a directory of modules, usually with `__init__.py`; without it, it's a *namespace package*. `import` finds, loads, executes and caches a module **once per process** in `sys.modules`.

**Why it matters:** Circular imports, `__name__ == "__main__"`, import-time side effects and project layout come up in both interviews and real debugging.

**Key concepts**

```python
import json                          # module object bound to the name json
import numpy as np                   # alias
from pathlib import Path             # import one name
from .models import User             # relative import (inside a package only)
from ..utils import slugify          # parent package
# from module import *               # avoid: pollutes the namespace; respects __all__

if __name__ == "__main__":           # run only when executed directly, not when imported
    main()
```

*How `import x` works:*
1. Check `sys.modules` (the cache). If found, return it; the module body never runs twice.
2. Find the module: run the finders over `sys.path` (script directory, `PYTHONPATH`, stdlib, site-packages).
3. Create the module object and insert it into `sys.modules` **before** executing it (this is what makes circular imports partially work).
4. Execute the module body top to bottom.
5. Bind the name in the importing namespace.

*Typical project layout (src layout):*

```
my-service/
├── pyproject.toml
├── src/
│   └── my_service/
│       ├── __init__.py          # package marker; can expose the public API, set __all__
│       ├── __main__.py          # enables `python -m my_service`
│       ├── api/
│       ├── domain/
│       └── infra/
└── tests/
```

*Circular imports:* `a` imports `b`, and `b` imports `a`. One of them sees a **partially initialized** module, which gives `ImportError: cannot import name X (most likely due to a circular import)`. Fixes, in order of preference:
1. Restructure: move the shared code into a third module, or merge the two.
2. Depend on abstractions (dependency inversion), so lower layers don't import upper ones.
3. `import a` (module-level) instead of `from a import X`, and access `a.X` at call time.
4. Import inside the function that needs it (lazy import).
5. For type-only imports: `from typing import TYPE_CHECKING` with `if TYPE_CHECKING: from a import X`.

*Other details:*
- `__all__` defines the public API for `import *` and documents intent.
- Import-time side effects (DB connections, network calls, heavy computation at module level) slow startup and break tests. Do that work lazily or in `main()` / app factories.
- `importlib.reload(mod)` re-executes a module, but objects that already reference old definitions keep them.
- Monkey patching (replacing module attributes at runtime) is mostly for tests (`unittest.mock.patch`). Patch where the name is **looked up**, not where it's defined.

### Interview Questions

**Q1. What does `if __name__ == "__main__":` do?** ⭐
Each module gets `__name__`: the module's import name when imported, `"__main__"` when run as a script (`python file.py`, `python -m pkg`). The guard lets a file be both importable and runnable without executing script code on import. It's also **required** with multiprocessing's `spawn` start method, where child processes re-import the main module.

**Q2. What happens when you import a module twice?**
The second import is just a lookup in `sys.modules`; the module body runs only once per process. That's why modules act as natural singletons, and why module-level state is shared by every importer (within one process, not across worker processes).

**Q3. What causes circular imports, and how do you fix them?** ⭐
Two modules import each other at module level, so one of them runs while the other is only partially initialized and the name doesn't exist yet. Fix the design: extract shared code into a third module, invert dependencies with interfaces or Protocols, or merge tightly coupled modules. Tactical fixes: import the module instead of the name, import inside functions, or use `TYPE_CHECKING` imports for type hints.

**Q4. Module vs package? What is `__init__.py` for?**
A module is one file; a package is a directory of modules that forms a namespace hierarchy. `__init__.py` marks a regular package, runs on first import of the package, and is commonly used to expose a curated public API (`from .client import Client`) and set `__all__`. Without it you get a namespace package (PEP 420), which can be spread across several directories.

**Q5. Absolute vs relative imports?**
Absolute (`from my_service.domain import Order`) is explicit and works everywhere; PEP 8 prefers it. Relative (`from .domain import Order`) is shorter inside a package and survives renaming the top-level package, but only works inside packages (not in scripts run directly). Most teams standardize on absolute imports, enforced with Ruff.

**Q6. How does Python find modules? What is `sys.path`?**
`sys.path` is a list of directories searched in order: the script's directory (or the current directory for `-m` and the REPL), `PYTHONPATH` entries, the stdlib, then `site-packages` of the active environment. A local file named like a stdlib module (`random.py`, `json.py`) shadows the real one, which is a classic bug. Virtual environments work by giving each project its own `site-packages`.

---

## 33. Type Hints & Static Typing

**What it is:** Optional annotations (PEP 484 and later) describing expected types. They're **not enforced at runtime**; static checkers (mypy, Pyright, ty) and IDEs use them, and libraries like Pydantic and FastAPI read them at runtime for validation.

**Why it matters:** Typed Python is the norm in production codebases. Interviewers ask about runtime vs static, Protocol vs ABC, generics, and how Pydantic uses hints.

**Key concepts**

```python
from typing import Any, Callable, Iterable, Iterator, Literal, Protocol, Self, TypedDict, \
    Final, ClassVar, TypeAlias, overload, TYPE_CHECKING, Annotated, NotRequired, override
from collections.abc import Sequence, Mapping, Awaitable

def greet(name: str, times: int = 1) -> str: ...
def find(user_id: int) -> "User | None": ...         # 3.10+ union syntax
def total(xs: Iterable[float]) -> float: ...         # accept the most general type you can
def handlers() -> dict[str, Callable[[bytes], None]]: ...
Mode = Literal["r", "w", "a"]

# Generics: new syntax (3.12+) and the classic TypeVar form
def first[T](items: Sequence[T]) -> T:
    return items[0]

class Repository[T]:
    def __init__(self) -> None: self._items: list[T] = []
    def add(self, item: T) -> None: self._items.append(item)
    def all(self) -> list[T]: return list(self._items)

type UserId = int                                    # type alias statement (3.12+)

from typing import TypeVar, Generic                  # pre-3.12 style
T = TypeVar("T")
class Box(Generic[T]): ...

# Protocol = structural typing
class Notifier(Protocol):
    def send(self, to: str, msg: str) -> None: ...

# TypedDict = typed shape for dicts (JSON payloads)
class UserDTO(TypedDict):
    id: int
    email: str
    nickname: NotRequired[str]

class Builder:
    def set(self, k: str, v: Any) -> Self: ...       # fluent APIs, subclass-friendly

class Base:
    def run(self) -> None: ...
class Child(Base):
    @override                                         # 3.12+: the checker verifies a parent method exists
    def run(self) -> None: ...

# Runtime reality
def double(n: int) -> int: return n * 2
double("ab")                                         # 'abab' — no runtime error!

# Runtime validation is a separate layer: Pydantic
from pydantic import BaseModel, EmailStr, Field
class SignUp(BaseModel):
    email: EmailStr
    age: int = Field(ge=18)
SignUp(email="a@x.com", age="21")                    # coerces "21" -> 21
# SignUp(email="bad", age=10)                        # ValidationError listing both problems
```

- `Any` switches off checking for that value; prefer `object` (accepts anything, but forces narrowing before use).
- **Narrowing**: `isinstance`, `is None` checks, `TypeGuard`/`TypeIs` functions and `match` all refine types for the checker.
- **Variance**: `list[int]` is *not* a `list[float]` (lists are invariant because they're mutable), but `Sequence[int]` *is* a `Sequence[float]` (covariant, read-only). So accept `Sequence` or `Iterable` in parameters.
- **Annotations evaluation**: since 3.14, annotations are evaluated lazily (PEP 649/749), so forward references no longer need quotes. Older code uses `from __future__ import annotations`.

### Interview Questions

**Q1. Are type hints enforced at runtime?** ⭐
No. CPython stores them in `__annotations__` and otherwise ignores them; `double("ab")` runs fine. Enforcement comes from **static checkers** in CI (mypy, Pyright, ty) and from **runtime libraries** that deliberately read annotations: Pydantic, FastAPI, dataclass-based validators, typeguard/beartype. Teams usually validate at system boundaries with Pydantic and rely on static checking inside.

**Q2. Protocol vs ABC?** ⭐
An ABC is **nominal**: a class must explicitly inherit to be compatible, and abstract methods are checked at instantiation. A Protocol is **structural**: any class with matching members is compatible, checked statically, with no inheritance. Protocols fit duck typing, third-party classes and dependency injection; ABCs fit owned hierarchies that share implementation.

**Q3. What are generics? Show an example.**
Generics parameterize types so relationships are preserved: `first(items: Sequence[T]) -> T` returns the element type the caller passed in, and `Repository[User]` only accepts and returns `User`. Python 3.12 added the inline syntax `def f[T](...)` and `class C[T]: ...`; older code uses `TypeVar` and `Generic[T]`.

**Q4. `Optional[X]` vs `X | None`? Does `Optional` mean the argument is optional?**
They're equivalent: the value may be `None`. `X | None` (3.10+) is the modern spelling. `Optional` does **not** mean the parameter can be omitted; that requires a default value (`x: int | None = None`).

**Q5. Why accept `Iterable` or `Sequence` instead of `list` in parameters?**
Accept the most general type you need and return the most specific. `list[float]` would reject tuples, generators and `list[int]` (invariance). `Iterable[float]` or `Sequence[float]` accept all of those, which makes functions more reusable without losing safety.

**Q6. How does FastAPI or Pydantic use type hints?**
At class or route definition time they introspect annotations (`typing.get_type_hints`) to build validators and serializers (Pydantic v2's core is written in Rust). They coerce and validate incoming data, generate JSON Schema / OpenAPI docs, and, in FastAPI, resolve dependency injection from parameter annotations. It's the clearest example of hints driving runtime behaviour.

---

# PART 6 — INTERNALS, CONCURRENCY & PERFORMANCE

---

## 34. CPython Execution Model

**What it is:** The pipeline from source to execution: **tokenize → parse → AST → compile to bytecode (code objects) → PVM eval loop**. Each function call creates a **frame** (locals, value stack, instruction pointer).

**Why it matters:** It explains performance behaviour (why local variables are faster, why built-ins beat Python loops), `.pyc` files, and the recent speed-ups and JIT work.

**Key concepts**

```python
import dis, ast

src = "x = a + b * 2"
ast.dump(ast.parse(src))             # the syntax tree
code = compile(src, "<demo>", "exec")
dis.dis(code)                        # bytecode: LOAD_NAME, BINARY_OP, STORE_NAME ...

def f(a, b):
    return a + b
f.__code__.co_varnames, f.__code__.co_consts   # code object metadata
```

- **`.pyc` caching**: imported modules are compiled once and cached in `__pycache__/` (keyed by source mtime or hash). The script you run directly is compiled each time but not cached.
- **The eval loop** is a big C loop dispatching opcodes. Overhead comes from dynamic dispatch, boxing every value as an object, and refcount updates.
- **3.11 "Faster CPython"**: the **specializing adaptive interpreter** rewrites hot opcodes into type-specialized versions (e.g. `BINARY_OP_ADD_INT`, `LOAD_ATTR_INSTANCE_VALUE`) based on observed types, plus cheaper frames and zero-cost exceptions. That gave 10–60% speed-ups.
- **3.13+ experimental JIT** (copy-and-patch) compiles hot traces to machine code. It's off by default; gains are still modest.
- **Why built-ins are fast**: `sum()`, `sorted()`, `str.join` and comprehensions run their loops in C, avoiding a bytecode dispatch per element.
- **Local lookup** is an array index (`LOAD_FAST`); global or built-in lookup is a dict lookup (`LOAD_GLOBAL`, cached since 3.11). Hoisting attribute lookups out of hot loops still helps a little.
- **Constant folding and peephole optimization**: `2 * 60 * 60` is computed at compile time.

### Interview Questions

**Q1. Walk through what happens when you run `python app.py`.** ⭐
The interpreter initializes the runtime (built-ins, `sys`, the import system, `site` / site-packages). It reads `app.py`, tokenizes and parses it into an AST, and compiles that into a code object of bytecode. It executes the module code object in a frame on the PVM eval loop with `__name__ == "__main__"`. Imports along the way load cached `.pyc` files or compile and cache them. At exit it runs `atexit` handlers and finalizes objects.

**Q2. Why is CPython slower than Java or Go, and what's being done about it?**
It interprets bytecode with dynamic typing: every operation checks types at runtime, every value is a heap-allocated object, refcounts are updated constantly, and there's no JIT by default. Java and Go compile to optimized machine code (JIT or AOT) with static types. Improvements: the 3.11 specializing interpreter, the experimental JIT in 3.13+, free-threading for parallelism, and in practice pushing hot loops into C, Rust or NumPy.

**Q3. What are `.pyc` files? Can you ship only `.pyc`?**
They're cached bytecode for imported modules, stored in `__pycache__` with the interpreter version in the name (e.g. `cpython-312`), and they speed up startup by skipping compilation. You *can* ship sourceless `.pyc`, but it's version-specific and only light obfuscation (it decompiles easily). It isn't real IP protection.

---

## 35. Memory Management & Garbage Collection

**What it is:** CPython manages memory with **reference counting** (immediate freeing when the count reaches 0) plus a **cyclic garbage collector** for reference cycles. Small objects come from the **pymalloc** allocator (arenas → pools → blocks for objects ≤ 512 bytes).

**Why it matters:** Memory leaks in long-running services, `__del__` behaviour, `weakref`, GC tuning and "how does Python manage memory?" are all classic senior questions.

**Key concepts**

```python
import sys, gc, weakref

a = []
sys.getrefcount(a)          # 2 — the name a + the temporary argument
b = a
sys.getrefcount(a)          # 3
del b                       # refcount drops; at 0 the object is freed IMMEDIATELY

# Reference cycle: refcounts never reach 0 on their own
class Node:
    def __init__(self): self.other = None
x, y = Node(), Node()
x.other, y.other = y, x
del x, y                    # still unreachable garbage until the cycle GC runs
gc.collect()                # returns the number of objects collected

# Weak references don't keep objects alive (caches, observers, parent pointers)
class Big: ...
obj = Big()
r = weakref.ref(obj)
r() is obj                  # True
del obj
r()                         # None
cache = weakref.WeakValueDictionary()   # entries vanish when the values die
```

- **Reference counting**: every object has `ob_refcnt`. Freeing is deterministic and prompt (files close when the last reference goes), but every assignment costs a refcount update, and refcounting can't collect cycles.
- **Cyclic GC**: tracks *container* objects (lists, dicts, instances) and periodically finds groups that are only reachable from each other. Classically it has 3 generations (0, 1, 2): young objects are scanned most often, because most objects die young. In **3.14** the cyclic GC became **incremental** (young and old generations, with work spread across collections) for shorter pauses.
- **Tuning**: `gc.set_threshold()`, `gc.disable()`; `gc.freeze()` before forking workers avoids copy-on-write memory blow-up. Instagram famously disabled or froze the GC in its Django workers.
- **`__del__` (finalizer)**: timing isn't guaranteed (cycles, interpreter shutdown). Don't rely on it for cleanup; use context managers or `weakref.finalize`.
- **Memory is not always returned to the OS**: pymalloc arenas are released only when completely empty, so RSS can stay high after freeing many small objects (fragmentation).
- **Common leak sources**: unbounded caches or global lists, `lru_cache` on methods (holds `self`), listeners never unregistered, cycles involving objects with `__del__` in old versions, C extensions, and keeping large tracebacks or frames alive.

**Finding leaks**

```python
import tracemalloc
tracemalloc.start()
# ... run the workload ...
snapshot = tracemalloc.take_snapshot()
for stat in snapshot.statistics("lineno")[:10]:
    print(stat)                # top allocation sites by source line
```

Production tools: `memray` (allocation flame graphs), `objgraph` (who references what), `py-spy dump`, and watching RSS growth per request in metrics.

### Interview Questions

**Q1. How does Python manage memory?** ⭐
Two layers. (1) **Reference counting**: each object counts its references and is freed as soon as the count hits zero. That's deterministic and covers most objects. (2) A **cyclic garbage collector** periodically detects reference cycles that refcounting can't free, using generations (young objects scanned more often). Underneath, pymalloc serves small objects from pools to reduce fragmentation and `malloc` overhead. Refcounting is also the reason the GIL exists.

**Q2. What is a reference cycle, and how is it collected?** ⭐
Objects that reference each other (parent ↔ child, a doubly linked list, an object referencing its own bound method), so their counts never reach 0. The cyclic GC looks at container objects: it subtracts references that come from *inside* the candidate set, and anything left with zero external references is unreachable and gets freed. Avoid cycles where you can, e.g. with `weakref` for back-pointers.

**Q3. How would you debug a memory leak in a long-running Python service?** ⭐
1. **Confirm** it's a leak: RSS grows steadily per request or job, not just a warm-up plateau.
2. **Locate** it: compare `tracemalloc` snapshots over time, or use `memray` for allocation flame graphs and `objgraph.show_growth()` / `show_backrefs()` to see which object types grow and who holds them.
3. **Check usual suspects**: unbounded caches and module-level dicts, `lru_cache` on methods, listeners never removed, global request state, C extensions.
4. **Fix** with bounded caches, weak references, or proper cleanup.
5. **Mitigate** meanwhile: Gunicorn `max_requests` (with jitter) to recycle workers.

**Q4. What are weak references used for?**
Referring to an object without keeping it alive: caches that shouldn't prevent cleanup (`WeakValueDictionary`), observer and listener lists, parent back-pointers in trees (which break cycles), and `weakref.finalize` for reliable cleanup callbacks. Not every object supports them (`int` and `tuple` don't; classes with `__slots__` need `__weakref__`).

**Q5. Why might memory not drop after deleting a big list of objects?**
The objects *are* freed, but pymalloc keeps the arenas for reuse and only returns an arena to the OS when it's completely empty. Fragmentation can keep arenas half-full. Large allocations (> 512 bytes, e.g. big buffers) go straight to `malloc` and are often returned. Fixes: process recycling, avoiding millions of tiny objects (use NumPy, Arrow, `array`), or doing big jobs in a separate subprocess.

---

## 36. The GIL & Free-Threading

**What it is:** The **Global Interpreter Lock** is a mutex in CPython that allows only **one thread to execute Python bytecode at a time** per interpreter. **Free-threaded CPython** (PEP 703) is a build without the GIL: experimental in 3.13 and **officially supported, though still optional, in 3.14** (PEP 779).

**Why it matters:** It's the single most-asked Python internals question, and it drives every concurrency decision.

**Key concepts**

- **Why it exists**: refcounting isn't thread-safe; two threads incrementing `ob_refcnt` at once would corrupt it. One global lock is simple and fast for single-threaded code, and it made writing C extensions easy. Removing it historically slowed single-threaded code.
- **When it's released**:
  - During **blocking I/O** (sockets, files, `time.sleep`), so I/O-bound threads overlap well.
  - By C extensions doing long computations (NumPy, hashlib, zlib, many ML libraries).
  - Periodically, every **switch interval of 5 ms** (`sys.getswitchinterval()`), so threads take turns.
- **Consequence**: CPU-bound pure-Python threads get **no parallel speed-up**, and contention can make them slower.
- **The GIL does not make your code thread-safe**: `counter += 1` is several bytecodes, and a thread switch can happen between them (§37).
- **Free-threaded build (`python3.14t`)**: uses biased reference counting, immortal objects and per-object locks. Pure-Python CPU work can now scale across cores; single-threaded overhead is now small (around 5–10%). Caveat: C extensions must be rebuilt and declare support, otherwise the GIL is re-enabled at import.
- **Other ways around the GIL**: multiprocessing; sub-interpreters with their own GIL (3.12 per-interpreter GIL; `concurrent.interpreters` in 3.14); native code that releases the GIL; asyncio for I/O.

```python
import threading, time

def cpu_task(n=20_000_000):
    while n: n -= 1

start = time.perf_counter()
cpu_task(); cpu_task()
print("sequential", time.perf_counter() - start)

start = time.perf_counter()
ts = [threading.Thread(target=cpu_task) for _ in range(2)]
[t.start() for t in ts]; [t.join() for t in ts]
print("2 threads ", time.perf_counter() - start)   # about the same (or slower) with the GIL
                                                   # roughly half on a free-threaded build
import sys
getattr(sys, "_is_gil_enabled", lambda: True)()    # False on a free-threaded build with the GIL off
```

### Interview Questions

**Q1. What is the GIL, and why does Python have it?** ⭐
A per-interpreter mutex in CPython that lets only one thread run Python bytecode at a time. It exists because CPython's memory management (reference counting) and much C-extension code aren't thread-safe. One big lock was simpler and faster for single-threaded programs than fine-grained locking, and it made C extensions easy to write. It's a CPython implementation detail, not part of the language (Jython and IronPython never had one).

**Q2. If there's a GIL, why use threads at all?** ⭐
Because the GIL is **released during blocking I/O** and inside many C extensions. Threads waiting on network, disk or DB calls overlap their waits, so I/O-bound workloads (HTTP calls, DB queries, file transfers) get large speed-ups. Only CPU-bound pure-Python work is serialized.

**Q3. Does the GIL make Python code thread-safe?** ⭐
No. It makes individual bytecode instructions atomic (and some built-in operations like `list.append` atomic in CPython), but compound operations such as `x += 1`, check-then-act sequences and read-modify-write on dicts can interleave between bytecodes. You still need locks, queues or other synchronization (§37). Free-threaded builds make this even more important.

**Q4. How do you speed up CPU-bound Python?** ⭐
In order: a better algorithm or data structure; vectorize with NumPy, Polars or pandas (C loops that release the GIL); `multiprocessing` / `ProcessPoolExecutor` to use all cores; native extensions (Cython, numba, PyO3/Rust); a free-threaded build with threads (3.14+), if your dependencies support it; or move the hot path into a separate service in a faster language.

**Q5. What is free-threaded Python, and what are its trade-offs?** ⭐
A CPython build (`python3.13t`/`3.14t`) with the GIL removed (PEP 703), officially supported in 3.14 but not yet the default. Threads can run Python bytecode in parallel on multiple cores. Trade-offs: slight single-threaded overhead; C extensions must be rebuilt and marked free-threading-safe (otherwise the GIL comes back); and latent race conditions in Python code now surface more often. The ecosystem (NumPy, etc.) is adopting it progressively.

---

## 37. Threading & Synchronization

**What it is:** `threading` runs multiple OS threads that share one process's memory. **Synchronization primitives** (`Lock`, `RLock`, `Semaphore`, `Event`, `Condition`, `Barrier`, `queue.Queue`) coordinate them.

**Why it matters:** I/O concurrency in sync code (legacy drivers, `requests`), plus race-condition and deadlock questions.

**Key concepts**

```python
import threading
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests

# Prefer executors over raw threads
urls = [...]
with ThreadPoolExecutor(max_workers=20) as pool:
    futures = {pool.submit(requests.get, u, timeout=5): u for u in urls}
    for fut in as_completed(futures):
        url = futures[fut]
        try:
            print(url, fut.result().status_code)
        except Exception as e:
            print(url, "failed", e)

# Race condition: += is not atomic
counter = 0
lock = threading.Lock()
def work():
    global counter
    for _ in range(100_000):
        with lock:                   # remove the lock -> the total may come out below 400000
            counter += 1

# Producer/consumer with queue.Queue (thread-safe, does its own locking)
import queue
q: queue.Queue[str | None] = queue.Queue(maxsize=100)   # bounded = backpressure
def producer():
    for item in items: q.put(item)
    q.put(None)                      # sentinel to stop the consumer
def consumer():
    while (item := q.get()) is not None:
        handle(item)
        q.task_done()

# Event for signalling / graceful shutdown
stop = threading.Event()
def poller():
    while not stop.is_set():
        poll()
        stop.wait(5)                 # sleeps, but wakes immediately when stop is set

# Thread-local state (e.g. a per-thread DB connection)
local = threading.local()
```

| Primitive | Use |
|---|---|
| `Lock` | Mutual exclusion for a critical section |
| `RLock` | Re-entrant: the same thread can acquire it again (recursive code) |
| `Semaphore(n)` | Allow at most N concurrent users of a resource (connection limits) |
| `Event` | One-shot or repeated signal ("ready", "stop") |
| `Condition` | Wait until a predicate becomes true (with notify) |
| `Barrier(n)` | Wait until N threads all reach the same point |
| `queue.Queue` | Hand off data between threads safely (preferred over shared state) |

- **Daemon threads** are killed abruptly at interpreter exit with no cleanup; non-daemon threads keep the process alive. Prefer explicit shutdown with an `Event`.
- **Deadlocks**: always acquire locks in a consistent global order, use timeouts (`lock.acquire(timeout=...)`), and keep critical sections small.
- Thread count is limited by memory (each thread has its own stack) and by context-switch costs. Hundreds is fine; tens of thousands means it's time for asyncio.

### Interview Questions

**Q1. Why do you need locks if Python has a GIL?** ⭐
The GIL makes single bytecode instructions atomic, but `counter += 1` is load, add and store, and a thread switch between them loses updates. The same applies to check-then-act logic (`if key not in d: d[key] = ...`). Locks make the whole critical section atomic. On free-threaded builds, even more operations can race.

**Q2. `Lock` vs `RLock`?**
A `Lock` can't be acquired twice, even by the same thread: re-acquiring deadlocks. An `RLock` counts acquisitions by the owning thread, so recursive or re-entrant code paths can take it again; it must be released the same number of times. `RLock` is slightly slower; use it only when re-entrancy is genuinely needed.

**Q3. How do you implement producer–consumer in Python?** ⭐
Use `queue.Queue`: producers `put()`, consumers `get()`, and the queue handles locking and blocking. Bound it with `maxsize` for backpressure, use sentinel values or `Event`s for shutdown, and `task_done()` / `join()` to wait for completion. The asyncio equivalent is `asyncio.Queue`; across processes it's `multiprocessing.Queue`, or Redis/Kafka across machines.

**Q4. How do you avoid deadlocks?**
Acquire multiple locks in a single global order, hold locks briefly (no I/O while locked), use `with lock:` so they're always released, add acquisition timeouts to detect problems, and prefer message passing (queues) over shared mutable state.

**Q5. `ThreadPoolExecutor` vs creating threads manually?**
The executor reuses a bounded pool of workers, returns `Future`s with results and exceptions, supports `map`, `as_completed` and cancellation, and shuts down cleanly as a context manager. Manual threads are only worth it for long-lived background workers with custom lifecycles.

---

## 38. Multiprocessing

**What it is:** `multiprocessing` and `concurrent.futures.ProcessPoolExecutor` run work in **separate processes**, each with its own interpreter, memory and GIL, giving true parallelism on multiple cores.

**Why it matters:** It's the standard answer for CPU-bound work in GIL Python, and its pitfalls (pickling, start methods, shared state) are frequent follow-ups.

**Key concepts**

```python
from concurrent.futures import ProcessPoolExecutor
import math

def is_prime(n: int) -> bool:          # must be a top-level function (picklable)
    if n < 2: return False
    return all(n % d for d in range(2, math.isqrt(n) + 1))

if __name__ == "__main__":             # REQUIRED with spawn/forkserver (child processes re-import main)
    nums = range(10_000_000, 10_000_500)
    with ProcessPoolExecutor() as pool:                 # defaults to os.cpu_count() workers
        results = list(pool.map(is_prime, nums, chunksize=50))  # chunksize cuts IPC overhead
```

- **Start methods**:
  - `fork`: copies the parent's memory, fast, but unsafe with threads and locks.
  - `spawn`: a fresh interpreter, safe but slower. The default on Windows and macOS.
  - `forkserver`: forks from a clean server process. **The default on Linux since 3.14**; earlier versions used `fork`.
- **Data passes by pickling** over pipes. Arguments and return values must be picklable (no lambdas, local functions or open sockets), and large objects are expensive to send. Send small inputs (IDs, file paths), or use `multiprocessing.shared_memory`, memory-mapped files or Arrow/Parquet on disk.
- **Shared state**: `Value`/`Array` (shared ctypes), `Manager()` (a proxy server, slower), `Queue`/`Pipe`. Prefer no shared state: map → reduce.
- **Costs**: process startup and memory per worker (each has a full interpreter plus imports). Good for coarse-grained tasks, bad for millions of tiny ones.
- **In production** this generalizes to *multiple worker processes*: Gunicorn/Uvicorn workers, Celery workers, Kubernetes replicas.

### Interview Questions

**Q1. Multithreading vs multiprocessing in Python?** ⭐
Threads share memory, are cheap, and are limited by the GIL, so use them for I/O-bound work. Processes have separate memory and interpreters, so they run in true parallel on all cores; use them for CPU-bound work. They cost more memory and startup time, and data must be pickled between processes. Rule of thumb: **threads (or asyncio) for waiting, processes for computing.**

**Q2. Why must multiprocessing code be under `if __name__ == "__main__":`?**
With `spawn` and `forkserver` (default on macOS and Windows, and on Linux since 3.14), each child process **imports the main module** to find the target function. Without the guard, the child re-runs the top-level code that creates the pool, which recursively spawns processes or raises a `RuntimeError`.

**Q3. What are the pitfalls of multiprocessing?** ⭐
- **Pickling**: lambdas, nested functions and some objects can't be sent; large payloads are slow.
- **Startup and memory** cost per worker.
- **`fork` with threads or locks** can deadlock in the child.
- **Shared state** is hard: mutations in a child aren't visible to the parent.
- **Errors in workers** surface only when you call `future.result()`.
- **Too many tiny tasks** let IPC overhead dominate; use `chunksize`.

**Q4. How do you share data between processes?**
Prefer passing messages (`Queue`, `Pipe`) or returning results. For large read-only data, load it before forking (copy-on-write with `fork`), or use `shared_memory` / memory-mapped files / NumPy on shared buffers. A `Manager` proxies dicts and lists through a server process: convenient but slow. Across machines, use Redis, a database or object storage.

---

## 39. Asyncio

**What it is:** Single-threaded **cooperative multitasking**. **Coroutines** (`async def`) voluntarily yield control at `await` points to an **event loop**, which uses OS I/O multiplexing (`epoll`, `kqueue`, IOCP) to resume whichever coroutine's I/O is ready.

**Why it matters:** It powers FastAPI, high-concurrency clients, WebSocket servers, scrapers and LLM agent orchestration. "How does asyncio work?" and "what happens if you block the loop?" are standard questions.

**Key concepts**

```python
import asyncio, httpx

async def fetch(client: httpx.AsyncClient, url: str) -> int:
    r = await client.get(url, timeout=10)      # yields to the loop while waiting on the network
    return r.status_code

async def main(urls: list[str]):
    sem = asyncio.Semaphore(20)                # limit concurrency (be kind to downstream services)
    async with httpx.AsyncClient() as client:

        async def bounded(u):
            async with sem:
                return await fetch(client, u)

        # Structured concurrency (3.11+): wait for all; if one fails, cancel the rest
        async with asyncio.TaskGroup() as tg:
            tasks = [tg.create_task(bounded(u)) for u in urls]
    return [t.result() for t in tasks]

asyncio.run(main(urls))                        # creates the loop, runs, cleans up
```

*Building blocks:*
- **Coroutine**: the object returned by calling an `async def` function. Nothing runs until it's awaited or scheduled.
- **Task**: a coroutine scheduled on the loop to run concurrently (`asyncio.create_task`, `TaskGroup.create_task`). **Keep a reference to it**; unreferenced tasks can be garbage-collected mid-flight.
- **Future**: a low-level placeholder for a result; Tasks are Futures.
- **Awaitable**: anything usable with `await` (coroutines, Tasks, Futures, objects with `__await__`).

```python
# gather vs TaskGroup
results = await asyncio.gather(*coros, return_exceptions=True)   # collect errors as values

# Timeouts (3.11+)
async with asyncio.timeout(5):
    await slow_call()

# Running blocking code without freezing the loop
data = await asyncio.to_thread(requests.get, url)                # blocking I/O -> thread pool
loop = asyncio.get_running_loop()
result = await loop.run_in_executor(process_pool, cpu_heavy, n)  # CPU work -> process pool

# Async iteration and queues
async def ticker():
    for i in range(3):
        await asyncio.sleep(1)
        yield i
async for t in ticker(): ...
q: asyncio.Queue[int] = asyncio.Queue(maxsize=100)

# Cancellation: CancelledError is raised at the current await point
try:
    await long_job()
except asyncio.CancelledError:
    await cleanup()
    raise                     # always re-raise CancelledError
```

*Blocking the event loop* is the #1 asyncio bug:

```python
async def handler():
    time.sleep(2)             # BLOCKS EVERYTHING for 2s: no other request progresses
    requests.get(url)         # same: a sync library inside async code
    sum(range(10**9))         # same: CPU work with no await points
    await asyncio.sleep(2)    # OK: yields to the loop
```

Detect it with `PYTHONASYNCIODEBUG=1` / `asyncio.run(main(), debug=True)`, which logs slow callbacks (> 100 ms).

### Interview Questions

**Q1. How does asyncio work under the hood?** ⭐
A single-threaded event loop keeps a queue of ready callbacks and a selector (`epoll`/`kqueue`/IOCP) watching sockets. A coroutine runs until it `await`s something that isn't ready. At that point it suspends (built on the generator machinery) and registers interest in an I/O event or timer. The loop then runs other ready tasks. When the OS reports the socket is ready, the loop resumes the waiting coroutine. Concurrency comes from overlapping waits, not from parallel execution.

**Q2. What happens if you call blocking code inside `async def`?** ⭐
The whole event loop stalls: every other coroutine, request and heartbeat waits until the blocking call returns, so latency spikes for all users. Use async libraries (`httpx.AsyncClient`, `asyncpg`, `aiofiles`), or offload blocking I/O with `asyncio.to_thread` and CPU work with `run_in_executor(ProcessPoolExecutor)`.

**Q3. `asyncio.gather` vs `TaskGroup`?** ⭐
`gather` runs awaitables concurrently and returns results in order. By default the first exception propagates, but the other tasks **keep running** unsupervised; `return_exceptions=True` returns errors as values instead. `TaskGroup` (3.11+) provides **structured concurrency**: if any task fails, the others are cancelled, and all errors are raised together as an `ExceptionGroup`. No orphaned tasks. Prefer `TaskGroup` for new code; `gather(return_exceptions=True)` still suits "best effort, collect everything".

**Q4. asyncio vs threads for I/O: when do you choose which?**
Asyncio handles **thousands to hundreds of thousands** of concurrent connections cheaply (tasks are small objects; no per-thread stack or context switches), with explicit yield points (fewer races). But it needs async libraries end to end. Threads work with existing **blocking** libraries and suit moderate concurrency (tens to hundreds). Mixed codebases use asyncio with `to_thread` for the leftover sync calls.

**Q5. What's the difference between a coroutine, a Task and a Future?**
A coroutine is a paused computation that runs only when awaited. A Task wraps a coroutine and schedules it on the loop immediately so it runs concurrently; you can await it, cancel it and read its result. A Future is the low-level "result will arrive later" object that Tasks build on; you rarely create one directly (it's used by low-level transports and bridging callbacks).

**Q6. How do you limit concurrency, add timeouts and handle cancellation?**
Concurrency: `asyncio.Semaphore` around the work, or a fixed pool of worker tasks consuming an `asyncio.Queue`. Timeouts: `async with asyncio.timeout(s)` or `asyncio.wait_for`. Cancellation: `task.cancel()` raises `CancelledError` at the task's current `await`; clean up in `try/finally` and **re-raise** it, since swallowing it breaks `TaskGroup` and shutdown.

**Q7. Why do you need `asyncio.Lock` if asyncio is single-threaded?**
Coroutines can interleave at any `await`. A check-then-act sequence spanning an `await` (read the cache → await the DB → write the cache) can be interrupted by another coroutine doing the same thing. `asyncio.Lock` protects *logical* invariants across await points; it doesn't protect memory, since there's no parallelism. Never use `threading.Lock` in async code: it blocks the loop.

---

## 40. Choosing a Concurrency Model

**What it is:** A decision framework: identify the bottleneck (**I/O-bound vs CPU-bound**), the scale of concurrency, and library constraints, then pick threads, processes, asyncio, or a combination.

**Why it matters:** "Design a service that does X concurrently" questions are judged on this reasoning.

| Model | Best for | Parallel CPU? | Memory per unit | Main pitfalls |
|---|---|---|---|---|
| `threading` / `ThreadPoolExecutor` | I/O-bound work with blocking libraries; tens to hundreds of tasks | ✗ with GIL (✓ free-threaded) | ~MBs of stack per thread | Races, deadlocks, GIL for CPU work |
| `multiprocessing` / `ProcessPoolExecutor` | CPU-bound work | ✓ | A full interpreter per worker | Pickling, startup cost, shared state |
| `asyncio` | Very high I/O concurrency (thousands of sockets) | ✗ | ~KBs per task | Blocking the loop; needs async libraries end to end |
| Free-threaded threads (3.14t) | CPU-bound work with shared memory | ✓ | MBs | Extension compatibility, races |
| Distributed (Celery, Kafka consumers, Ray) | Work beyond one machine, durable background jobs | ✓ across machines | — | Ops complexity, idempotency |

**Decision flow**

```
Is the work mostly waiting on network/disk/DB?
 ├─ yes → many concurrent connections (≫100) and async libraries available? → asyncio
 │        otherwise (blocking libraries, moderate scale)                     → ThreadPoolExecutor
 └─ no (CPU-bound) → can it be vectorized (NumPy/Polars)?                   → do that first
                     otherwise → ProcessPoolExecutor (or free-threading / native code)
Hybrid (an async API that occasionally does CPU work) → asyncio + run_in_executor(ProcessPool)
Long or durable jobs, retries, scheduling      → a task queue (Celery/RQ/Temporal) + workers
```

### Interview Questions

**Q1. Multiprocessing vs multithreading vs asyncio: when do you use each?** ⭐
Identify the bottleneck. **CPU-bound** (image processing, parsing, model inference in Python): processes, since each has its own GIL. **I/O-bound with blocking libraries**: threads, because the GIL is released while waiting. **I/O-bound at very high concurrency** with async-native libraries: asyncio, because tasks are cheap and there are no thread switches. Real systems combine them: async web servers run as multiple worker processes and offload CPU work to process pools.

**Q2. Design a scraper that fetches 100k URLs and parses the HTML.**
Use asyncio with `httpx.AsyncClient` (connection pooling, HTTP/2), a `Semaphore` for per-host and global concurrency limits, timeouts and retries with exponential backoff and jitter, and an `asyncio.Queue` for backpressure. Parsing is CPU-bound, so hand pages to a `ProcessPoolExecutor` (or a separate worker fleet via a queue). Stream results to storage instead of holding them in memory. Respect robots.txt and rate limits, and make the work resumable (track completed URLs).

**Q3. How would you handle 10k concurrent slow clients (long polling, WebSockets)?**
An ASGI server (Uvicorn) with asyncio. Each connection is a lightweight task, not a thread, so memory stays at KBs per client. Avoid blocking calls, use async DB and Redis drivers, set timeouts, scale horizontally behind a load balancer with sticky sessions or pub/sub (Redis) for fan-out, and run multiple worker processes per machine.

---

## 41. Performance & Profiling

**What it is:** A methodology: **measure → find the bottleneck → fix the biggest cost → measure again**, plus Python-specific optimization techniques.

**Why it matters:** "This endpoint is slow, what do you do?" is a classic senior question. The method matters more than tricks.

**Key concepts — tools**

| Tool | What it does | When |
|---|---|---|
| `timeit` | Micro-benchmarks | Comparing two snippets |
| `cProfile` + `pstats` / `snakeviz` | Deterministic function-level profile | Dev/staging; has significant overhead |
| **`py-spy`** | Sampling profiler; attaches to a running PID, flame graphs | **Production-safe**, no code changes |
| `scalene` | CPU + memory + GPU, line level, separates Python vs native time | Deep dives |
| `line_profiler` | Per-line timings for decorated functions | A known hot function |
| `tracemalloc`, `memray` | Memory allocations and leaks | Memory growth |
| APM / OpenTelemetry tracing | Where request time goes (DB, HTTP, code) | Production first look |

```python
import cProfile, pstats
with cProfile.Profile() as prof:
    handle_request()
pstats.Stats(prof).sort_stats("cumulative").print_stats(10)

# $ py-spy top --pid 12345
# $ py-spy record -o profile.svg --pid 12345
# $ python -m timeit "'-'.join(str(n) for n in range(100))"
```

**Optimization checklist (in order of impact)**

1. **I/O first**: most "slow Python" is slow I/O. N+1 ORM queries (`select_related`/`prefetch_related`, `selectinload`), missing DB indexes, sequential HTTP calls that could run concurrently, missing caching (Redis, `lru_cache`), chatty APIs (batch them).
2. **Algorithms and data structures**: O(n²) → O(n log n); list membership → set; repeated sorts → heap or bisect.
3. **Push loops into C**: built-ins (`sum`, `min`, `sorted`, `any`), comprehensions, `str.join`, `itertools`, `collections.Counter`, and **NumPy/Polars vectorization** instead of per-row Python loops.
4. **Avoid repeated work**: hoist invariants out of loops, cache attribute lookups in very hot loops, precompile regexes, memoize pure functions.
5. **Memory and data layout**: generators for streaming, `__slots__` for many small objects, Arrow/NumPy instead of lists of dicts.
6. **Concurrency**: async or threads for I/O, processes for CPU (§40).
7. **Native code** for the remaining hot spots: Cython, numba, PyO3/Rust, or a C library.
8. **Runtime**: upgrade Python (3.11+ is much faster), try PyPy for pure-Python CPU code, tune server workers.

```python
# Slow: a per-row Python loop
total = 0
for price, qty in rows:
    total += price * qty
# Fast: vectorized
import numpy as np
prices, qtys = np.array(prices), np.array(qtys)
total = float((prices * qtys).sum())
```

### Interview Questions

**Q1. A Python API endpoint is slow. How do you approach it?** ⭐
1. **Measure, don't guess.** Check APM traces or metrics to see where the latency goes (DB, external calls, CPU, waiting for a worker), and look at p50 vs p99 under realistic load.
2. **If it's I/O**: look for N+1 queries, missing indexes, sequential external calls (make them concurrent), and missing caching.
3. **If it's CPU**: profile with `py-spy` (safe in production) or `cProfile` in staging, find the hot functions, and fix the algorithm or vectorize. Move heavy work to background jobs.
4. **Verify** with a benchmark or load test, and add a regression alert.

**Q2. `cProfile` vs `py-spy`?**
`cProfile` is a deterministic tracer: it hooks every call, so it's accurate about call counts but adds significant overhead, which can distort timings. Use it in development. `py-spy` is a sampling profiler that reads the process's stacks from outside (written in Rust): very low overhead, attaches to a live PID with no restart or code change, and produces flame graphs. That makes it the production choice.

**Q3. Why is NumPy so much faster than Python loops?**
NumPy arrays store raw, typed values contiguously, not pointers to boxed Python objects. Operations run as compiled C loops (often SIMD and multi-threaded BLAS) over that memory, with no per-element type checks, refcounting or bytecode dispatch, and they release the GIL. A Python loop pays interpreter overhead on every element.

**Q4. When is Python simply the wrong tool for performance?**
Hard real-time or ultra-low-latency paths (trading engines, game loops), CPU-heavy logic that can't be vectorized and runs on every request at huge scale, or tight memory limits (embedded). Even then, the usual answer is "Python orchestrates, native code does the hot loop", or a dedicated service in Go, Rust or Java behind an API.

---

# PART 7 — TOOLING & ECOSYSTEM

---

## 42. Packaging, Environments & uv

**What it is:** How Python projects isolate dependencies (**virtual environments**), declare them (**`pyproject.toml`**), pin them (**lockfiles**), and get built into distributable **wheels**. **uv** (from Astral, written in Rust) has become the go-to all-in-one tool, replacing pip, pip-tools, virtualenv, pyenv and pipx, and in many teams Poetry too.

**Why it matters:** Reproducible builds, fast CI, Docker images and dependency conflicts are daily backend concerns.

**Key concepts**

```bash
# Classic workflow
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pip freeze > requirements.txt

# Modern workflow with uv
uv python install 3.13          # manage interpreter versions
uv init my-service              # create pyproject.toml
uv add fastapi "sqlalchemy>=2"  # add a dependency, resolve it, update uv.lock, install into .venv
uv add --dev pytest ruff mypy   # dev dependency group
uv sync --frozen                # install exactly what's in the lockfile (CI/Docker)
uv run pytest                   # run inside the project's environment
uvx ruff check .                # run a tool in an ephemeral environment (like pipx)
uv lock --upgrade-package httpx
```

```toml
# pyproject.toml — the single config file (PEP 517/518/621)
[project]
name = "my-service"
version = "1.4.0"
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.115",
    "sqlalchemy>=2.0,<3",
    "pydantic>=2.7",
]

[dependency-groups]
dev = ["pytest>=8", "ruff", "mypy"]

[project.scripts]
my-service = "my_service.cli:main"      # console entry point

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 100

[tool.pytest.ini_options]
testpaths = ["tests"]
```

- **Virtual environment**: an isolated `site-packages` plus a pointer to an interpreter, so projects don't conflict. Never install project dependencies into the system Python.
- **Applications vs libraries**: applications **pin exact versions with a lockfile** (`uv.lock`, `poetry.lock`, `requirements.txt` with hashes) for reproducible deploys. Libraries declare **ranges** (`>=2,<3`) so they don't force conflicts on their users.
- **Wheel vs sdist**: a wheel (`.whl`) is a pre-built archive that just gets unpacked (fast, no compiler needed, can contain compiled extensions for a platform). An sdist is source that's built at install time.
- **Dependency resolution** finds versions satisfying every transitive constraint (a SAT-like problem). uv is 10–100× faster than pip thanks to Rust, parallel downloads and caching.
- **Docker**: multi-stage build; copy `pyproject.toml` + `uv.lock` first and run `uv sync --frozen --no-dev` (layer caching); copy the app afterwards; run as non-root; use slim base images.
- Private packages: internal indexes (AWS CodeArtifact, Artifactory). Supply-chain safety: hash-pinned lockfiles, `pip-audit`, Dependabot/Renovate.

### Interview Questions

**Q1. Why use virtual environments?** ⭐
They isolate each project's dependencies and interpreter version, so project A can use `requests==2.28` while project B uses `2.32`, and system tools don't break. They make environments reproducible and disposable. In containers the image itself provides isolation, but many teams still use a venv inside it for consistency with uv/Poetry.

**Q2. How do you make Python builds reproducible?** ⭐
Pin the full transitive dependency set in a **lockfile** with hashes (`uv.lock`, `poetry.lock`, `pip-compile --generate-hashes`), pin the Python version (`requires-python`, `.python-version`, the Docker base image tag), install with `--frozen` / `--require-hashes` in CI, and build wheels once and promote the same artifact through environments.

**Q3. What is `pyproject.toml`? What did it replace?**
The standard project file (PEP 518/621) for build-system requirements, project metadata, dependencies, and tool configuration (Ruff, pytest, mypy). It replaced `setup.py` (executable code, unsafe and non-declarative), `setup.cfg`, `requirements*.txt` for declared dependencies, and a pile of per-tool config files.

**Q4. How do you resolve a dependency conflict?**
Read the resolver's error to find the conflicting transitive requirement (`uv tree`, `pipdeptree`). Then: upgrade the package with the tight pin, relax your own constraints, use an override or constraint file as a temporary measure, or replace or vendor the offending library. Avoid over-pinning in libraries, which is the usual root cause.

**Q5. Why has uv become popular?**
Speed (Rust, parallelism, a global cache with hard links), one tool for interpreters, venvs, resolution, locking, running tools and building, cross-platform lockfiles, and pip-compatible commands for gradual migration. In CI and Docker it cuts install times from minutes to seconds.

---

## 43. Code Quality: Linting, Formatting & Type Checking

**What it is:** Automated tools that keep a codebase consistent and catch bugs before runtime:
- **Ruff**: linter and formatter, replacing flake8, isort, pyupgrade, many pylint rules, and largely Black.
- **Type checkers**: mypy, Pyright/Pylance, and Astral's newer `ty`.
- **pre-commit** hooks.
- **Security scanners**: bandit rules (inside Ruff), pip-audit.

**Why it matters:** Senior engineers are expected to set up and justify the quality toolchain and CI gates.

**Key concepts**

```toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "B", "UP", "SIM", "S", "N", "RUF"]   # pycodestyle, pyflakes, isort,
                                                              # bugbear, pyupgrade, simplify, bandit...
[tool.mypy]
strict = true
plugins = ["pydantic.mypy"]
```

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.x.y            # pin to a real release tag
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
```

CI pipeline: `ruff check` → `ruff format --check` → `mypy` / `pyright` → `pytest --cov` → `pip-audit` → build the image.

**Adopting typing in a legacy codebase:** start non-strict; turn on strict per module for new code; add types at module boundaries first; use `# type: ignore[code]` with reasons; ratchet the strictness up over time.

### Interview Questions

**Q1. What does your Python quality toolchain look like?** ⭐
Ruff for linting and formatting (fast, one config), mypy or Pyright for static types (strict on new code), pytest with coverage, pre-commit so developers get feedback before pushing, and the same checks enforced in CI. Add pip-audit or Dependabot for vulnerable dependencies and secret scanning. The goal is that review time is spent on design, not style.

**Q2. Linter vs formatter vs type checker?**
A formatter rewrites layout deterministically (no opinions left to argue about). A linter flags suspicious patterns and style violations: unused imports, mutable defaults (`B006`), bare `except`, insecure calls. A type checker verifies that types flow correctly through code paths (wrong argument types, possible `None`, missing attributes). They catch different classes of bugs, so you want all three.

**Q3. mypy vs Pyright?**
Both implement the typing specs. mypy is the reference implementation with a plugin system (Django, Pydantic, SQLAlchemy plugins). Pyright (Microsoft, powering Pylance in VS Code) is faster, infers more, and gives great IDE feedback. Many teams run Pyright in the editor and mypy or Pyright in CI. `ty` (Astral) is a newer, very fast Rust-based checker.

---

## 44. Testing with Pytest

**What it is:** **pytest** is the standard Python test framework: plain `assert` statements, **fixtures** for setup and dependency injection, **parametrization**, plugins (`pytest-asyncio`, `pytest-cov`, `pytest-xdist`, `pytest-mock`), plus `unittest.mock` for test doubles.

**Why it matters:** Senior candidates are expected to describe a testing strategy, write fixtures and mocks, and avoid flaky tests.

**Key concepts**

```python
# tests/conftest.py — shared fixtures, discovered automatically
import pytest

@pytest.fixture
def user():
    return {"id": 1, "email": "a@x.com"}

@pytest.fixture(scope="session")             # created once per test run
def db_engine():
    engine = create_engine(TEST_DB_URL)
    yield engine                             # teardown runs after the yield
    engine.dispose()

@pytest.fixture
def db_session(db_engine):                   # fixtures can depend on fixtures
    conn = db_engine.connect(); tx = conn.begin()
    yield Session(bind=conn)
    tx.rollback(); conn.close()              # each test is isolated by a rollback

# tests/test_pricing.py
@pytest.mark.parametrize(
    ("amount", "coupon", "expected"),
    [(100, None, 100), (100, "SAVE10", 90), (0, "SAVE10", 0)],
)
def test_apply_coupon(amount, coupon, expected):
    assert apply_coupon(amount, coupon) == expected

def test_rejects_negative():
    with pytest.raises(ValueError, match="negative"):
        apply_coupon(-1, None)

# Mocking: patch where the name is LOOKED UP, not where it's defined
from unittest.mock import patch, AsyncMock

def test_sends_email():
    with patch("app.services.signup.send_email") as send:   # signup.py did `from mail import send_email`
        signup("a@x.com")
        send.assert_called_once_with("a@x.com", subject="Welcome")

def test_env(monkeypatch, tmp_path):         # built-in fixtures
    monkeypatch.setenv("APP_ENV", "test")
    (tmp_path / "cfg.toml").write_text("debug = true")

# Async tests (pytest-asyncio)
@pytest.mark.asyncio
async def test_fetch():
    client = AsyncMock()
    client.get.return_value.status_code = 200
    assert await fetch(client, "/x") == 200

# FastAPI: override dependencies instead of patching
def test_api(client_app):
    app.dependency_overrides[get_repo] = lambda: FakeRepo()
```

**Testing pyramid:** many fast **unit** tests (pure logic, fakes); fewer **integration** tests (real DB via Testcontainers, real Redis); a few **end-to-end** tests. Add **contract tests** between services and property-based tests with **Hypothesis** for parsers and invariants.

**Flaky-test killers:** shared global state, test-order dependence, real time (use `freezegun`/`time-machine`), real network calls (use `respx` / `responses`), randomness without a seed, sleeps in async tests.

### Interview Questions

**Q1. What are pytest fixtures? How are they better than setUp/tearDown?** ⭐
Fixtures are functions that provide test dependencies, injected by **parameter name**. They're composable (fixtures use fixtures), scoped (`function`, `class`, `module`, `session`), handle teardown via `yield`, and are shared through `conftest.py`. Compared with `unittest`'s `setUp`, they're explicit about what each test needs, reusable across files, and avoid inheritance-heavy test classes.

**Q2. Where should you patch with `mock.patch`?** ⭐
Where the object is **looked up**, not where it's defined. If `app/signup.py` does `from app.mail import send_email`, patch `"app.signup.send_email"`; patching `"app.mail.send_email"` won't affect the reference `signup` already holds. Better still, inject dependencies (function parameters, FastAPI `dependency_overrides`) so tests need fewer patches.

**Q3. Mock vs stub vs fake?**
A **stub** returns canned answers. A **mock** also records calls so you can assert on interactions. A **fake** is a working lightweight implementation (an in-memory repository, SQLite instead of Postgres). Prefer fakes and state-based assertions for domain logic; over-mocking couples tests to implementation details and makes refactoring painful. Use mocks at true boundaries (email, payment gateways).

**Q4. How do you test code that uses a database?**
Unit-test the domain logic with a fake repository. Integration-test the repository against a **real** database engine of the same kind (Testcontainers or docker-compose Postgres), not SQLite pretending to be Postgres. Isolate each test with a transaction that's rolled back, or truncate tables between tests. Run migrations in the test setup so schema drift is caught.

**Q5. How do you speed up a slow test suite?**
Profile it (`pytest --durations=20`), run in parallel with `pytest-xdist`, widen fixture scopes for expensive resources (session-scoped DB engine, per-test transactions), replace sleeps with event-driven waits, mock slow external calls, split unit and integration jobs in CI, and only run affected tests on pull requests where possible.

---

## 45. Logging & Configuration

**What it is:** The stdlib `logging` module (loggers → handlers → formatters, with levels), structured logging (`structlog`, JSON output), and configuration via environment variables with typed settings (`pydantic-settings`).

**Why it matters:** Production readiness: debuggable logs, no secrets in code, 12-factor configuration.

**Key concepts**

```python
import logging

logger = logging.getLogger(__name__)          # one logger per module, hierarchical names

def charge(order_id: str, amount: int):
    logger.info("charging order", extra={"order_id": order_id, "amount": amount})
    try:
        gateway.charge(amount)
    except GatewayError:
        logger.exception("charge failed order_id=%s", order_id)   # includes the traceback
        raise

# Configure ONCE, at the application entry point — never in library code
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(name)s %(message)s",
)
# Libraries: logging.getLogger(__name__).addHandler(logging.NullHandler())

# Lazy formatting: arguments are formatted only if the record is emitted
logger.debug("payload=%s", big_payload)       # good
logger.debug(f"payload={big_payload}")        # always formats, even when DEBUG is off
```

```python
# Typed configuration from environment variables
from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import SecretStr

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_prefix="APP_")
    database_url: str
    redis_url: str = "redis://localhost:6379/0"
    debug: bool = False
    api_key: SecretStr                         # masked in repr and logs

settings = Settings()                          # fails fast at startup if config is missing or invalid
```

- **Levels**: DEBUG < INFO < WARNING < ERROR < CRITICAL. Production usually runs at INFO; turn on DEBUG per logger temporarily.
- **Structured (JSON) logs** with request or trace IDs (via `contextvars`) make logs searchable in ELK, Datadog or CloudWatch.
- **Never log** secrets, tokens, passwords or full PII; mask or redact them.
- **`print` vs `logging`**: logging has levels, routing, formatting, timestamps and module names, and libraries can integrate with it. `print` is for CLIs and scripts.

### Interview Questions

**Q1. How do you set up logging in a production Python service?** ⭐
Use module-level loggers (`getLogger(__name__)`); configure them once at startup (`dictConfig`) to write structured JSON to stdout (containers collect stdout); include a timestamp, level, logger name, a request/trace ID (via `contextvars` or OpenTelemetry) and key business IDs; use `logger.exception` in error handlers; keep production at INFO with sampling for noisy paths; redact secrets and PII. Libraries should only add a `NullHandler` and never configure logging.

**Q2. Why use `logger.info("x=%s", x)` instead of an f-string?**
The `%`-style arguments are formatted **only if** the record passes the level and filter checks, so disabled DEBUG logs cost almost nothing. Log aggregators can also group messages by the constant template. f-strings always format, which is wasteful for expensive `repr`s in hot paths. Ruff has a rule that flags f-strings in logging calls.

**Q3. How do you manage configuration and secrets?**
Follow 12-factor: configuration lives in the **environment**, not in code. Validate it at startup with `pydantic-settings` (typed, fails fast). Keep secrets in a secret manager (AWS Secrets Manager, Vault, SSM) and inject them as environment variables or files at runtime. Use `.env` only for local development (git-ignored), and `SecretStr` to avoid logging secrets by accident. Different environments differ only in configuration, never in code.

---

## 46. Web & Backend Python

**What it is:** The Python backend stack.
- **Frameworks**: Django (batteries-included), FastAPI (async, type-driven), Flask (minimal).
- **Interfaces**: WSGI vs ASGI. **Servers**: Gunicorn, Uvicorn, Granian.
- **Data**: SQLAlchemy 2.x and the Django ORM; migrations with Alembic.
- **Background jobs**: Celery, RQ, Dramatiq, Temporal.
- **HTTP clients**: httpx, requests.

**Why it matters:** Most Python backend interviews include framework trade-offs, ORM performance (N+1) and deployment models.

**Key concepts**

| | Django | FastAPI | Flask |
|---|---|---|---|
| Style | Full-stack, batteries included | API-first, type-hint driven | Micro-framework |
| Interface | WSGI (ASGI support growing) | ASGI (async-native) | WSGI |
| ORM / admin / auth | Built in | Bring your own (SQLAlchemy/SQLModel) | Extensions |
| Validation / docs | DRF serializers / Django Ninja | Pydantic + automatic OpenAPI | Extensions |
| Best for | CRUD-heavy products, admin, fast team delivery | High-concurrency APIs, microservices, ML/LLM serving | Small services, legacy code |

```python
# FastAPI: types drive validation, docs and dependency injection
from fastapi import FastAPI, Depends, HTTPException
from pydantic import BaseModel

app = FastAPI()

class OrderIn(BaseModel):
    sku: str
    qty: int

class OrderOut(OrderIn):
    id: int

async def get_repo() -> OrderRepo:          # a dependency (overridable in tests)
    return OrderRepo(session_factory())

@app.post("/orders", response_model=OrderOut, status_code=201)
async def create_order(body: OrderIn, repo: OrderRepo = Depends(get_repo)):
    if body.qty <= 0:
        raise HTTPException(422, "qty must be positive")
    return await repo.create(body)
```

- **WSGI vs ASGI**: WSGI (PEP 3333) is synchronous, one request per worker thread or process, using Gunicorn. ASGI is async and also supports WebSockets, long polling and streaming (SSE, which LLM token streaming relies on), using Uvicorn or Granian.
- **Deployment**: `gunicorn -k uvicorn.workers.UvicornWorker -w <N>` (or `uvicorn --workers N`). Multiple **processes** use all cores; inside each, asyncio handles concurrency. In Kubernetes, it's common to run 1–2 workers per pod and scale with replicas.
- **Sync code inside FastAPI**: a plain `def` endpoint runs in a thread pool; an `async def` endpoint runs on the event loop and **must not block**.
- **The N+1 query problem**: loading N parents, then lazily querying children one at a time. Fix it with eager loading (Django `select_related` for FK/one-to-one via a JOIN, `prefetch_related` for many-to-many and reverse relations via a second query; SQLAlchemy `joinedload`/`selectinload`).
- **Background work**: don't do slow work inside the request. Push it to Celery/RQ/Temporal through a broker (Redis, RabbitMQ, SQS). Make tasks **idempotent** (they may run twice), and use retries with backoff.
- **Connection pooling**: a SQLAlchemy pool per process; size it as roughly workers × pool size against the DB's connection limit; consider PgBouncer.

### Interview Questions

**Q1. Django vs FastAPI vs Flask: how do you choose?** ⭐
Django suits data-heavy products that benefit from the ORM, admin, auth, migrations and a mature ecosystem, and it's the fastest way to ship CRUD with a team. FastAPI suits API-only services, high concurrency, async I/O, streaming (LLM responses), and type-driven validation with automatic OpenAPI docs. Flask suits small, simple services or existing codebases. The decision rests on team familiarity, sync vs async needs, and how much you want built in versus assembled.

**Q2. WSGI vs ASGI?** ⭐
WSGI is a synchronous interface: the server calls the app once per request and blocks a worker until it returns, which is fine for classic request/response. ASGI is an asynchronous interface built around events (`receive`/`send`), supporting thousands of concurrent connections per worker, WebSockets, HTTP/2 and streaming. FastAPI and Starlette are ASGI; Django supports both.

**Q3. What is the N+1 problem, and how do you fix it?** ⭐
Fetching a list of N objects with 1 query, then running 1 query per object to load a relationship, so N+1 round trips. It's invisible in development and fatal at scale. Detect it with query logging, `django-debug-toolbar`, `nplusone`, or APM traces. Fix it with eager loading: Django `select_related` (a JOIN for FK/one-to-one) and `prefetch_related` (a second `IN` query for many-to-many and reverse FKs); SQLAlchemy `joinedload` / `selectinload`. Add pagination and select only the columns you need.

**Q4. How do you run a Python web app in production?** ⭐
Behind a load balancer and reverse proxy, run Gunicorn with Uvicorn workers (or Uvicorn/Granian directly): multiple worker **processes** per container, sized to CPU cores and memory. Add graceful shutdown (SIGTERM handling, connection draining), health and readiness endpoints, request timeouts, DB connection pooling sized to worker count, worker recycling (`max_requests`) to contain leaks, structured logs, metrics and tracing (OpenTelemetry), and horizontal scaling through replicas.

**Q5. How do you handle long-running work triggered by an API call?**
Return `202 Accepted` with a job ID, enqueue the job (Celery, RQ, SQS consumers or Temporal for durable workflows), let workers process it with retries, timeouts and idempotency keys, and expose status through polling, webhooks or WebSocket/SSE notifications. FastAPI's `BackgroundTasks` is only for tiny fire-and-forget work, since tasks die if the process restarts.

**Q6. `requests` vs `httpx`?**
`requests` is sync-only, mature and ubiquitous. `httpx` has a requests-like API in both sync and **async** forms, plus HTTP/2, strict timeouts by default and transport mocking. Inside async code you must use an async client (`httpx.AsyncClient`, `aiohttp`). Always set timeouts and reuse a client or session for connection pooling.

---

## 47. Data, ML & GenAI Ecosystem

**What it is:** The libraries that made Python dominant in data and AI:
- **Scientific stack**: NumPy, pandas, Polars, DuckDB, PyArrow.
- **ML**: scikit-learn, XGBoost/LightGBM, PyTorch, Hugging Face.
- **Pipelines**: Airflow, Dagster, Prefect, dbt, PySpark.
- **GenAI**: LLM SDKs (Anthropic, OpenAI, Google), LangGraph/LangChain, Pydantic AI, LlamaIndex, MCP, vector databases, evals.

**Why it matters:** Even backend roles increasingly involve data pipelines and LLM features, and interviewers ask about the trade-offs.

**Key concepts**

| Area | Tools | Know this |
|---|---|---|
| Arrays | **NumPy** | Vectorization, broadcasting, dtypes, views vs copies |
| DataFrames | **pandas**, **Polars** | pandas: ubiquitous, eager, single-threaded core. Polars: Rust, multi-threaded, lazy query optimization, Arrow memory |
| In-process SQL | **DuckDB** | SQL directly on Parquet/CSV/DataFrames; great for analytics without a server |
| Columnar format | **PyArrow / Parquet** | Columnar, compressed, zero-copy interchange between tools |
| Distributed | **PySpark**, Dask, Ray | Spark for huge ETL; Ray for distributed Python and ML |
| Orchestration | **Airflow**, Dagster, Prefect, dbt | DAGs, scheduling, retries, data lineage |
| Classical ML | **scikit-learn**, XGBoost, LightGBM | Pipelines, cross-validation, gradient boosting for tabular data |
| Deep learning | **PyTorch**, JAX | Tensors, autograd, GPU |
| LLMs and models | **Hugging Face** `transformers`, `peft`; **vLLM**, Ollama | Model hub, fine-tuning (LoRA), high-throughput serving |
| LLM apps / agents | Anthropic/OpenAI SDKs, **LangGraph**, **Pydantic AI**, LlamaIndex, Claude Agent SDK | Tool calling, structured output, RAG, agent loops |
| Tool protocol | **MCP** (Model Context Protocol) Python SDK, FastMCP | Expose tools and data to LLM clients in a standard way |
| Vector search | pgvector, Qdrant, Chroma, Pinecone | Embeddings, ANN indexes, hybrid search |
| Evals and observability | Langfuse, LangSmith, Ragas, DeepEval | Measure quality, trace LLM calls |

```python
# Vectorization & broadcasting
import numpy as np
a = np.arange(6).reshape(2, 3)
a * 10                     # element-wise, no Python loop
a - a.mean(axis=0)         # broadcasting: (2,3) - (3,)
mask = a > 2
a[mask]                    # boolean indexing

# pandas: avoid row loops
import pandas as pd
df = pd.read_parquet("orders.parquet")
df["total"] = df["price"] * df["qty"]                         # vectorized
summary = df.groupby("region", as_index=False)["total"].sum()
# df.apply(lambda r: ..., axis=1)  -> a slow Python loop in disguise

# Polars lazy: the query optimizer pushes filters and column selection down into the scan
import polars as pl
out = (pl.scan_parquet("orders/*.parquet")
         .filter(pl.col("status") == "paid")
         .group_by("region").agg(pl.col("total").sum())
         .collect())

# DuckDB: SQL over files
import duckdb
duckdb.sql("SELECT region, sum(total) FROM 'orders/*.parquet' GROUP BY region").df()

# Structured LLM output with Pydantic (provider-agnostic idea)
from pydantic import BaseModel
class Ticket(BaseModel):
    category: str
    priority: int
    summary: str
# Pass Ticket's JSON schema as the tool/output schema, then Ticket.model_validate_json(response)
```

**A typical RAG pipeline:** load documents → chunk them (by size or semantics, with overlap) → embed → store in a vector DB with metadata → at query time embed the question → hybrid retrieval (vector + keyword) → rerank → build a prompt with the retrieved context → LLM → cite sources. Evaluate retrieval recall and answer faithfulness offline, and trace everything in production.

### Interview Questions

**Q1. pandas vs Polars vs DuckDB?** ⭐
pandas is the default, with the biggest ecosystem; it's eager and mostly single-threaded and memory-heavy, fine up to a few GB. Polars (Rust, Arrow-based) is multi-threaded with a **lazy** query optimizer (predicate and projection pushdown), often several times faster and lighter on memory, and good for bigger-than-RAM work via streaming. DuckDB is an in-process **SQL** OLAP engine that queries Parquet, CSV and DataFrames directly; great when the team thinks in SQL. All three interoperate through Arrow.

**Q2. Why are row-wise `apply` / `iterrows` slow in pandas?**
They run Python code per row: interpreter overhead, boxing values into objects, building a Series per row. Vectorized column operations run in C over contiguous arrays. Replace them with column arithmetic, `np.where` / `np.select`, `.str` / `.dt` accessors, `merge` instead of lookups, and `groupby`/`agg`.

**Q3. How would you build an LLM-powered feature in a Python backend?** ⭐
FastAPI (async) calling the provider SDK with timeouts, retries with backoff, and streaming responses over SSE. Pydantic models for **structured outputs and tool schemas**, validated before use. RAG for grounding (embeddings + pgvector or a vector DB, reranking), and prompt and versions managed in code. Guardrails: input and output validation, PII redaction, prompt-injection awareness when tools have side effects. Cache repeated prompts, enforce token and cost budgets per user, run evals in CI and trace in production (Langfuse/OTel). Run heavy or long agent runs as background jobs.

**Q4. What is MCP?**
The Model Context Protocol is an open protocol for exposing **tools, resources and prompts** to LLM applications in a standard way, through MCP servers over stdio or HTTP. Instead of writing custom integrations for each model or framework, you write one MCP server (e.g. with FastMCP in Python) and any MCP-capable client or agent can use it. It's roughly what LSP did for editors, applied to AI tools.

---

# PART 8 — ARCHITECTURE & PRODUCTION

---

## 48. Design Patterns in Python

**What it is:** The classic Gang-of-Four patterns, translated **idiomatically**. Python's first-class functions, modules, duck typing and decorators make many patterns much lighter than in Java, or unnecessary.

**Why it matters:** Interviewers check whether you apply patterns with judgment rather than copying Java-style ceremony.

**Key concepts**

| Pattern | Java-style | Pythonic version |
|---|---|---|
| **Singleton** | Private constructor + `getInstance()` | A **module** (imported once, cached in `sys.modules`), or `@cache` on a factory function |
| **Strategy** | Interface + one class per algorithm | **Pass a function** (or a callable object); a dict of functions |
| **Factory** | A factory class hierarchy | A function or `@classmethod` (`from_json`), or a registry dict |
| **Decorator** | Wrapper classes | `@decorator` for functions; composition + `__getattr__` for objects |
| **Iterator** | Iterator classes | Generators |
| **Observer** | Subject/Observer interfaces | A list of callbacks; signals (Django, blinker); asyncio events |
| **Command** | Command classes | Functions, closures or `functools.partial` |
| **Template method** | Abstract base class | ABC with hooks, or pass functions in |
| **Adapter** | Adapter class | A small wrapper class, or a Protocol plus a thin wrapper |
| **Builder** | Builder class | Keyword arguments with defaults, dataclasses, `replace()` |
| **Repository** | DAO interfaces | A Protocol (`OrderRepo`) with SQL and in-memory implementations |
| **Dependency injection** | DI containers | Constructor arguments, FastAPI `Depends`, a simple composition root |

```python
# Strategy: functions are strategies
def standard(order): return order.base
def festive(order): return order.base * 0.8
PRICING = {"standard": standard, "festive": festive}
price = PRICING[campaign](order)

# Singleton: module-level instance, or a cached factory
from functools import cache
@cache
def get_settings() -> Settings:
    return Settings()               # created on first call, reused afterwards

# Factory with a registry (open/closed: add a type without editing the factory)
EXPORTERS: dict[str, type["Exporter"]] = {}
def register(name):
    def deco(cls):
        EXPORTERS[name] = cls
        return cls
    return deco

@register("csv")
class CsvExporter: ...
def make_exporter(kind: str) -> "Exporter":
    return EXPORTERS[kind]()

# Observer: callbacks
class EventBus:
    def __init__(self): self._subs: dict[str, list[Callable]] = defaultdict(list)
    def subscribe(self, event, fn): self._subs[event].append(fn)
    def publish(self, event, payload):
        for fn in self._subs[event]:
            fn(payload)

# Adapter: wrap a third-party client behind your own Protocol
class SmsSender(Protocol):
    def send(self, to: str, text: str) -> None: ...
class TwilioAdapter:
    def __init__(self, client): self._c = client
    def send(self, to, text): self._c.messages.create(to=to, body=text, from_=FROM)
```

### Interview Questions

**Q1. How do you implement a Singleton in Python, and should you?** ⭐
The Pythonic singleton is a **module**: it's imported once per process and cached in `sys.modules`, so module-level objects are shared. For lazy creation, use a `@functools.cache` factory. Overriding `__new__` or using a metaclass works, but it's rarely needed. Caveats: a "singleton" exists once **per process** (each Gunicorn/Celery worker has its own), global state hurts testability, and dependency injection of a shared instance is usually better.

**Q2. Show the Strategy pattern in Python.** ⭐
Pass the behaviour in as a function: `sorted(items, key=...)` is itself a strategy. For selectable strategies, use a dict mapping names to functions. Use classes only when a strategy needs its own state or several methods, typed with a Protocol.

**Q3. Which GoF patterns become unnecessary in Python, and why?**
Iterator (generators), Strategy, Command and Template Method (first-class functions and closures), Singleton (modules), Builder (keyword arguments and dataclasses), Decorator (language-level `@`), and the visitor or overloading boilerplate (`singledispatch`, `match`). Dynamic typing and duck typing remove the interface ceremony. The *ideas* still matter; the class hierarchies mostly don't.

**Q4. How do you implement the Repository pattern?**
Define a Protocol (`get`, `add`, `list_by_customer`) in the domain layer, implement it with SQLAlchemy in the infrastructure layer, and with an in-memory dict for tests. Services depend on the Protocol, not the ORM, so domain logic is testable without a database and the storage can change. Pair it with a Unit of Work for transactions.

---

## 49. Clean Architecture, SOLID & Dependency Injection

**What it is:** Principles for structuring maintainable services:
- **SOLID**.
- **Layered / clean / hexagonal architecture**: the domain is at the centre, adapters sit at the edges, and dependencies point inward.
- **Dependency injection**: pass collaborators in instead of constructing them inside.

**Why it matters:** Senior and lead interviews include "how would you structure this service?" and code-review-style questions.

**Key concepts**

```
src/order_service/
├── domain/          # entities, value objects, domain rules — pure Python, no frameworks
│   ├── models.py
│   └── ports.py     # Protocols: OrderRepo, PaymentGateway, EventPublisher
├── application/     # use cases / services orchestrating the domain through ports
│   └── place_order.py
├── adapters/        # implementations: SQLAlchemy repo, Stripe gateway, Kafka publisher
├── api/             # FastAPI routers, request/response schemas (Pydantic)
└── main.py          # composition root: wires concrete adapters into the use cases
```

```python
# domain/ports.py
class OrderRepo(Protocol):
    def add(self, order: Order) -> None: ...
    def get(self, order_id: str) -> Order | None: ...

class PaymentGateway(Protocol):
    def charge(self, customer_id: str, amount: int) -> str: ...

# application/place_order.py — depends only on abstractions
class PlaceOrder:
    def __init__(self, repo: OrderRepo, payments: PaymentGateway):
        self.repo, self.payments = repo, payments

    def __call__(self, cmd: PlaceOrderCmd) -> str:
        order = Order.create(cmd.customer_id, cmd.items)      # domain rules
        order.mark_paid(self.payments.charge(cmd.customer_id, order.total))
        self.repo.add(order)
        return order.id

# api — FastAPI wires the dependencies
def get_place_order() -> PlaceOrder:
    return PlaceOrder(SqlOrderRepo(session()), StripeGateway(settings.stripe_key))

@router.post("/orders")
def place(body: PlaceOrderIn, uc: PlaceOrder = Depends(get_place_order)):
    return {"id": uc(body.to_cmd())}

# tests — no DB, no Stripe
def test_place_order():
    uc = PlaceOrder(InMemoryOrderRepo(), FakePayments(txn="t1"))
    assert uc(cmd)
```

**SOLID in Python terms**
- **S**ingle responsibility: a module or class has one reason to change (don't mix HTTP parsing, business rules and SQL).
- **O**pen/closed: extend through registries, strategies and plugins instead of editing `if/elif` chains.
- **L**iskov substitution: subclasses and Protocol implementations must honour the contract (same exceptions, no stronger preconditions).
- **I**nterface segregation: small, focused Protocols (`Reader`, `Writer`) instead of one god interface.
- **D**ependency inversion: high-level policy depends on Protocols; adapters implement them; the composition root wires them together.

### Interview Questions

**Q1. How would you structure a medium-sized Python backend service?** ⭐
Layer it: **domain** (pure business rules, entities, value objects), **application** (use cases orchestrating the domain through ports/Protocols), **adapters** (DB, external APIs, messaging), and **API** (FastAPI/Django views, Pydantic schemas). Dependencies point inward; wiring happens in one composition root. Benefits: fast unit tests with fakes, framework-independent business logic, and swappable infrastructure. Keep it pragmatic: a simple CRUD service doesn't need every layer.

**Q2. What is dependency injection in Python? Do you need a DI framework?** ⭐
Pass collaborators in (constructor or function parameters) instead of instantiating them inside or importing globals. Duck typing and Protocols make this lightweight, so most projects need no container: a composition root plus FastAPI's `Depends` is enough. Libraries like `dependency-injector` or `punq` help only in very large applications. The payoff is testability (inject fakes) and decoupling.

**Q3. Give an example of violating and then fixing the open/closed principle.**
Violation: `if fmt == "csv": ... elif fmt == "json": ...` inside `export()`, which must be edited for every new format. Fix: a registry of exporter classes or functions keyed by format, with new formats registering themselves (decorator or `__init_subclass__`), so `export()` never changes.

**Q4. How do you keep business logic independent of Django/FastAPI/SQLAlchemy?**
Put the rules in plain Python domain classes and functions that don't import the framework. Access persistence and external services through Protocols. Convert between ORM models or Pydantic schemas and domain objects at the edges. In Django, a common pragmatic middle ground is "fat services, thin views", with domain logic in service modules rather than models or views.

---

## 50. Security Pitfalls

**What it is:** Python-specific and common web security mistakes and how to prevent them.

**Why it matters:** Security questions show up in senior backend loops, and many Python APIs (`pickle`, `eval`, `yaml.load`, `subprocess` with `shell=True`) are dangerous by default.

**Key concepts**

| Risk | Unsafe | Safe |
|---|---|---|
| SQL injection | `f"SELECT ... WHERE id={uid}"` | Parameterized queries or the ORM: `execute("... WHERE id = %s", (uid,))` |
| Command injection | `os.system(f"convert {name}")`, `shell=True` | `subprocess.run(["convert", name], check=True)` (argument list, no shell) |
| Code execution | `eval(user_input)`, `exec` | `ast.literal_eval` for literals; a real parser |
| Deserialization | `pickle.loads(untrusted)`, `yaml.load` | JSON / MessagePack, `yaml.safe_load` |
| Path traversal | `open(BASE + user_path)` | `resolved = (BASE / p).resolve(); resolved.is_relative_to(BASE)` |
| Weak randomness | `random.random()` for tokens | `secrets.token_urlsafe(32)`, `secrets.compare_digest` |
| Password storage | md5/sha256 hashes | `argon2-cffi` / `bcrypt` (slow, salted) |
| Secrets in code | Hard-coded keys, committed `.env` | Secret manager + env vars; secret scanning in CI |
| SSRF | Fetching user-provided URLs blindly | Allow-lists, block internal IP ranges and metadata endpoints |
| XML attacks | `xml.etree` on untrusted input (XXE, billion laughs) | `defusedxml` |
| Supply chain | Unpinned dependencies, typosquatting | Lockfiles with hashes, `pip-audit`, trusted indexes |
| `assert` for checks | `assert user.is_admin` | `if not ...: raise PermissionError` (asserts vanish under `-O`) |
| Timing attacks | `token == expected` | `hmac.compare_digest(token, expected)` |

```python
import secrets, hmac, subprocess, ast
from pathlib import Path

api_token = secrets.token_urlsafe(32)
hmac.compare_digest(provided, expected)                 # constant-time comparison

subprocess.run(["git", "clone", repo_url, dest], check=True, timeout=60)   # no shell

ast.literal_eval("[1, 2, {'a': 3}]")                    # literals only, nothing executes

BASE = Path("/srv/uploads").resolve()
def safe_path(user_path: str) -> Path:
    p = (BASE / user_path).resolve()
    if not p.is_relative_to(BASE):                      # 3.9+
        raise PermissionError("path traversal")
    return p
```

### Interview Questions

**Q1. What are the most dangerous Python functions to use with untrusted input?** ⭐
`eval`/`exec` (arbitrary code), `pickle.loads` / `shelve` / `joblib` loading (code execution during deserialization), `yaml.load` without `SafeLoader`, `subprocess` with `shell=True` or `os.system` (command injection), string-built SQL (injection), `xml` parsers on untrusted XML (XXE/DoS; use `defusedxml`), and `tarfile.extractall` without filters (path traversal; use the `filter="data"` argument).

**Q2. How do you prevent SQL injection in Python?** ⭐
Never interpolate input into SQL strings, including with f-strings or `.format`. Use **parameterized queries** (driver placeholders like `%s` or `?` with an argument tuple) or an ORM / query builder (SQLAlchemy Core or ORM, Django ORM), which parameterize for you. For dynamic identifiers (column or table names), validate against an allow-list. Add least-privilege DB users as defence in depth.

**Q3. `random` vs `secrets`?**
`random` is a Mersenne Twister PRNG: fast and reproducible, but **predictable** (its state can be recovered from its outputs), so it's for simulations and tests only. `secrets` uses the OS CSPRNG and is the module for tokens, password resets, API keys and session IDs. Compare secrets with `hmac.compare_digest` to avoid timing leaks.

---

## 51. Running Python Services in Production

**What it is:** Everything around the code that keeps a Python service reliable: process model and worker sizing, containers, health checks and graceful shutdown, observability, resilience patterns and scaling.

**Why it matters:** Senior interviews probe operational maturity: "Your service's p99 doubled after a deploy. What do you do?"

**Key concepts**

- **Process model**: N worker processes per container, with roughly `(2 × cores) + 1` as the Gunicorn sync-worker heuristic; async workers usually equal the core count. On Kubernetes, prefer fewer workers per pod and more pods, and set CPU/memory requests and limits consistently.
- **Worker hygiene**: `--max-requests` with jitter (recycles workers to contain leaks), `--timeout`, `gc.freeze()` before fork to reduce copy-on-write memory, and preloading the app (`--preload`) to share memory.
- **Graceful shutdown**: handle SIGTERM; stop accepting new requests; finish in-flight ones within the termination grace period; close DB pools and flush telemetry. Use FastAPI lifespan events for startup and shutdown resources.
- **Health checks**: *liveness* (the process is responsive) vs *readiness* (dependencies reachable, warmed up). Don't make liveness depend on the DB, or one DB blip restarts every pod at once.
- **Observability**: structured logs, metrics (request rate, errors, duration via Prometheus/OTel; worker saturation; event-loop lag), distributed tracing (OpenTelemetry auto-instrumentation for FastAPI, SQLAlchemy, httpx), error tracking (Sentry).
- **Resilience**: timeouts on **every** outbound call, retries with exponential backoff and jitter (only for idempotent operations; `tenacity`), circuit breakers, bulkheads (separate pools), rate limiting, idempotency keys for POSTs, backpressure through bounded queues.
- **Data layer**: connection pools sized to workers × pool size within DB limits, PgBouncer, read replicas, caching (Redis) with TTLs and stampede protection.
- **Containers**: slim base image, multi-stage build with uv, non-root user, `PYTHONUNBUFFERED=1` (logs flush immediately), `PYTHONDONTWRITEBYTECODE=1`, pinned base image digest, and an image vulnerability scan.

```dockerfile
FROM python:3.13-slim AS build
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
WORKDIR /app
COPY pyproject.toml uv.lock ./
RUN uv sync --frozen --no-dev --no-install-project
COPY src ./src
RUN uv sync --frozen --no-dev

FROM python:3.13-slim
WORKDIR /app
COPY --from=build /app /app
ENV PATH="/app/.venv/bin:$PATH" PYTHONUNBUFFERED=1
USER 1000
CMD ["gunicorn", "my_service.main:app", "-k", "uvicorn.workers.UvicornWorker", "-w", "2", "-b", "0.0.0.0:8000"]
```

### Interview Questions

**Q1. How many workers should a Python web service run?** ⭐
It depends on the workload. Sync (WSGI) workers block on I/O, so the classic heuristic is `2 × cores + 1` (or add threads per worker). Async (ASGI) workers handle many concurrent requests each, so roughly one per core. Constraints: memory per worker (each is a full interpreter plus the app), DB connection limits (workers × pool size), and in Kubernetes it's often better to run 1–2 workers per pod and scale pods horizontally. Validate with load tests by watching latency, CPU and memory.

**Q2. p99 latency doubled after a deploy. How do you investigate?** ⭐
1. **Correlate**: was it only this deploy? Which endpoints are affected? Compare the trace breakdown before and after (DB time, external calls, CPU, queueing).
2. **Common Python culprits**: a new N+1 query, a blocking call in an async path (event-loop lag metrics), a missing index, a reduced worker count or pool size, a heavier serialization path, GC pauses from more allocations, or an outbound call with no timeout.
3. **Mitigate first**: roll back or apply a feature flag.
4. **Root-cause** with `py-spy` on a canary pod plus the traces, add a regression test or load test, and alert on the SLO.

**Q3. How do you make outbound calls resilient?**
Always set connect and read timeouts. Retry only idempotent operations, with exponential backoff, jitter and a cap (`tenacity`). Use circuit breakers to fail fast when a dependency is down, bulkheads (separate pools or semaphores per dependency), and fallbacks (cached or default responses). Add idempotency keys for writes so retries don't double-charge, and monitor the error rate and latency of each dependency.

**Q4. How do you shut down a Python service gracefully?**
On SIGTERM: fail the readiness check so traffic drains, stop accepting new work, let in-flight requests and tasks finish (bounded by the grace period), cancel asyncio tasks and await their cleanup, close DB and HTTP pools, and flush logs, metrics and traces. Celery workers should use warm shutdown and `acks_late`, so unfinished tasks are redelivered rather than lost.

---

# APPENDICES

---

## Appendix A. Predict the Output

Cover the answer, predict it, then check. These are the traps interviewers use most.

**1. Mutable default**
```python
def f(x, acc=[]):
    acc.append(x); return acc
f(1); print(f(2))
```
<details><summary>Answer</summary>

`[1, 2]`: the default list is created once, at definition time (§15).
</details>

**2. Late-binding closures**
```python
fs = [lambda: i for i in range(3)]
print([f() for f in fs])
```
<details><summary>Answer</summary>

`[2, 2, 2]`: closures look up `i` when called (§17).
</details>

**3. Shared rows**
```python
grid = [[0] * 2] * 2
grid[0][0] = 9
print(grid)
```
<details><summary>Answer</summary>

`[[9, 0], [9, 0]]`: both rows are the same list object (§14).
</details>

**4. `+=` on a list vs rebinding**
```python
a = [1]; b = a
a += [2]
a = a + [3]
print(a, b)
```
<details><summary>Answer</summary>

`[1, 2, 3] [1, 2]`: `+=` mutates in place (`b` sees it); `a + [3]` creates a new list (§6).
</details>

**5. Tuple containing a list**
```python
t = ([1],)
try:
    t[0] += [2]
except TypeError as e:
    print("error")
print(t)
```
<details><summary>Answer</summary>

Prints `error` and then `([1, 2],)`. The list is extended in place, *then* assigning the result back into the tuple fails.
</details>

**6. Floor division and modulo**
```python
print(-7 // 2, -7 % 3, round(2.5), round(3.5))
```
<details><summary>Answer</summary>

`-4 2 2 4`: floor toward −∞; modulo takes the divisor's sign; banker's rounding (§5, §6).
</details>

**7. `and` / `or` return operands**
```python
print(0 or [] or "x", "a" and 0 and "b")
```
<details><summary>Answer</summary>

`x 0`
</details>

**8. Class attribute shared by instances**
```python
class A:
    items = []
a1, a2 = A(), A()
a1.items.append(1)
a2.items = [9]
print(A.items, a1.items, a2.items)
```
<details><summary>Answer</summary>

`[1] [1] [9]`: the mutation goes to the shared class list; the assignment creates an instance attribute on `a2` only (§21).
</details>

**9. `strip` takes a set of characters**
```python
print("docs.csv".rstrip(".csv"))
```
<details><summary>Answer</summary>

`do` (§8). Use `removesuffix`.
</details>

**10. Exhausted generator**
```python
g = (x * x for x in range(3))
print(sum(g), sum(g))
```
<details><summary>Answer</summary>

`5 0`: generators are single-use (§28).
</details>

**11. `finally` overrides `return`**
```python
def f():
    try:
        return "try"
    finally:
        return "finally"
print(f())
```
<details><summary>Answer</summary>

`finally`. A return in `finally` overrides the earlier one and would also swallow exceptions. Don't do this.
</details>

**12. MRO**
```python
class A:
    def who(self): return "A"
class B(A):
    def who(self): return "B" + super().who()
class C(A):
    def who(self): return "C" + super().who()
class D(B, C):
    def who(self): return "D" + super().who()
print(D().who())
```
<details><summary>Answer</summary>

`DBCA`: the MRO is D → B → C → A, and `super()` follows it (§22).
</details>

**13. Dict keys: `1`, `1.0` and `True`**
```python
d = {1: "int", 1.0: "float", True: "bool"}
print(d)
```
<details><summary>Answer</summary>

`{1: 'bool'}`. `1 == 1.0 == True` and their hashes are equal, so it's one key: the first key object is kept and the last value wins (§10).
</details>

**14. Scope**
```python
x = 1
def f():
    print(x)
    x = 2
f()
```
<details><summary>Answer</summary>

`UnboundLocalError`: the assignment makes `x` local for the whole function (§4).
</details>

**15. `is` with equal values**
```python
a = [1, 2]; b = [1, 2]
print(a == b, a is b, None is None)
```
<details><summary>Answer</summary>

`True False True`
</details>

**16. Loop `else`**
```python
for n in [1, 3, 5]:
    if n % 2 == 0:
        break
else:
    print("no even")
```
<details><summary>Answer</summary>

`no even`: `else` runs because the loop never hit `break` (§7).
</details>

**17. Comprehension with a condition**
```python
print([x if x % 2 else -x for x in range(4)])
```
<details><summary>Answer</summary>

`[0, 1, -2, 3]`
</details>

**18. Unpacking**
```python
a, *b, c = "hello"
print(a, b, c)
```
<details><summary>Answer</summary>

`h ['e', 'l', 'l'] o`
</details>

**19. Equal but unhashable**
```python
class P:
    def __init__(self, x): self.x = x
    def __eq__(self, o): return self.x == o.x
print(P(1) == P(1))
try:
    print({P(1)})
except TypeError as e:
    print("TypeError")
```
<details><summary>Answer</summary>

`True`, then `TypeError`: defining `__eq__` sets `__hash__ = None` (§10).
</details>

**20. Blocking inside async code**
```python
import asyncio, time
async def job(n):
    time.sleep(1); return n
async def main():
    t = time.perf_counter()
    await asyncio.gather(job(1), job(2), job(3))
    print(round(time.perf_counter() - t))
asyncio.run(main())
```
<details><summary>Answer</summary>

`3`: `time.sleep` blocks the event loop, so the jobs run one after another. With `await asyncio.sleep(1)` it would be `1` (§39).
</details>

---

## Appendix B. Revision Cheat Sheet

| # | Topic | One-line takeaway |
|---|---|---|
| 1 | Compiled or interpreted | Compiled to bytecode, interpreted by the PVM; PyPy and the 3.13+ JIT go further |
| 1 | Why Python for AI | Ecosystem + readability + C/GPU interop; Python orchestrates, native code computes |
| 3 | Objects & names | Everything is an object; variables are names bound to objects; assignment never copies |
| 3 | `is` vs `==` | Identity vs value; `is` only for `None` and singletons |
| 3 | Typing | Dynamic (types on objects) + strong (no implicit coercion) |
| 4 | Scope | LEGB; assignment makes a name local; no block scope; `global`/`nonlocal` to rebind |
| 5 | Mutability | Immutable: numbers, str, tuple, frozenset, bytes. Mutable: list, dict, set, most objects |
| 5 | Floats | `0.1 + 0.2 != 0.3`; `math.isclose`; `Decimal` for money; `round` uses banker's rounding |
| 6 | Operators | `//` floors toward −∞; `and`/`or` return operands; no `++`; list `+=` mutates in place |
| 8 | Strings | Immutable; `"".join` not `+=`; `strip` takes a char set; `str` ≠ `bytes`; never f-string SQL |
| 9 | list / dict / set | list: append O(1)*, insert(0) O(n); dict/set: O(1) avg hash tables; dict keeps insertion order |
| 10 | Hashing | Equal ⇒ same hash; `__eq__` without `__hash__` ⇒ unhashable; mutable keys get lost |
| 11 | Comprehension vs genexpr | Eager list (O(n) memory, reusable) vs lazy generator (O(1), single-use) |
| 12 | Sorting | Timsort, stable, O(n log n), O(n) on sorted data; `key=` tuples for multi-key sorts |
| 13 | collections | `Counter` counting; `defaultdict` grouping; `deque` O(1) ends; `heapq` top-k; `bisect` binary search |
| 14 | Copying | Assignment = alias; shallow shares nested objects; deepcopy is independent (and handles cycles) |
| 15 | Argument passing | Pass by object reference: mutation is visible to the caller, rebinding is not |
| 15 | Mutable defaults | Evaluated once at def time; use a `None` sentinel / `default_factory` |
| 15 | Parameters | `def f(pos, /, normal, *, kw)`; `*args` is a tuple, `**kwargs` a dict |
| 17 | Closures | Capture variables (cells), not values; late binding in loops; fix with a default arg or factory |
| 18 | Decorators | `f = d(f)` at definition time; factory for arguments; always `functools.wraps` |
| 19 | lru_cache | Hashable args; per process; leaks `self` on methods; only for pure functions |
| 20 | Recursion | ~1000 limit, no tail-call optimization; go iterative or memoize |
| 21 | Methods | instance (`self`), classmethod (`cls`, alternative constructors), staticmethod (namespaced util) |
| 21 | `__new__` vs `__init__` | Create vs initialize; `__new__` for immutables and singletons |
| 22 | MRO / super | C3 linearization; `super()` = next class in the MRO, not "parent"; pass `**kwargs` along |
| 23 | Interfaces | Duck typing; ABC = nominal (inherit); Protocol = structural (static) |
| 24 | Data model | Dunders plug objects into syntax; `__repr__` first; return `NotImplemented` for unknown types |
| 25 | Descriptors | `__get__`/`__set__` power property, methods, ORM fields; data descriptors beat the instance dict |
| 26 | dataclass / slots | Generated `__init__`/`__repr__`/`__eq__`; frozen ⇒ hashable; slots ⇒ less memory; Pydantic at boundaries |
| 27 | Metaclasses | The class of a class; prefer decorators / `__init_subclass__` first |
| 28 | Generators | Lazy iterators that pause at `yield`; O(1) memory; single pass; `yield from` delegates |
| 29 | Context managers | `__enter__`/`__exit__` or `@contextmanager`; cleanup guaranteed; a truthy `__exit__` suppresses |
| 30 | Exceptions | Catch specific; no bare `except`; `raise ... from e`; domain hierarchy; EAFP |
| 31 | Files & serialization | `with open(..., encoding=)`; stream big files; pickle is unsafe on untrusted data |
| 32 | Imports | Executed once, cached in `sys.modules`; `__main__` guard; fix circular imports by restructuring |
| 33 | Type hints | Not enforced at runtime; mypy/Pyright statically; Pydantic validates at runtime |
| 34 | CPython | Source → AST → bytecode → eval loop; 3.11 specializing interpreter; built-ins run loops in C |
| 35 | Memory | Refcounting (immediate) + cyclic GC; weakref for caches and back-pointers; tracemalloc/memray for leaks |
| 36 | GIL | One thread runs bytecode at a time; released on I/O; not thread safety; free-threading in 3.14t |
| 37 | Threads | For blocking I/O; `Lock` for compound ops; `queue.Queue` for hand-off; consistent lock order |
| 38 | Processes | For CPU work; separate GILs; pickling cost; `__main__` guard; Linux default is forkserver from 3.14 |
| 39 | asyncio | Single-thread event loop; `await` yields; never block it; TaskGroup; `to_thread` / executors |
| 40 | Choosing | I/O → asyncio or threads; CPU → processes or native code; durable jobs → task queue |
| 41 | Performance | Measure (py-spy) → fix I/O and algorithms → vectorize → native code |
| 42 | Packaging | `pyproject.toml` + lockfile; apps pin, libraries use ranges; uv for speed |
| 43 | Quality | Ruff (lint + format) + mypy/Pyright + pre-commit + CI |
| 44 | Testing | pytest fixtures, parametrize; patch where it's looked up; fakes over mocks; real DB in integration tests |
| 45 | Logging & config | `getLogger(__name__)`; lazy `%s` arguments; JSON logs; config from env via pydantic-settings |
| 46 | Web | Django = batteries, FastAPI = async + types; WSGI vs ASGI; fix N+1 with eager loading |
| 47 | Data & AI | Vectorize; Polars/DuckDB for speed; Pydantic for LLM structured output; RAG = retrieve → rerank → generate |
| 48 | Patterns | Module = singleton, function = strategy, generator = iterator, registry = factory |
| 49 | Architecture | Domain at the centre, Protocol ports, adapters at the edges, a composition root for DI |
| 50 | Security | No eval/pickle/shell=True on input; parameterized SQL; `secrets` for tokens |
| 51 | Production | Worker processes sized to cores and DB limits; timeouts and retries; graceful SIGTERM; OTel |

---

*Practice: explain each ⭐ answer out loud in about 60–90 seconds, then expect a follow-up asking "why?" or "what are the trade-offs?".*
