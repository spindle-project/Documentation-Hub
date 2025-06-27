---
type: page
title: Defining The Built In Functions
listed: true
slug: defining-the-built-in-functions
description: 
index_title: Defining The Built In Functions
hidden: 
keywords: 
tags: 
---
````markdown
# Built-in Functions

This section documents the `BuiltInFunction` class, which serves as a blueprint for functions pre-defined and available in the language, as well as the specific implementations of these built-in functions.

## Class: `BuiltInFunction`

The `BuiltInFunction` class inherits from `BaseFunction` (which is not provided in this snippet but is assumed to define common function properties like name, context, and position). It provides a generic mechanism to execute built-in functions by dynamically dispatching to methods based on the function's name.

### `__init__(self, name)`

**Purpose:** Initializes a `BuiltInFunction` instance.

**Parameters:**
* `name`: A string representing the name of the built-in function (e.g., "display", "input").

**Explanation:**
```python
super().__init__(name)
````

Calls the constructor of the `BaseFunction` class, passing the `name` to initialize common function attributes.

### `execute(self, args)`

**Purpose:** The core method for executing a built-in function. This method is called by the interpreter when a built-in function is invoked.

**Parameters:**

  * `args`: A list of arguments (evaluated `Value` objects, such as `Number`, `String`, `List`) passed to the built-in function.

**Explanation:**

```python
res = RTResult()
exec_ctx = self.generate_new_context()

method_name = f'execute_{self.name}'
method = getattr(self,method_name,self.no_visit_method)
res.register(self.check_and_populate_args(method.arg_names, args, exec_ctx))
if res.should_return(): return res

return_value = res.register(method(exec_ctx))
if res.should_return(): return res
return res.success(return_value)
```

1.  **Result Object:** Initializes an `RTResult` object to manage the execution outcome (success/failure, return/break/continue signals).
2.  **New Execution Context:**
    ```python
    exec_ctx = self.generate_new_context()
    ```
    Generates a new execution context for the built-in function call. This is important for isolating the function's local variables (its arguments) from the caller's context.
3.  **Dynamic Method Dispatch:**
    ```python
    method_name = f'execute_{self.name}'
    method = getattr(self,method_name,self.no_visit_method)
    ```
    Constructs the name of the specific execution method (e.g., `execute_display`, `execute_input`) based on the `BuiltInFunction`'s `name`. `getattr()` retrieves this method from `self`, or defaults to `self.no_visit_method` if not found.
4.  **Argument Checking and Population:**
    ```python
    res.register(self.check_and_populate_args(method.arg_names, args, exec_ctx))
    if res.should_return(): return res
    ```
    Calls `self.check_and_populate_args` (presumably a method from `BaseFunction`) to:
      * Verify that the number and types of provided `args` match the `method.arg_names` (defined later for each built-in function).
      * Populate the `exec_ctx.symbol_table` with the argument values under their respective names.
        If this process encounters an error (e.g., wrong number of arguments), it registers the error and returns immediately.
5.  **Execute Built-in Logic:**
    ```python
    return_value = res.register(method(exec_ctx))
    if res.should_return(): return res
    ```
    Calls the specific `execute_` method (e.g., `execute_display`) with the `exec_ctx`. This is where the actual logic of the built-in function is performed. The return value is registered.
6.  **Return Result:** Returns the `return_value` as a successful `RTResult`.

### `no_visit_method(self, node, context)`

**Purpose:** A fallback method called if a specific `execute_` method for a built-in function is not defined.

**Parameters:**

  * `node`: (Unused in this context, but present due to the `getattr` default signature.)
  * `context`: (Unused in this context, but present due to the `getattr` default signature.)

**Explanation:**

```python
raise Exception(f'No execute_{self.name} method defined')
```

Raises an `Exception` indicating that the interpreter could not find the implementation for the built-in function being called.

### `copy(self)`

**Purpose:** Creates a copy of the `BuiltInFunction` instance. This is typically used when functions are passed around or stored, to ensure that the function object itself can carry its context and position information.

**Parameters:** None.

**Explanation:**

```python
copy = BuiltInFunction(self.name)
copy.set_context(self.context)
copy.set_pos(self.pos_start, self.pos_end)
return copy
```

Creates a new `BuiltInFunction` with the same `name`, and then copies the `context` and positional information from the original instance to the new copy.

### `__repr__(self)`

**Purpose:** Provides a string representation of the `BuiltInFunction` object, useful for debugging and display.

**Parameters:** None.

**Explanation:**

```python
return f"<built-in function {self.name}>"
```

Returns a formatted string indicating that it's a built-in function and its name.

-----

## Built-in Functions List

This section details the implementations of various built-in functions available in the language. Each function is a method within the `BuiltInFunction` class, named `execute_functionName`. After each method definition, a class attribute `arg_names` is set to a list of strings, indicating the expected parameter names for the function. These parameter names are used by `check_and_populate_args` to match arguments passed during a function call.

### `execute_display(self, exec_ctx)`

**Purpose:** Displays a value to the console (standard output).

**Parameters:**

  * `exec_ctx`: The execution context for this function call. Expected to contain a variable named `value` in its symbol table.

**Explanation:**

```python
display_res(str(exec_ctx.symbol_table.get('value')))
return RTResult().success(Number.null)
```

1.  Retrieves the `value` argument from the `exec_ctx`'s symbol table.
2.  Converts the value to a string using `str()`.
3.  Calls `display_res()` (presumably an external utility function for printing output).
4.  Returns `Number.null` as a successful result, as `display` typically doesn't return a meaningful value.

**Arguments:**

  * `value`: The value to be displayed.

<!-- end list -->

```python
execute_display.arg_names = ["value"]
```

### `execute_input(self, exec_ctx)`

**Purpose:** Reads a line of text from standard input. It attempts to convert the input to a number if possible; otherwise, it returns it as a string.

**Parameters:**

  * `exec_ctx`: The execution context.

**Explanation:**

```python
text = input()
try:
    val = int(text)
    return RTResult().success(Number(val))
