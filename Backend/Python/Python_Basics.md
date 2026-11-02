# Python Interview Prep

**Goal:** Become interview ready of 4 year experience

## Flow of topics/questions

- A. Language Fundamentals
- B. Variables & Constants
- C. Data Types
- D. Operators
- E. Conditional Statements
- F. Loops & Iteration
- G. Functions
- H. Input & Output
- I. Collections / Data Structures
- J. Strings
- K. Scope & Lifetime
- L. Error & Exception Handling
- M. Modules, Packages & Imports
- N. Object-Oriented Programming
- O. Memory & Execution
- P. Concurrency & Asynchronous Programming
- Q. Functional Programming Concepts
- R. File & Resource Handling
- S. Language-Specific Features

---

## A. Language Fundamentals

### Why, when and by whom the Language was developed?

Python was created by Guido van Rossum and first released in 1991 while he was working at CWI (Centrum Wiskunde & Informatica) in the Netherlands. He began the project in December 1989 as a hobby to keep himself occupied during the Christmas holidays, as a successor to the ABC language he had worked on earlier. ABC was easy to learn but not extensible enough for real-world use, so Guido wanted a language that combined ABC's readability with the extensibility and practicality needed for systems programming. He named it "Python" after the British comedy show "Monty Python's Flying Circus," not the snake, reflecting his intent to make programming fun. Python's development has since been guided by the Python Software Foundation, with major milestones being Python 2.0 (2000) and Python 3.0 (2008), the latter being a backward-incompatible redesign to clean up language inconsistencies.

> **Interview one-liner:** Python started as a general-purpose scripting and glue language. It became the main language for data science and AI only in the 2000s–2010s, thanks to NumPy, pandas, scikit-learn, TensorFlow and PyTorch. Before that the leaders were Lisp/Prolog for AI, R/MATLAB/SAS for data science, Perl/Java for NLP, and Java/Scala for big data.

### Core problem it solved?

Before Python, developers often had to choose between low-level languages like C, which offered performance and control but demanded manual memory management and verbose syntax, and scripting languages that were easy to use but lacked robustness for larger programs. Python aimed to solve the problem of developer productivity versus power: writing correct, readable, and maintainable code quickly without sacrificing the ability to build serious, scalable applications. It addressed the steep learning curve and boilerplate-heavy nature of languages like C/C++/Java by offering a clean, English-like syntax, automatic memory management, and a "batteries included" standard library. This made it possible to prototype fast, glue different systems together, and still scale to production use, reducing the gap between an idea and working code.

### How it solved?

Python solved these problems through several deliberate design choices:

- **Readability via significant indentation**
  - Uses indentation instead of braces.
  - Makes code visually consistent across projects.
- **Dynamic typing & automatic memory management**
  - No manual type declarations.
  - Garbage collection removes manual memory handling.
- **Huge standard library ("batteries included")**
  - Covers file I/O, networking, text processing, and more.
  - Reduces dependence on external tools.
- **Interpreted, high-level language**
  - Rapid iteration through a REPL.
  - No separate compile step.
- **Multi-paradigm support**
  - Procedural, object-oriented, and functional.
  - Developers pick the best approach per problem.
- **Simple, extensible C API**
  - Performance-critical code can be written in C/C++ and exposed to Python.
  - Gives the best of both: readability and speed.

### What is the language?

Python is a high-level, general-purpose, interpreted programming language known for its clean, readable syntax and dynamic typing. It supports multiple programming paradigms, including procedural, object-oriented, and functional programming, and is designed to be highly readable and expressive, often described as "executable pseudocode." Python emphasizes code readability through the use of significant whitespace/indentation rather than curly braces or keywords to delimit blocks. It comes with an extensive standard library and a massive ecosystem of third-party packages (via PyPI), making it suitable for web development, data science, automation, scripting, artificial intelligence, and more. The reference implementation, CPython, compiles source code to bytecode which is then executed by the Python Virtual Machine, though alternative implementations (PyPy, Jython, IronPython) also exist.

### What are its key characteristics?

Python's key characteristics include: (1) simple, readable syntax using indentation for blocks; (2) dynamic typing, where variable types are determined at runtime; (3) interpreted execution via bytecode on a virtual machine; (4) automatic memory management through reference counting plus a cyclic garbage collector; (5) multi-paradigm support for procedural, object-oriented, and functional styles; (6) "batteries included" standard library covering common tasks; (7) strong typing, meaning implicit unsafe conversions between unrelated types are disallowed; (8) extensibility via C/C++ extension modules; (9) cross-platform portability, running on Windows, macOS, Linux, and more; (10) a large, active open-source community and package ecosystem (PyPI); and (11) first-class support for interactive development through the REPL and notebooks, which makes exploration and rapid prototyping easy.

### What are its use cases? when to choose it?

Python is widely used in web development (Django, Flask, FastAPI), data science and machine learning (NumPy, pandas, scikit-learn, PyTorch, TensorFlow), automation and scripting (system administration, DevOps tooling), test automation, scientific computing, artificial intelligence, backend APIs, and rapid prototyping. It's also popular for education due to its gentle learning curve. Choose Python when developer productivity, readability, and time-to-market matter more than raw execution speed, when you need access to rich data science/ML libraries, when building glue code integrating multiple systems, or when the team values maintainability over micro-optimized performance. It's less ideal for hard real-time systems, mobile app development, or scenarios demanding maximum CPU performance without relying on compiled extensions, where languages like C++, Rust, or Go may be better suited.

> **Interview one-liner (Why Python for Data Science/AI, not Java/JS?):** Python dominates data science and AI because of its ecosystem (NumPy, pandas, PyTorch, Hugging Face), its readable syntax suited to quick experiments in Jupyter, and its easy C/C++ interop. The heavy computation runs in optimized native or GPU code, so Python's slowness doesn't matter. Java is too verbose and its ML ecosystem is thin. JavaScript lacks strong numeric types and a scientific-computing base. Both are used around Python: Java for big-data infrastructure and enterprise serving, JS for in-browser inference.

### What are its advantages and limitations?

**Advantages:** easy to learn and read, which speeds up development and onboarding; a vast standard library and third-party ecosystem; strong community support; multi-paradigm flexibility; excellent for prototyping and data science; cross-platform; and dynamic typing that speeds up iteration.

**Limitations:** slower execution speed compared to compiled languages like C++, Java, or Go, since it's interpreted and dynamically typed; the Global Interpreter Lock (GIL) in CPython restricts true multi-threaded CPU-bound parallelism; higher memory consumption due to object overhead; runtime type errors that a statically typed language would catch at compile time; and weaker mobile/front-end development support compared to JavaScript or Swift/Kotlin.

These trade-offs are usually acceptable because Python's productivity gains outweigh performance costs for most non-performance-critical applications, and performance-critical parts can be offloaded to C extensions.

### Is it compiled, interpreted, or both?

Python is technically both, though it's commonly labeled "interpreted." When you run a Python script, the CPython interpreter first compiles the source code into an intermediate, platform-independent bytecode (`.pyc` files), similar in spirit to Java's bytecode. This bytecode is then executed by the Python Virtual Machine (PVM), which interprets it line by line at runtime. This differs from purely compiled languages (like C), where source is translated directly into native machine code ahead of time, and from purely interpreted languages that execute source line-by-line without an intermediate form. This hybrid model gives Python portability (bytecode runs anywhere the PVM exists) while retaining a fast edit-run cycle. Just-In-Time compilers like PyPy go further and compile bytecode to native machine code at runtime for speed.

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)     # shows the bytecode the PVM actually executes
# LOAD_FAST    a
# LOAD_FAST    b
# BINARY_OP    0 (+)     # exact opcode names vary by Python version
# RETURN_VALUE

# Compile ahead of time (imported modules are cached like this automatically):
# $ python -m py_compile app.py   ->  __pycache__/app.cpython-312.pyc
```

### What is the runtime/execution environment?

Python's runtime environment is the Python Virtual Machine (PVM), which executes the compiled bytecode. The most common implementation is CPython (written in C), which includes the compiler (source-to-bytecode), the PVM (bytecode interpreter), memory manager, and the Global Interpreter Lock that serializes thread execution for memory safety. Other implementations include PyPy (uses JIT compilation for speed), Jython (runs on the JVM), and IronPython (runs on .NET's CLR). The runtime also manages the import system, module caching (`sys.modules`), object allocation/deallocation via reference counting and cyclic garbage collection, and exception handling. Virtual environments (`venv`, `virtualenv`, `conda`) isolate runtime dependencies per project, ensuring consistent execution environments across different machines and deployments.

```python
import sys, platform, json

platform.python_implementation()   # 'CPython' (or 'PyPy', 'Jython', ...)
sys.version_info                    # sys.version_info(major=3, minor=12, ...)
sys.executable                      # path of the interpreter running this code
"json" in sys.modules               # True — imported modules are cached here

# Isolate dependencies per project:
# $ python -m venv .venv
# $ source .venv/bin/activate
# $ pip install requests
```

### What is the difference between syntax and semantics?

Syntax refers to the set of rules that define how valid statements and expressions must be structured, i.e., the grammar of the language, such as correct indentation, use of colons after `if`/`for`/`def`, and proper parenthesis matching. A syntax error occurs when code violates these structural rules, e.g., missing a colon after `if x > 5`, and is caught before the program runs. Semantics refers to the meaning behind syntactically correct code, i.e., what the code actually does when executed. Code can be syntactically valid but semantically wrong, such as dividing by zero, using an undefined variable, or applying incorrect logic that runs without crashing but produces the wrong result. In short, syntax is about "is this written correctly?" while semantics is about "does this do what I intended?"

```python
# Syntax error — caught when the file is parsed, before any line runs
# if x > 5          # SyntaxError: expected ':'
#     print(x)

# Syntactically valid, semantically wrong — fails only at runtime
def average(nums):
    return sum(nums) / len(nums)
average([])          # ZeroDivisionError

# Syntactically valid, runs without error, but the logic is wrong
def is_adult(age):
    return age > 18  # bug: 18-year-olds are excluded (should be >=)
```

### What are keywords and reserved words?

Keywords (also called reserved words) are predefined, reserved identifiers that have special meaning in the language and cannot be used as variable, function, or class names. In Python, there are 35 keywords (as of Python 3.x), including `False`, `None`, `True`, `and`, `as`, `assert`, `async`, `await`, `break`, `class`, `continue`, `def`, `del`, `elif`, `else`, `except`, `finally`, `for`, `from`, `global`, `if`, `import`, `in`, `is`, `lambda`, `nonlocal`, `not`, `or`, `pass`, `raise`, `return`, `try`, `while`, `with`, and `yield`. These words are part of the language's core syntax and grammar, used to define control flow, declare functions/classes, handle exceptions, and more. You can check them programmatically via the `keyword` module using `keyword.kwlist`. Attempting to use a keyword as a variable name (e.g., `class = 5`) raises a `SyntaxError`, since the interpreter cannot distinguish it from its structural role.

```python
import keyword

len(keyword.kwlist)          # 35 (3.10+)
keyword.iskeyword("class")   # True
keyword.softkwlist           # ['_', 'case', 'match', 'type'] (3.12+)

# class = 5                  # SyntaxError — hard keyword
match = 5                    # OK — soft keywords are only special in context
```

### What are identifiers?

Identifiers are the names used to identify variables, functions, classes, modules, or other objects in code. In Python, valid identifiers must start with a letter (a-z, A-Z) or an underscore (`_`), followed by any combination of letters, digits (0-9), or underscores; they cannot start with a digit and cannot be a reserved keyword. Python identifiers are case-sensitive, so `myVar` and `myvar` are distinct. Since Python 3, identifiers can also include Unicode characters, allowing names in non-English scripts. Identifiers beginning with an underscore (e.g., `_private`) conventionally signal "internal use," a double underscore prefix (e.g., `__mangled`) triggers name mangling in classes, and dunder names (e.g., `__init__`) are reserved for special/magic methods. Choosing meaningful identifiers is a best practice for readability, even though the interpreter itself has no such requirement.

```python
user_name = "a"        # valid
_count = 0             # valid — leading underscore signals internal use
total2 = 10            # valid
café = 3.5             # valid — Unicode letters allowed in Python 3
# 2total = 10          # SyntaxError — can't start with a digit
# user-name = "a"      # SyntaxError — '-' is the minus operator

