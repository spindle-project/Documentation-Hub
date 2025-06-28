---
type: page
title: Other Values: Strings and Lists
listed: true
slug: other-values--strings-and-lists
description: 
index_title: Other Values: Strings and Lists
hidden: 
keywords: 
tags: 
---

# Strings and Lists

This section documents the `String` and `List` classes, which are concrete implementations of the `Value` base class for handling text and ordered collections of values, respectively. Like `Number`, these classes override the generic `Value` methods to define their specific behaviors for various operations.

## Class: `String`

The `String` class encapsulates textual data. It inherits from `Value` and implements string-specific operations.

### `__init__(self, value)`

**Purpose:** Initializes a `String` instance.

**Parameters:**
* `value`: The Python string value that this `String` object will encapsulate.

**Explanation:**
```python
super().__init__()
self.value = value
````

Calls the `Value` class constructor to set up position and context, then stores the provided Python `value` (the actual string content) in `self.value`.

### `added_to(self, other)`

**Purpose:** Implements string concatenation using the `+` operator.

**Parameters:**

  * `other`: The other `Value` object to be added (concatenated).

**Explanation:**

```python
if isinstance(other, String):
    return String(self.value + other.value).set_context(self.context), None
else:
    return None, Value.illegal_operation(self, other)
```

1.  **Type Check:** Verifies that `other` is also a `String` instance.
2.  **Concatenation:** If it is, it performs Python's string concatenation (`self.value + other.value`), wraps the result in a new `String` object, sets its context, and returns it with no error.
3.  **Error Handling:** If `other` is not a `String`, it returns an `illegal_operation` error.

### `multed_by(self, other)`

**Purpose:** Implements string repetition using the `*` operator, similar to Python's string multiplication.

**Parameters:**

  * `other`: The `Value` object representing the multiplier.

**Explanation:**

```python
if isinstance(other, Number):
    return String(self.value * other.value).set_context(self.context), None
else:
    return None, Value.illegal_operation(self, other)
```

1.  **Type Check:** Verifies that `other` is a `Number` instance.
2.  **Repetition:** If it is, it performs Python's string multiplication (`self.value * other.value`)—which repeats the string `other.value` times. The result is wrapped in a new `String` object, its context is set, and it's returned with no error.
3.  **Error Handling:** If `other` is not a `Number`, it returns an `illegal_operation` error.

### `is_true(self)`

**Purpose:** Determines the "truthiness" of a `String` object.

**Parameters:** None.

**Explanation:**

```python
return len(self.value) > 0
```

A `String` is considered "true" if its underlying Python string `self.value` has a length greater than zero (i.e., it's not an empty string). This aligns with Python's truthiness rules for strings.

### `copy(self)`

**Purpose:** Creates a copy of the `String` instance.

**Parameters:** None.

**Explanation:**

```python
copy = String(self.value)
copy.set_pos(self.pos_start, self.pos_end)
copy.set_context(self.context)
return copy
```

Creates a new `String` object with the same underlying `value`, and then copies its position and context from the original instance.

### `__str__(self)`

**Purpose:** Provides the informal string representation of the `String` object. Used by `print()` and `str()` functions.

**Parameters:** None.

**Explanation:**

```python
return self.value
```

Directly returns the encapsulated Python string `self.value`. This means that when a `String` object is printed, only its raw content is displayed, without quotation marks.

### `__repr__(self)`

**Purpose:** Provides the formal string representation of the `String` object. Used by `repr()` and for debugging.

**Parameters:** None.

**Explanation:**

```python
return f'"{self.value}"'
```

Returns a string representation that includes double quotes around the `self.value`. This helps distinguish string values from other types when inspecting them (e.g., in a debugger or interactive shell).

-----

## Class: `List`

The `List` class encapsulates ordered collections of `Value` objects, analogous to arrays or lists in other programming languages. It inherits from `Value` and provides list-specific operations.

### `__init__(self, elements)`

**Purpose:** Initializes a `List` instance.

**Parameters:**

  * `elements`: A Python list containing `Value` objects that will form the elements of this `List`.

**Explanation:**

```python
super().__init__()
self.elements = elements
```

Calls the `Value` class constructor to set up position and context, then stores the provided `elements` list in `self.elements`. This `self.elements` is the internal Python list that holds the actual `Value` objects.

### `added_to(self, other)`

**Purpose:** Implements appending an element to the list using the `+` operator.

**Parameters:**

  * `other`: The `Value` object to be appended to the list.

**Explanation:**

```python
new_list = self.copy()
new_list.elements.append(other)
return new_list, None
```

1.  **Create Copy:** Creates a `copy` of the current `List` instance. This is crucial to ensure that the original list is not modified (lists are mutable, so operations should ideally return a new list if immutability is desired, or explicitly modify in-place if that's the language's design).
2.  **Append:** Appends the `other` `Value` object to the `elements` list of the `new_list`.
3.  Returns the `new_list` with no error.

### `subbed_by(self, other)`

**Purpose:** Implements removal of an element from the list at a specific index using the `-` operator.

**Parameters:**

  * `other`: A `Number` object representing the index of the element to remove.

**Explanation:**

```python
if isinstance(other, Number):
    new_list = self.copy()
    try:
        new_list.elements.pop(other.value-1) # Adjust for 1-based indexing if desired
        return new_list, None
    except: # Broad exception handling for index out of bounds
        return None, RTError(
            other.pos_start, other.pos_end,
            'Element at this index could not be removed from list because index is out of bounds',
            self.context
        )
