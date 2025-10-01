---
type: page
title: The Value Class and it's Child Number Class
listed: true
slug: the-value-and-number-classes
description: 
index_title: The Value Class and it's Child Number Class
hidden: 
keywords: 
tags: 
---

## Values and Numbers

This section details the `Value` class, which serves as the base for all data types in the language, and the `Number` class, a concrete implementation for numerical values. The design choice to separate a generic `Value` class from specific types like `Number` is fundamental for extensibility and consistency in a custom programming language.

## Design Philosophy: Separation of `Value` and `Number`

The separation of `Value` and `Number` (and other potential types like `String`, `List`, `Boolean`, `Function`) is a classic example of **object-oriented design principles**, specifically **inheritance and polymorphism**.

**Why Separate?**

1. **Abstraction and Common Interface:**

- The `Value` class acts as an **abstract base class** (though not formally declared using `abc` module in Python, it serves that purpose). It defines a **common interface** that all concrete data types in the language are expected to adhere to.
- This interface includes methods like `set_pos`, `set_context`, `copy`, `is_true`, and all the arithmetic/comparison/logical operation stubs (`added_to`, `get_comparison_eq`, etc.).
- By defining these methods in `Value`, the interpreter can interact with _any_ value type in a consistent manner, without needing to know its specific underlying type. It simply calls `value.added_to(other_value)`, and the specific `Number` or `String` class handles the operation correctly.

2. **Type-Specific Behavior:**

- Each specific data type (like `Number`, `String`, `List`) will have unique rules for how operations apply to them. For example, adding two numbers results in a number, while adding two strings might concatenate them.
- The `Number` class _overrides_ the generic `illegal_operation` methods from `Value` with concrete implementations that perform numerical operations.
- If you were to combine them, the `Number` class would become bloated with conditional logic (`if type is Number: do this; elif type is String: do that;`). Separating them allows each type to manage its own behavior.

3. **Extensibility:**

- This design makes it easy to add new data types to the language in the future (e.g., `Boolean`, `List`, `Dictionary`, `Function`). You simply create a new class that inherits from `Value` and implements (overrides) the necessary operation methods.
- The core interpreter logic doesn't need to change when new types are introduced, as long as they adhere to the `Value` interface.

4. **Error Handling:**

- The `Value` class provides a default `illegal_operation` error. This means if a specific type doesn't implement an operation (e.g., trying to divide a string by a number), the base `Value` class's method will be called, resulting in a clear "Illegal operation" error, rather than a cryptic Python `AttributeError`.

**Why They Go Hand-in-Hand (and are Documented Together):**

While separated conceptually for design purposes, `Value` and its concrete subclasses like `Number` are intrinsically linked because `Value` defines _what_ a data type in the language _can do_, and `Number` defines _how_ a number _does_ those things.

- **Foundation and Implementation:** `Value` lays the foundation, and `Number` builds upon it, providing the actual operational logic for numerical data.
- **Unified Concept:** From the perspective of the language user, they interact with "numbers." The underlying class hierarchy is an implementation detail that allows the system to correctly handle these "numbers" in various contexts.
- **Documentation Flow:** Documenting them together makes sense because one defines the contract, and the other fulfills it for a specific type. It shows the complete picture of how numerical values are handled within the language's runtime.

In essence, `Value` is the abstract concept of a manipulable piece of data in your language, and `Number` is one specific, concrete instance of that concept.

---

## Class: `Value`

The `Value` class is the base class for all data types in the language. It provides fundamental attributes such as position (for error reporting) and context (for scope), along with placeholder methods for common operations.

### `__init__(self)`

**Purpose:** Initializes a `Value` instance.

**Parameters:** None.

**Explanation:**

{% code %}
{% tab language="python" %}
self.set_pos()
self.set_context()
{% /tab %}
{% /code %}

