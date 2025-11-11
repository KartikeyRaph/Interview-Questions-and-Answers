# Python Interview Q&A

## Overview
This section covers Python fundamentals and advanced concepts for interview preparation.

## Topics Covered
- Basic Syntax & Data Types
- Functions & Decorators
- Object-Oriented Programming (OOP)
- Functional Programming
- Error Handling
- File I/O & Modules
- Built-in Data Structures
- Advanced Python Concepts
- Best Practices

## Q&A

---

## Basic Syntax & Data Types

### Q1: What are the differences between mutable and immutable objects in Python? Give examples.

**Answer:**

**Immutable Objects:** Cannot be modified after creation. Any modification creates a new object.
- Examples: `int`, `float`, `str`, `tuple`, `frozenset`, `bool`, `bytes`

**Mutable Objects:** Can be modified in place.
- Examples: `list`, `dict`, `set`, `bytearray`

**Practical Implications:**

```python
# Immutable
x = 5
y = x
y = 10  # Creates new object, x remains 5
print(x, y)  # Output: 5 10

# Mutable
list1 = [1, 2, 3]
list2 = list1
list2.append(4)  # Modifies original list
print(list1)  # Output: [1, 2, 3, 4]

# Strings are immutable
s = "hello"
# s[0] = 'H'  # TypeError - cannot modify
s = "H" + s[1:]  # Creates new string
```

**Interview Tip:** Understanding mutability is crucial for:
- Function parameters and side effects
- Dictionary key restrictions (must be immutable)
- Memory efficiency considerations

---

### Q2: Explain the difference between `==` and `is` operators.

**Answer:**

- **`==`**: Checks **value equality** (compares content)
- **`is`**: Checks **identity equality** (compares memory address/object identity)

```python
# Value equality
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True (same values)
print(a is b)  # False (different objects)

# Identity equality
c = d = [1, 2, 3]
print(c is d)  # True (same object reference)

# Special case: Small integers are cached
x = 256
y = 256
print(x is y)  # True (cached by Python)

z = 257
w = 257
print(z is w)  # False (not cached)

# None comparison
value = None
print(value is None)  # Correct (use is/is not for None)
print(value == None)  # Works but not recommended
```

**Best Practice:** Use `is` for None, True, False comparisons. Use `==` for value comparisons.

---

### Q3: What is the difference between list, tuple, and set? When would you use each?

**Answer:**

| Feature | List | Tuple | Set |
|---------|------|-------|-----|
| **Mutable** | Yes | No | Yes |
| **Ordered** | Yes | Yes | No |
| **Indexed** | Yes | Yes | No |
| **Duplicates** | Allowed | Allowed | Not allowed |
| **Hashable** | No | Yes (if contents are) | No |
| **Use Case** | General collection | Fixed data, dict keys | Unique items, membership tests |

```python
# List - Flexible, mutable
my_list = [1, 2, 3, 2]
my_list.append(4)
print(my_list[0])  # 1

# Tuple - Immutable, faster
my_tuple = (1, 2, 3)
# my_tuple[0] = 5  # TypeError
my_dict = {my_tuple: "value"}  # Can use as key

# Set - Unique items, mathematical operations
my_set = {1, 2, 3, 2}
print(my_set)  # {1, 2, 3}
set1 = {1, 2, 3}
set2 = {2, 3, 4}
print(set1 & set2)  # {2, 3} intersection
print(set1 | set2)  # {1, 2, 3, 4} union
print(set1 - set2)  # {1} difference
```

**Performance Tip:** Sets have O(1) membership testing vs O(n) for lists.

---

## Functions & Decorators

### Q4: What are *args and **kwargs? Provide practical examples.

**Answer:**

- **`*args`**: Allows function to accept variable number of positional arguments (tuple)
- **`**kwargs`**: Allows function to accept variable number of keyword arguments (dictionary)

```python
# *args example
def sum_numbers(*args):
    print(f"Type: {type(args)}")  # <class 'tuple'>
    return sum(args)

print(sum_numbers(1, 2, 3, 4))  # 10

# **kwargs example
def print_config(**kwargs):
    print(f"Type: {type(kwargs)}")  # <class 'dict'>
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_config(host="localhost", port=8000, debug=True)
# host: localhost
# port: 8000
# debug: True

# Combined usage
def flexible_function(name, *args, **kwargs):
    print(f"Name: {name}")
    print(f"Args: {args}")
    print(f"Kwargs: {kwargs}")

flexible_function("John", 1, 2, 3, age=30, city="NYC")
# Name: John
# Args: (1, 2, 3)
# Kwargs: {'age': 30, 'city': 'NYC'}

# Unpacking with * and **
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
print(add(*numbers))  # 6

params = {'a': 1, 'b': 2, 'c': 3}
print(add(**params))  # 6
```

---

### Q5: What is a decorator? Write a decorator that measures function execution time.

**Answer:**

A **decorator** is a function that takes another function as input and extends/modifies its behavior without permanently changing it.