myVar, myvar = 1, 2    # two different names (case-sensitive)
"total2".isidentifier()   # True — check validity programmatically
```

### What are comments?

Comments are non-executable lines in source code used to explain logic, document intent, or temporarily disable code, and are ignored by the interpreter. In Python, single-line comments start with a hash symbol (`#`), extending to the end of the line, e.g., `# this calculates total price`. Python doesn't have a dedicated multi-line comment syntax; instead, developers commonly use consecutive single-line `#` comments, or (less correctly) use triple-quoted strings (`"""..."""`) as an unassigned string literal, which is technically a string expression, not a true comment, though it's ignored functionally when not assigned. A special form of a triple-quoted string right after a function, class, or module definition becomes a "docstring," accessible via the `__doc__` attribute, used for documentation tools. Good commenting practice explains "why" something is done rather than restating "what" the code already makes obvious.

```python
# Retry because the payment API sometimes times out under load
retries = 3  # inline comment

def area(r):
    """Return the area of a circle with radius r."""   # docstring
    return 3.14159 * r ** 2

area.__doc__   # 'Return the area of a circle with radius r.'
help(area)     # docstrings are what help() and doc tools display
```

### What are naming conventions?

Python's naming conventions are largely defined by PEP 8, the official style guide. Variables and function names should use "snake_case" (lowercase words separated by underscores), e.g., `total_price`, `calculate_sum()`. Class names should use "PascalCase" (also called CapWords), e.g., `class UserAccount:`. Constants are written in "UPPER_SNAKE_CASE", e.g., `MAX_RETRIES = 5`, though Python has no true immutable constants—it's purely a convention. Module and package names should be short, all-lowercase, and can use underscores if needed, e.g., `data_utils.py`. A single leading underscore (e.g., `_helper`) indicates an internal/protected use of a name by convention, and a double leading underscore (e.g., `__value`) triggers name mangling inside classes to avoid subclass attribute clashes. Dunder names like `__init__` are reserved for Python's special methods. Following these conventions improves readability and signals intent to other developers.

```python
MAX_RETRIES = 5                 # constant
user_count = 0                  # variable

def calculate_total(): ...      # function

class UserAccount:              # class
    def __init__(self):
        self._balance = 0       # internal by convention
        self.__pin = 1234       # name-mangled to _UserAccount__pin

acct = UserAccount()
# acct.__pin                    # AttributeError
acct._UserAccount__pin          # 1234 — mangling avoids clashes, it doesn't secure
```

---

## B. Variables & Constants

### How to declare a variable?

Python doesn't have a separate variable declaration step like `int x;` in C/Java. A variable is created the moment you assign a value to a name, e.g., `x = 10`. There's no need to specify a type ahead of time because Python is dynamically typed—the interpreter infers the type from the assigned value at runtime. The name (identifier) simply becomes a reference/label pointing to an object in memory. You can also declare a variable's intended type using type hints for readability and static-analysis tools, e.g., `x: int = 10`, but this is optional and not enforced by the interpreter itself. If you reference a name before ever assigning it, Python raises a `NameError`, since there's no concept of an "empty" declared-but-unassigned variable as in statically typed languages.

```python
x = 10                # created on first assignment; type inferred as int
name: str = "Bob"     # type hint — for readers and tools like mypy
age: int = "ten"      # still runs! hints are not enforced at runtime
print(y)              # NameError: name 'y' is not defined
```

### How to initialize a variable?

Initialization in Python happens simultaneously with declaration, since assignment creates the variable and binds it to a value in one step, e.g., `count = 0` or `name = 'Alice'`. You can initialize multiple variables in a single line using tuple unpacking, e.g., `a, b, c = 1, 2, 3`, or assign the same value to multiple variables at once with chained assignment, e.g., `x = y = z = 0`. Variables can also be initialized conditionally, e.g., `status = 'active' if is_valid else 'inactive'`, or left uninitialized in a logical sense by assigning `None`, e.g., `result = None`, to indicate "no value yet." Since Python has no uninitialized-variable state, every variable is bound to some object as soon as it exists.

```python
count = 0
a, b, c = 1, 2, 3              # tuple unpacking
first, *rest = [1, 2, 3, 4]    # extended unpacking: first=1, rest=[2, 3, 4]
x = y = z = 0                  # chained: all three names -> same object
result = None                  # "no value yet"

p = q = []                     # gotcha: both names share ONE list
p.append(1)
q                              # [1]
```

### How to assign/reassign a value?

Assignment uses the `=` operator, binding a name to an object, e.g., `x = 5`. Reassignment simply points the same name to a new object, e.g., `x = 'hello'`—Python allows this because variables are just labels/references, not fixed-type containers, so a name can be rebound to a value of a completely different type. Python also supports augmented assignment operators for updating in place conceptually, e.g., `x += 1` (equivalent to `x = x + 1`), `x *= 2`, etc. Multiple assignment and unpacking work too, e.g., `a, b = b, a` to swap values without a temp variable. Importantly, reassignment doesn't mutate the original object (for immutable types like int/str); it just changes what the name refers to, leaving any other references to the old object unaffected.

```python
x = 5
x = "hello"        # rebinding: same name, new object, different type
x += " world"      # augmented assignment -> new str object

a, b = 1, 2
a, b = b, a        # swap without a temp variable -> a=2, b=1

s = "hi"
t = s              # t and s point to the same object
s += "!"           # s is rebound to a new object
t                  # 'hi' — t still points to the old one
```

### How does variable scope work?

Python resolves variable names using the LEGB rule: Local, Enclosing, Global, Built-in, searched in that order. A variable assigned inside a function is local to that function by default and isn't visible outside it. If a name isn't found locally, Python checks enclosing (outer) function scopes for nested functions, then the global (module-level) scope, and finally Python's built-in namespace (e.g., `len`, `print`). Unlike block-scoped languages (C, Java), Python doesn't create new scope for `if`/`for`/`while` blocks—variables defined inside them remain accessible in the enclosing function or module scope after the block ends. To modify a global variable inside a function, you must explicitly declare it with the `global` keyword; similarly, `nonlocal` is used to modify a variable from an enclosing (non-global) function scope inside a nested function.

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)       # 'local'     — L
    inner()
    print(x)           # 'enclosing' — E
outer()
print(x)               # 'global'    — G
print(len)             # built-in    — B

for i in range(3):
    pass
print(i)               # 2 — no block scope, loop variable survives

counter = 0
def increment():
    global counter     # without this: UnboundLocalError
    counter += 1

def make_counter():
    n = 0
    def inc():
        nonlocal n     # rebind the enclosing function's n
        n += 1
        return n
    return inc
```

### What are local, global, and module/class-level variables?

Local variables are defined inside a function and exist only during that function's execution, destroyed once it returns. Global variables are defined at the top level of a module (script file) and are accessible throughout that module, including inside functions (read-only by default, unless declared `global` for writing). Module-level variables are effectively the same as global variables scoped to that particular `.py` file's namespace, accessed from other files via imports (e.g., `module.variable`). Class-level variables (also called class/static attributes) are defined directly inside a class body but outside any method, shared across all instances of that class unless shadowed by an instance attribute of the same name, e.g., `class Dog: species = 'Canine'`. Instance variables, by contrast, are typically set inside `__init__` using `self.attribute = value` and are unique per object.

```python
APP_NAME = "shop"            # global / module-level (other files: config.APP_NAME)

class Dog:
    species = "Canine"       # class-level — shared by all instances

    def __init__(self, name):
        self.name = name     # instance-level — unique per object

    def bark(self):
        sound = "Woof"       # local — gone after bark() returns
        return f"{self.name}: {sound}"

a, b = Dog("Rex"), Dog("Max")
Dog.species = "Canis"        # visible through both a and b
a.species = "Wolf"           # creates an instance attr that shadows it on a only
b.species                    # 'Canis'
```

### Does the language support constants?

Python doesn't have a built-in, enforced constant mechanism like `const` in JavaScript or `final` in Java—there's no way to make a variable truly immutable at the language level. Instead, constants are a convention: developers name them in UPPER_SNAKE_CASE (e.g., `MAX_CONNECTIONS = 100`, `PI = 3.14159`) to signal that the value shouldn't be changed, but the interpreter won't prevent reassignment. Some approaches simulate stricter constant behavior: using a `NamedTuple` or an `Enum` class for grouped constants, overriding `__setattr__` in a custom class to raise an error on reassignment, or using `typing.Final` (from Python 3.8+) as a type-checker hint, e.g., `MAX: Final = 100`, which tools like mypy will flag if violated, though it still doesn't stop reassignment at runtime.

```python
from typing import Final
from enum import Enum
from dataclasses import dataclass

MAX_CONNECTIONS = 100
MAX_CONNECTIONS = 200        # allowed — the naming convention isn't enforced

TIMEOUT: Final = 30
TIMEOUT = 60                 # runs; mypy reports "Cannot assign to final name"

class Color(Enum):
    RED = 1
Color.RED = 3                # AttributeError — enum members can't be reassigned

@dataclass(frozen=True)
class Config:
    host: str = "localhost"
Config().host = "x"          # FrozenInstanceError
```

### How does variable declaration differ from variable definition?

In statically typed languages, declaration announces a variable's name and type without necessarily giving it a value (e.g., `int x;` in C), while definition allocates storage and often assigns an initial value. Python collapses these two concepts into one step: since there's no separate type declaration, simply writing `x = 10` both "declares" the name `x` in the current scope and "defines" it by binding it to the integer object `10` immediately. There's no Python syntax to declare a variable without giving it some initial value or reference. The closest conceptual equivalent to "declaring without defining" is using type hints without assignment inside class bodies for dataclasses or typing purposes (e.g., `x: int` inside a class), which doesn't create an actual variable until it's assigned.

```python
x: int                 # annotation only — no variable is created
print(x)               # NameError: name 'x' is not defined
x = 10                 # declared AND defined in one step

class Settings:
    port: int          # annotation only — no attribute created
Settings.__annotations__   # {'port': <class 'int'>} — only the hint is stored
Settings.port              # AttributeError

from dataclasses import dataclass
@dataclass
class Point:
    x: int             # dataclasses turn annotations into __init__ params
    y: int = 0
Point(1)               # Point(x=1, y=0)
```

### How are variables stored/referenced internally?

In Python (CPython), variables are names stored in a namespace (implemented as a dictionary) that map to references (pointers) to objects in memory, rather than the variable directly holding the object's data. When you write `x = 10`, Python creates an integer object `10` somewhere in memory and stores a reference to it under the key `x` in the current scope's namespace (e.g., accessible via `locals()` or `globals()`). Every object carries a reference count, tracking how many names/containers point to it; when a name is reassigned or deleted, its reference count on the old object decreases, and the object is garbage-collected once the count hits zero (plus a cyclic garbage collector for reference cycles). Small integers and interned strings are cached and reused by CPython for efficiency, which is why multiple variables can sometimes share the same object identity.

```python
import sys

a = [1, 2, 3]
b = a                    # second reference to the SAME list
id(a) == id(b)           # True
sys.getrefcount(a)       # e.g. 3 — a, b, + the call's temporary reference
del b                    # removes the name; refcount drops by 1
globals()["a"]           # [1, 2, 3] — module namespace is a dict

