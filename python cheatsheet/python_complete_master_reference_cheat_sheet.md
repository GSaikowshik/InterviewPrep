# Python Master Reference Cheat Sheet

---

## Table of Contents
1. [Syntax Fundamentals & Primitive Types](#1-syntax-fundamentals--primitive-types)
2. [Data Structures & Collections](#2-data-structures--collections)
3. [Control Flow & Comprehensions](#3-control-flow--comprehensions)
4. [Functions & Argument Parsing Mechanics](#4-functions--argument-parsing-mechanics)
5. [Scoping, Closures & Decorators](#5-scoping-closures--decorators)
6. [Object-Oriented Programming (OOP) & Dunder Methods](#6-object-oriented-programming-oop--dunder-methods)
7. [Iterators, Generators & Context Managers](#7-iterators-generators--context-managers)
8. [Type Annotations & Structural Subtyping](#8-type-annotations--structural-subtyping)
9. [Concurrency & Parallelism](#9-concurrency--parallelism)
10. [Metaprogramming & Descriptors](#10-metaprogramming--descriptors)
11. [Memory Model & Performance Internals](#11-memory-model--performance-internals)

---

## 1. Syntax Fundamentals & Primitive Types

### Core Primitives & Conversions
```python
# Scalar Types
val_int: int = 42
val_float: float = 3.14159
val_bool: bool = True
val_none: None = None
val_complex: complex = 2 + 3j

# Type conversions
int_val = int("101", base=2)      # Binary string to int -> 5
float_val = float("1.25e-2")      # Scientific notation -> 0.0125
bool_truthy = bool("non-empty")   # True (0, "", [], {}, None evaluate to False)
```

### String Operations & Formatting
```python
text = " Python Architecture "

# Slicing syntax: [start:stop:step]
stripped = text.strip()           # 'Python Architecture'
sliced = stripped[0:6]            # 'Python'
reversed_str = stripped[::-1]     # 'erutcetihcrA nohtyP'

# f-Strings (Formatting & Debugging)
version = 3.12
price = 149.998
print(f"{version=}")              # 'version=3.12'
print(f"Price: ${price:.2f}")     # 'Price: $150.00'
print(f"{'Centered':^20}")        # Centered padding with spaces

# Transformations
tokens = "a,b,c".split(",")       # ['a', 'b', 'c']
joined = " -> ".join(tokens)      # 'a -> b -> c'
```

---

## 2. Data Structures & Collections

```python
# --- LIST (Mutable, Ordered sequence) ---
items = [1, 2, 3]
items.append(4)                   # In-place append
items.extend([5, 6])              # Concatenate iterable
items.insert(0, 0)                # Insert at index (O(n))
popped = items.pop()              # Removes last item (O(1))
items.sort(reverse=True)          # Timsort algorithm (O(n log n))

# --- TUPLE (Immutable sequence) ---
point = (10, 20, "start")
x, y, label = point               # Destructuring / Unpacking

# --- DICTIONARY (Hash map, O(1) avg lookup) ---
user = {"id": 1, "name": "Dev"}
user["role"] = "Admin"
email = user.get("email", "N/A")  # Safe retrieval with fallback
keys = user.keys()                # Dynamic dict_keys view
items_view = user.items()         # Dynamic dict_items view

# --- SET (Unique, unordered, hashable elements) ---
s1 = {1, 2, 3, 4}
s2 = {3, 4, 5, 6}
union_s = s1 | s2                 # {1, 2, 3, 4, 5, 6}
inter_s = s1 & s2                 # {3, 4}
diff_s = s1 - s2                  # {1, 2}

# --- COLLECTIONS MODULE HIGHLIGHTS ---
from collections import deque, Counter, defaultdict

dq = deque([1, 2], maxlen=3)      # O(1) appends and pops on both ends
dq.appendleft(0)
counts = Counter("mississippi")    # Frequency table: {'i': 4, 's': 4, 'p': 2, 'm': 1}
tree = defaultdict(list)          # Missing keys auto-initialized to []
```

---

## 3. Control Flow & Comprehensions

### Conditional Execution & Pattern Matching
```python
# Walrus Operator (Assignment expression: := )
if (n := len([1, 2, 3, 4])) > 3:
    print(f"List is large: {n} items")

# Structural Pattern Matching (Python 3.10+)
def dispatch_action(event: dict):
    match event:
        case {"type": "click", "coords": (x, y)}:
            return f"Clicked at {x}, {y}"
        case {"type": "keypress", "key": "Enter" | "Return"}:
            return "Submit triggered"
        case {"type": "scroll", "offset": int(offset)} if offset > 100:
            return "Fast scroll detected"
        case _:
            return "Unrecognized event"
```

### Comprehensions
```python
# List comprehension
evens = [x for x in range(10) if x % 2 == 0]

# Dict comprehension
square_map = {x: x**2 for x in range(5)}

# Set comprehension
distinct_lens = {len(w) for w in ["apple", "pear", "banana", "kiwi"]}

# Generator Expression (Evaluated lazily, low memory footprint)
stream = (x**2 for x in range(100_000_000))  # Does not allocate in memory
next(stream)                                  # 0
```

---

## 4. Functions & Argument Parsing Mechanics

### Unified Signature Specification
```text
def signature(pos_only_1, /, standard, *args, kw_only, **kwargs):
                 ▲        ▲    ▲         ▲        ▲        ▲
Positional-Only ─┘        │    │         │        │        │
Slash Boundary ───────────┘    │         │        │        │
Standard (Pos/Kw) ─────────────┘         │        │        │
Var-Positional (Tuple) ──────────────────┘        │        │
Keyword-Only (Named) ─────────────────────────────┘        │
Var-Keyword (Dict) ────────────────────────────────────────┘
```

### Argument Syntax Examples
```python
def complete_func(pos1, pos2, /, regular, *args, kw_only=True, **kwargs):
    return {
        "pos": (pos1, pos2),
        "reg": regular,
        "extra_args": args,
        "kw_only": kw_only,
        "extra_kwargs": kwargs
    }

# Invocation:
res = complete_func(
    10, 20,                  # pos1, pos2 MUST be positional
    "regular_val",           # Can be positional or keyword
    100, 200,                # *args
    kw_only=False,           # MUST be passed as keyword
    env="prod", retries=3    # **kwargs
)
```

### Parameter Unpacking
```python
def execute(a, b, c):
    return a + b + c

seq = [1, 2, 3]
params = {"a": 10, "b": 20, "c": 30}

print(execute(*seq))     # Iterable unpacking -> 6
print(execute(**params)) # Mapping unpacking  -> 60
```

### Mutable Default Trap & Sentinel Fix
```python
# WRONG: Default list evaluated once when module imports
def broken_append(item, store=[]):
    store.append(item)
    return store

# CORRECT: Use None as immutable sentinel
def safe_append(item, store=None):
    if store is None:
        store = []
    store.append(item)
    return store
```

---

## 5. Scoping, Closures & Decorators

### LEGB Rule & Scope Modifiers
Variables resolve in order: **L**ocal $\rightarrow$ **E**nclosing $\rightarrow$ **G**lobal $\rightarrow$ **B**uilt-in.

```python
outer_var = "global"

def parent():
    enclosed_var = "enclosing"
    
    def child():
        nonlocal enclosed_var
        global outer_var
        enclosed_var = "modified enclosing"
        outer_var = "modified global"
        
    child()
    return enclosed_var

parent()
```

### Closures & Parameterized Decorators
```python
import functools
import time

def retry(times: int, backoff: float = 0.1):
    """Parameterized decorator template."""
    def decorator(func):
        @functools.wraps(func)  # Retains __name__, __doc__, annotations
        def wrapper(*args, **kwargs):
            last_exception = None
            for attempt in range(times):
                try:
                    return func(*args, **kwargs)
                except Exception as exc:
                    last_exception = exc
                    time.sleep(backoff * (2 ** attempt))
            raise last_exception
        return wrapper
    return decorator

@retry(times=3, backoff=0.05)
def call_api(endpoint: str) -> str:
    # Simulated failing call
    return f"Response from {endpoint}"
```

---

## 6. Object-Oriented Programming (OOP) & Dunder Methods

### Encapsulation, Methods & Inheritance
```python
from abc import ABC, abstractmethod

class BaseEntity(ABC):
    """Abstract Base Class enforcing interface."""
    @abstractmethod
    def get_id(self) -> str:
        pass

class User(BaseEntity):
    domain = "internal.corp"  # Class variable

    def __init__(self, user_id: str, username: str, salary: float):
        self._user_id = user_id         # Protected
        self.username = username        # Public
        self.__salary = salary          # Private (Mangled to _User__salary)

    def get_id(self) -> str:
        return self._user_id

    # Property Getters / Setters
    @property
    def salary(self) -> float:
        return self.__salary

    @salary.setter
    def salary(self, value: float):
        if value < 0:
            raise ValueError("Salary cannot be negative")
        self.__salary = value

    @classmethod
    def from_domain(cls, user_id: str, name: str):
        """Alternative constructor."""
        return cls(user_id, f"{name}@{cls.domain}", 0.0)

    @staticmethod
    def is_valid_name(name: str) -> bool:
        return len(name) >= 3
```

### Essential Magic (Dunder) Methods
```python
class Vector2D:
    __slots__ = ("x", "y")  # Suppresses __dict__ for memory efficiency

    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        """Machine/Debug string representation."""
        return f"Vector2D(x={self.x}, y={self.y})"

    def __str__(self) -> str:
        """Human-readable representation."""
        return f"({self.x}, {self.y})"

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Vector2D):
            return NotImplemented
        return self.x == other.x and self.y == other.y

    def __add__(self, other: "Vector2D") -> "Vector2D":
        return Vector2D(self.x + other.x, self.y + other.y)

    def __len__(self) -> int:
        return 2

    def __getitem__(self, index: int) -> float:
        if index == 0: return self.x
        if index == 1: return self.y
        raise IndexError("Index out of bounds")
```

### Modern Idiom: `@dataclass`
```python
from dataclasses import dataclass, field

@dataclass(slots=True, order=True)
class DataPacket:
    priority: int
    payload: str = field(compare=False)
    timestamp: float = field(default=0.0, compare=False)
```

---

## 7. Iterators, Generators & Context Managers

### Custom Iterator Protocol
```python
class StepRange:
    def __init__(self, limit: int, step: int):
        self.limit = limit
        self.step = step
        self.current = 0

    def __iter__(self):
        return self  # Iterator must return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration
        result = self.current
        self.current += self.step
        return result
```

### Generators & Sub-Generator Delegation
```python
def number_stream():
    yield 1
    yield 2

def integrated_stream():
    yield "Start"
    yield from number_stream()  # Delegates iteration to number_stream()
    yield "End"
```

### Context Managers (`__enter__` / `__exit__` & `contextlib`)
```python
from contextlib import contextmanager

# Class-based Context Manager
class ManagedResource:
    def __enter__(self):
        # Setup phase
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        # Teardown phase. Return True to swallow exceptions, False to raise.
        return False

# Functional generator context manager
@contextmanager
def temporary_lock(lock_id: str):
    print(f"Acquired lock {lock_id}")
    try:
        yield lock_id
    finally:
        print(f"Released lock {lock_id}")
```

---

## 8. Type Annotations & Structural Subtyping

```python
from typing import Callable, TypeVar, Generic, Protocol

# Type Aliases and Union (Python 3.10+ Pipe Syntax)
Scalar = int | float
JsonValue = dict[str, "JsonValue"] | list["JsonValue"] | str | int | float | bool | None

# Generics
T = TypeVar("T")

class Stack(Generic[T]):
    def __init__(self):
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

# Protocol (Static Duck Typing / Structural Subtyping)
class Serializable(Protocol):
    def serialize(self) -> str: ...

def transmit(item: Serializable) -> None:
    data = item.serialize()
```

---

## 9. Concurrency & Parallelism

### Paradigm Selection Grid

| Paradigm | Mechanism | Best Used For | GIL Bound? |
| :--- | :--- | :--- | :--- |
| **`asyncio`** | Cooperative single-thread Event Loop | Large-scale concurrent I/O (Web, Sockets) | Yes |
| **`threading`** | Preemptive OS Threads | Blocking I/O, C-extensions releasing GIL | Yes (for Python bytecode) |
| **`multiprocessing`**| Distinct OS processes with isolated heaps | CPU-heavy computation, Data Processing | No (bypasses GIL) |

### `asyncio` & Structured Task Groups (Python 3.11+)
```python
import asyncio

async def fetch_record(record_id: int) -> dict:
    await asyncio.sleep(0.05)
    return {"id": record_id, "data": "payload"}

async def async_pipeline():
    # Structured Concurrency: Automatically cancels remaining tasks on exception
    async with asyncio.TaskGroup() as tg:
        task1 = tg.create_task(fetch_record(1))
        task2 = tg.create_task(fetch_record(2))

    print(task1.result(), task2.result())

# Running asynchronous entry point
asyncio.run(async_pipeline())
```

### Threading with `ThreadPoolExecutor`
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def sync_io_task(n: int) -> str:
    time.sleep(0.01)
    return f"Done: {n}"

with ThreadPoolExecutor(max_workers=4) as executor:
    futures = [executor.submit(sync_io_task, i) for i in range(4)]
    for f in as_completed(futures):
        res = f.result()
```

### Multiprocessing with `ProcessPoolExecutor`
```python
from concurrent.futures import ProcessPoolExecutor
import os

def compute_square(num: int) -> int:
    return num * num

# Guard strictly required to prevent recursive process spawning
if __name__ == "__main__":
    with ProcessPoolExecutor() as executor:
        inputs = [10_000, 20_000, 30_000]
        results = list(executor.map(compute_square, inputs))
```

---

## 10. Metaprogramming & Descriptors

### Descriptors (Attribute Access Interception)
A descriptor is an object that defines `__get__`, `__set__`, or `__delete__`.

```python
class ValidatedString:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = f"_{name}"

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.private_name, "")

    def __set__(self, instance, value):
        if not isinstance(value, str):
            raise TypeError(f"{self.public_name} must be a string")
        setattr(instance, self.private_name, value)

class Person:
    name = ValidatedString()  # Managed via Descriptor

    def __init__(self, name: str):
        self.name = name
```

### Custom Metaclass
Metaclasses control class generation (`type` is the default metaclass).

```python
class InterfaceEnforcerMeta(type):
    """Metaclass verifying class attributes during class creation."""
    def __new__(mcs, name, bases, namespace):
        if name != "BasePlugin" and "execute" not in namespace:
            raise TypeError(f"Class '{name}' must implement 'execute' method")
        return super().__new__(mcs, name, bases, namespace)

class BasePlugin(metaclass=InterfaceEnforcerMeta):
    pass

# class FailingPlugin(BasePlugin):
#     pass  # Raises TypeError at definition/import time
```

---

## 11. Memory Model & Performance Internals

### Global Interpreter Lock (GIL) & Reference Counting
* Python uses **Reference Counting** supplemented by a **Generational Garbage Collector** (detecting cyclic references).
* The **GIL** ensures only one thread executes Python bytecode at a time, protecting memory safety for reference counters (`ob_refcnt`).

### Inspecting References
```python
import sys

ref_target = []
print(sys.getrefcount(ref_target))  # Typically returns 2 (target + arg to getrefcount)
```

### Copying Semantics: Shallow vs Deep
```python
import copy

nested = [[1, 2], [3, 4]]
shallow = list(nested)        # Outer list is copied, inner lists remain identical references
deep = copy.deepcopy(nested)  # Recursively copies full object graph

nested[0][0] = 999
print(shallow[0][0])          # 999 (Mutated)
print(deep[0][0])             # 1 (Isolated copy)
```

### Memory Optimization with `__slots__`
Instances normally have an internal dictionary `__dict__` for dynamic attribute storage. Declaring `__slots__` stores variables in a static array:
```python
class HeavyClass:
    def __init__(self, x, y):
        self.x = x
        self.y = y

class LeanClass:
    __slots__ = ("x", "y")  # Eliminates __dict__ & __weakref__ overhead
    def __init__(self, x, y):
        self.x = x
        self.y = y
```