```python
import time
from functools import wraps

# Basic decorator pattern
def my_decorator(func):
    def wrapper(*args, **kwargs):
        # Code before function execution
        result = func(*args, **kwargs)
        # Code after function execution
        return result
    return wrapper

# Practical example: Timing decorator
def timer_decorator(func):
    @wraps(func)  # Preserves original function metadata
    def wrapper(*args, **kwargs):
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        print(f"{func.__name__} took {end_time - start_time:.4f} seconds")
        return result
    return wrapper

@timer_decorator
def slow_function():
    time.sleep(2)
    return "Done"

slow_function()
# slow_function took 2.0001 seconds

# Parametrized decorator
def retry_decorator(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed, retrying...")
            return None
        return wrapper
    return decorator

@retry_decorator(max_attempts=3)
def unstable_function():
    import random
    if random.random() < 0.7:
        raise ValueError("Random failure")
    return "Success"

# Chaining decorators
def decorator_a(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Before A")
        result = func(*args, **kwargs)
        print("After A")
        return result
    return wrapper

def decorator_b(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print("Before B")
        result = func(*args, **kwargs)
        print("After B")
        return result
    return wrapper

@decorator_a
@decorator_b
def my_function():
    print("Function execution")

my_function()
# Before A
# Before B
# Function execution
# After B
# After A
```

**Key Points:**
- Always use `@wraps` to preserve metadata
- Decorators are applied bottom-to-top
- Common use cases: logging, timing, caching, authentication, validation

---

## Object-Oriented Programming (OOP)

### Q6: Explain the concepts of inheritance, polymorphism, and encapsulation with examples.

**Answer:**

### **Inheritance**
Allows a class to inherit properties and methods from a parent class.

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return f"{self.name} makes a sound"

class Dog(Animal):
    def speak(self):  # Override parent method
        return f"{self.name} barks"

class Cat(Animal):
    def speak(self):
        return f"{self.name} meows"

dog = Dog("Buddy")
print(dog.speak())  # Buddy barks
print(isinstance(dog, Animal))  # True
```

### **Polymorphism**
Objects of different classes respond to the same method call differently.

```python
def animal_sound(animal):
    print(animal.speak())

dog = Dog("Rex")
cat = Cat("Whiskers")
animal_sound(dog)   # Rex barks
animal_sound(cat)   # Whiskers meows
```

### **Encapsulation**
Bundling data and methods while hiding internal details (data hiding).

```python
class BankAccount:
    def __init__(self, account_number, balance):
        self.__account_number = account_number  # Private attribute
        self.__balance = balance  # Private attribute
    
    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount
            return True
        return False
    
    def withdraw(self, amount):
        if 0 < amount <= self.__balance:
            self.__balance -= amount
            return True
        return False
    
    @property
    def balance(self):  # Getter
        return self.__balance
    
    @balance.setter
    def balance(self, amount):  # Setter with validation
        if amount >= 0:
            self.__balance = amount

account = BankAccount("12345", 1000)
print(account.balance)  # 1000
account.deposit(500)
print(account.balance)  # 1500
# account.__balance  # AttributeError - cannot access directly
```

---

### Q7: What is the Method Resolution Order (MRO) and how does multiple inheritance work?

**Answer:**

**MRO (Method Resolution Order)** determines the order in which Python looks for methods and attributes in a hierarchy of classes (especially important in multiple inheritance).

```python
class A:
    def method(self):
        print("Method from A")

class B(A):
    def method(self):
        print("Method from B")

class C(A):
    def method(self):
        print("Method from C")

class D(B, C):
    pass

d = D()
d.method()  # Method from B

# Check MRO
print(D.mro())
# [<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>]

print(D.__mro__)  # Same as above

# Using super() with MRO
class Animal:
    def speak(self):
        return "Animal sound"

class Mammal(Animal):
    def speak(self):
        return f"{super().speak()} - Mammal"

class Dog(Mammal):
    def speak(self):
        return f"{super().speak()} - Dog"

dog = Dog()
print(dog.speak())
# Animal sound - Mammal - Dog

# Practical multiple inheritance example (Mixins)
class TimestampMixin:
    def get_timestamp(self):
        from datetime import datetime
        return datetime.now()

class LogMixin:
    def log(self, message):
        print(f"[LOG] {message}")

class User(TimestampMixin, LogMixin):
    def __init__(self, name):
        self.name = name
        self.log(f"User {name} created at {self.get_timestamp()}")

user = User("John")
```

**Key Points:**
- Python uses C3 Linearization for MRO
- Use `super()` to call parent methods
- Mixins are a common pattern for multiple inheritance
- Diamond problem is handled by MRO

---

### Q8: What are class methods and static methods? When would you use each?

**Answer:**

```python
class MyClass:
    class_variable = "shared across instances"
    
    # Instance method - most common
    def instance_method(self):
        return f"Instance method: {self.class_variable}"
    
    # Class method - receives class as first argument
    @classmethod
    def class_method(cls):
        return f"Class method: {cls.class_variable}"
    
    # Static method - no implicit first argument
    @staticmethod
    def static_method():
        return "Static method - no access to instance or class"