x = 256; y = 256
x is y                   # True — small ints (-5..256) are cached
```

### What is dynamic vs static typing?

In statically typed languages (Java, C, Go), variable types are declared explicitly and checked at compile time, so type mismatches are caught before the program runs, e.g., `int x = 5;` and assigning a string to x later would fail to compile. In dynamically typed languages like Python, types are determined and checked at runtime, not compile time—the same variable name can be rebound to different types over its lifetime, e.g., `x = 5` then later `x = 'hello'` is perfectly valid. Python defers all type resolution to execution, which speeds up development and increases flexibility but shifts type-related bugs from compile-time errors to runtime exceptions (like `TypeError`). Optional static type hints (via the `typing` module and tools like mypy) let Python developers opt into some compile-time-style checks without changing its fundamentally dynamic nature.

```python
x = 5            # int
x = "hello"      # now str — valid; the name just points elsewhere
# Java equivalent:  int x = 5;  x = "hello";   -> compile error

def double(n: int) -> int:
    return n * 2

double("ab")     # 'abab' — no error; hints aren't checked at runtime
double(None)     # TypeError — only discovered when this line executes
```

### What is strong vs weak typing?

Strong vs weak typing describes how strictly a language enforces type rules during operations, independent of static/dynamic typing. A strongly typed language disallows implicit, unsafe conversions between incompatible types, raising an error instead, e.g., in Python, `'5' + 3` raises a `TypeError` rather than silently converting one type to match the other. A weakly typed language (like JavaScript or C) permits such implicit coercions, e.g., in JavaScript, `'5' + 3` silently produces `'53'`. Python is dynamically typed AND strongly typed: types aren't declared upfront (dynamic), but once an object has a type, Python enforces that type's rules strictly during operations and requires explicit conversions (e.g., `str(3)` or `int('5')`) to mix types. This strong typing helps catch bugs early by failing loudly on nonsensical operations rather than silently producing unexpected values.

```python
"5" + 3          # TypeError: can only concatenate str (not "int") to str
"5" + str(3)     # '53' — explicit conversion
int("5") + 3     # 8    — explicit conversion
1 + True         # 2    — not coercion: bool is a subclass of int
# JavaScript (weak):  "5" + 3 -> "53",  "5" - 3 -> 2
```

---

## C. Data Types

### What are the built-in/primitive data types?

Python doesn't have "primitives" in the C/Java sense—every value, even a simple number, is an object with a type, identity, and methods. The core scalar (single-value) built-in types are: `int` (arbitrary-precision whole numbers), `float` (double-precision decimals), `complex` (numbers with real/imaginary parts, e.g. `3+4j`), `bool` (`True`/`False`, technically a subclass of `int`), `str` (immutable sequence of Unicode characters), and `NoneType` (the singleton `None`, representing absence of a value). Because `int` has arbitrary precision, Python integers never silently overflow the way fixed-width integers do in C or Java.

```python
i = 10          # int
f = 3.14        # float
c = 2 + 3j      # complex
b = True        # bool (subclass of int)
s = "hello"     # str
n = None        # NoneType
```

### What are reference/composite/complex data types?

Composite (container) types hold collections of other objects rather than a single scalar value. The built-in ones are `list` (ordered, mutable sequence), `tuple` (ordered, immutable sequence), `dict` (key-value mapping), `set` and `frozenset` (unordered unique collections, mutable and immutable respectively), and `bytes`/`bytearray` (sequences of raw bytes). User-defined classes also produce composite objects. These are called "reference" types because a variable holding one doesn't store the data inline—it stores a reference/pointer to the object in memory, so multiple names can refer to, and mutate, the same underlying container.

```python
lst = [1, 2, 3]              # list
tup = (1, 2, 3)              # tuple
d = {"a": 1, "b": 2}          # dict
st = {1, 2, 3}                 # set
fs = frozenset({1, 2, 3})     # frozenset
```

### How do you declare/create each data type?

Python has no separate declaration step—creating an object and binding it to a name happens in one assignment. Most types can be created via a literal syntax or their constructor function, both of which produce equivalent objects.

```python
i = 42; i2 = int("42")
f = 3.14; f2 = float("3.14")
s = "hi"; s2 = str(42)
lst = [1, 2]; lst2 = list((1, 2))
tup = (1, 2); tup2 = tuple([1, 2])
d = {"a": 1}; d2 = dict(a=1)
st = {1, 2}; st2 = set([1, 2])
b = True
n = None
```

Literals are idiomatic and faster; constructors are mainly used for conversions (e.g. `list()` on a generator) rather than creating a value from scratch.

### What are the properties of each type?

Every type carries a combination of properties: **mutability** (can its contents change after creation—`list`/`dict`/`set` yes, `int`/`str`/`tuple` no), **ordering** (does it preserve insertion/sequence order—`list`/`tuple`/`str` yes, `set`/`dict` conceptually unordered though dicts preserve insertion order as an implementation detail since 3.7), **hashability** (can it be used as a dict key or set member—only immutable types with a stable `__hash__`), and **iterability** (can you loop over it—all containers and strings, but not scalars like `int`). For example, a `tuple` is ordered, immutable, and hashable only if all its elements are hashable, while a `list` is ordered, mutable, and never hashable.

### What operations are supported by each type?

Numeric types (`int`, `float`, `complex`) support arithmetic operators (`+ - * / // % **`) and comparisons. Sequence types (`str`, `list`, `tuple`) support concatenation (`+`), repetition (`*`), indexing (`seq[i]`), slicing (`seq[a:b]`), membership (`in`), and `len()`. `dict` supports key lookup (`d[key]`), membership on keys (`in`), and merging (`|` since 3.9). `set`/`frozenset` support mathematical set operations: union (`|`), intersection (`&`), difference (`-`), and symmetric difference (`^`).

```python
[1, 2] + [3, 4]      # [1, 2, 3, 4]  concatenation
"ab" * 3              # 'ababab'      repetition
{1, 2} | {2, 3}       # {1, 2, 3}     union
{1, 2} & {2, 3}       # {2}           intersection
```

### What methods are available on each type?

Each built-in type ships with methods tailored to its purpose. `str` has methods like `.upper()`, `.split()`, `.join()`, `.strip()`, `.replace()`, `.format()`. `list` has `.append()`, `.extend()`, `.pop()`, `.sort()`, `.remove()`, `.insert()`. `dict` has `.get()`, `.keys()`, `.values()`, `.items()`, `.update()`, `.pop()`. `set` has `.add()`, `.discard()`, `.union()`, `.intersection()`. Numeric types have relatively few methods (e.g. `int.bit_length()`, `float.is_integer()`) since most of their behavior is expressed through operators instead. You can always inspect any object's available methods interactively with `dir(obj)`, and read docs with `help(obj)`.

```python
"  Hi  ".strip().lower()   # 'hi'
[3, 1, 2].sort()            # in-place: [1, 2, 3]
{"a": 1}.get("b", 0)        # 0 (default if missing)
```

### Mutable vs immutable data types?

Immutable types (`int`, `float`, `bool`, `str`, `tuple`, `frozenset`, `bytes`) cannot be changed after creation—any "modification" actually creates a new object. Mutable types (`list`, `dict`, `set`, `bytearray`, and most custom objects) can be changed in place without creating a new object, so their identity (`id()`) stays constant across the mutation. This distinction matters for three practical reasons: only immutable objects are hashable and can be used as dict keys or set members; mutable default arguments in functions are a classic bug source since the same object persists across calls; and passing a mutable object into a function lets that function modify the caller's data, while passing an immutable one cannot.

```python
s = "abc"
s += "d"        # creates a new str object; id(s) changes

lst = [1, 2]
lst.append(3)   # mutates in place; id(lst) unchanged
```

### Value types vs reference types?

Python doesn't have true stack-allocated "value types" like C#'s `struct`—every variable is a name bound to a reference to an object on the heap, including numbers. What creates the *illusion* of value semantics is immutability: since `int`/`str`/`tuple` can't be mutated, reassigning or passing them around always looks like a fresh independent value, because any change produces a new object rather than altering the shared one. Mutable objects (`list`, `dict`, custom classes) behave like true reference types: if you pass one into a function and mutate it there, the caller sees the change, because both names point to the same underlying object.

```python
def modify(lst): lst.append(4)
a = [1, 2, 3]; modify(a); print(a)   # [1, 2, 3, 4] — caller sees the change

def reassign(x): x = x + 1
n = 5; reassign(n); print(n)          # 5 — caller unaffected, int is immutable
```

### How does type conversion work?

Type conversion (casting) transforms an object of one type into an equivalent object of another type, using built-in constructor functions: `int()`, `float()`, `str()`, `list()`, `tuple()`, `set()`, `dict()`, `bool()`. These functions don't mutate the original object—they return a brand-new object of the target type. Conversions can fail at runtime if the value isn't compatible, e.g. `int("abc")` raises a `ValueError`. Conversions between container types (like `list()` on a `dict`) follow well-defined rules—converting a `dict` to a `list` yields a list of its keys, while converting it via `.items()` yields key-value tuples.

```python
int("42")        # 42
str(3.14)        # '3.14'
list("abc")      # ['a', 'b', 'c']
dict([("a", 1)]) # {'a': 1}
```

### Implicit vs explicit type conversion?

Implicit conversion (type coercion) happens automatically, without any function call, when Python can safely widen a type without losing information—this mainly applies to the numeric tower, where `bool` → `int` → `float` → `complex` mix freely in expressions, e.g. `True + 1.5` evaluates to `2.5` automatically. Explicit conversion requires the programmer to call a conversion function deliberately, and Python demands it whenever an operation would otherwise be ambiguous or unsafe, e.g. `"5" + 3` raises a `TypeError` because Python (being strongly typed) refuses to guess whether you meant string concatenation or numeric addition—you must write `int("5") + 3` or `"5" + str(3)`.

```python
True + 1.5        # 2.5   (implicit: bool -> float)
"5" + str(3)       # '53'  (explicit conversion required)
int("5") + 3       # 8     (explicit conversion required)
```

### How do you check the type of a variable?

`type(obj)` returns an object's exact class, useful for debugging or strict type checks, e.g. `type(5) == int`. `isinstance(obj, cls)` checks whether an object is an instance of a class *or any of its subclasses*, and is the preferred way to check types in real code because it respects inheritance and polymorphism—e.g. `isinstance(True, int)` is `True` since `bool` subclasses `int`, while `type(True) == int` is `False`. `isinstance()` also accepts a tuple of types for an "any of these" check. The `__class__` attribute (`obj.__class__`) gives the same result as `type(obj)` and is what `type()` uses internally.

```python
type(5) is int             # True
isinstance(True, int)      # True  (bool is a subclass of int)
isinstance(5, (int, float)) # True — matches any type in the tuple
```

## D. Operators

### Arithmetic operators

Python provides `+` (addition), `-` (subtraction), `*` (multiplication), `/` (true division, always returns a `float`), `//` (floor division, rounds toward negative infinity), `%` (modulo/remainder), and `**` (exponentiation). Unlike some languages, `/` never truncates—`7 / 2` gives `3.5`, not `3`—so `//` exists specifically for integer-style division. The `%` operator's sign follows the divisor, which can surprise developers coming from C, e.g. `-7 % 3` is `2` in Python, not `-1`. `**` has right-to-left associativity and works with negative/fractional exponents, e.g. `4 ** 0.5` gives `2.0`.

```python
7 / 2    # 3.5
7 // 2   # 3
-7 % 3   # 2
2 ** 10  # 1024
```

### Assignment operators