The constructor calls `set_pos()` and `set_context()` without arguments, effectively initializing `pos_start`, `pos_end`, and `context` to `None`. This ensures that every `Value` object has these fundamental attributes from its creation.

### `set_pos(self, pos_start=None, pos_end=None)`

**Purpose:** Sets the start and end position of the value in the source code. This is crucial for accurate error reporting.

**Parameters:**

- `pos_start`: An object representing the starting position (e.g., line, column). Defaults to `None`.
- `pos_end`: An object representing the ending position. Defaults to `None`.

**Explanation:**

{% code %}
{% tab language="python" %}
self.pos_start = pos_start
self.pos_end = pos_end
return self
{% /tab %}
{% /code %}

Assigns the provided position objects to the instance attributes. It returns `self` to allow for method chaining (e.g., `Number(5).set_pos(...).set_context(...)`).

### `set_context(self, context=None)`

**Purpose:** Sets the execution context in which the value exists. This helps in understanding the scope and environment when an error occurs involving this value.

**Parameters:**

- `context`: The `Context` object representing the current scope. Defaults to `None`.

**Explanation:**

{% code %}
{% tab language="python" %}
self.context = context
return self
{% /tab %}
{% /code %}

Assigns the provided `context` object to the instance attribute. It returns `self` for method chaining.

### Arithmetic Operations

The following methods are **placeholder methods** in the `Value` class. They are designed to be overridden by subclasses (like `Number`, `String`, etc.) that actually support these operations. If a type does not override one of these and an operation is attempted, the `illegal_operation` method will be called.

- `added_to(self, other)`: Handles addition (`+`).
- `subbed_by(self, other)`: Handles subtraction (`-`).
- `multed_by(self, other)`: Handles multiplication (`*`).
- `dived_by(self, other)`: Handles division (`/`).
- `powed_by(self, other)`: Handles exponentiation (`**`).

**Signature for Arithmetic Operations:**

{% code %}
{% tab language="python" %}
def operation_name(self, other):
    return None, self.illegal_operation(other)
{% /tab %}
{% /code %}

**Return Value:** All these methods are expected to return a tuple: `(result_value, error)`. In the base `Value` class, they always return `None` for the `result_value` and an `RTError` generated by `illegal_operation`.

### Comparison Operations

Similar to arithmetic operations, these are placeholder methods for comparisons, to be overridden by concrete value types.

- `get_comparison_eq(self, other)`: Handles equality comparison (`==`).
- `get_comparison_ne(self, other)`: Handles inequality comparison (`!=`).
- `get_comparison_lt(self, other)`: Handles less than comparison (`<`).
- `get_comparison_gt(self, other)`: Handles greater than comparison (`>`).
- `get_comparison_lte(self, other)`: Handles less than or equal to comparison (`<=`).
- `get_comparison_gte(self, other)`: Handles greater than or equal to comparison (`>=`).

**Signature for Comparison Operations:**

{% code %}
{% tab language="python" %}
def get_comparison_name(self, other):
    return None, self.illegal_operation(other)
{% /tab %}
{% /code %}

**Return Value:** All these methods are expected to return a tuple: `(result_value, error)`. In the base `Value` class, they always return `None` for the `result_value` and an `RTError` generated by `illegal_operation`.

### Logical Operations

Placeholder methods for logical operations.

- `anded_by(self, other)`: Handles logical AND.
- `ored_by(self, other)`: Handles logical OR.
- `notted(self, other)`: Handles logical NOT. (Note: The `other` parameter for `notted` is unusual as NOT is typically a unary operation. This might be a mistake in the provided code or a design choice for a specific AST structure.)

**Signature for Logical Operations:**

{% code %}
{% tab language="python" %}
def logical_operation_name(self, other):
    return None, self.illegal_operation(other)
{% /tab %}
{% /code %}

**Return Value:** All these methods are expected to return a tuple: `(result_value, error)`. In the base `Value` class, they always return `None` for the `result_value` and an `RTError` generated by `illegal_operation`.