# Usage
obj = MyClass()
print(obj.instance_method())  # Instance method: shared across instances
print(obj.class_method())      # Class method: shared across instances
print(obj.static_method())     # Static method - no access to instance or class

print(MyClass.class_method())  # Works without instance
print(MyClass.static_method()) # Works without instance

# Practical examples
class Database:
    _instances = {}
    
    @classmethod
    def get_instance(cls, name):
        # Singleton pattern using classmethod
        if name not in cls._instances:
            cls._instances[name] = cls(name)
        return cls._instances[name]
    
    def __init__(self, name):
        self.name = name

class FileUtil:
    @staticmethod
    def read_file(path):
        # Utility function with no state
        with open(path, 'r') as f:
            return f.read()
    
    @staticmethod
    def write_file(path, content):
        with open(path, 'w') as f:
            f.write(content)

# Usage
db1 = Database.get_instance("main")
db2 = Database.get_instance("main")
print(db1 is db2)  # True - same instance

content = FileUtil.read_file("file.txt")
```

**When to Use:**
- **Instance methods:** Most operations, access/modify instance state
- **Class methods:** Factory methods, alternative constructors, class-level data
- **Static methods:** Utility functions, no instance/class state needed

---

## Functional Programming

### Q9: What is a lambda function? How is it different from a regular function?

**Answer:**

A **lambda** is a small anonymous function created with the `lambda` keyword.

```python
# Regular function
def add(x, y):
    return x + y

# Lambda equivalent
add_lambda = lambda x, y: x + y

print(add(3, 4))          # 7
print(add_lambda(3, 4))   # 7

# Lambda with single argument
square = lambda x: x ** 2
print(square(5))  # 25

# Lambda with no arguments
get_constant = lambda: 42
print(get_constant())  # 42

# Practical uses with map, filter, sorted
numbers = [1, 2, 3, 4, 5]

# map - apply function to each element
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# filter - keep elements where condition is true
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4]

# sorted - custom sorting
people = [
    {'name': 'Alice', 'age': 30},
    {'name': 'Bob', 'age': 25},
    {'name': 'Charlie', 'age': 35}
]
sorted_by_age = sorted(people, key=lambda p: p['age'])
print(sorted_by_age)

# Nested lambda
multiply_by_factor = lambda factor: lambda x: x * factor
double = multiply_by_factor(2)
triple = multiply_by_factor(3)
print(double(5))  # 10
print(triple(5))  # 15
```

**Differences from Regular Functions:**

| Aspect | Lambda | Regular Function |
|--------|--------|------------------|
| **Syntax** | Single expression | Multiple statements |
| **Return** | Implicit | Explicit with `return` |
| **Name** | Anonymous | Named |
| **Usage** | Quick callbacks | Complex logic |
| **Readability** | Less readable for complex logic | More readable |

**Best Practice:** Use lambda for short, simple operations. Use regular functions for complex logic.

---

### Q10: Explain map, filter, and reduce functions with examples.

**Answer:**

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]

# MAP - transform each element
def double(x):
    return x * 2

doubled = list(map(double, numbers))
print(doubled)  # [2, 4, 6, 8, 10]

# Using lambda with map
squared = list(map(lambda x: x ** 2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# Mapping with multiple iterables
list1 = [1, 2, 3]
list2 = [4, 5, 6]
sums = list(map(lambda x, y: x + y, list1, list2))
print(sums)  # [5, 7, 9]

# FILTER - keep elements matching condition
def is_even(x):
    return x % 2 == 0

evens = list(filter(is_even, numbers))
print(evens)  # [2, 4]

# Using lambda with filter
greater_than_two = list(filter(lambda x: x > 2, numbers))
print(greater_than_two)  # [3, 4, 5]

# REDUCE - accumulate result
def sum_function(acc, x):
    return acc + x

total = reduce(sum_function, numbers)
print(total)  # 15

# Using lambda with reduce
product = reduce(lambda acc, x: acc * x, numbers)
print(product)  # 120

# Finding maximum using reduce
maximum = reduce(lambda acc, x: x if x > acc else acc, numbers)
print(maximum)  # 5

# Combining map, filter, and reduce
data = [1, 2, 3, 4, 5]
result = reduce(
    lambda acc, x: acc + x,
    filter(lambda x: x % 2 == 0,  # Keep even
           map(lambda x: x ** 2, data))  # Square first
)
print(result)  # (2^2) + (4^2) = 4 + 16 = 20
```

**Important Note:** Python style prefers list comprehensions over map/filter:

```python
# Recommended style
squared = [x ** 2 for x in numbers]
evens = [x for x in numbers if x % 2 == 0]

# These are more readable and Pythonic than:
squared = list(map(lambda x: x ** 2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))
```

---

## Error Handling