The basic assignment operator `=` binds a name to an object. Python also supports augmented assignment operators that combine an operation with assignment: `+=`, `-=`, `*=`, `/=`, `//=`, `%=`, `**=`, `&=`, `|=`, `^=`, `>>=`, `<<=`. For immutable types, `x += 1` is shorthand for `x = x + 1` and creates a new object; for mutable types like `list`, `+=` can trigger an in-place mutation (via `__iadd__`) rather than creating a new object, which is a subtle but important distinction. Python also supports chained assignment (`a = b = 0`) and unpacking assignment (`a, b = 1, 2`), and since 3.8 the walrus operator `:=` allows assignment as part of an expression.

```python
x = 5
x += 3          # 8
lst = [1]; lst += [2]   # mutates lst in place -> [1, 2]
if (n := 10) > 5:       # walrus: assign and test in one line
    print(n)
```

### Comparison operators

Comparison operators evaluate a relationship between two values and always return a `bool`: `==` (equal), `!=` (not equal), `<`, `>`, `<=`, `>=`. Python supports **chained comparisons**, where `1 < x < 10` is evaluated as `1 < x and x < 10`, but `x` is only evaluated once, unlike writing it out manually. `==` compares value equality (calls `__eq__`), which is different from identity comparison (`is`)—two distinct objects can be `==` equal but not the same object in memory. Comparing genuinely incompatible types (e.g. `3 < "3"`) raises a `TypeError` since Python is strongly typed and won't guess an ordering.

```python
1 < 5 <= 10        # True (chained)
[1, 2] == [1, 2]    # True (value equality)
[1, 2] is [1, 2]    # False (different objects)
```

### Logical operators

Python uses the English words `and`, `or`, and `not` instead of symbols like `&&`/`||`/`!`. Crucially, `and`/`or` don't just return `True`/`False`—they return one of the actual operand values, based on short-circuit evaluation: `a and b` returns `a` if `a` is falsy, otherwise `b`; `a or b` returns `a` if `a` is truthy, otherwise `b`. This makes them useful for default-value idioms, e.g. `name = user_input or "Guest"`. `not` negates an operand's truthiness and always returns a proper `bool`. Truthiness itself follows Python's rules: `0`, `0.0`, `""`, `[]`, `{}`, `None`, and `False` are all falsy; virtually everything else is truthy.

```python
0 or "default"     # 'default'
[] and "unreached"  # [] (short-circuits, returns the falsy value)
not None            # True
```

### Bitwise operators