except ValueError:
    return RTResult().success(String(text))
```

1.  Uses Python's built-in `input()` function to read a line of text.
2.  **Type Coercion:**
      * Attempts to convert the `text` to an integer using `int()`. If successful, it wraps it in a `Number` object and returns it.
      * If `int()` raises a `ValueError` (meaning the input is not a valid integer), it treats the input as a string, wraps it in a `String` object, and returns it.

**Arguments:** None.

```python
execute_input.arg_names = []
```

### `execute_clear(self, exec_ctx)`

**Purpose:** Clears the terminal screen.

**Parameters:**

  * `exec_ctx`: The execution context.

**Explanation:**

```python
os.system('cls' if os.name == 'nt' else 'cls') 
return RTResult().success(Number.null)
```

1.  Uses Python's `os.system()` to execute a system command.
2.  `os.name == 'nt'` checks if the operating system is Windows. If true, it runs `cls`; otherwise, it also runs `cls`. (Note: For non-Windows systems, `clear` is the more common command, so this might be an oversight for cross-platform compatibility).
3.  Returns `Number.null` as a successful result.

**Arguments:** None.

```python
execute_clear.arg_names = []
```

### `execute_is_number(self, exec_ctx)`

**Purpose:** Checks if the provided argument is a `Number` type.

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain a variable named `value`.

**Explanation:**

```python
is_number = isinstance(exec_ctx.symbol_table.get("value"), Number)
return RTResult().success(Number.true if is_number else Number.false)
```

1.  Retrieves the `value` argument.
2.  Uses `isinstance()` to check if the `value` is an instance of the `Number` class.
3.  Returns `Number.true` if it's a number, otherwise `Number.false`.

**Arguments:**

  * `value`: The value to check.

<!-- end list -->

```python
execute_is_number.arg_names = ["value"]
```

### `execute_is_string(self, exec_ctx)`

**Purpose:** Checks if the provided argument is a `String` type.

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain a variable named `value`.

**Explanation:**

```python
is_number = isinstance(exec_ctx.symbol_table.get("value"), String)
return RTResult().success(Number.true if is_number else Number.false)
```

Similar to `execute_is_number`, but checks for `String` type. (Note: The variable name `is_number` is a copy-paste error and should ideally be `is_string`).

**Arguments:**

  * `value`: The value to check.

<!-- end list -->

```python
execute_is_string.arg_names = ["value"]
```

### `execute_is_list(self, exec_ctx)`

**Purpose:** Checks if the provided argument is a `List` type.

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain a variable named `value`.

**Explanation:**

```python
is_number = isinstance(exec_ctx.symbol_table.get("value"), List)
return RTResult().success(Number.true if is_number else Number.false)
```

Similar to `execute_is_number`, but checks for `List` type. (Note: The variable name `is_number` is a copy-paste error and should ideally be `is_list`).

**Arguments:**

  * `value`: The value to check.

<!-- end list -->

```python
execute_is_list.arg_names = ["value"]
```

### `execute_is_function(self, exec_ctx)`

**Purpose:** Checks if the provided argument is a function type (specifically, an instance of `BaseFunction`).

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain a variable named `value`.

**Explanation:**

```python
is_number = isinstance(exec_ctx.symbol_table.get("value"), BaseFunction)
return RTResult().success(Number.true if is_number else Number.false)
```

Similar to `execute_is_number`, but checks for `BaseFunction` type. (Note: The variable name `is_number` is a copy-paste error and should ideally be `is_function`).

**Arguments:**

  * `value`: The value to check.

<!-- end list -->

```python
execute_is_function.arg_names = ["value"]
```

### `execute_append(self, exec_ctx)`

**Purpose:** Appends a `value` to a `list`.

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain variables named `list` and `value`.

**Explanation:**

```python
list_ = exec_ctx.symbol_table.get("list")
value = exec_ctx.symbol_table.get("value")

