# Python Examples With Solutions

This workbook supplements `python_complete_master_reference_cheat_sheet.md`.
The original cheat sheet is unchanged. Each exercise includes a short task,
one possible solution, and the expected result.

## Table of Contents

1. [Syntax Fundamentals and Primitive Types](#1-syntax-fundamentals-and-primitive-types)
2. [Data Structures and Collections](#2-data-structures-and-collections)
3. [Control Flow and Comprehensions](#3-control-flow-and-comprehensions)
4. [Functions and Argument Parsing](#4-functions-and-argument-parsing)
5. [Scoping, Closures, and Decorators](#5-scoping-closures-and-decorators)
6. [OOP and Dunder Methods](#6-oop-and-dunder-methods)
7. [Iterators, Generators, and Context Managers](#7-iterators-generators-and-context-managers)
8. [Type Annotations and Structural Subtyping](#8-type-annotations-and-structural-subtyping)
9. [Concurrency and Parallelism](#9-concurrency-and-parallelism)
10. [Metaprogramming and Descriptors](#10-metaprogramming-and-descriptors)
11. [Memory Model and Performance](#11-memory-model-and-performance)

---

## 1. Syntax Fundamentals and Primitive Types

### Example 1.1: Convert and classify input

**Task:** Convert a binary string to an integer, parse a decimal, and report
the truthiness of a user-provided value.

```python
binary_text = "10110"
decimal_text = "3.75"
value = "ready"

number = int(binary_text, 2)
decimal = float(decimal_text)
is_present = bool(value)

print(number, decimal, is_present)
```

**Output:**

```text
22 3.75 True
```

`bool("")`, `bool(0)`, `bool([])`, and `bool(None)` are false; most non-empty
values are true.

### Example 1.2: Slice, transform, and format text

**Task:** Clean a name, take its first three characters, reverse it, and
create a display string.

```python
raw_name = "  Ada Lovelace  "
name = raw_name.strip()
first_three = name[:3]
reversed_name = name[::-1]
display = f"{name=}; prefix={first_three!r}; reversed={reversed_name!r}"

print(display)
print("Ada,Lovelace".split(","))
print(" / ".join(["Python", "SQL", "Linux"]))
```

**Output:**

```text
name='Ada Lovelace'; prefix='Ada'; reversed='ecalevoL adA'
['Ada', 'Lovelace']
Python / SQL / Linux
```

**Slicing diagram:**

```text
text:  P  y  t  h  o  n
index: 0  1  2  3  4  5
text[1:5] -> y  t  h  o
text[::-1] -> n  o  h  t  y  P
```

### Example 1.3: Use a complex number

**Task:** Calculate the magnitude of `3 + 4j`.

```python
z = 3 + 4j
magnitude = abs(z)
print(magnitude)
```

**Output:**

```text
5.0
```

---

## 2. Data Structures and Collections

### Example 2.1: Choose the right collection

**Task:** Maintain an ordered task list, immutable coordinates, user fields,
and unique tags.

```python
tasks = ["read", "code"]
tasks.append("test")
tasks[0] = "review"

point = (10, 20)
x, y = point

user = {"id": 7, "name": "Mina"}
user["role"] = "admin"
email = user.get("email", "not supplied")

tags = {"python", "backend", "python"}

print(tasks)
print(x, y)
print(user, email)
print(sorted(tags))
```

**Output:**

```text
['review', 'code', 'test']
10 20
{'id': 7, 'name': 'Mina', 'role': 'admin'} not supplied
['backend', 'python']
```

Lists are mutable and ordered, tuples are immutable sequences, dictionaries
map hashable keys to values, and sets remove duplicates.

### Example 2.2: Set operations

**Task:** Find common skills, skills unique to one candidate, and all skills.

```python
alice = {"python", "sql", "git"}
bob = {"python", "docker", "git"}

print(alice & bob)  # intersection
print(alice - bob)  # difference
print(alice | bob)  # union
```

**Output:**

```text
{'python', 'git'}
{'sql'}
{'python', 'sql', 'git', 'docker'}
```

### Example 2.3: Use `deque`, `Counter`, and `defaultdict`

**Task:** Keep the last three events, count letters, and group names by team.

```python
from collections import Counter, defaultdict, deque

recent = deque(maxlen=3)
for event in ["login", "search", "logout", "login"]:
    recent.append(event)

counts = Counter("banana")
teams = defaultdict(list)
for name, team in [("Ada", "eng"), ("Lin", "eng"), ("Grace", "qa")]:
    teams[team].append(name)

print(list(recent))
print(counts["a"])
print(dict(teams))
```

**Output:**

```text
['search', 'logout', 'login']
3
{'eng': ['Ada', 'Lin'], 'qa': ['Grace']}
```

---

## 3. Control Flow and Comprehensions

### Example 3.1: Validate with the walrus operator

**Task:** Read a value once, convert it, and branch on the converted result.

```python
text = "42"
if (number := int(text)) >= 40:
    message = f"{number} is large enough"
else:
    message = f"{number} is too small"

print(message)
```

**Output:**

```text
42 is large enough
```

Use `:=` when assigning and testing in one expression improves clarity; do
not use it merely to shorten ordinary code.

### Example 3.2: Dispatch events with pattern matching

**Task:** Return a useful message for different event shapes.

```python
def dispatch(event: dict) -> str:
    match event:
        case {"type": "click", "coords": (x, y)}:
            return f"click at {x},{y}"
        case {"type": "keypress", "key": ("Enter" | "Return")}:
            return "submit"
        case {"type": "scroll", "offset": int(offset)} if offset > 100:
            return "fast scroll"
        case _:
            return "unknown event"


print(dispatch({"type": "click", "coords": (4, 9)}))
print(dispatch({"type": "scroll", "offset": 120}))
```

**Output:**

```text
click at 4,9
fast scroll
```

### Example 3.3: Replace loops with readable comprehensions

**Task:** Produce even numbers, squares, unique word lengths, and a lazy
sequence.

```python
evens = [n for n in range(10) if n % 2 == 0]
squares = {n: n * n for n in range(4)}
lengths = {len(word) for word in ["cat", "horse", "dog"]}
lazy_squares = (n * n for n in range(1_000_000))

print(evens)
print(squares)
print(sorted(lengths))
print(next(lazy_squares), next(lazy_squares))
```

**Output:**

```text
[0, 2, 4, 6, 8]
{0: 0, 1: 1, 2: 4, 3: 9}
[3, 5]
0 1
```

```text
list comprehension:  input -> filter -> list in memory
generator expression: input -> one value at a time
```

---

## 4. Functions and Argument Parsing

### Example 4.1: Understand every parameter category

**Task:** Write a function with positional-only, regular, variadic,
keyword-only, and keyword-variadic arguments.

```python
def describe(first, second, /, label, *values, scale=1, **metadata):
    return {
        "pair": (first, second),
        "label": label,
        "values": tuple(value * scale for value in values),
        "metadata": metadata,
    }


result = describe(2, 3, "numbers", 4, 5, scale=10, source="test")
print(result)
```

**Output:**

```text
{'pair': (2, 3), 'label': 'numbers', 'values': (40, 50), 'metadata': {'source': 'test'}}
```

**Signature diagram:**

```text
def f(pos_only, /, regular, *args, keyword_only, **kwargs)
       |       |      |       |       |             |
       |       |      |       |       +------------- named keyword only
       |       |      |       +--------------------- extra positional tuple
       |       |      +----------------------------- positional or keyword
       |       +------------------------------------ slash boundary
       +-------------------------------------------- positional only
```

### Example 4.2: Unpack arguments

**Task:** Call one function from a sequence and a mapping.

```python
def total(a: int, b: int, c: int) -> int:
    return a + b + c


numbers = [1, 2, 3]
named_numbers = {"a": 10, "b": 20, "c": 30}
print(total(*numbers))
print(total(**named_numbers))
```

**Output:**

```text
6
60
```

### Example 4.3: Fix a mutable default argument

**Task:** Ensure separate calls do not share an accidental list.

```python
def add_bad(item, bucket=[]):
    bucket.append(item)
    return bucket


def add_good(item, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(item)
    return bucket


print(add_bad("a"))
print(add_bad("b"))
print(add_good("a"))
print(add_good("b"))
```

**Output:**

```text
['a']
['a', 'b']
['a']
['b']
```

The default expression is evaluated once when the function is defined. A
`None` sentinel creates a fresh list for each call.

---

## 5. Scoping, Closures, and Decorators

### Example 5.1: Apply LEGB, `global`, and `nonlocal`

**Task:** Update an enclosing counter without creating a global counter.

```python
message = "module"


def make_counter():
    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment


counter = make_counter()
print(counter(), counter())
print(message)
```

**Output:**

```text
1 2
module
```

```text
local -> enclosing -> global -> built-in
```

Use `nonlocal` for an enclosing variable and `global` only when changing a
module-level binding is genuinely intended.

### Example 5.2: Build a parameterized decorator

**Task:** Log calls while preserving the wrapped function's name and docstring.

```python
from functools import wraps


def announce(prefix: str):
    def decorator(function):
        @wraps(function)
        def wrapper(*args, **kwargs):
            print(f"{prefix}: calling {function.__name__}")
            result = function(*args, **kwargs)
            print(f"{prefix}: result={result}")
            return result

        return wrapper

    return decorator


@announce("TRACE")
def add(a: int, b: int) -> int:
    """Return the sum of two numbers."""
    return a + b


print(add(2, 3))
print(add.__name__)
```

**Output:**

```text
TRACE: calling add
TRACE: result=5
5
add
```

The decorator factory receives configuration, the decorator receives the
function, and the wrapper receives the function's call arguments.

---

## 6. OOP and Dunder Methods

### Example 6.1: Encapsulation, inheritance, and alternative constructors

**Task:** Protect a salary invariant while exposing a convenient constructor.

```python
from abc import ABC, abstractmethod


class Entity(ABC):
    @abstractmethod
    def get_id(self) -> str:
        raise NotImplementedError


class User(Entity):
    domain = "example.test"

    def __init__(self, user_id: str, username: str, salary: float):
        self._user_id = user_id
        self.username = username
        self.__salary = salary

    def get_id(self) -> str:
        return self._user_id

    @property
    def salary(self) -> float:
        return self.__salary

    @salary.setter
    def salary(self, value: float) -> None:
        if value < 0:
            raise ValueError("salary cannot be negative")
        self.__salary = value

    @classmethod
    def from_name(cls, user_id: str, name: str):
        return cls(user_id, f"{name}@{cls.domain}", 0.0)

    @staticmethod
    def valid_name(name: str) -> bool:
        return len(name) >= 3


user = User.from_name("u1", "ada")
user.salary = 100.0
print(user.get_id(), user.username, user.salary, User.valid_name("Ada"))
```

**Output:**

```text
u1 ada@example.test 100.0 True
```

### Example 6.2: Implement useful dunder methods

**Task:** Make a 2D point printable, comparable, addable, indexable, and
usable with `len`.

```python
class Point:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y

    def __repr__(self) -> str:
        return f"Point(x={self.x}, y={self.y})"

    def __str__(self) -> str:
        return f"({self.x}, {self.y})"

    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Point):
            return NotImplemented
        return (self.x, self.y) == (other.x, other.y)

    def __add__(self, other: "Point") -> "Point":
        return Point(self.x + other.x, self.y + other.y)

    def __len__(self) -> int:
        return 2

    def __getitem__(self, index: int) -> float:
        return (self.x, self.y)[index]


first = Point(1, 2)
second = Point(3, 4)
print(str(first), repr(first), first + second, first == Point(1, 2))
print(len(first), first[0])
```

**Output:**

```text
(1, 2) Point(x=1, y=2) (4, 6) True
2 1
```

### Example 6.3: Use a dataclass

**Task:** Compare packets by priority but ignore payload when ordering.

```python
from dataclasses import dataclass, field


@dataclass(order=True, slots=True)
class Packet:
    priority: int
    payload: str = field(compare=False)


packets = [Packet(2, "slow"), Packet(1, "urgent")]
print(sorted(packets))
```

**Output:**

```text
[Packet(priority=1, payload='urgent'), Packet(priority=2, payload='slow')]
```

---

## 7. Iterators, Generators, and Context Managers

### Example 7.1: Implement the iterator protocol

**Task:** Iterate over `0, 2, 4` without materializing a list.

```python
class StepRange:
    def __init__(self, limit: int, step: int):
        self.limit = limit
        self.step = step
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.limit:
            raise StopIteration
        value = self.current
        self.current += self.step
        return value


print(list(StepRange(6, 2)))
```

**Output:**

```text
[0, 2, 4]
```

`iter(obj)` calls `__iter__`; repeated `next(obj)` calls `__next__` until
`StopIteration`.

### Example 7.2: Delegate with `yield from`

**Task:** Add framing around another generator.

```python
def numbers():
    yield 1
    yield 2


def framed_numbers():
    yield "begin"
    yield from numbers()
    yield "end"


print(list(framed_numbers()))
```

**Output:**

```text
['begin', 1, 2, 'end']
```

### Example 7.3: Guarantee cleanup with a context manager

**Task:** Acquire and release a named resource even if the body raises.

```python
from contextlib import contextmanager


@contextmanager
def resource(name: str):
    print(f"open {name}")
    try:
        yield name
    finally:
        print(f"close {name}")


with resource("cache") as name:
    print(f"use {name}")
```

**Output:**

```text
open cache
use cache
close cache
```

The `finally` block runs on normal exit and during exception unwinding.
Class-based managers use `__enter__` and `__exit__` for the same protocol.

---

## 8. Type Annotations and Structural Subtyping

### Example 8.1: Use aliases, unions, and generics

**Task:** Create a type-safe stack that can store one chosen item type.

```python
from typing import Generic, TypeVar

Scalar = int | float
T = TypeVar("T")


class Stack(Generic[T]):
    def __init__(self):
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()


def double(value: Scalar) -> Scalar:
    return value * 2


numbers = Stack[int]()
numbers.push(4)
print(numbers.pop(), double(2.5))
```

**Output:**

```text
4 5.0
```

Annotations guide readers and static type checkers; Python does not enforce
them automatically at runtime.

### Example 8.2: Accept behavior with a protocol

**Task:** Accept any object that provides `serialize() -> str`.

```python
from typing import Protocol


class Serializable(Protocol):
    def serialize(self) -> str:
        ...


class Invoice:
    def serialize(self) -> str:
        return "invoice:001"


def transmit(item: Serializable) -> str:
    return item.serialize()


print(transmit(Invoice()))
```

**Output:**

```text
invoice:001
```

`Invoice` does not inherit from `Serializable`; it matches structurally because
it supplies the required method.

---

## 9. Concurrency and Parallelism

### Choosing a model

```text
mostly waiting for I/O  -> asyncio or threads
blocking library calls  -> ThreadPoolExecutor
CPU-heavy Python work   -> ProcessPoolExecutor
```

### Example 9.1: Run independent I/O tasks with `asyncio`

**Task:** Fetch two records concurrently and collect both results.

```python
import asyncio


async def fetch(record_id: int) -> dict[str, int]:
    await asyncio.sleep(0.01)
    return {"id": record_id}


async def main():
    async with asyncio.TaskGroup() as group:
        first = group.create_task(fetch(1))
        second = group.create_task(fetch(2))
    return first.result(), second.result()


print(asyncio.run(main()))
```

**Output:**

```text
({'id': 1}, {'id': 2})
```

`await` gives the event loop an opportunity to run another task while this task
is waiting.

### Example 9.2: Run blocking I/O in a thread pool

**Task:** Execute several blocking operations without blocking the caller's
main flow.

```python
from concurrent.futures import ThreadPoolExecutor
import time


def blocking_job(number: int) -> str:
    time.sleep(0.01)
    return f"done:{number}"


with ThreadPoolExecutor(max_workers=2) as pool:
    results = list(pool.map(blocking_job, [1, 2, 3]))

print(results)
```

**Output:**

```text
['done:1', 'done:2', 'done:3']
```

### Example 9.3: Parallelize CPU work with processes

**Task:** Compute squares in worker processes.

```python
from concurrent.futures import ProcessPoolExecutor


def square(number: int) -> int:
    return number * number


if __name__ == "__main__":
    with ProcessPoolExecutor() as pool:
        print(list(pool.map(square, [2, 3, 4])))
```

**Output:**

```text
[4, 9, 16]
```

The `__main__` guard is required because some platforms start child
interpreters by importing the module again.

---

## 10. Metaprogramming and Descriptors

### Example 10.1: Validate attributes with a descriptor

**Task:** Allow only strings for `Person.name`.

```python
class StringField:
    def __set_name__(self, owner, name):
        self.public_name = name
        self.private_name = f"_{name}"

    def __get__(self, instance, owner=None):
        if instance is None:
            return self
        return getattr(instance, self.private_name, "")

    def __set__(self, instance, value):
        if not isinstance(value, str):
            raise TypeError(f"{self.public_name} must be a string")
        setattr(instance, self.private_name, value)


class Person:
    name = StringField()

    def __init__(self, name: str):
        self.name = name


person = Person("Ada")
print(person.name)
try:
    person.name = 42
except TypeError as error:
    print(error)
```

**Output:**

```text
Ada
name must be a string
```

```text
person.name = value
       |
       +--> descriptor.__set__(person, value)
```

### Example 10.2: Enforce a class interface with a metaclass

**Task:** Reject plugin classes that do not define `execute`.

```python
class PluginMeta(type):
    def __new__(metaclass, name, bases, namespace):
        if name != "Plugin" and "execute" not in namespace:
            raise TypeError(f"{name} must define execute")
        return super().__new__(metaclass, name, bases, namespace)


class Plugin(metaclass=PluginMeta):
    pass


class GoodPlugin(Plugin):
    def execute(self) -> str:
        return "ok"


print(GoodPlugin().execute())
```

**Output:**

```text
ok
```

The metaclass runs while the class object is being created, before instances
exist.

---

## 11. Memory Model and Performance

### Example 11.1: Observe references

**Task:** Show that two names can refer to the same list.

```python
import sys

items = []
alias = items
print(items is alias)
print(sys.getrefcount(items) >= 2)
```

**Output:**

```text
True
True
```

`getrefcount` itself temporarily adds a reference, so exact counts should not
be treated as stable application logic.

### Example 11.2: Compare shallow and deep copies

**Task:** Copy nested data so inner lists are isolated.

```python
import copy

original = [[1, 2], [3, 4]]
shallow = list(original)
deep = copy.deepcopy(original)

original[0][0] = 99
print(shallow[0][0])
print(deep[0][0])
```

**Output:**

```text
99
1
```

```text
original ----> outer list ----> inner list A
    |                              ^
shallow --------------------------+

deep ----> different outer list ----> different inner list A
```

### Example 11.3: Use `__slots__` deliberately

**Task:** Prevent arbitrary attributes on a small, fixed-shape object.

```python
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x: int, y: int):
        self.x = x
        self.y = y


point = Point(2, 3)
print(point.x + point.y)
try:
    point.z = 4
except AttributeError:
    print("z is not an allowed attribute")
```

**Output:**

```text
5
z is not an allowed attribute
```

`__slots__` can reduce per-instance overhead and removes the normal instance
`__dict__`; use it only when the fixed layout is an intentional design choice.

### Example 11.4: Understand the GIL when selecting concurrency

**Task:** Select an appropriate tool for each workload.

```python
workloads = {
    "HTTP requests": "asyncio or threads",
    "file reads through a blocking API": "ThreadPoolExecutor",
    "CPU-heavy pure Python": "ProcessPoolExecutor",
}

for workload, tool in workloads.items():
    print(f"{workload}: {tool}")
```

**Output:**

```text
HTTP requests: asyncio or threads
file reads through a blocking API: ThreadPoolExecutor
CPU-heavy pure Python: ProcessPoolExecutor
```

The GIL limits simultaneous execution of Python bytecode in regular threads.
Processes use separate interpreters, while I/O-oriented async code avoids
waiting idly on one event loop.

---

## Coverage Checklist

This companion workbook covers every section and source subtopic:

- primitives, conversions, strings, slicing, formatting, and transformations
- lists, tuples, dictionaries, sets, `deque`, `Counter`, and `defaultdict`
- walrus expressions, `match`, guards, comprehensions, and generators
- positional-only, keyword-only, variadic arguments, unpacking, and defaults
- LEGB, `global`, `nonlocal`, closures, decorator factories, and `wraps`
- abstract classes, properties, class/static methods, inheritance, dunder
  methods, slots, and dataclasses
- iterator protocol, `yield from`, class context managers, and `contextlib`
- aliases, unions, generics, `TypeVar`, and protocols
- `asyncio`, task groups, threads, processes, and the main guard
- descriptors, `__set_name__`, validation, metaclasses, and class creation
- references, reference counting, shallow/deep copying, slots, and the GIL