Bitwise operators act on the binary representation of integers: `&` (AND), `|` (OR), `^` (XOR), `~` (NOT/complement), `<<` (left shift), `>>` (right shift). Because Python integers have arbitrary precision, `~x` is defined as `-x - 1` (two's-complement semantics without a fixed bit width) rather than flipping a fixed number of bits. Bitwise operators are commonly used for flag/permission bitmasks, low-level protocol parsing, and performance tricks like `x << 1` instead of `x * 2`. They should not be confused with the logical operators `and`/`or`—using `&`/`|` on booleans works but doesn't short-circuit, since they're evaluated as bitwise operations on `0`/`1`.

```python
5 & 3    # 1   (0101 & 0011)
5 | 3    # 7   (0101 | 0011)
5 ^ 3    # 6   (0101 ^ 0011)
1 << 3   # 8   (shift left)
```

### Increment/decrement operators, if supported

Python has **no** `++` or `--` operators—this was a deliberate design choice to avoid the ambiguity and bugs associated with pre/post-increment in C-like languages. Instead, you increment or decrement using augmented assignment: `x += 1` or `x -= 1`. Writing `++x` in Python doesn't error, but it doesn't do what a C programmer expects either—it's parsed as two unary `+` operators applied to `x` (`+(+x)`), which is a no-op, not an increment. This is a common gotcha for developers coming from C/Java/JavaScript.

```python
x = 5
x += 1   # 6, the idiomatic way to "increment"
++x      # still 6 — this is unary plus applied twice, NOT increment
```

### Membership operators

`in` and `not in` test whether a value exists within a container—`str`, `list`, `tuple`, `set`, `dict` (tests keys by default), or any object implementing `__contains__`. For `list`/`tuple`, membership testing is O(n) since it scans sequentially; for `set`/`dict`, it's O(1) average case since they're hash-based, which is why sets are preferred for frequent membership checks on large collections. Membership operators return a `bool` and are commonly used in conditionals and comprehensions to filter data.

```python
3 in [1, 2, 3]         # True
"a" in {"a": 1}          # True (checks keys)
5 not in {1, 2, 3}      # True
```

### Identity operators

`is` and `is not` check object **identity**—whether two names refer to the exact same object in memory (equivalent to comparing `id(a) == id(b)`)—as opposed to `==`, which checks value equality. `is` should be used for singleton comparisons, most notably `x is None` (the idiomatic, PEP 8-recommended way to check for `None`, rather than `x == None`), and for checking `True`/`False` identity. A common pitfall is using `is` to compare numbers or strings expecting it to behave like `==`: CPython caches small integers (-5 to 256) and some string literals, so `is` may accidentally appear to work for small values but fail unpredictably for larger ones—this caching is an implementation detail, not a language guarantee.

```python
x = None
x is None          # True — idiomatic None check

a = 1000; b = 1000
a == b              # True (same value)
a is b              # False (different objects; not small-int cached)
```

### Ternary/conditional operators

Python's ternary expression has the form `value_if_true if condition else value_if_false`, evaluated inline as a single expression rather than a statement. It's read left-to-right differently from C's `condition ? a : b`, which can trip up developers switching languages. It's commonly used for concise conditional assignment, and can be chained (though nesting too deeply hurts readability). Unlike an `if`/`else` block, it always produces a value and can be used directly inside function calls, list literals, or return statements.

```python
status = "active" if is_valid else "inactive"
label = "big" if x > 100 else "medium" if x > 10 else "small"  # chained
```

### Operator precedence

Operator precedence determines which operator is evaluated first when an expression mixes multiple operators without parentheses. Python's precedence (highest to lowest, simplified) roughly follows: parentheses/grouping → exponentiation (`**`) → unary `+`/`-`/`~` → multiplication/division/floor-division/modulo → addition/subtraction → bitwise shifts → bitwise AND → bitwise XOR → bitwise OR → comparisons → `not` → `and` → `or` → ternary → assignment. For example, `2 + 3 * 4` evaluates to `14`, not `20`, because `*` binds tighter than `+`. When precedence isn't obvious or the expression is complex, using explicit parentheses is best practice for readability, even when not strictly required.

```python
2 + 3 * 4        # 14, not 20 (* before +)
(2 + 3) * 4       # 20, parentheses override precedence
```

### Operator associativity

Associativity determines evaluation order when operators of the *same* precedence appear together. Most binary operators in Python are left-associative, meaning they evaluate left to right, e.g. `10 - 3 - 2` evaluates as `(10 - 3) - 2 = 5`. The notable exception is exponentiation (`**`), which is right-associative, e.g. `2 ** 3 ** 2` evaluates as `2 ** (3 ** 2) = 2 ** 9 = 512`, not `(2 ** 3) ** 2 = 64`. Comparison chaining (`a < b < c`) is a special case that isn't strictly left/right associative in the operator sense—it's syntactic sugar evaluated as combined `and` conditions.

```python
10 - 3 - 2     # 5   -> (10 - 3) - 2, left-associative
2 ** 3 ** 2    # 512 -> 2 ** (3 ** 2), right-associative
```

## E. Conditional Statements

### How to declare an if statement?

An `if` statement executes a block of code only when its condition evaluates to truthy. Python uses a colon followed by an indented block instead of braces—indentation is not just style, it's syntactically required and defines the block's boundaries. The condition can be any expression; Python doesn't require it to be strictly a `bool`, since any object's truthiness is checked automatically (via `__bool__` or `__len__`). No parentheses are required around the condition, though they're allowed for grouping complex expressions.

```python
age = 20
if age >= 18:
    print("Adult")
```

### How to declare else?

`else` provides a fallback block that runs when the preceding `if` (and any `elif` clauses) all evaluate to falsy. It's optional—an `if` doesn't require an `else`. Notably, Python also allows an `else` clause on `for`, `while`, and `try` blocks, which is unusual compared to most languages; there, `else` runs when the loop completes normally without hitting a `break`, or when a `try` block raises no exception—a feature distinct from the conditional `else` shown here.

```python
age = 15
if age >= 18:
    print("Adult")
else:
    print("Minor")
```

### How to declare else if?

Python spells "else if" as a single keyword, `elif`, avoiding the nested nature of writing `else: if ...:` explicitly. You can chain as many `elif` blocks as needed between an `if` and an optional final `else`; Python evaluates them top-to-bottom and executes only the first branch whose condition is truthy, skipping the rest.

```python
score = 75
if score >= 90:
    grade = "A"
elif score >= 75:
    grade = "B"
elif score >= 50:
    grade = "C"
else:
    grade = "F"
```

### How to create nested conditions?

Nested conditionals are `if` statements placed inside the body of another `if`/`elif`/`else` block, each nested level requiring one additional level of indentation (conventionally 4 spaces). They're used when a decision depends on multiple, hierarchical conditions. Deep nesting (more than 2-3 levels) is generally considered a readability smell in Python; it's often refactored using early returns (guard clauses), combining conditions with `and`/`or`, or extracting the logic into a separate function.

```python
if user.is_authenticated:
    if user.is_admin:
        print("Welcome, admin")
    else:
        print("Welcome, user")
else:
    print("Please log in")
```

### How does the ternary/conditional expression work?

Python's ternary form is `value_if_true if condition else value_if_false`—an expression, not a statement, meaning it evaluates to a value and can appear anywhere an expression is valid (assignments, function arguments, list literals, f-strings). It's evaluated lazily: only the branch matching the condition is actually computed, so `x if cond else expensive_call()` won't call `expensive_call()` when `cond` is true. It exists as syntactic sugar for a simple `if/else` that assigns a single value, and is discouraged for anything beyond one level of nesting since it hurts readability.

```python
n = 7
parity = "even" if n % 2 == 0 else "odd"   # 'odd'
```

### How are multiple conditions evaluated?

Multiple conditions are combined with the logical operators `and`, `or`, and `not`, and Python evaluates them left to right using short-circuit evaluation—stopping as soon as the overall result is determined. Parentheses can group sub-conditions to control precedence explicitly (`and` binds tighter than `or`), which is good practice for clarity even when not strictly required. Chained comparisons like `0 <= x < 100` are also a concise way to express a "between" condition as a single evaluated expression rather than combining two separate comparisons manually.

```python
age, has_id = 20, True
if age >= 18 and has_id:
    print("Entry allowed")

if 0 <= age < 100:   # chained comparison
    print("Valid age range")
```

### Short-circuit evaluation?

Short-circuit evaluation means Python stops evaluating a boolean expression as soon as the final result is already determined, skipping the remaining operands entirely. For `and`, if the first operand is falsy, the whole expression is falsy and the second operand is never evaluated; for `or`, if the first operand is truthy, the second is never evaluated. This isn't just a performance optimization—it's often relied upon for correctness, such as guarding against errors (`obj is not None and obj.value > 0` avoids an `AttributeError` on `None` because the second check never runs if the first is `False`).

```python
def risky():
    print("called!")
    return True

False and risky()   # False — risky() is never called
True or risky()      # True  — risky() is never called
```

### switch/case or equivalent?

Python historically had no `switch`/`case` statement—the idiomatic alternatives were chains of `if`/`elif`/`else`, or a dictionary mapping cases to functions/values (a "dispatch table") for cleaner multi-branch logic. This changed in Python 3.10, which introduced `match`/`case` as a structural pattern matching statement, offering switch-like behavior plus much more powerful destructuring (matching on types, sequence/mapping shapes, and guards), rather than a plain value-equality switch.

```python
# dict dispatch, common pre-3.10 idiom
actions = {"start": start_fn, "stop": stop_fn}
actions.get(command, default_fn)()

# match/case, 3.10+
match command:
    case "start":
        start_fn()
    case "stop":
        stop_fn()
    case _:
        default_fn()
```

### Pattern matching, if supported?

Python 3.10 introduced structural pattern matching via `match`/`case`. Beyond simple value matching, it can destructure sequences and mappings, bind variables from matched parts, match against types/classes, and apply guard conditions with `if`. The wildcard `_` acts as a catch-all default case. It's structurally closer to pattern matching in languages like Rust or Haskell than to a C-style `switch`, since it can match on shape and structure, not just equality, making it powerful for parsing structured data (like API responses or ASTs).

```python
def handle(command):
    match command:
        case ["move", x, y]:
            print(f"Move to {x},{y}")
        case {"action": "greet", "name": name}:
            print(f"Hello, {name}")
        case int() if command > 0:
            print("Positive number")
        case _:
            print("Unknown command")
```

## F. Loops & Iteration

### What types of loops are available?

Python has two looping constructs: `for` and `while`. `for` iterates over the elements of any iterable (list, tuple, string, dict, range, generator, file object, etc.) until the iterable is exhausted, making it the go-to choice for "for each item" logic. `while` repeats a block as long as a condition remains truthy, making it suited for cases where the number of iterations isn't known ahead of time (e.g. polling until a state changes). There's no separate C-style counting `for (i=0; i<n; i++)` form—Python expresses that pattern using `for i in range(n)` instead.

```python
for x in [1, 2, 3]:
    print(x)

i = 0
while i < 3:
    print(i)
    i += 1
```

### How to declare a for loop?

A `for` loop has the form `for item in iterable:` followed by an indented block, binding `item` to each element of `iterable` in turn. Since `for` works on any iterable rather than a numeric range by default, counting from 0 to n-1 requires the built-in `range()` function, e.g. `for i in range(5):`. `range()` is lazy—it produces values on demand rather than building a full list in memory, which matters for large ranges.

```python
for fruit in ["apple", "banana", "cherry"]:
    print(fruit)

for i in range(5):     # 0, 1, 2, 3, 4
    print(i)
```

### How to declare a while loop?

A `while` loop has the form `while condition:` followed by an indented block, and keeps executing that block as long as `condition` evaluates to truthy, re-checking the condition before every iteration (including the very first). If the condition is false from the start, the body never runs at all. The programmer is responsible for eventually making the condition false (e.g. incrementing a counter or updating a flag) to avoid an infinite loop, unless an infinite loop is intentional and exited via `break`.

```python
count = 0
while count < 3:
    print(count)
    count += 1
```

### Is there a do-while loop?

Python has no built-in `do...while` construct (a loop that always runs its body at least once before checking the condition). The standard idiom to simulate it is an infinite `while True:` loop with a conditional `break` at the point where the check would normally happen, guaranteeing at least one execution before the exit condition is evaluated.

```python
while True:
    response = input("Continue? (y/n): ")
    if response != "y":
        break
```

### How does iteration work internally?

Python's iteration is driven by the **iterator protocol**. An *iterable* is any object implementing `__iter__()`, which returns an *iterator*—an object implementing `__next__()` that returns the next value each call and raises `StopIteration` when exhausted. A `for` loop is essentially syntactic sugar: internally it calls `iter(obj)` once to get an iterator, then repeatedly calls `next()` on it, catching `StopIteration` to know when to stop. This uniform protocol is why `for` works identically on lists, dicts, files, generators, and any custom object that implements `__iter__`/`__next__`.

```python
it = iter([10, 20])
next(it)   # 10
next(it)   # 20
next(it)   # raises StopIteration
```

### How to iterate over collections?

Any built-in collection—`list`, `tuple`, `str`, `set`, `dict`, `range`—can be iterated directly with `for item in collection:` since they all implement the iterator protocol. Iterating a `dict` by default walks over its keys; use `.values()` for values or `.items()` for key-value pairs. Iterating a `set` visits elements in an arbitrary (hash-based) order, unlike `list`/`tuple`, which preserve insertion/sequence order.

```python
for k in {"a": 1, "b": 2}:          # keys: a, b
    print(k)
for v in {"a": 1, "b": 2}.values(): # values: 1, 2
    print(v)
```

### Nested loops?

Loops can be nested inside one another, with the inner loop completing all of its iterations for every single iteration of the outer loop—commonly used for 2D grids, matrices, or combinations of two collections. Each nesting level requires an additional level of indentation. A `break` or `continue` inside a nested loop only affects the innermost loop it's directly inside, not the outer one; exiting multiple nested loops at once typically requires a flag variable, restructuring into a function with `return`, or (rarely) exceptions.

```python
for i in range(2):
    for j in range(2):
        print(i, j)   # (0,0) (0,1) (1,0) (1,1)
```

### break

`break` immediately exits the nearest enclosing `for` or `while` loop entirely, skipping any remaining iterations and any loop's `else` clause. It's typically used to stop searching once a match is found, avoiding unnecessary further iterations.

```python
for n in range(10):
    if n == 3:
        break
    print(n)   # prints 0, 1, 2, then stops
```

### continue

`continue` skips the rest of the current iteration's body and jumps straight to the next iteration's condition check (for `while`) or next element (for `for`), rather than exiting the loop entirely as `break` does. It's used to skip processing for items that don't meet some criteria without needing a nested `if` to wrap the rest of the loop body.

```python
for n in range(5):
    if n % 2 == 0:
        continue
    print(n)   # prints 1, 3 (odd numbers only)
```

### pass / equivalent

`pass` is a null operation—a statement that does nothing and exists purely to satisfy Python's syntactic requirement that every block (loop, function, class, conditional) must contain at least one statement. It's used as a placeholder when code is syntactically required but not yet written, e.g. stubbing out a function during design, or when a branch genuinely should do nothing.

```python
for x in range(5):
    if x == 2:
        pass   # placeholder — do nothing for now
    else:
        print(x)
```

### How to terminate a loop?

A loop terminates naturally when its condition becomes false (`while`) or its iterable is exhausted (`for`), or it can be terminated early with `break`. Additionally, `return` inside a function will exit the loop along with the entire function, and an uncaught exception will also abort the loop (and propagate up the call stack) unless caught within it.

```python
def find_first_even(nums):
    for n in nums:
        if n % 2 == 0:
            return n   # exits both the loop and the function
    return None
```

### How to skip an iteration?

Use `continue` to skip the remainder of the current iteration and move to the next one, as shown above. This is the standard way to implement "process only items matching X" logic inline, without wrapping the entire loop body in a conditional.

```python
for n in range(10):
    if n % 3 != 0:
        continue
    print(n)   # prints 0, 3, 6, 9
```

### How to iterate with index?

The built-in `enumerate(iterable, start=0)` wraps any iterable and yields `(index, value)` pairs lazily, which is the idiomatic way to get an index while iterating—preferred over manually managing a counter variable or using `range(len(seq))`, both of which are considered less "Pythonic" and more error-prone.

```python
for i, fruit in enumerate(["apple", "banana"], start=1):
    print(i, fruit)   # 1 apple, 2 banana
```

### How to iterate over key/value pairs?

For dictionaries, `.items()` returns a view of `(key, value)` tuples, which can be unpacked directly in a `for` loop—this is the standard idiom rather than looking up `d[key]` inside a loop over `.keys()`. Views like `.items()` are dynamic; they reflect changes to the dictionary made after the view was created (though mutating the dict's size during iteration raises a `RuntimeError`).

```python
scores = {"Alice": 90, "Bob": 85}
for name, score in scores.items():
    print(name, score)
```

### Language's iteration protocol

The iteration protocol is the formal contract Python's `for` loops (and functions like `list()`, `sum()`, `sorted()`) rely on. An object is **iterable** if it defines `__iter__()`, returning an **iterator**. An iterator must define both `__iter__()` (returning itself) and `__next__()` (returning the next value or raising `StopIteration` when done). This two-part design lets a single collection be iterated multiple times independently (each `iter()` call can return a fresh iterator), while the iterator itself is a single-use, forward-only cursor. Generators (functions using `yield`) automatically implement this protocol without manually writing `__next__`.

```python
class Countdown:
    def __init__(self, n): self.n = n
    def __iter__(self): return self
    def __next__(self):
        if self.n <= 0: raise StopIteration
        self.n -= 1
        return self.n + 1

for x in Countdown(3):   # 3, 2, 1
    print(x)
```

## G. Functions

### How to declare a function?

Functions are declared with the `def` keyword, a name, a parenthesized parameter list, and a colon, followed by an indented body. An optional docstring can appear as the first statement, and a `return` statement sends a value back to the caller—if omitted, the function implicitly returns `None`. Type hints can annotate parameters and the return type for readability and static analysis, though they aren't enforced at runtime.

```python
def greet(name: str) -> str:
    """Return a greeting for the given name."""
    return f"Hello, {name}!"
```

### How to call a function?

A function is invoked by writing its name followed by parentheses containing any arguments, e.g. `greet("Alice")`. Calling triggers execution of the function body with the given arguments bound to its parameters, and the call expression evaluates to whatever the function returns (or `None` if there's no explicit `return`). Functions are objects themselves, so a reference to a function (without parentheses) can be passed around and called later via that reference.

```python
greet("Alice")          # "Hello, Alice!"
say_hi = greet          # reference to the function, not a call
say_hi("Bob")            # "Hello, Bob!"
```

### Parameters vs arguments?

**Parameters** are the names listed in a function's definition, acting as placeholders for the values it expects, e.g. `name` in `def greet(name):`. **Arguments** are the actual values supplied at the call site, e.g. `"Alice"` in `greet("Alice")`. The distinction is subtle but commonly tested in interviews: parameters exist at definition time; arguments exist at call time and get bound to the corresponding parameters.

```python
def add(a, b):     # a, b are parameters
    return a + b
add(2, 3)            # 2, 3 are arguments
```

### Positional arguments?

Positional arguments are matched to parameters strictly by their order in the call, not by name—the first argument fills the first parameter, and so on. This is the default and most common way to pass arguments, but it requires the caller to know and respect the exact parameter order, which can hurt readability for functions with many parameters.

```python
def describe(name, age):
    print(f"{name} is {age}")
describe("Alice", 30)   # name='Alice', age=30 — matched by position
```

### Keyword/named arguments?

Keyword arguments are passed as `name=value` at the call site, matched to parameters by name rather than position, allowing arguments to be supplied in any order and making call sites self-documenting. Positional and keyword arguments can be mixed in one call, but all positional arguments must come before any keyword arguments.

```python
def describe(name, age):
    print(f"{name} is {age}")
describe(age=30, name="Alice")   # order doesn't matter with keywords
```

### Default parameters?

A parameter can specify a default value in the function definition (`def f(x=10):`), making it optional at the call site—if the caller omits it, the default is used. Default values are evaluated **once**, at function definition time, not on every call—this is a well-known gotcha when the default is a mutable object like a `list`, since the same object would persist and accumulate state across calls. The standard workaround is defaulting to `None` and creating a new mutable object inside the function body.

```python
def add_item(item, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(item)
    return bucket
```

### Variable-length arguments?

Python supports collecting an arbitrary number of arguments using `*args` (gathers extra positional arguments into a tuple) and `**kwargs` (gathers extra keyword arguments into a dict). These allow a function to accept a flexible, unknown-ahead-of-time number of inputs, and are heavily used for wrapper/decorator functions that need to forward arbitrary arguments to another function.

```python
def total(*args, **kwargs):
    print(args)     # (1, 2, 3)
    print(kwargs)   # {'tax': 0.1}
    return sum(args)

total(1, 2, 3, tax=0.1)
```

### Return values?

The `return` statement ends a function's execution and sends a value back to the caller. A function without an explicit `return`, or a bare `return` with no value, implicitly returns `None`. Because Python is dynamically typed, a function isn't restricted to returning one fixed type—different branches could return different types, though that's generally discouraged for maintainability. `return` can appear anywhere in the function body, including inside loops or conditionals, and immediately exits as soon as it executes.

```python
def is_even(n):
    if n % 2 == 0:
        return True
    return False
```

### Multiple return values?

Python doesn't literally return more than one object—instead, a function returns a single `tuple` when you separate values with commas after `return`, and the caller typically unpacks that tuple into multiple variables. This is a widely used idiom for functions that logically produce more than one related result, like a min and max.

```python
def min_max(nums):
    return min(nums), max(nums)   # returns a tuple

low, high = min_max([3, 1, 4, 1, 5])   # unpacked at the call site
```

### Function scope?

Each function call creates its own local scope/namespace: parameters and any variables assigned inside the function body are local by default, invisible outside the function, and discarded when the function returns. Name resolution inside a function follows the LEGB rule (Local, Enclosing, Global, Built-in). To modify a variable from an outer scope rather than shadowing it locally, the `global` or `nonlocal` keyword must be used explicitly.

```python
x = 10
def f():
    x = 5       # local x, doesn't affect the global one
    print(x)
f()             # 5
print(x)        # 10 — unchanged
```

### Nested functions?

A function can be defined inside another function's body; the inner (nested) function has access to the outer function's variables via lexical (enclosing) scope, and exists only within the outer function's scope unless explicitly returned or exposed. Nested functions are the foundation for closures and decorators, letting you encapsulate helper logic that shouldn't be visible outside its parent function.

```python
def outer():
    msg = "hi"
    def inner():
        print(msg)   # accesses outer's variable (enclosing scope)
    inner()
outer()   # 'hi'
```

### Anonymous/lambda functions?

A `lambda` creates a small, unnamed function inline using the form `lambda args: expression`, restricted to a single expression (no statements, no multiple lines) whose result is implicitly returned. Lambdas are commonly used for short, throwaway functions passed as arguments to higher-order functions like `sorted()`, `map()`, or `filter()`, where defining a full `def` would be unnecessarily verbose.

```python
square = lambda x: x ** 2
square(5)   # 25
sorted([(1, 'b'), (2, 'a')], key=lambda pair: pair[1])   # sort by 2nd element
```

### First-class functions?

Functions in Python are "first-class citizens," meaning they can be treated like any other value: assigned to variables, stored in data structures (lists, dicts), passed as arguments to other functions, and returned from other functions. This capability underpins decorators, callbacks, and functional-style programming patterns like `map`/`filter`/`reduce`.

```python
def shout(text): return text.upper()
funcs = [shout, str.lower]        # functions stored in a list
funcs[0]("hi")                    # 'HI'
```

### Higher-order functions?

A higher-order function either accepts one or more functions as arguments, returns a function, or both—a direct consequence of functions being first-class. Built-in examples include `map()`, `filter()`, `sorted(key=...)`, and decorators (which take a function and return a wrapped version of it). They're central to functional-style composition in Python.

```python
def apply_twice(f, x):
    return f(f(x))
apply_twice(lambda x: x + 3, 10)   # 16
```

### Recursion?

A recursive function is one that calls itself, typically to break a problem down into smaller subproblems of the same shape, always requiring a base case to stop the recursion and prevent infinite calls. Python doesn't optimize tail recursion (no tail-call elimination), and has a default recursion depth limit (`sys.getrecursionlimit()`, typically 1000), raising a `RecursionError` if exceeded—so deeply recursive algorithms in Python are often rewritten iteratively for very large inputs.

```python
def factorial(n):
    if n <= 1:          # base case
        return 1
    return n * factorial(n - 1)   # recursive case
```

### Function overloading?

Python does not support traditional function overloading (defining multiple functions with the same name but different parameter types/counts, resolved at compile time)—defining a function with the same name twice simply replaces the earlier one, since functions are just names bound to objects. Overload-like behavior is instead achieved dynamically: via default/variable-length arguments, checking argument types inside the function (`isinstance`), using `functools.singledispatch` for type-based dispatch, or (for static-analysis purposes only) the `typing.overload` decorator, which doesn't change runtime behavior.

```python
from functools import singledispatch

@singledispatch
def process(value): print("generic", value)

@process.register
def _(value: int): print("int:", value)
```

### Function overriding?

Overriding happens in the context of inheritance: a subclass defines a method with the same name as one in its superclass, replacing (overriding) that behavior for instances of the subclass. Python resolves method calls dynamically at runtime via the Method Resolution Order (MRO), so calling `obj.method()` always uses the most specific (subclass) version available, unless `super().method()` is used to explicitly invoke the parent's implementation.

```python
class Animal:
    def speak(self): return "..."

class Dog(Animal):
    def speak(self): return "Woof"   # overrides Animal.speak

Dog().speak()   # 'Woof'
```

### Call by value vs reference/reference semantics?

Python uses a model often called "call by object reference" (or "call by assignment"), which is neither pure call-by-value nor pure call-by-reference. When a function is called, its parameters are bound to references to the same objects the caller passed in—no copies are made. Consequently, mutating a **mutable** object (like a `list`) inside the function is visible to the caller, since both names point to the same object; but **reassigning** the parameter to a new object (regardless of mutability) only rebinds the local name and has no effect on the caller's variable, since reassignment just changes what the local name points to.

```python
def mutate(lst): lst.append(4)
def reassign(lst): lst = [9, 9]

a = [1, 2, 3]
mutate(a); print(a)     # [1, 2, 3, 4] — caller sees the mutation
reassign(a); print(a)   # [1, 2, 3, 4] — caller unaffected by reassignment
```

## H. Input & Output

### How to print/output data?

The built-in `print()` function writes text to standard output (the console by default), automatically converting arguments to strings and separating multiple arguments with a space, followed by a newline. Its behavior is customizable via keyword arguments: `sep` changes the separator between arguments, `end` changes what's appended after the output (default `"\n"`), and `file` redirects output to a different stream (like a file object) instead of the console.

```python
print("a", "b", "c")                  # a b c
print("a", "b", sep="-", end="!\n")   # a-b!
```

### How to take user input?

The built-in `input(prompt)` function reads a line of text typed by the user from standard input, displaying an optional prompt string first, and always returns the result as a `str`—even if the user types a number, it must be explicitly converted with `int()`/`float()` if numeric use is intended. Execution pauses at `input()` until the user presses Enter.

```python
name = input("Enter your name: ")
age = int(input("Enter your age: "))
```

### How to format output?

Python offers several ways to format strings for output: f-strings (`f"{value}"`, Python 3.6+, the modern preferred approach), the `.format()` method (`"{}".format(value)`), and old-style `%` formatting (`"%s" % value`, inherited from C-style formatting). F-strings support inline expressions and format specifiers for things like decimal precision, padding, and alignment, directly inside the braces.

```python
name, pi = "Alice", 3.14159
f"{name} owes ${pi:.2f}"       # 'Alice owes $3.14'
"{} owes ${:.2f}".format(name, pi)   # same result, .format() style
```

### String interpolation?

String interpolation embeds variable values or expressions directly inside a string template. Python's primary mechanism is the f-string (`f"Hello, {name}!"`), prefixed with `f`, where anything inside `{}` is evaluated as a live Python expression, not just a variable lookup—so `f"{2 + 2}"` and `f"{name.upper()}"` both work. F-strings are generally faster and more readable than `.format()` or `%`-based interpolation, which is why they've become the idiomatic default since Python 3.6.

```python
x, y = 3, 4
f"{x} + {y} = {x + y}"   # '3 + 4 = 7'
```

### Reading from standard input?

Beyond the simple `input()` function, Python provides `sys.stdin` for lower-level or streaming access to standard input—useful for reading multiple lines, piped data, or when processing input without an interactive prompt (e.g. in a script fed data via a Unix pipe). Iterating over `sys.stdin` yields one line at a time, similar to iterating over a file object.

```python
import sys
for line in sys.stdin:
    print(line.strip())
```

### Writing to standard output?

While `print()` is the common high-level way to write to standard output, `sys.stdout.write()` offers a lower-level alternative that writes a string exactly as given, without the automatic space-separation or trailing newline that `print()` adds—giving finer control in performance-sensitive or precisely formatted output scenarios.

```python
import sys
sys.stdout.write("no auto newline")
sys.stdout.write("\n")
```

### File input/output?

Files are opened with the built-in `open(path, mode)`, where `mode` controls behavior: `"r"` (read, default), `"w"` (write, truncates existing content), `"a"` (append), `"x"` (exclusive create), each optionally combined with `"b"` for binary mode. The idiomatic approach is a `with` block, which guarantees the file is properly closed even if an exception occurs inside it, since `open()` returns a context manager.

```python
with open("data.txt", "r") as f:
    content = f.read()          # whole file as one string

with open("data.txt", "w") as f:
    f.write("new content")      # overwrites the file
```

### Serialization/deserialization?

Serialization converts an in-memory Python object into a storable/transmittable format (like a JSON string or a byte stream); deserialization reverses that process, reconstructing the object from that format. The `json` module handles JSON (`json.dumps()`/`json.loads()` for strings, `json.dump()`/`json.load()` for files), the go-to format for interoperable, human-readable data exchange. The `pickle` module serializes arbitrary Python objects into a Python-specific binary format, more powerful (handles custom classes, functions) but not language-interoperable and unsafe to unpickle data from untrusted sources, since it can execute arbitrary code during deserialization.

```python
import json
data = {"name": "Alice", "age": 30}
s = json.dumps(data)     # serialize to a JSON string
obj = json.loads(s)       # deserialize back to a dict
```

## I. Collections / Data Structures

### Arrays

Python has no dedicated low-level "array" type in everyday use the way C or Java do—the built-in `list` serves as a dynamic, general-purpose array that can hold mixed types and resizes automatically. For memory-efficient, fixed-type numeric arrays, the standard library offers the `array` module, and for serious numerical work, the third-party `numpy.ndarray` is the de facto standard, offering vectorized operations and much better performance/memory characteristics for large numeric datasets.

```python
import array
a = array.array('i', [1, 2, 3])   # typed array of ints, more compact than list
```

### Lists

A `list` is Python's built-in ordered, mutable, resizable sequence, able to hold elements of any (even mixed) type, and implemented internally as a dynamic array of object references. Lists support indexing, slicing, concatenation, and a rich set of in-place methods (`.append()`, `.insert()`, `.remove()`, `.sort()`), making them the default general-purpose container for ordered collections.

```python
nums = [1, 2, 3]
nums.append(4)         # [1, 2, 3, 4]
nums[0]                 # 1 (indexing)
```

### Tuples

A `tuple` is an ordered, **immutable** sequence—once created, its elements can't be added, removed, or replaced. Because of this immutability, tuples are hashable (if their contents are) and can be used as dictionary keys or set members, unlike lists. They're commonly used for fixed collections of heterogeneous data (like coordinate pairs) and for function returns that bundle multiple values.

```python
point = (3, 4)
point[0]         # 3
# point[0] = 5   # TypeError — tuples are immutable
```

### Sets

A `set` is an unordered collection of unique, hashable elements, backed by a hash table, giving average O(1) membership testing and automatic de-duplication. Sets support mathematical operations like union (`|`), intersection (`&`), and difference (`-`). `frozenset` is the immutable, hashable counterpart, usable as a dict key or a member of another set.

```python
s = {1, 2, 2, 3}     # {1, 2, 3} — duplicates removed automatically
s.add(4)
{1, 2} & {2, 3}       # {2} — intersection
```

### Maps/Dictionaries

A `dict` is Python's built-in hash-map/associative-array type, storing unique keys mapped to values, with average O(1) lookup, insertion, and deletion by key. Since Python 3.7, dicts officially preserve insertion order as a language guarantee (previously an implementation detail). Keys must be hashable (so immutable types like `str`, `int`, `tuple`), while values can be any type.

```python
d = {"name": "Alice", "age": 30}
d["age"]           # 30
d["city"] = "NYC"    # add a new key
```

### Stacks

Python has no dedicated `Stack` type—a `list` is commonly used as a stack (LIFO: last-in, first-out), using `.append()` to push and `.pop()` to pop from the end, both O(1) operations. For thread-safe or more specialized use, `collections.deque` (with `.append()`/`.pop()`) or `queue.LifoQueue` are alternatives.

```python
stack = []
stack.append(1); stack.append(2)
stack.pop()   # 2 — LIFO
```

### Queues

A plain `list` is a poor choice for a FIFO queue since removing from the front (`.pop(0)`) is O(n). The idiomatic structure is `collections.deque` (double-ended queue), which supports O(1) appends/pops from **both** ends, making it efficient for FIFO queues (`.append()` + `.popleft()`). For multi-threaded producer/consumer scenarios, `queue.Queue` provides a thread-safe FIFO implementation with built-in locking.

```python
from collections import deque
q = deque()
q.append(1); q.append(2)
q.popleft()   # 1 — FIFO
```

### Linked lists, if language-supported

Python has no built-in linked-list type—`list` (a dynamic array) and `deque` (typically implemented as a doubly-linked list of fixed-size blocks internally) cover the practical use cases that linked lists solve in lower-level languages. A true singly/doubly linked list can be implemented manually with a simple node class when the specific access pattern (O(1) insert/delete at arbitrary positions given a reference to a node) is genuinely needed, which is rare in everyday Python code.

```python
class Node:
    def __init__(self, value, next=None):
        self.value = value
        self.next = next
```

### Trees/graphs, where relevant

Python has no built-in tree or graph type either—these are typically built from custom classes (nodes with child references, or adjacency lists/dicts for graphs), or via third-party libraries suited to the domain, e.g. `heapq` for a binary-heap-based priority queue, or `networkx` for full-featured graph algorithms. Because Python objects and references naturally form arbitrary graphs (an object's attribute can reference any other object, including itself for cycles), these structures are straightforward to hand-build.

```python
graph = {"A": ["B", "C"], "B": ["D"], "C": [], "D": []}   # adjacency-list graph
```

### How to add/remove/update elements?

Each collection type exposes its own idiomatic operations. Lists: `.append()`, `.insert(i, x)`, `.remove(x)`, `.pop(i)`, or index assignment `lst[i] = x`. Dicts: `d[key] = value` to add/update, `del d[key]` or `.pop(key)` to remove. Sets: `.add(x)`, `.discard(x)`/`.remove(x)` (the latter raises `KeyError` if missing, `.discard()` doesn't). Tuples support none of these—since they're immutable, "modifying" one means building a new tuple.

```python
lst = [1, 2, 3]; lst.append(4); lst.remove(2)   # [1, 3, 4]
d = {"a": 1}; d["b"] = 2; del d["a"]              # {'b': 2}
```

### Searching?

`in` performs membership testing on any collection—O(n) for `list`/`tuple` (linear scan), O(1) average for `set`/`dict` (hash-based lookup). `list.index(x)` finds the position of the first matching element, raising `ValueError` if not found. For sorted sequences, the `bisect` module provides O(log n) binary search.

```python
3 in [1, 2, 3]            # True, O(n)
[1, 2, 3].index(2)         # 1
3 in {1, 2, 3}              # True, O(1) average
```

### Sorting?

`sorted(iterable)` returns a **new** sorted list without modifying the original, and works on any iterable. `list.sort()` sorts a list **in place** and returns `None`. Both accept `key=` (a function producing the sort value per element) and `reverse=True` for descending order. Python's sort algorithm, Timsort, is stable—elements that compare equal retain their relative original order.

```python
sorted([3, 1, 2])                          # [1, 2, 3], new list
["bb", "a", "ccc"].sort()                    # in place
sorted(["bb", "a", "ccc"], key=len)          # ['a', 'bb', 'ccc'] by length
```

### Iteration?

All collections support direct iteration via `for item in collection:`, since each implements the iterator protocol (`__iter__`). Dicts iterate over keys by default (use `.items()`/`.values()` for pairs/values); sets iterate in an arbitrary hash-based order; lists/tuples iterate in their defined sequence order.

```python
for item in [1, 2, 3]:
    print(item)
```

### Slicing?

Slicing extracts a sub-sequence using `seq[start:stop:step]`, where `start` is inclusive, `stop` is exclusive, and `step` controls the interval (negative `step` reverses direction). It works on any sequence type (`list`, `tuple`, `str`) and always returns a new object of the same type, never mutating the original. Omitting `start`/`stop` defaults to the beginning/end of the sequence.

```python
nums = [0, 1, 2, 3, 4, 5]
nums[1:4]     # [1, 2, 3]
nums[::2]      # [0, 2, 4]  — every 2nd element
nums[::-1]     # [5, 4, 3, 2, 1, 0]  — reversed
```

### Copying?

A plain assignment (`b = a`) doesn't copy anything—it just creates a second name pointing to the same object, so mutating one affects the other. A true (shallow) copy is made with `.copy()`, `list(a)`, or the slice trick `a[:]`, producing a new outer container while still sharing references to the same nested objects.

```python
a = [1, 2, 3]
b = a           # b is the same object as a
c = a.copy()    # c is a new list, same elements
```

### Shallow vs deep copy?

A **shallow copy** (`.copy()`, `copy.copy()`) creates a new outer container but still references the *same* nested/inner objects as the original—so mutating a nested mutable object (like a list inside a list) affects both copies. A **deep copy** (`copy.deepcopy()`) recursively copies every nested object too, producing a fully independent structure with no shared references at any level, at the cost of more time/memory.

```python
import copy
original = [[1, 2], [3, 4]]
shallow = copy.copy(original)
shallow[0].append(99)
print(original)   # [[1, 2, 99], [3, 4]] — inner list is shared!

deep = copy.deepcopy(original)
deep[0].append(100)
print(original)   # unaffected — fully independent
```

### Comprehensions / collection expressions?

Comprehensions build a new collection from an iterable in a single, concise expression, combining a loop, an optional filter condition, and a transformation into one line: list comprehensions `[expr for x in iterable if cond]`, dict comprehensions `{k: v for ...}`, set comprehensions `{expr for ...}`, and generator expressions `(expr for ...)` (lazy, memory-efficient, don't build the whole collection upfront). They're generally faster and more idiomatic ("Pythonic") than building a collection with an explicit loop and repeated `.append()` calls.

```python
squares = [x**2 for x in range(5)]                 # list comprehension
evens = {x for x in range(10) if x % 2 == 0}         # set comprehension
lookup = {x: x**2 for x in range(5)}                  # dict comprehension
lazy = (x**2 for x in range(5))                        # generator expression
```

## J. Strings

### How to create strings?

A Python string (`str`) is an immutable sequence of Unicode code points. Strings can be created with single quotes, double quotes (both identical—pick one so the other can appear inside without escaping), or triple quotes for multi-line text. A raw string prefix `r"..."` disables backslash escapes, which is useful for regex patterns and Windows paths. There's no separate `char` type—a single character is just a `str` of length 1. The `str()` constructor converts any object to its string form via `__str__`. Note that `b"..."` creates `bytes`, a different type from `str`. Adjacent string literals are joined automatically at compile time, which is handy for splitting long strings across lines.

```python
s1 = 'single'
s2 = "double — lets you write 'quotes' inside"
s3 = """triple quotes
span multiple lines"""
path = r"C:\new\table"       # raw: backslashes kept literally
b = b"bytes"                 # bytes, not str
str(42)                      # '42'
msg = ("adjacent literals "
       "are joined at compile time")
type("a")                    # <class 'str'> — no separate char type
```

### How does string concatenation work?

`+` joins two strings into a new string, and `*` repeats a string. Because strings are immutable, every `+` creates a new object—so building a string with `+=` inside a loop can degrade to O(n²) as each step copies everything built so far (CPython has an in-place optimization for this, but it's an implementation detail you shouldn't rely on). The idiomatic approach for many pieces is to collect them in a list and call `"".join(parts)` once, which is O(n). Concatenating a `str` with a non-`str` raises a `TypeError`, since Python is strongly typed—convert explicitly with `str()` or use an f-string.

```python
"Hello, " + "World"       # 'Hello, World'
"ab" * 3                  # 'ababab'
# "Age: " + 30            # TypeError — use "Age: " + str(30)

# Slow pattern for many pieces
result = ""
for word in words:
    result += word

# Idiomatic: O(n)
result = "".join(words)
```

### How does string interpolation work?

The basics are covered in section H; the details below are what interviewers usually probe. f-strings are evaluated at runtime and can contain any expression, function call, or format spec. The `=` specifier (3.8+) prints both the expression and its value, which is handy for debugging, and `!r` uses `repr()` instead of `str()`. Two practical rules: for logging, pass arguments to the logger (`logger.info("x=%s", x)`) so formatting only happens when the message is actually emitted; and never build SQL queries with f-strings—use parameterized queries to prevent SQL injection. Python 3.14 added template strings (`t"..."`), which return a `Template` object instead of a `str`, so libraries can safely escape the interpolated values.

```python
name, items = "Alice", 3
f"{name} has {items} items"      # 'Alice has 3 items'
f"{items * 2 = }"                # 'items * 2 = 6' — debug specifier (3.8+)
f"{name!r}"                      # "'Alice'" — uses repr()

logger.info("User %s logged in", name)   # formatted only if the level is enabled
cursor.execute("SELECT * FROM users WHERE name = %s", (name,))  # never f-string SQL
```

### How does string indexing work?

Strings are zero-indexed sequences, so `s[0]` is the first character and negative indices count from the end (`s[-1]` is the last). Indexing returns a new `str` of length 1, and an index outside the range raises `IndexError`. Since strings are immutable, assigning to an index raises `TypeError`. Indexing works on Unicode code points, not bytes, so `"é"` counts as one character even though it takes two bytes in UTF-8.

```python
s = "Python"
s[0]             # 'P'
s[-1]            # 'n'
# s[10]          # IndexError: string index out of range
# s[0] = "J"     # TypeError: 'str' object does not support item assignment
len("héllo")     # 5 — counts code points, not bytes
```

### How does string slicing work?

Slicing uses `s[start:stop:step]`, where `start` is inclusive, `stop` is exclusive, and omitted values default to the beginning/end. Unlike indexing, slicing never raises `IndexError`—out-of-range bounds are clamped to the string's length. Every slice returns a new string. A negative step walks backwards, so `s[::-1]` is the idiomatic way to reverse a string, which comes up often in interview problems like palindrome checks.

```python
s = "Python"
s[0:2]           # 'Py'
s[2:]            # 'thon'
s[:-2]           # 'Pyth'
s[::2]           # 'Pto' — every 2nd character
s[::-1]          # 'nohtyP' — reversed
s[2:100]         # 'thon' — out-of-range slice is clamped, no error

def is_palindrome(t):
    t = t.lower()
    return t == t[::-1]
```

### What are the commonly used string methods?

String methods never modify the original—they always return a new string (or a bool/list/int), which is why they can be chained. They fall into groups: case conversion (`upper`, `lower`, `title`), searching (`find`, `count`, `startswith`), testing content (`isdigit`, `isalpha`, `isalnum`, `isspace`), trimming (`strip`), splitting/joining, replacing, and padding/alignment (`center`, `ljust`, `rjust`, `zfill`). Python 3.9 added `removeprefix()`/`removesuffix()`. `startswith()`/`endswith()` accept a tuple to check several options at once.

```python
s = "  Hello World  "
s.strip().lower()                         # 'hello world'
"abc123".isalnum()                        # True
"42".isdigit()                            # True
"report.pdf".endswith((".pdf", ".doc"))   # True — accepts a tuple
"7".zfill(3)                              # '007'
"title".center(11, "*")                   # '***title***'
"v1.2".removeprefix("v")                  # '1.2' (3.9+)
```

### How to search within a string?

The `in` operator is the most readable way to check whether a substring exists. `find()` returns the index of the first match or `-1` if not found, while `index()` does the same but raises `ValueError` when missing—use `index()` when absence is a bug, `find()` when it's an expected case. `rfind()`/`rindex()` search from the right, and `count()` counts non-overlapping occurrences. For patterns rather than fixed text, use the `re` module: `re.search()` finds the first match anywhere, `re.match()` only at the start, `re.fullmatch()` the whole string, and `re.findall()` all matches. Compile a pattern with `re.compile()` when it's reused.

```python
s = "banana"
"nan" in s           # True
s.find("an")         # 1
s.rfind("an")        # 3
s.find("x")          # -1
# s.index("x")       # ValueError: substring not found
s.count("a")         # 3

import re
re.findall(r"\d+", "order 12, qty 30")          # ['12', '30']
EMAIL = re.compile(r"[\w.]+@[\w.]+")
EMAIL.search("mail me at a.b@x.com").group()    # 'a.b@x.com'
```

### How to replace within a string?

`replace(old, new, count)` returns a new string with every occurrence of `old` replaced (or only the first `count` occurrences); the original is unchanged. For replacing or deleting many individual characters at once, `str.translate()` with a table from `str.maketrans()` is faster than chaining several `replace()` calls. For pattern-based replacement, `re.sub()` is the tool, and it can take a function to compute each replacement dynamically.

```python
s = "a-b-c"
s.replace("-", "_")          # 'a_b_c'
s.replace("-", "_", 1)       # 'a_b-c' — only the first occurrence
s                            # 'a-b-c' — original unchanged

table = str.maketrans({"a": "4", "e": "3"})
"leet speak".translate(table)                             # 'l33t sp34k'
"hello world".translate(str.maketrans("", "", "aeiou"))   # 'hll wrld' — delete chars

import re
re.sub(r"\s+", " ", "too    many   spaces")                      # 'too many spaces'
re.sub(r"\d+", lambda m: str(int(m.group()) * 2), "a1 b22")      # 'a2 b44'
```

### How to split a string?

`split()` with no argument splits on any run of whitespace (spaces, tabs, newlines) and drops empty strings, which makes it ideal for splitting words. With an explicit separator, `split(",")` splits on exactly that separator and keeps empty strings between consecutive separators—a common source of bugs. `maxsplit` limits the number of splits, and `rsplit()` splits from the right. `splitlines()` splits on line boundaries, and `partition()` splits once into a 3-tuple `(before, sep, after)`. For real CSV data with quoted fields, use the `csv` module instead of `split(",")`.

```python
"a  b\tc\n".split()             # ['a', 'b', 'c'] — any whitespace, empties dropped
"a,,b".split(",")               # ['a', '', 'b'] — explicit sep keeps empties
"k=v=w".split("=", 1)           # ['k', 'v=w'] — maxsplit
"a/b/c.txt".rsplit("/", 1)      # ['a/b', 'c.txt']
"line1\nline2".splitlines()     # ['line1', 'line2']
"user@host".partition("@")      # ('user', '@', 'host')
```

### How to join strings?

`sep.join(iterable)` is a method on the separator string, not on the list—a design choice that lets it accept any iterable (list, tuple, generator). Every element must already be a `str`, otherwise it raises `TypeError`; convert with `map(str, ...)` first. `join()` calculates the total size once and builds the result in a single pass, which is why it's the recommended way to combine many strings.

```python
", ".join(["a", "b", "c"])           # 'a, b, c'
"".join(reversed("abc"))             # 'cba'
# ", ".join([1, 2, 3])               # TypeError: sequence item 0: expected str instance, int found
", ".join(map(str, [1, 2, 3]))       # '1, 2, 3'
"\n".join(f"{k}={v}" for k, v in {"a": 1}.items())   # generators work too
```

### How to trim whitespace?

`strip()` removes leading and trailing whitespace, while `lstrip()` and `rstrip()` trim only the left or right side. When given an argument, these methods treat it as a **set of characters** to remove, not as a substring—a classic gotcha that can remove more than intended. To remove an exact prefix or suffix, use `removeprefix()`/`removesuffix()` (3.9+).

```python
"  hi  ".strip()                     # 'hi'
"  hi  ".lstrip()                    # 'hi  '
"xxhixx".strip("x")                  # 'hi'
"www.example.com".strip("w.moc")     # 'example' — chars, not a substring
"docs.csv".rstrip(".csv")            # 'do'   — gotcha! also strips 's' and 'c'
"docs.csv".removesuffix(".csv")      # 'docs' — correct
```

### How to convert case?

`upper()`, `lower()`, `title()` (first letter of every word), `capitalize()` (first letter of the string only), and `swapcase()` change letter case and return new strings. For case-insensitive comparison, `casefold()` is more correct than `lower()` because it handles special Unicode rules—e.g. the German `ß` folds to `ss`. `title()` treats any non-letter as a word boundary, so it produces odd results with apostrophes.

```python
"hello world".upper()          # 'HELLO WORLD'
"Hello".lower()                # 'hello'
"hello world".title()          # 'Hello World'
"hello world".capitalize()     # 'Hello world'
"Hello".swapcase()             # 'hELLO'
"they're".title()              # "They'Re" — gotcha

"Straße".casefold() == "STRASSE".casefold()   # True
"Straße".lower() == "STRASSE".lower()         # False
```

### How to format strings (format specification)?

f-strings, `str.format()`, and `format()` all share the same format specification mini-language after the colon: `{value:[fill][align][sign][width][,][.precision][type]}`. It controls decimal places, thousands separators, percentages, padding, alignment, and number bases. Objects can define their own format specs via `__format__`, which is why `datetime` accepts `strftime` codes directly in an f-string. Width and precision can themselves be nested expressions.

```python
n, pi = 1234567.891, 3.14159
f"{pi:.2f}"              # '3.14'
f"{n:,.2f}"              # '1,234,567.89' — thousands separator
f"{0.256:.1%}"           # '25.6%'
f"{42:>8}"               # '      42' — right-align in width 8
f"{'hi':*^8}"            # '***hi***' — fill + center
f"{7:03d}"               # '007' — zero-pad
f"{255:x} {255:b}"       # 'ff 11111111' — hex, binary

from datetime import date
f"{date(2026, 11, 2):%d %b %Y}"   # '02 Nov 2026'

width = 10
f"{'x':>{width}}"        # nested spec: dynamic width
```

### How do encoding and decoding work?

`str` holds Unicode text, while `bytes` holds raw bytes—Python 3 keeps them strictly separate, and mixing them raises `TypeError`. **Encoding** converts text to bytes (`str.encode()`), and **decoding** converts bytes back to text (`bytes.decode()`), both defaulting to UTF-8. Because UTF-8 is variable-length, a string's `len()` can differ from its encoded length. Decoding with the wrong codec raises `UnicodeDecodeError`; the `errors` argument (`"strict"`, `"ignore"`, `"replace"`) controls that behavior. Always pass `encoding="utf-8"` explicitly when opening text files, since the default depends on the operating system. `ord()` and `chr()` convert between a character and its code point.

```python
s = "café"
b = s.encode("utf-8")              # b'caf\xc3\xa9'
len(s), len(b)                     # (4, 5) — é takes 2 bytes in UTF-8
b.decode("utf-8")                  # 'café'
# b.decode("ascii")                # UnicodeDecodeError
b.decode("ascii", errors="replace")   # 'caf��'
ord("A"), chr(65)                  # (65, 'A')
# "a" + b"b"                       # TypeError: can only concatenate str (not "bytes") to str

with open("notes.txt", encoding="utf-8") as f:
    text = f.read()
```

### Mutable vs immutable strings?

Python's `str` is immutable: you can't change a character in place, and every method returns a new string. This design has real benefits—strings are hashable (usable as dict keys and set members), safe to share between threads, and can be cached and reused by the interpreter (string interning, done automatically for many identifier-like strings, or manually with `sys.intern()`). Because of interning, `is` sometimes appears to work for string comparison, but you should always compare strings with `==`. When you need to modify text heavily, convert to a list of characters and `join` it back, use `io.StringIO` as a text buffer, or use `bytearray` for mutable raw bytes.

```python
s = "hello"
# s[0] = "H"                     # TypeError
s = "H" + s[1:]                  # build a new string -> 'Hello'

chars = list("hello")
chars[0] = "J"
"".join(chars)                   # 'Jello'

import io
buf = io.StringIO()
buf.write("line 1\n")
buf.write("line 2\n")
buf.getvalue()                   # 'line 1\nline 2\n'

ba = bytearray(b"cat")
ba[0] = ord("b")                 # mutable bytes -> bytearray(b'bat')
```

## K. Scope & Lifetime

- What is scope?
- Local scope
- Global scope
- Block scope
- Function scope
- Module/package scope
- Lexical scope
- Variable lifetime
- Closures
- Shadowing
- Name resolution

## L. Error & Exception Handling

- What is an error?
- What is an exception?
- Compile-time vs runtime errors?
- How to handle exceptions?
- try/catch or equivalent?
- finally
- How to throw/raise an exception?
- Custom exceptions?
- Exception propagation?
- Multiple exception handling?
- Best practices for exception handling?

## M. Modules, Packages & Imports

- What is a module?
- What is a package?
- How to import code?
- Different import syntaxes?
- Exporting code?
- Namespace?
- Package management?
- Dependency management?
- Standard library?
- Third-party libraries?

## N. Object-Oriented Programming

- What is a class?
- What is an object?
- How to declare a class?
- How to create an object?
- Constructor?
- Instance variables?
- Class/static variables?
- Instance methods?
- Static methods?
- Class methods?
- Encapsulation?
- Inheritance?
- Polymorphism?
- Abstraction?
- Method overriding?
- Method overloading?
- Access modifiers?
- Interfaces/traits/protocols?
- Composition vs inheritance?

## O. Memory & Execution

- Stack vs heap?
- How is memory allocated?
- How are objects stored?
- References/pointers?
- Garbage collection?
- Reference counting?
- Memory management?
- Object lifetime?
- Pass-by-value/reference semantics?
- Copy vs reference?
- Mutable vs immutable objects?

## P. Concurrency & Asynchronous Programming

- Process
- Thread
- Concurrency
- Parallelism
- Async programming
- Event loop
- Futures/Promises
- Callbacks
- async/await
- Synchronization
- Locks/mutexes
- Race conditions
- Deadlocks
- Thread safety

## Q. Functional Programming Concepts

- First-class functions
- Higher-order functions
- Lambda/anonymous functions
- Closures
- Map
- Filter
- Reduce
- Immutability
- Pure functions
- Function composition

## R. File & Resource Handling

- Open/create files
- Read files
- Write files
- Append files
- Close files
- Context/resource management
- File paths
- Directory operations
- Streams/buffers

## S. Language-Specific Features

Finally, give every language a section for its unique machinery.

### For Python

- List comprehensions
- Dictionary comprehensions
- Generator expressions
- Generators
- Iterators
- Decorators
- Context managers
- with
- yield
- \*args
- \*\*kwargs
- `__dunder__` methods
- Dataclasses
- Type hints
- Pattern matching
- property
- Descriptors
- Metaclasses

### For JavaScript

- Prototype chain
- Closures
- Event loop
- Promises
- this
- Destructuring
- Modules
- Generators

### For Go

- Goroutines
- Channels
- Interfaces
- Structs
- Pointers
- Defer
- Embedding