### Q11: Explain exception handling in Python with try, except, else, and finally blocks.

**Answer:**

```python
# Basic try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")

# Multiple except blocks
try:
    data = {'name': 'John'}
    age = data['age']  # KeyError
except KeyError as e:
    print(f"Key error: {e}")
except ValueError:
    print("Value error")
except (TypeError, IndexError):
    print("Type or Index error")
except Exception:
    print("Generic exception")

# Using else block (executes if no exception)
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Division error")
else:
    print(f"Result: {result}")  # This runs

# Using finally block (always executes)
try:
    file = open("data.txt")
    content = file.read()
except FileNotFoundError:
    print("File not found")
else:
    print(f"Read {len(content)} characters")
finally:
    file.close()  # Always executed

# Context manager (better for file handling)
try:
    with open("data.txt") as file:
        content = file.read()
except FileNotFoundError:
    print("File not found")
# File automatically closed

# Raising custom exceptions
class InsufficientFundsError(Exception):
    pass

class BankAccount:
    def __init__(self, balance):
        self.balance = balance
    
    def withdraw(self, amount):
        if amount > self.balance:
            raise InsufficientFundsError(f"Cannot withdraw {amount}, balance is {self.balance}")
        self.balance -= amount

try:
    account = BankAccount(100)
    account.withdraw(150)
except InsufficientFundsError as e:
    print(f"Error: {e}")

# Chaining exceptions
try:
    result = 10 / 0
except ZeroDivisionError as e:
    raise ValueError("Invalid operation") from e

# Getting traceback
import traceback
try:
    result = int("abc")
except ValueError:
    print(traceback.format_exc())
```

**Best Practices:**
- Catch specific exceptions, not generic `Exception`
- Use `else` for code that depends on try block success
- Use `finally` for cleanup code
- Use context managers (`with` statement) for resource management
- Create custom exceptions for domain-specific errors

---

### Q12: What are context managers? How do you create one using `with` statement and the `contextlib` module?

**Answer:**

A **context manager** automatically handles setup and cleanup of resources.

```python
# Using context manager with file
with open("file.txt") as f:
    content = f.read()
# File automatically closed

# Creating a context manager using class
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None
    
    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
        # Return True to suppress exception, False to propagate
        return False

# Usage
with FileManager("data.txt", "r") as f:
    content = f.read()

# Creating context manager using contextlib decorator
from contextlib import contextmanager

@contextmanager
def database_connection(db_url):
    print("Connecting to database")
    connection = f"Connection to {db_url}"
    try:
        yield connection
    finally:
        print("Closing connection")

with database_connection("localhost:5432") as conn:
    print(f"Using: {conn}")

# Resource manager with error handling
@contextmanager
def timer():
    import time
    start = time.time()
    try:
        yield
    finally:
        end = time.time()
        print(f"Elapsed: {end - start:.4f} seconds")

with timer():
    import time
    time.sleep(1)

# Multiple context managers
with open("input.txt") as infile, open("output.txt", "w") as outfile:
    for line in infile:
        outfile.write(line.upper())

# Nested context managers
from contextlib import ExitStack

with ExitStack() as stack:
    files = [stack.enter_context(open(f"file{i}.txt")) for i in range(3)]
    # All files available here
    # All automatically closed when exiting block
```

**Key Points:**
- `__enter__()`: Called when entering `with` block
- `__exit__()`: Called when exiting (cleanup)
- `@contextmanager`: Decorator to create context managers from functions
- `ExitStack`: Manage multiple context managers dynamically

---

## File I/O & Modules

### Q13: How do you work with files in Python? Explain different read/write modes.

**Answer:**

```python
# File modes
modes = {
    'r': 'Read only (default)',
    'w': 'Write (creates new, overwrites existing)',
    'a': 'Append (add to end)',
    'x': 'Create (fails if exists)',
    'b': 'Binary mode (can combine with above)',
    't': 'Text mode (default, can combine with above)',
    '+': 'Read and write (can combine with above)'
}

# Reading files
with open("file.txt", "r") as f:
    # Read entire file
    content = f.read()
    
    # Read line by line
    f.seek(0)  # Reset position
    line = f.readline()  # Read single line
    
    # Read all lines as list
    f.seek(0)
    lines = f.readlines()
    
    # Iterate through lines (memory efficient)
    f.seek(0)
    for line in f:
        print(line.strip())

# Writing files
with open("output.txt", "w") as f:
    f.write("Hello, World!\n")
    f.writelines(["Line 1\n", "Line 2\n"])

# Appending to files
with open("output.txt", "a") as f:
    f.write("Appended line\n")

# Working with binary files
with open("image.jpg", "rb") as f:
    binary_data = f.read()

with open("copy.jpg", "wb") as f:
    f.write(binary_data)

# File pointer position
with open("file.txt", "r") as f:
    print(f.tell())  # Current position
    f.read(5)  # Read 5 characters
    print(f.tell())  # Position advanced
    f.seek(0)  # Reset to beginning
    f.seek(10, 1)  # Move 10 bytes from current
    f.seek(0, 2)  # Go to end of file

# Checking if file exists
import os
if os.path.exists("file.txt"):
    print("File exists")

# Modern way using pathlib
from pathlib import Path
path = Path("file.txt")
if path.exists():
    content = path.read_text()
    path.write_text("New content")

# Working with CSV
import csv
with open("data.csv", "w", newline='') as f:
    writer = csv.writer(f)
    writer.writerow(["Name", "Age"])
    writer.writerows([["John", 30], ["Jane", 25]])

with open("data.csv", "r") as f:
    reader = csv.reader(f)
    for row in reader:
        print(row)

# Working with JSON
import json
data = {"name": "John", "age": 30}

# Write JSON
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# Read JSON
with open("data.json", "r") as f:
    loaded_data = json.load(f)
```