else:
    return None, Value.illegal_operation(self, other)
```

1.  **Type Check:** Verifies that `other` is a `Number` (representing the index).
2.  **Create Copy:** Creates a `copy` of the list to avoid modifying the original directly.
3.  **Pop Element:** Uses a `try-except` block to safely attempt to remove an element.
      * `new_list.elements.pop(other.value - 1)`: Removes the element at the index specified by `other.value`. The `-1` suggests that the language uses 1-based indexing for lists (e.g., the first element is at index 1, not 0), converting it to Python's 0-based indexing for the `pop` operation.
4.  **Error Handling:** If the index is out of bounds or any other error occurs during `pop`, it returns an `RTError`.
5.  Returns the `new_list` (without the popped element) with no error.

### `multed_by(self, other)`

**Purpose:** Implements list extension (combining two lists) using the `*` operator.

**Parameters:**

  * `other`: A `List` object whose elements will be added to the current list.

**Explanation:**

```python
if isinstance(other, List):
    new_list = self.copy()
    new_list.elements.extend(other.elements)
    return new_list, None
else:
    return None, Value.illegal_operation(self, other)
```

1.  **Type Check:** Verifies that `other` is also a `List` instance.
2.  **Create Copy:** Creates a `copy` of the current `List`.
3.  **Extend:** Uses Python's `list.extend()` method to append all elements from `other.elements` to `new_list.elements`.
4.  Returns the `new_list` (which now contains elements from both lists) with no error.

### `dived_by(self, other)`

**Purpose:** Implements element access (getting an element at a specific index) using the `/` operator. The comment clarifies that this is conceptually like `list[index]`.

**Parameters:**

  * `other`: A `Number` object representing the index of the element to retrieve.

**Explanation:**

```python
if isinstance(other, Number):
    try:
        return self.elements[other.value-1], None # Adjust for 1-based indexing
    except: # Broad exception handling for index out of bounds
        return None, RTError(
            other.pos_start, other.pos_end,
            'Element at this index could not be retrieved from list because index is out of bounds',
            self.context
        )
else:
    return None, Value.illegal_operation(self, other)
```

1.  **Type Check:** Verifies that `other` is a `Number` (representing the index).
2.  **Access Element:** Uses a `try-except` block to safely attempt to access an element.
      * `self.elements[other.value - 1]`: Retrieves the element at the index specified by `other.value`, adjusting for 1-based indexing (if applicable).
3.  **Error Handling:** If the index is out of bounds or any other error occurs during access, it returns an `RTError`.
4.  Returns the retrieved element (`Value` object) with no error.

### `copy(self)`

**Purpose:** Creates a copy of the `List` instance.

**Parameters:** None.

**Explanation:**

```python
copy = List(self.elements) # This is a shallow copy of the 'elements' list.
copy.set_pos(self.pos_start, self.pos_end)
copy.set_context(self.context)
return copy
```

Creates a new `List` object. **Important Note:** `List(self.elements)` performs a *shallow copy* of the underlying Python list of elements. This means the new list (`copy.elements`) will contain references to the *same* `Value` objects as the original list. If these `Value` objects are mutable (e.g., other `List` objects), changes to them in the copy will affect the original, and vice-versa. For a true deep copy (where all nested `Value` objects are also copied), a more involved copying mechanism would be needed. It then copies its position and context.

### `__repr__(self)`

**Purpose:** Provides the formal string representation of the `List` object, often used for debugging.

**Parameters:** None.

**Explanation:**

```python
return ", ".join([str(x) for x in self.elements])
```

Returns a string where each element's `str()` representation is joined by a comma and space. This is a common representation but might not include the `[]` brackets that typically denote a list.

### `__str__(self)`

**Purpose:** Provides the informal string representation of the `List` object, used by `print()` and `str()`.

**Parameters:** None.

**Explanation:**

```python
return f'[{", ".join([str(x) for x in self.elements])}]'
```

Returns a formatted string that includes square brackets `[]` around the comma-separated string representations of its elements. This provides a more conventional list representation for display to the user.