### `execute(self, args)`

**Purpose:** This method is intended to be overridden by callable `Value` types (e.g., functions). In the base `Value` class, it signifies that the value cannot be executed.

**Parameters:**

- `args`: A list of arguments passed during an attempted execution.

**Explanation:**

{% code %}
{% tab language="python" %}
return RTResult().failure(self.illegal_operation())
{% /tab %}
{% /code %}

Returns an `RTResult` indicating a failure due to an `Illegal operation`, as a generic `Value` is not executable.

### `copy(self)`

**Purpose:** This method is intended to be overridden by subclasses to provide a deep or shallow copy of the value object.

**Parameters:** None.

**Explanation:**

{% code %}
{% tab language="python" %}
raise Exception('No copy method defined')
{% /tab %}
{% /code %}

Raises an `Exception` because the base `Value` class cannot provide a generic copy mechanism; each subclass must define how to copy itself.

### `is_true(self)`

**Purpose:** Determines the "truthiness" of a value. This is used in conditional statements (e.g., `if` statements, `while` loops).

**Parameters:** None.

**Explanation:**

{% code %}
{% tab language="python" %}
return False
{% /tab %}
{% /code %}

By default, a generic `Value` is considered `False`. Subclasses will override this (e.g., `Number(0)` is false, `Number(non-zero)` is true; empty list is false, non-empty list is true).

### `illegal_operation(self, other=None)`

**Purpose:** Generates an `RTError` for an illegal operation between `self` and `other` values.

**Parameters:**

- `other`: The other `Value` object involved in the illegal operation. Defaults to `self` if not provided (for unary operations).

**Explanation:**

{% code %}
{% tab language="python" %}
if not other: other = self
return RTError(
    self.pos_start, other.pos_end,
    'Illegal operation',
    self.context
)
{% /tab %}
{% /code %}

Creates and returns an `RTError` indicating an "Illegal operation". It uses the `pos_start` of `self` and `pos_end` of `other` (or `self` if `other` is not provided) to pinpoint the location of the error, along with the current `context`.

---

## Class: `Number`

The `Number` class inherits from `Value` and represents numerical data within the language. It overrides the generic `Value` methods to provide specific implementations for arithmetic, comparison, and logical operations on numbers.

### `__init__(self, value)`

**Purpose:** Initializes a `Number` instance.

**Parameters:**

- `value`: The Python numerical value (int or float) that this `Number` object will encapsulate.

**Explanation:**

{% code %}
{% tab language="python" %}
super().__init__()
self.value = value
{% /tab %}
{% /code %}

Calls the constructor of the `Value` class to initialize position and context. Then, it stores the actual Python `value` in `self.value`.

### Arithmetic Operations

These methods implement the actual arithmetic logic for `Number` objects. They perform type checking to ensure the `other` operand is also a `Number`.

- `added_to(self, other)`:
  if isinstance(other, Number):
  return Number(self.value + other.value).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)

If `other` is a `Number`, performs addition and returns a new `Number` object with the result. Otherwise, returns an `illegal_operation` error.

- `subbed_by(self, other)`:
  if isinstance(other, Number):
  return Number(self.value - other.value).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)

If `other` is a `Number`, performs subtraction and returns a new `Number` object. Otherwise, returns an `illegal_operation` error.

- `multed_by(self, other)`:
  if isinstance(other, Number):
  return Number(self.value * other.value).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)

If `other` is a `Number`, performs multiplication and returns a new `Number` object. Otherwise, returns an `illegal_operation` error.

- `dived_by(self, other)`:
  if isinstance(other, Number):
  if other.value == 0:
  return None, RTError(
  other.pos_start, other.pos_end,
  'Division by zero',
  self.context
  )
  return Number(self.value / other.value).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)

If `other` is a `Number`, performs division. Includes specific error handling for **division by zero**. Otherwise, returns an `illegal_operation` error.