**Best Practices:**
- Always use context managers (`with` statement)
- Use `pathlib.Path` for modern file handling
- Close files automatically with context managers
- Use appropriate encoding (specify `encoding='utf-8'`)

---

### Q14: What is a module and a package in Python? How do you import them?

**Answer:**

**Module:** A file containing Python code (`.py` file)
**Package:** A directory with a `__init__.py` file containing modules

```python
# Simple import
import math
print(math.pi)

# From import
from math import sqrt, pi
print(sqrt(16))  # 4.0

# Aliasing
import numpy as np
from collections import defaultdict as dd

# Import all (generally not recommended)
from math import *

# Relative imports (inside packages)
# If package structure is:
# mypackage/
#   __init__.py
#   module1.py
#   module2.py
#   subpackage/
#     __init__.py
#     module3.py

# Inside module2.py
from . import module1  # Import sibling
from .subpackage import module3  # Import from subpackage
from .. import other_package  # Import from parent package

# Import with __all__ (controls what * imports)
# In mymodule.py
__all__ = ['function1', 'function2']

# sys.path to add custom module locations
import sys
sys.path.append('/path/to/modules')
import custom_module

# Lazy importing
def my_function():
    import expensive_module  # Only imported when function called
    return expensive_module.calculate()

# Dynamic import
module_name = "math"
module = __import__(module_name)
print(module.pi)

# Or use importlib
import importlib
module = importlib.import_module('math')

# Checking if module exists
try:
    import some_module
except ImportError:
    print("Module not found")

# Module metadata
print(math.__name__)  # 'math'
print(math.__file__)  # Path to module file
print(dir(math))  # List of attributes/functions
```

**Creating a Package:**

```
mypackage/
├── __init__.py
├── module1.py
├── module2.py
└── subpackage/
    ├── __init__.py
    └── module3.py
```

```python
# mypackage/__init__.py
from .module1 import function1
from .module2 import function2

# Now users can do:
from mypackage import function1, function2

# Or
import mypackage.module1
mypackage.module1.some_function()
```

---

## Built-in Data Structures

### Q15: Explain list comprehensions, dictionary comprehensions, and set comprehensions.

**Answer:**

**List Comprehensions:** Concise way to create lists

```python
# Basic syntax: [expression for item in iterable if condition]

numbers = [1, 2, 3, 4, 5]

# Simple list comprehension
squared = [x**2 for x in numbers]
print(squared)  # [1, 4, 9, 16, 25]

# With condition
evens = [x for x in numbers if x % 2 == 0]
print(evens)  # [2, 4]

# With transformation and condition
result = [x*2 for x in numbers if x > 2]
print(result)  # [6, 8, 10]

# Nested list comprehension
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [x for row in matrix for x in row]
print(flattened)  # [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Creating matrix
matrix = [[i*j for j in range(3)] for i in range(3)]
print(matrix)  # [[0, 0, 0], [0, 1, 2], [0, 2, 4]]

# Dictionary Comprehensions
numbers = [1, 2, 3, 4, 5]

# Simple dictionary
squared_dict = {x: x**2 for x in numbers}
print(squared_dict)  # {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# With condition
even_dict = {x: x**2 for x in numbers if x % 2 == 0}
print(even_dict)  # {2: 4, 4: 16}

# From two lists
keys = ['a', 'b', 'c']
values = [1, 2, 3]
dict_from_lists = {k: v for k, v in zip(keys, values)}
print(dict_from_lists)  # {'a': 1, 'b': 2, 'c': 3}

# Swapping keys and values
original = {'a': 1, 'b': 2, 'c': 3}
swapped = {v: k for k, v in original.items()}
print(swapped)  # {1: 'a', 2: 'b', 3: 'c'}

# Filtering dictionary
data = {'name': 'John', 'age': 30, 'email': 'john@email.com'}
filtered = {k: v for k, v in data.items() if k != 'email'}
print(filtered)  # {'name': 'John', 'age': 30}

# Set Comprehensions
numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]

# Simple set comprehension (removes duplicates)
unique = {x for x in numbers}
print(unique)  # {1, 2, 3, 4}

# With condition
evens = {x for x in numbers if x % 2 == 0}
print(evens)  # {2, 4}

# Set operations
set1 = {x for x in range(5)}
set2 = {x for x in range(3, 8)}
print(set1 & set2)  # {3, 4} intersection
print(set1 | set2)  # {0, 1, 2, 3, 4, 5, 6, 7} union
```

