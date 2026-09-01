Goal: Become interview ready of 4 year experience
Flow of topics/questions:
Basics Of Any language
    A. Language Fundamentals
        Why, when and by whom the Language was developed?
            Python was created by Guido van Rossum and first released in 1991 while he was working at CWI (Centrum Wiskunde & Informatica) in the Netherlands. He began the project in December 1989 as a hobby to keep himself occupied during the Christmas holidays, as a successor to the ABC language he had worked on earlier. ABC was easy to learn but not extensible enough for real-world use, so Guido wanted a language that combined ABC's readability with the extensibility and practicality needed for systems programming. He named it "Python" after the British comedy show "Monty Python's Flying Circus," not the snake, reflecting his intent to make programming fun. Python's development has since been guided by the Python Software Foundation, with major milestones being Python 2.0 (2000) and Python 3.0 (2008), the latter being a backward-incompatible redesign to clean up language inconsistencies.

        Core problem it solved?
            Before Python, developers often had to choose between low-level languages like C, which offered performance and control but demanded manual memory management and verbose syntax, and scripting languages that were easy to use but lacked robustness for larger programs. Python aimed to solve the problem of developer productivity versus power: writing correct, readable, and maintainable code quickly without sacrificing the ability to build serious, scalable applications. It addressed the steep learning curve and boilerplate-heavy nature of languages like C/C++/Java by offering a clean, English-like syntax, automatic memory management, and a "batteries included" standard library. This made it possible to prototype fast, glue different systems together, and still scale to production use, reducing the gap between an idea and working code.

        How it solved?
            Python solved these problems through several deliberate design choices. First, it enforced readability via significant indentation instead of braces, making code visually consistent across projects. Second, it adopted dynamic typing and automatic memory management (garbage collection), removing the need for manual type declarations and memory handling. Third, it provided a huge standard library ("batteries included") covering file I/O, networking, text processing, and more, reducing dependence on external tools. Fourth, it was designed as an interpreted, high-level language, enabling rapid iteration through a REPL without a separate compile step. Fifth, it embraced multi-paradigm support (procedural, object-oriented, functional), letting developers pick the best approach per problem. Finally, its simple, extensible C API allowed performance-critical code to be written in C/C++ and exposed to Python, giving the best of both readability and speed.

        What is the language?
            Python is a high-level, general-purpose, interpreted programming language known for its clean, readable syntax and dynamic typing. It supports multiple programming paradigms, including procedural, object-oriented, and functional programming, and is designed to be highly readable and expressive, often described as "executable pseudocode." Python emphasizes code readability through the use of significant whitespace/indentation rather than curly braces or keywords to delimit blocks. It comes with an extensive standard library and a massive ecosystem of third-party packages (via PyPI), making it suitable for web development, data science, automation, scripting, artificial intelligence, and more. The reference implementation, CPython, compiles source code to bytecode which is then executed by the Python Virtual Machine, though alternative implementations (PyPy, Jython, IronPython) also exist.

        What are its key characteristics?
            Python's key characteristics include: (1) simple, readable syntax using indentation for blocks; (2) dynamic typing, where variable types are determined at runtime; (3) interpreted execution via bytecode on a virtual machine; (4) automatic memory management through reference counting plus a cyclic garbage collector; (5) multi-paradigm support for procedural, object-oriented, and functional styles; (6) "batteries included" standard library covering common tasks; (7) strong typing, meaning implicit unsafe conversions between unrelated types are disallowed; (8) extensibility via C/C++ extension modules; (9) cross-platform portability, running on Windows, macOS, Linux, and more; (10) a large, active open-source community and package ecosystem (PyPI); and (11) first-class support for interactive development through the REPL and notebooks, which makes exploration and rapid prototyping easy.

        What are its use cases? when to choose it?
            Python is widely used in web development (Django, Flask, FastAPI), data science and machine learning (NumPy, pandas, scikit-learn, PyTorch, TensorFlow), automation and scripting (system administration, DevOps tooling), test automation, scientific computing, artificial intelligence, backend APIs, and rapid prototyping. It's also popular for education due to its gentle learning curve. Choose Python when developer productivity, readability, and time-to-market matter more than raw execution speed, when you need access to rich data science/ML libraries, when building glue code integrating multiple systems, or when the team values maintainability over micro-optimized performance. It's less ideal for hard real-time systems, mobile app development, or scenarios demanding maximum CPU performance without relying on compiled extensions, where languages like C++, Rust, or Go may be better suited.

        What are its advantages and limitations?
            Advantages: easy to learn and read, which speeds up development and onboarding; a vast standard library and third-party ecosystem; strong community support; multi-paradigm flexibility; excellent for prototyping and data science; cross-platform; and dynamic typing that speeds up iteration. Limitations: slower execution speed compared to compiled languages like C++, Java, or Go, since it's interpreted and dynamically typed; the Global Interpreter Lock (GIL) in CPython restricts true multi-threaded CPU-bound parallelism; higher memory consumption due to object overhead; runtime type errors that a statically typed language would catch at compile time; and weaker mobile/front-end development support compared to JavaScript or Swift/Kotlin. These trade-offs are usually acceptable because Python's productivity gains outweigh performance costs for most non-performance-critical applications, and performance-critical parts can be offloaded to C extensions.

        Is it compiled, interpreted, or both?
            Python is technically both, though it's commonly labeled "interpreted." When you run a Python script, the CPython interpreter first compiles the source code into an intermediate, platform-independent bytecode (.pyc files), similar in spirit to Java's bytecode. This bytecode is then executed by the Python Virtual Machine (PVM), which interprets it line by line at runtime. This differs from purely compiled languages (like C), where source is translated directly into native machine code ahead of time, and from purely interpreted languages that execute source line-by-line without an intermediate form. This hybrid model gives Python portability (bytecode runs anywhere the PVM exists) while retaining a fast edit-run cycle. Just-In-Time compilers like PyPy go further and compile bytecode to native machine code at runtime for speed.

        What is the runtime/execution environment?
            Python's runtime environment is the Python Virtual Machine (PVM), which executes the compiled bytecode. The most common implementation is CPython (written in C), which includes the compiler (source-to-bytecode), the PVM (bytecode interpreter), memory manager, and the Global Interpreter Lock that serializes thread execution for memory safety. Other implementations include PyPy (uses JIT compilation for speed), Jython (runs on the JVM), and IronPython (runs on .NET's CLR). The runtime also manages the import system, module caching (sys.modules), object allocation/deallocation via reference counting and cyclic garbage collection, and exception handling. Virtual environments (venv, virtualenv, conda) isolate runtime dependencies per project, ensuring consistent execution environments across different machines and deployments.

        What is the difference between syntax and semantics?
            Syntax refers to the set of rules that define how valid statements and expressions must be structured, i.e., the grammar of the language, such as correct indentation, use of colons after if/for/def, and proper parenthesis matching. A syntax error occurs when code violates these structural rules, e.g., missing a colon after "if x > 5", and is caught before the program runs. Semantics refers to the meaning behind syntactically correct code, i.e., what the code actually does when executed. Code can be syntactically valid but semantically wrong, such as dividing by zero, using an undefined variable, or applying incorrect logic that runs without crashing but produces the wrong result. In short, syntax is about "is this written correctly?" while semantics is about "does this do what I intended?"

        What are keywords and reserved words?
            Keywords (also called reserved words) are predefined, reserved identifiers that have special meaning in the language and cannot be used as variable, function, or class names. In Python, there are 35 keywords (as of Python 3.x), including False, None, True, and, as, assert, async, await, break, class, continue, def, del, elif, else, except, finally, for, from, global, if, import, in, is, lambda, nonlocal, not, or, pass, raise, return, try, while, with, and yield. These words are part of the language's core syntax and grammar, used to define control flow, declare functions/classes, handle exceptions, and more. You can check them programmatically via the "keyword" module using keyword.kwlist. Attempting to use a keyword as a variable name (e.g., "class = 5") raises a SyntaxError, since the interpreter cannot distinguish it from its structural role.

        What are identifiers?
            Identifiers are the names used to identify variables, functions, classes, modules, or other objects in code. In Python, valid identifiers must start with a letter (a-z, A-Z) or an underscore (_), followed by any combination of letters, digits (0-9), or underscores; they cannot start with a digit and cannot be a reserved keyword. Python identifiers are case-sensitive, so "myVar" and "myvar" are distinct. Since Python 3, identifiers can also include Unicode characters, allowing names in non-English scripts. Identifiers beginning with an underscore (e.g., "_private") conventionally signal "internal use," a double underscore prefix (e.g., "__mangled") triggers name mangling in classes, and dunder names (e.g., "__init__") are reserved for special/magic methods. Choosing meaningful identifiers is a best practice for readability, even though the interpreter itself has no such requirement.

        What are comments?
            Comments are non-executable lines in source code used to explain logic, document intent, or temporarily disable code, and are ignored by the interpreter. In Python, single-line comments start with a hash symbol (#), extending to the end of the line, e.g., "# this calculates total price". Python doesn't have a dedicated multi-line comment syntax; instead, developers commonly use consecutive single-line "#" comments, or (less correctly) use triple-quoted strings ("""...""") as an unassigned string literal, which is technically a string expression, not a true comment, though it's ignored functionally when not assigned. A special form of a triple-quoted string right after a function, class, or module definition becomes a "docstring," accessible via the __doc__ attribute, used for documentation tools. Good commenting practice explains "why" something is done rather than restating "what" the code already makes obvious.

        What are naming conventions?
            Python's naming conventions are largely defined by PEP 8, the official style guide. Variables and function names should use "snake_case" (lowercase words separated by underscores), e.g., "total_price", "calculate_sum()". Class names should use "PascalCase" (also called CapWords), e.g., "class UserAccount:". Constants are written in "UPPER_SNAKE_CASE", e.g., "MAX_RETRIES = 5", though Python has no true immutable constants—it's purely a convention. Module and package names should be short, all-lowercase, and can use underscores if needed, e.g., "data_utils.py". A single leading underscore (e.g., "_helper") indicates an internal/protected use of a name by convention, and a double leading underscore (e.g., "__value") triggers name mangling inside classes to avoid subclass attribute clashes. Dunder names like "__init__" are reserved for Python's special methods. Following these conventions improves readability and signals intent to other developers.

    B. Variables & Constants
        How to declare a variable?
            Python doesn't have a separate variable declaration step like "int x;" in C/Java. A variable is created the moment you assign a value to a name, e.g., "x = 10". There's no need to specify a type ahead of time because Python is dynamically typed—the interpreter infers the type from the assigned value at runtime. The name (identifier) simply becomes a reference/label pointing to an object in memory. You can also declare a variable's intended type using type hints for readability and static-analysis tools, e.g., "x: int = 10", but this is optional and not enforced by the interpreter itself. If you reference a name before ever assigning it, Python raises a NameError, since there's no concept of an "empty" declared-but-unassigned variable as in statically typed languages.

        How to initialize a variable?
            Initialization in Python happens simultaneously with declaration, since assignment creates the variable and binds it to a value in one step, e.g., "count = 0" or "name = 'Alice'". You can initialize multiple variables in a single line using tuple unpacking, e.g., "a, b, c = 1, 2, 3", or assign the same value to multiple variables at once with chained assignment, e.g., "x = y = z = 0". Variables can also be initialized conditionally, e.g., "status = 'active' if is_valid else 'inactive'", or left uninitialized in a logical sense by assigning None, e.g., "result = None", to indicate "no value yet." Since Python has no uninitialized-variable state, every variable is bound to some object as soon as it exists.

        How to assign/reassign a value?
            Assignment uses the "=" operator, binding a name to an object, e.g., "x = 5". Reassignment simply points the same name to a new object, e.g., "x = 'hello'"—Python allows this because variables are just labels/references, not fixed-type containers, so a name can be rebound to a value of a completely different type. Python also supports augmented assignment operators for updating in place conceptually, e.g., "x += 1" (equivalent to "x = x + 1"), "x *= 2", etc. Multiple assignment and unpacking work too, e.g., "a, b = b, a" to swap values without a temp variable. Importantly, reassignment doesn't mutate the original object (for immutable types like int/str); it just changes what the name refers to, leaving any other references to the old object unaffected.

        How does variable scope work?
            Python resolves variable names using the LEGB rule: Local, Enclosing, Global, Built-in, searched in that order. A variable assigned inside a function is local to that function by default and isn't visible outside it. If a name isn't found locally, Python checks enclosing (outer) function scopes for nested functions, then the global (module-level) scope, and finally Python's built-in namespace (e.g., len, print). Unlike block-scoped languages (C, Java), Python doesn't create new scope for if/for/while blocks—variables defined inside them remain accessible in the enclosing function or module scope after the block ends. To modify a global variable inside a function, you must explicitly declare it with the "global" keyword; similarly, "nonlocal" is used to modify a variable from an enclosing (non-global) function scope inside a nested function.

        What are local, global, and module/class-level variables?
            Local variables are defined inside a function and exist only during that function's execution, destroyed once it returns. Global variables are defined at the top level of a module (script file) and are accessible throughout that module, including inside functions (read-only by default, unless declared "global" for writing). Module-level variables are effectively the same as global variables scoped to that particular .py file's namespace, accessed from other files via imports (e.g., "module.variable"). Class-level variables (also called class/static attributes) are defined directly inside a class body but outside any method, shared across all instances of that class unless shadowed by an instance attribute of the same name, e.g., "class Dog: species = 'Canine'". Instance variables, by contrast, are typically set inside "__init__" using "self.attribute = value" and are unique per object.

        Does the language support constants?
            Python doesn't have a built-in, enforced constant mechanism like "const" in JavaScript or "final" in Java—there's no way to make a variable truly immutable at the language level. Instead, constants are a convention: developers name them in UPPER_SNAKE_CASE (e.g., "MAX_CONNECTIONS = 100", "PI = 3.14159") to signal that the value shouldn't be changed, but the interpreter won't prevent reassignment. Some approaches simulate stricter constant behavior: using a NamedTuple or an Enum class for grouped constants, overriding "__setattr__" in a custom class to raise an error on reassignment, or using "typing.Final" (from Python 3.8+) as a type-checker hint, e.g., "MAX: Final = 100", which tools like mypy will flag if violated, though it still doesn't stop reassignment at runtime.

        How does variable declaration differ from variable definition?
            In statically typed languages, declaration announces a variable's name and type without necessarily giving it a value (e.g., "int x;" in C), while definition allocates storage and often assigns an initial value. Python collapses these two concepts into one step: since there's no separate type declaration, simply writing "x = 10" both "declares" the name x in the current scope and "defines" it by binding it to the integer object 10 immediately. There's no Python syntax to declare a variable without giving it some initial value or reference. The closest conceptual equivalent to "declaring without defining" is using type hints without assignment inside class bodies for dataclasses or typing purposes (e.g., "x: int" inside a class), which doesn't create an actual variable until it's assigned.

        How are variables stored/referenced internally?
            In Python (CPython), variables are names stored in a namespace (implemented as a dictionary) that map to references (pointers) to objects in memory, rather than the variable directly holding the object's data. When you write "x = 10", Python creates an integer object 10 somewhere in memory and stores a reference to it under the key "x" in the current scope's namespace (e.g., accessible via locals() or globals()). Every object carries a reference count, tracking how many names/containers point to it; when a name is reassigned or deleted, its reference count on the old object decreases, and the object is garbage-collected once the count hits zero (plus a cyclic garbage collector for reference cycles). Small integers and interned strings are cached and reused by CPython for efficiency, which is why multiple variables can sometimes share the same object identity.

        What is dynamic vs static typing?
            In statically typed languages (Java, C, Go), variable types are declared explicitly and checked at compile time, so type mismatches are caught before the program runs, e.g., "int x = 5;" and assigning a string to x later would fail to compile. In dynamically typed languages like Python, types are determined and checked at runtime, not compile time—the same variable name can be rebound to different types over its lifetime, e.g., "x = 5" then later "x = 'hello'" is perfectly valid. Python defers all type resolution to execution, which speeds up development and increases flexibility but shifts type-related bugs from compile-time errors to runtime exceptions (like TypeError). Optional static type hints (via the "typing" module and tools like mypy) let Python developers opt into some compile-time-style checks without changing its fundamentally dynamic nature.

        What is strong vs weak typing?
            Strong vs weak typing describes how strictly a language enforces type rules during operations, independent of static/dynamic typing. A strongly typed language disallows implicit, unsafe conversions between incompatible types, raising an error instead, e.g., in Python, "'5' + 3" raises a TypeError rather than silently converting one type to match the other. A weakly typed language (like JavaScript or C) permits such implicit coercions, e.g., in JavaScript, "'5' + 3" silently produces "'53'". Python is dynamically typed AND strongly typed: types aren't declared upfront (dynamic), but once an object has a type, Python enforces that type's rules strictly during operations and requires explicit conversions (e.g., "str(3)" or "int('5')") to mix types. This strong typing helps catch bugs early by failing loudly on nonsensical operations rather than silently producing unexpected values.

    C. Data Types
        What are the built-in/primitive data types?
        What are reference/composite/complex data types?
        How do you declare/create each data type?
        What are the properties of each type?
        What operations are supported by each type?
        What methods are available on each type?
        Mutable vs immutable data types?
        Value types vs reference types?
        How does type conversion work?
        Implicit vs explicit type conversion?
        How do you check the type of a variable?

    D. Operators
        Arithmetic operators
        Assignment operators
        Comparison operators
        Logical operators
        Bitwise operators
        Increment/decrement operators, if supported
        Membership operators
        Identity operators
        Ternary/conditional operators
        Operator precedence
        Operator associativity

    E. Conditional Statements
        How to declare an if statement?
        How to declare else?
        How to declare else if?
        How to create nested conditions?
        How does the ternary/conditional expression work?
        How are multiple conditions evaluated?
        Short-circuit evaluation?
        switch/case or equivalent?
        Pattern matching, if supported?

    F. Loops & Iteration
        What types of loops are available?
        How to declare a for loop?
        How to declare a while loop?
        Is there a do-while loop?
        How does iteration work internally?
        How to iterate over collections?
        Nested loops?
        break
        continue
        pass / equivalent
        How to terminate a loop?
        How to skip an iteration?
        How to iterate with index?
        How to iterate over key/value pairs?
        Language's iteration protocol

    G. Functions
        How to declare a function?
        How to call a function?
        Parameters vs arguments?
        Positional arguments?
        Keyword/named arguments?
        Default parameters?
        Variable-length arguments?
        Return values?
        Multiple return values?
        Function scope?
        Nested functions?
        Anonymous/lambda functions?
        First-class functions?
        Higher-order functions?
        Recursion?
        Function overloading?
        Function overriding?
        Call by value vs reference/reference semantics?

    H. Input & Output
        How to print/output data?
        How to take user input?
        How to format output?
        String interpolation?
        Reading from standard input?
        Writing to standard output?
        File input/output?
        Serialization/deserialization?

    I. Collections / Data Structures
        Arrays
        Lists
        Tuples
        Sets
        Maps/Dictionaries
        Stacks
        Queues
        Linked lists, if language-supported
        Trees/graphs, where relevant
        How to add/remove/update elements?
        Searching?
        Sorting?
        Iteration?
        Slicing?
        Copying?
        Shallow vs deep copy?
        Comprehensions / collection expressions?

    J. Strings
        Creating strings
        String concatenation
        String interpolation
        String indexing
        String slicing
        String methods
        Searching
        Replacing
        Splitting
        Joining
        Trimming
        Case conversion
        Formatting
        Encoding/decoding
        Mutable vs immutable strings

    K. Scope & Lifetime
        What is scope?
        Local scope
        Global scope
        Block scope
        Function scope
        Module/package scope
        Lexical scope
        Variable lifetime
        Closures
        Shadowing
        Name resolution

    L. Error & Exception Handling
        What is an error?
        What is an exception?
        Compile-time vs runtime errors?
        How to handle exceptions?
        try/catch or equivalent?
        finally
        How to throw/raise an exception?
        Custom exceptions?
        Exception propagation?
        Multiple exception handling?
        Best practices for exception handling?

    M. Modules, Packages & Imports
        What is a module?
        What is a package?
        How to import code?
        Different import syntaxes?
        Exporting code?
        Namespace?
        Package management?
        Dependency management?
        Standard library?
        Third-party libraries?

    N. Object-Oriented Programming
        What is a class?
        What is an object?
        How to declare a class?
        How to create an object?
        Constructor?
        Instance variables?
        Class/static variables?
        Instance methods?
        Static methods?
        Class methods?
        Encapsulation?
        Inheritance?
        Polymorphism?
        Abstraction?
        Method overriding?
        Method overloading?
        Access modifiers?
        Interfaces/traits/protocols?
        Composition vs inheritance?

    O. Memory & Execution
        Stack vs heap?
        How is memory allocated?
        How are objects stored?
        References/pointers?
        Garbage collection?
        Reference counting?
        Memory management?
        Object lifetime?
        Pass-by-value/reference semantics?
        Copy vs reference?
        Mutable vs immutable objects?

    P. Concurrency & Asynchronous Programming
        Process
        Thread
        Concurrency
        Parallelism
        Async programming
        Event loop
        Futures/Promises
        Callbacks
        async/await
        Synchronization
        Locks/mutexes
        Race conditions
        Deadlocks
        Thread safety

    Q. Functional Programming Concepts
        First-class functions
        Higher-order functions
        Lambda/anonymous functions
        Closures
        Map
        Filter
        Reduce
        Immutability
        Pure functions
        Function composition
        
    R. File & Resource Handling
        Open/create files
        Read files
        Write files
        Append files
        Close files
        Context/resource management
        File paths
        Directory operations
        Streams/buffers
        
    S. Language-Specific Features
        Finally, give every language a section for its unique machinery.

        For Python:
        List comprehensions
        Dictionary comprehensions
        Generator expressions
        Generators
        Iterators
        Decorators
        Context managers
        with
        yield
        *args
        **kwargs
        __dunder__ methods
        Dataclasses
        Type hints
        Pattern matching
        property
        Descriptors
        Metaclasses

        For JavaScript:
        Prototype chain
        Closures
        Event loop
        Promises
        this
        Destructuring
        Modules
        Generators

        For Go:
        Goroutines
        Channels
        Interfaces
        Structs
        Pointers
        Defer
        Embedding
                