- `powed_by(self, other)`:
  if isinstance(other, Number):
  return Number(self.value ** other.value).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)

If `other` is a `Number`, performs exponentiation (`**`) and returns a new `Number` object. Otherwise, returns an `illegal_operation` error.

### Comparison Operations

These methods implement the actual comparison logic for `Number` objects, returning `Number(1)` for true and `Number(0)` for false.

- `get_comparison_eq(self, other)`: Handles `==`.
  if isinstance(other, Number):
  return Number(int(self.value == other.value)).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)
- `get_comparison_ne(self, other)`: Handles `!=`.
  if isinstance(other, Number):
  return Number(int(self.value != other.value)).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)
- `get_comparison_lt(self, other)`: Handles `<`.
  if isinstance(other, Number):
  return Number(int(self.value &lt; other.value)).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)
- `get_comparison_gt(self, other)`: Handles `>`.
  if isinstance(other, Number):
  return Number(int(self.value &gt; other.value)).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)
- `get_comparison_lte(self, other)`: Handles `<=`.
  if isinstance(other, Number):
  return Number(int(self.value &lt;= other.value)).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)
- `get_comparison_gte(self, other)`: Handles `>=`.
  if isinstance(other, Number):
  return Number(int(self.value &gt;= other.value)).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)

### Logical Operations

These methods implement logical AND, OR, and NOT for `Number` objects, treating `0` as false and non-zero as true.

- `anded_by(self, other)`: Handles logical AND.
  if isinstance(other, Number):
  return Number(int(self.value and other.value)).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)
- `ored_by(self, other)`: Handles logical OR.
  if isinstance(other, Number):
  return Number(int(self.value or other.value)).set_context(self.context), None
  else:
  return None, Value.illegal_operation(self, other)
- `notted(self)`: Handles logical NOT.
  return Number(1 if self.value == 0 else 0).set_context(self.context), None

This method correctly implements logical NOT: if the number's value is `0` (false), it returns `Number(1)` (true); otherwise, it returns `Number(0)` (false). Note that unlike the `Value` class stub, this method for `Number` does not take an `other` argument, which is correct for a unary operation.

### `copy(self)`

**Purpose:** Creates a copy of the `Number` instance.

**Parameters:** None.

**Explanation:**

{% code %}
{% tab language="python" %}
copy = Number(self.value)
copy.set_pos(self.pos_start, self.pos_end)
copy.set_context(self.context)
return copy
{% /tab %}
{% /code %}

Creates a new `Number` object with the same underlying `value`, and then copies its position and context from the original instance.

### `is_true(self)`

**Purpose:** Determines the "truthiness" of a `Number`.

**Parameters:** None.

**Explanation:**

{% code %}
{% tab language="python" %}
return self.value != 0
{% /tab %}
{% /code %}

A `Number` is considered "true" if its `value` is not equal to `0`.

### `__repr__(self)`

**Purpose:** Provides a string representation of the `Number` object, useful for debugging and display.

**Parameters:** None.

**Explanation:**

{% code %}
{% tab language="python" %}
return str(self.value)
{% /tab %}
{% /code %}

Returns the string representation of the underlying Python numerical value.

### Built-in Number Constants

The `Number` class defines several static attributes that represent common numerical constants, including `null`, boolean `false` and `true`, and mathematical PI.

- `Number.null = Number(-1.010203040506071)`: A special constant to represent the absence of a value, or `null`. The specific floating-point value is likely a chosen "magic number" that is unlikely to be generated by normal calculations, allowing it to be uniquely identified and potentially replaced with a more user-friendly string "null" in output.
- `Number.false = Number(0)`: Represents the boolean false value as a `Number(0)`.
- `Number.true = Number(1)`: Represents the boolean true value as a `Number(1)`.
- `Number.math_PI = PI`: Assumes `PI` is a predefined constant (likely `math.pi` imported from Python's `math` module). This provides access to the mathematical constant $\pi$.