**Generator Expressions:** Memory efficient (similar syntax to list comprehension)

```python
# Use parentheses instead of brackets
squares_gen = (x**2 for x in range(1000000))

# Doesn't create list, generates values on demand
for square in squares_gen:
    if square > 100:
        break

# Converting to list when needed
squares_list = list(x**2 for x in range(10))
```

---

### Q16: How does Python's slice notation work? Explain [:], [start:stop:step].

**Answer:**

```python
# Basic slicing
numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

# [start:stop:step]
# start: beginning index (inclusive), default 0
# stop: ending index (exclusive), default len
# step: increment, default 1

print(numbers[2:5])      # [2, 3, 4]
print(numbers[:5])       # [0, 1, 2, 3, 4]
print(numbers[5:])       # [5, 6, 7, 8, 9]
print(numbers[::2])      # [0, 2, 4, 6, 8] (every 2nd)
print(numbers[1::2])     # [1, 3, 5, 7, 9] (start at 1, every 2nd)

# Negative indices
print(numbers[-1])       # 9 (last element)
print(numbers[-3:])      # [7, 8, 9] (last 3)
print(numbers[:-2])      # [0, 1, 2, 3, 4, 5, 6, 7] (all but last 2)
print(numbers[-5:-2])    # [5, 6, 7]

# Reversing with negative step
print(numbers[::-1])     # [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
print(numbers[9:0:-1])   # [9, 8, 7, 6, 5, 4, 3, 2, 1]

# With strings
text = "Hello, World!"
print(text[0:5])         # "Hello"
print(text[::2])         # "Hlo ol!"
print(text[::-1])        # "!dlroW ,olleH"

# With tuples
t = (10, 20, 30, 40, 50)
print(t[1:4])            # (20, 30, 40)
print(t[::-1])           # (50, 40, 30, 20, 10)

# Using slice() object
s = slice(2, 7, 2)
print(numbers[s])        # [2, 4, 6]

# Dynamic slicing
def get_slice_of_list(lst, start=None, stop=None, step=None):
    return lst[start:stop:step]

print(get_slice_of_list(numbers, 2, 8, 2))  # [2, 4, 6]

# Modifying slices
numbers_copy = list(numbers)
numbers_copy[2:5] = [20, 30, 40]
print(numbers_copy)  # [0, 1, 20, 30, 40, 5, 6, 7, 8, 9]

numbers_copy = list(numbers)
numbers_copy[::2] = [100]*5  # Replace every 2nd element
print(numbers_copy)  # [100, 1, 100, 3, 100, 5, 100, 7, 100, 9]

# Deleting slices
numbers_copy = list(numbers)
del numbers_copy[2:5]
print(numbers_copy)  # [0, 1, 5, 6, 7, 8, 9]
```

