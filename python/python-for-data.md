# Python for Data Study Guide

## 1. Core Python Syntax & Control Flow (loops, conditionals, functions)

### 1.1 Variables & Assignment

- Python is **dynamically typed**, meaning does not require you to declare variable types ahead of time. The type of a variable is determined at runtime, based on the value assigned to it. You can even reassign a variable to a different type later.
- Multiple assignment & unpacking are common. **Unpacking** lets you assign parts of an iterable to multiple variables. With a `*`, you can capture "the rest" of the items into a list.

```python
x = 42
name = "Alena"
temp, humidity = 23.5, 0.67               # tuple unpacking
a, b, *rest = [1, 2, 3, 4, 5]             # starred unpacking
first, *middle, last = [10, 20, 30, 40]   # starred unpacking
```

### 1.2 Comments & Docstrings

```python
# Single-line comment

def add(a, b):
    """Return the sum of a and b."""
    return a + b
```

### 1.3 Indentation & Blocks

- Indentation defines scope (PEP 8: 4 spaces).
- No curly braces.

```python
if True:
    msg = "Indented block"
```

### 1.4 Truthiness & Comparisons

- **Falsy** means a value that evaluates to `False` when used in a boolean context (like an `if` statement). Everything else is considered truthy.
- Falsy: `0, 0.0, "", [], {}, set(), None, False`

### 1.5 Conditionals (if / elif / else) + Ternary

- A **ternary expression** is a one-line conditional expression that chooses between two values based on a condition. The word “ternary” means it involves three parts, and here they are condition, value if true, and value if false.

```python
x = 10
if x > 10:
    status = "high"
elif x == 10:
    status = "equal"
else:
    status = "low"

# Ternary expression
label = "large" if x >= 100 else "small"
```

### 1.6 Loops (for / while) + break / continue / else

- In a for-else (or while-else) loop, the `else` block runs only if the loop ends normally (i.e., it did not hit a `break`). The `else` is often used when you’re searching for something, and this pattern avoids need a "not found" flag.

```python
# for loop over iterables
for i in range(3):
    print(i)   # 0, 1, 2

# while loop
n = 3
while n > 0:
    n -= 1

# break: exit loop early
for x in [1, 2, 3, 99, 4]:
    if x == 99:
        break
    print(x)   # prints 1, 2, 3

# continue: skip to next iteration
for num in range(6):
    if num % 2 == 0:
        continue   # skip evens
    print(num)     # prints 1, 3, 5

# loop-else runs only if loop didn't break
for x in [1, 2, 3]:
    if x == 99:
        break
else:
    print("No break encountered")
```

- **enumerate** returns an iterator that yields pairs of (index, value) when looping over an iterable. You can set the start value for the index with `start`.
- **zip** returns an iterator that pairs up elements from two or more iterables into tuples of corresponding items.

Useful loop helpers
```python
# enumerate
names = ["Ana", "Bo", "Cy"]
for i, name in enumerate(names, start=1):
    print(i, name)

# zip
a = [1, 2, 3]; b = [10, 20, 30]
for x, y in zip(a, b):
    print(x, y)   # (1,10), (2,20), (3,30)
```

### 1.7 Functions (defs, args, scope, pitfalls)

- A **variable positional argument** (`*args`) collects any extra *unnamed* arguments passed to a function into a tuple.
- A **variable keyword argument** (`**kwargs`) collects any extra *named* arguments passed to a function into a dictionary.

```python
def greet(name: str) -> str:          # type hints optional but interview-friendly
    return f"Hello, {name}!"

def power(base, exp=2):               # default arg
    return base ** exp

def add_all(*args):                   # variable positional args
    return sum(args)

def print_kv(**kwargs):               # variable keyword args
    for k, v in kwargs.items():
        print(k, v)

def min_max(nums):                    # multiple returns (tuples)
    return min(nums), max(nums)

lo, hi = min_max([3, 1, 5])
```

Full argument ordering example
```python
def demo(a, b, *args, c=10, d=20, **kwargs):
    print("a:", a)           # first required positional arg
    print("b:", b)           # second required positional arg
    print("args:", args)     # extra unnamed args (tuple)
    print("c:", c)           # keyword-only arg with default
    print("d:", d)           # another keyword-only arg with default
    print("kwargs:", kwargs) # extra named args (dict)

# Call example
demo(1, 2, 3, 4, 5, c=99, d=100, x=7, y=8)
```