if not isinstance(list_, List):
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        "First argument must be list",
        exec_ctx
    ))

list_.elements.append(value)
return RTResult().success(Number.null)
```

1.  Retrieves `list_` and `value` arguments.
2.  **Type Check:** Checks if `list_` is indeed a `List` type. If not, it returns an `RTError`.
3.  **Append:** Uses Python's `list.append()` method on the internal `elements` list of the `list_` object.
4.  Returns `Number.null` as a successful result.

**Arguments:**

  * `list`: The list to which the value will be appended.
  * `value`: The value to append.

<!-- end list -->

```python
execute_append.arg_names = ["list", "value"]
```

### `execute_pop(self, exec_ctx)`

**Purpose:** Removes and returns an element from a list at a specified `index`.

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain variables named `list` and `index`.

**Explanation:**

```python
list_ = exec_ctx.symbol_table.get("list")
index = exec_ctx.symbol_table.get("index")

if not isinstance(list_, List):
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        "First argument must be list",
        exec_ctx
    ))

if not isinstance(index, Number):
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        "Second argument must be number",
        exec_ctx
    ))

try:
    element = list_.elements.pop(index.value)
except: # Broad exception handling
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        'Element at this index could not be removed from list because index is out of bounds',
        exec_ctx
    ))
return RTResult().success(element)
```

1.  Retrieves `list_` and `index` arguments.
2.  **Type Checks:** Verifies that `list_` is a `List` and `index` is a `Number`. Returns `RTError` if types are incorrect.
3.  **Pop Element:**
      * Uses a `try-except` block to handle potential `IndexError` (or other exceptions) if `index.value` is out of bounds.
      * `list_.elements.pop(index.value)` removes the element at the specified index from the underlying Python list.
4.  **Error Handling:** If an exception occurs, it returns an `RTError` indicating an out-of-bounds index.
5.  Returns the `element` that was popped as a successful `RTResult`.

**Arguments:**

  * `list`: The list from which to pop an element.
  * `index`: The numerical index of the element to pop.

<!-- end list -->

```python
execute_pop.arg_names = ["list", "index"]
```

### `execute_extend(self, exec_ctx)`

**Purpose:** Extends `listA` by appending all elements from `listB`.

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain variables named `listA` and `listB`.

**Explanation:**

```python
listA = exec_ctx.symbol_table.get("listA")
listB = exec_ctx.symbol_table.get("listB")