**Key Points:**
- Slicing creates new objects (doesn't modify original for lists)
- Negative indices count from end
- Step of -1 reverses sequences
- Stop index is exclusive
- Can use slice() object for reusable slices

---

## Advanced Python Concepts

### Q17: What is the Global Interpreter Lock (GIL)? How does it affect multi-threading?

**Answer:**

The **GIL (Global Interpreter Lock)** is a mutex that protects access to Python objects in CPython. Only one thread can execute Python bytecode at a time.

```python
import threading
import time

# GIL Impact Demonstration
def cpu_bound_task():
    total = 0
    for i in range(50000000):
        total += i
    return total

# Single thread
start = time.time()
result1 = cpu_bound_task()
result2 = cpu_bound_task()
print(f"Sequential: {time.time() - start:.2f} seconds")

# Two threads - not faster due to GIL
start = time.time()
thread1 = threading.Thread(target=cpu_bound_task)
thread2 = threading.Thread(target=cpu_bound_task)
thread1.start()
thread2.start()
thread1.join()
thread2.join()
print(f"Threading: {time.time() - start:.2f} seconds")  # Similar to sequential

# I/O bound tasks benefit from threading
def io_bound_task():
    time.sleep(1)  # Simulates I/O
    return "Done"

start = time.time()
result1 = io_bound_task()
result2 = io_bound_task()
print(f"Sequential I/O: {time.time() - start:.2f} seconds")  # 2 seconds

start = time.time()
thread1 = threading.Thread(target=io_bound_task)
thread2 = threading.Thread(target=io_bound_task)
thread1.start()
thread2.start()
thread1.join()
thread2.join()
print(f"Threading I/O: {time.time() - start:.2f} seconds")  # ~1 second

# Workarounds for GIL with CPU-bound tasks

# 1. Multiprocessing
from multiprocessing import Process

start = time.time()
p1 = Process(target=cpu_bound_task)
p2 = Process(target=cpu_bound_task)
p1.start()
p2.start()
p1.join()
p2.join()
print(f"Multiprocessing: {time.time() - start:.2f} seconds")  # Runs in parallel

# 2. Use libraries written in C (NumPy, Pandas)
import numpy as np
array = np.arange(50000000)
np.sum(array)  # Released GIL during computation

# 3. Asyncio for I/O-bound tasks
import asyncio

async def async_task():
    await asyncio.sleep(1)
    return "Done"

async def main():
    start = time.time()
    await asyncio.gather(async_task(), async_task())
    print(f"Asyncio: {time.time() - start:.2f} seconds")

# asyncio.run(main())  # ~1 second
```

**Key Points:**
- GIL prevents true parallel execution of Python bytecode
- Affects CPU-bound tasks in multi-threading
- I/O-bound tasks benefit from threading
- Use `multiprocessing` for CPU-bound tasks
- Use `asyncio` for I/O-bound tasks
- C extensions can release GIL

---

### Q18: Explain generators and how they differ from regular functions.

**Answer:**

A **generator** is a function that returns an iterator object using `yield`. It produces values lazily.

```python
# Regular function
def regular_function():
    result = []
    for i in range(5):
        result.append(i)
    return result

print(regular_function())  # [0, 1, 2, 3, 4]

# Generator function
def generator_function():
    for i in range(5):
        yield i

gen = generator_function()
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 2

# Iterate through generator
for value in generator_function():
    print(value)  # 0, 1, 2, 3, 4

# Generator expression (shorthand)
gen_expr = (x for x in range(5))
print(next(gen_expr))  # 0

# Practical examples

# Fibonacci generator
def fibonacci(n):
    a, b = 0, 1
    for _ in range(n):
        yield a
        a, b = b, a + b

print(list(fibonacci(7)))  # [0, 1, 1, 2, 3, 5, 8]

# Reading large files line by line
def read_large_file(filepath):
    with open(filepath) as f:
        for line in f:
            yield line.rstrip('\n')

# for line in read_large_file('huge_file.txt'):
#     process(line)  # Memory efficient

# Generator with send()
def echo_generator():
    value = None
    while True:
        value = (yield value)
        if value is not None:
            value = value.upper()

gen = echo_generator()
next(gen)  # Prime the generator
print(gen.send('hello'))  # HELLO
print(gen.send('world'))  # WORLD

# Generator with return value
def countdown(n):
    while n > 0:
        yield n
        n -= 1
    return "Finished!"

gen = countdown(3)
try:
    while True:
        print(next(gen))
except StopIteration as e:
    print(e.value)  # "Finished!"

# Using itertools
from itertools import count, cycle, repeat, islice

# Infinite count
counter = count(1)
print(next(counter))  # 1
print(next(counter))  # 2

# Get first n items
first_five = list(islice(count(1), 5))
print(first_five)  # [1, 2, 3, 4, 5]

# Memory comparison
def memory_comparison():
    import sys
    
    # List comprehension - stores all in memory
    list_result = [x for x in range(1000000)]
    print(f"List size: {sys.getsizeof(list_result)} bytes")
    
    # Generator expression - generates on demand
    gen_result = (x for x in range(1000000))
    print(f"Generator size: {sys.getsizeof(gen_result)} bytes")
```

**Differences:**

| Feature | Regular Function | Generator |
|---------|-----------------|-----------|
| **Return Type** | Value | Iterator |
| **Keyword** | `return` | `yield` |
| **Memory** | Stores all data | Generates on demand |
| **State** | Fresh on each call | Maintains state |
| **Speed** | Immediate | Lazy evaluation |

---

### Q19: What is the `@property` decorator? How does it work?

**Answer:**

The `@property` decorator allows you to define methods that are accessed like attributes.

```python
# Without property
class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    def get_radius(self):
        return self._radius
    
    def set_radius(self, value):
        if value <= 0:
            raise ValueError("Radius must be positive")
        self._radius = value

# Usage
circle = Circle(5)
radius = circle.get_radius()
circle.set_radius(10)

# With property decorator
class CircleWithProperty:
    def __init__(self, radius):
        self._radius = radius
    
    @property
    def radius(self):
        """Getter"""
        return self._radius
    
    @radius.setter
    def radius(self, value):
        """Setter with validation"""
        if value <= 0:
            raise ValueError("Radius must be positive")
        self._radius = value
    
    @radius.deleter
    def radius(self):
        """Deleter"""
        del self._radius
    
    @property
    def area(self):
        """Computed property"""
        import math
        return math.pi * self._radius ** 2

# Usage - looks like attribute access
circle = CircleWithProperty(5)
print(circle.radius)  # 5 (calls getter)
circle.radius = 10    # calls setter
print(circle.area)    # 314.159... (computed property)
del circle.radius     # calls deleter

# Complex example with lazy computation
class DataProcessor:
    def __init__(self, filename):
        self.filename = filename
        self._data = None
    
    @property
    def data(self):
        """Lazy loading - load only when accessed"""
        if self._data is None:
            self._data = self._load_data()
        return self._data
    
    def _load_data(self):
        # Expensive operation
        with open(self.filename) as f:
            return f.read()

# Data only loaded when .data is accessed
processor = DataProcessor("large_file.txt")
# File not loaded yet
content = processor.data  # Now loaded

# Using property() function (older style)
class OldStyle:
    def __init__(self, value):
        self._value = value
    
    def get_value(self):
        return self._value
    
    def set_value(self, v):
        if v < 0:
            raise ValueError("Negative not allowed")
        self._value = v
    
    value = property(get_value, set_value)

obj = OldStyle(10)
print(obj.value)
obj.value = 20
```

**Benefits:**
- Cleaner API (attribute-like access)
- Validation and computation
- Lazy evaluation
- Future-proof (can add logic without changing interface)

---

## Best Practices

### Q20: What are some Python best practices and PEP 8 style guidelines?

**Answer:**

```python
# PEP 8 Style Guide

# 1. Naming Conventions
MY_CONSTANT = 100  # Constants: ALL_CAPS
my_variable = 10  # Variables: lowercase_with_underscores
my_function = lambda x: x  # Functions: lowercase_with_underscores
MyClass = object  # Classes: PascalCase
_private_var = 5  # Private: single underscore
__dunder_var = 5  # Name mangling: double underscore

# 2. Indentation - 4 spaces (never tabs)
def function():
    if True:
        x = 1
        y = 2
    return x + y

# 3. Line length - 79 characters for code, 72 for comments
# Bad
very_long_variable_name = very_long_function_name(arg1, arg2, arg3, arg4, arg5)

# Good
very_long_variable_name = very_long_function_name(
    arg1, arg2, arg3, arg4, arg5
)

# 4. Imports
import os  # Standard library
import sys
from datetime import datetime

import numpy as np  # Third-party

from mymodule import MyClass  # Local

# 5. Whitespace
# Good
x = 1  # Space around operators
y = x + 2

dict_data = {"key": "value"}  # Space after colon in dicts
list_data = [1, 2, 3]  # Space after comma

def function(param1, param2):  # No space before params
    pass

# Bad
x=1  # No space around operators
dict_data = { "key" : "value" }  # Extra spaces

# 6. Comments and Docstrings
def calculate_total(items):
    """
    Calculate total sum of items.
    
    Args:
        items: List of numbers
    
    Returns:
        Sum of all items
    
    Raises:
        TypeError: If items is not iterable
    """
    # This is a comment explaining complex logic
    return sum(items)

# 7. String formatting
name = "John"
age = 30

# Bad
message = "My name is " + name + " and I'm " + str(age)

# Good
message = f"My name is {name} and I'm {age}"
message = "My name is {} and I'm {}".format(name, age)

# 8. Avoid overly complex statements
# Bad
result = [x*2 for x in range(100) if x % 2 == 0 and x > 50 and x < 90]

# Good
result = [
    x * 2 
    for x in range(100) 
    if x % 2 == 0 and 50 < x < 90
]

# 9. Type hints (Python 3.5+)
def add_numbers(a: int, b: int) -> int:
    return a + b

from typing import List, Dict, Optional

def process_data(data: List[int]) -> Dict[str, int]:
    return {"sum": sum(data)}

def find_user(user_id: int) -> Optional[str]:
    return "John" if user_id == 1 else None

# 10. Exception handling
# Bad
try:
    result = risky_operation()
except:  # Catches all exceptions
    pass

# Good
try:
    result = risky_operation()
except ValueError as e:  # Specific exception
    print(f"Invalid value: {e}")
except Exception as e:  # Only if necessary
    print(f"Unexpected error: {e}")

# 11. Boolean checks
# Bad
if len(items) > 0:
    pass

if items == []:
    pass

if is_active == True:
    pass

# Good
if items:
    pass

if not items:
    pass

if is_active:
    pass

# 12. Context managers for resources
# Bad
f = open("file.txt")
content = f.read()
f.close()

# Good
with open("file.txt") as f:
    content = f.read()

# 13. List comprehensions over loops
# Bad
squares = []
for x in range(10):
    squares.append(x ** 2)

# Good
squares = [x ** 2 for x in range(10)]

# 14. Use enumerate() for index and value
# Bad
for i in range(len(items)):
    print(i, items[i])

# Good
for i, item in enumerate(items):
    print(i, item)

# 15. Use zip() for multiple iterables
# Bad
for i in range(len(list1)):
    print(list1[i], list2[i])

# Good
for item1, item2 in zip(list1, list2):
    print(item1, item2)
```

**Key Principles:**
- Readability counts
- Explicit is better than implicit
- Simple is better than complex
- Use type hints
- Write docstrings
- Follow PEP 8
- Use linting tools (pylint, flake8)
- Use formatters (black, autopep8)

---

**Last Updated:** November 11, 2025