Variables assigned from the example are
```
a: 1
b: 2
args: (3, 4, 5)
c: 99
d: 100
kwargs: {'x': 7, 'y': 8}
```

Common pitfall: mutable defaults
```python
# BAD: list persists across calls
def append_bad(x, bucket=[]):
    bucket.append(x)
    return bucket

# GOOD: use None sentinel
def append_good(x, bucket=None):
    if bucket is None:
        bucket = []
    bucket.append(x)
    return bucket
```

Anonymous / lambda functions

- A **lambda function** is an anonymous, inline function, often used for short, throwaway operations.

```python
double = lambda x: x * 2
sorted_words = sorted(["bb", "a", "ccc"], key=lambda s: len(s))
squared = series.apply(lambda x: x**2)
```

### 1.8 Additional Concepts

- You can use exceptions as control flow. Instead of checking `if count == 0`, we let it fail and handle it.

```python
try:
    rate = total / count
except ZeroDivisionError:
    rate = 0.0
```

## 2. Data Types

### 2.1 Data Type Definitions

| Type | Example | Definition |
| -- | -- | -- |
| `int` | `42` | Whole numbers (positive, negative, or zero) without decimals |
| `float` | `3.14` | Numbers with decimal points, or scientific notation |
| `bool` | `True`, `False` | A logical type with only two values, True or False |
| `str` | `"SKU_123"` | An ordered, immutable sequence of characters used for text |
| `list` | `[1, 2, 3]` | An ordered, mutable collection of items |
| `tuple` | `(lat, lon)` | An ordered, immutable collection of items |
| `dict` | `{"sku": "X1", "qty": 2}` | A collection of key–value pairs |
| `set` | `{1, 2, 3}` | An unordered collection of unique items |
| `datetime` | `datetime(2025, 8, 19, 9, 0)` | Represents a specific point in time, including date and optionally hours, minutes, seconds, and microseconds |
| `timedelta` | `timedelta(days=7)` | Represents a duration or difference between two dates or times |
| numpy `ndarray` | `np.array([1,2,3])` | A multi-dimensional, homogeneous array for efficient numerical computation and vectorized operations |
| pandas `Series` | `pd.Series()` | A one-dimensional labeled array that can hold any data type and includes an index |
| pandas `DataFrame` | `pd.DataFrame([["bananas", 3, 0.5],["apples", 4, 0.75],["oranges", 2, 0.4]])` | A two-dimensional labeled table (rows and columns) for structured data, similar to a spreadsheet or SQL table |

Important data type concepts:
- Collections are heterogeneous by default, meaning you can mix types freely. This is true for lists, tuples, and sets. A numpy array, however, is homogenous.
  - A pandas Series is technically homogenous at dtype level, but a Series with `dtype=object` can store mixed objects.
- We prefer lists when the collection needs frequent updates, otherwise tuples are slightly more performant. Tuples can signal the programmer intends the data should not change.

### 2.2 Comprehensions

### 2.3 String Handling

### 2.4 Datetime Handling

## 3.0 Importing and Manipulating Files

### 3.1 Context Manager / Read Write Files

- A **context manager** is an object that defines setup and teardown behavior around a block of code, ensuring resources are properly managed (e.g., files closed automatically). The below is considered best practice because it automatically closes the file, even if errors occur. This is cleaner and safer than manually calling `f.close()`.
- In `open(filename, mode, encoding)`, the most commonly used modes are:
  - `"r"` -> read
  - `"w"` -> write
  - `"a"` -> append

Write to file example
```python
with open("data.csv", "w") as f:
    f.write("Your text")
```

- `f.read()` reads the entire file into a single string.
- `f.readline()` reads just one line at a time (useful for looping).

Read file examples
```python
# read whole file
with open("data.txt", "r") as f:
    data = f.read()

# read line by line
data_list = []
with open("data.txt", "r") as f:
    header = f.readline()  # save only first line

    # loop through remaining lines
    for line in f:
        data_list.append(line)
```

### 3.2 CSVs

### 3.3 JSON

## 4.0 Numpy

arrays, vectorized operations

## 5.0 Pandas

merge, groupby, filtering, pivoting

## 6.0 Basic Plotting

matplotlib

## 7.0 Itertools & Collections

## 8.0 Predictions & Evaluations

## 9.0 Annex

```python

```

#### `is` vs `==`

- Use `is` / `None` for object identity, checking if they are the exact same object in memory. Use `==` for value equality, checking if the contents are the same.

```python
if not items:          # empty list is falsy
    print("No items")

if value is None:      # correct None check
    ...
```