if not isinstance(listA, List):
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        "First argument must be list",
        exec_ctx
    ))

if not isinstance(listB, List):
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        "Second argument must be list",
        exec_ctx
    ))

listA.elements.extend(listB.elements)
return RTResult().success(Number.null)
```

1.  Retrieves `listA` and `listB` arguments.
2.  **Type Checks:** Verifies that both `listA` and `listB` are `List` types. Returns `RTError` if types are incorrect.
3.  **Extend:** Uses Python's `list.extend()` method to add all elements from `listB.elements` to `listA.elements`.
4.  Returns `Number.null` as a successful result.

**Arguments:**

  * `listA`: The list to be extended.
  * `listB`: The list whose elements will be appended to `listA`.

<!-- end list -->

```python
execute_extend.arg_names = ["listA", "listB"]
```

### `execute_length(self, exec_ctx)`

**Purpose:** Returns the number of elements in a list.

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain a variable named `list`.

**Explanation:**

```python
list_ = exec_ctx.symbol_table.get("list")

if not isinstance(list_, List):
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        "Argument must be list",
        exec_ctx
    ))

return RTResult().success(Number(len(list_.elements)))
```

1.  Retrieves the `list_` argument.
2.  **Type Check:** Verifies that `list_` is a `List` type. Returns `RTError` if incorrect.
3.  **Get Length:** Uses Python's `len()` function on the internal `elements` list.
4.  Wraps the length in a `Number` object and returns it as a successful `RTResult`.

**Arguments:**

  * `list`: The list for which to get the length.

<!-- end list -->

```python
execute_length.arg_names = ["list"]
```

### `execute_run(self, exec_ctx)`

**Purpose:** Executes a script from a file specified by a string filename.

**Parameters:**

  * `exec_ctx`: The execution context. Expected to contain a variable named `fn` (filename).

**Explanation:**

```python
fn = exec_ctx.symbol_table.get("fn")

if not isinstance(fn, String):
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        "Second argument must be string", # Note: Comment says "Second argument", but `fn` is the only argument expected by arg_names
        exec_ctx
    ))

fn = fn.value
try:
    with open(fn, "r") as f:
        script = f.read()
except Exception as e: # Broad exception handling
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        f"Failed to load script \"{fn}\"\n" + str(e),
        exec_ctx
    ))

_, error = run(fn, script) # Assumes 'run' is an external function that compiles and executes the script

if error:
    return RTResult().failure(RTError(
        self.pos_start, self.pos_end,
        f"Failed to finish executing script \"{fn}\"\n" +
        error.as_string(),
        exec_ctx
    ))

return RTResult().success(Number.null)
```

1.  Retrieves the `fn` (filename) argument.
2.  **Type Check:** Verifies that `fn` is a `String`. Returns `RTError` if incorrect.
3.  **Read File:**
      * Attempts to open and read the content of the file specified by `fn.value`.
      * Uses a `try-except` block to catch any `Exception` during file operations (e.g., `FileNotFoundError`). If an error occurs, it returns an `RTError`.
4.  **Execute Script:**
    ```python
    _, error = run(fn, script)
    ```
    Calls an external `run()` function (presumably the main entry point of the interpreter's pipeline: lexer -\> parser -\> interpreter). This `run()` function takes the filename and the script content and returns a result and an error (if any).
5.  **Handle Script Execution Errors:** If `run()` returns an `error`, it wraps that error in an `RTError` and returns it.
6.  Returns `Number.null` as a successful result if the script executes without errors.

**Arguments:**

  * `fn`: The filename (as a string) of the script to execute.

<!-- end list -->

```python
execute_run.arg_names = ["fn"]
```
