---
type: page
title: Functions: Base Function and Function Class
listed: true
slug: functions--base-function-and-function-class
description: 
index_title: Functions: Base Function and Function Class
hidden: 
keywords: 
tags: 
---
# Function Classes: `BaseFunction` and `Function`

This section documents the `BaseFunction` and `Function` classes, which are crucial for handling callable entities within the language. `BaseFunction` establishes the fundamental properties and behaviors common to all functions (both user-defined and built-in), while `Function` specifically implements user-defined functions parsed from the source code.

## Class: `BaseFunction`

The `BaseFunction` class serves as the abstract base for all callable objects in the language, including user-defined functions and built-in functions. It inherits from `Value`, meaning functions themselves are first-class values that can be passed around, stored in variables, and returned from other functions.

### `__init__(self, name)`

**Purpose:** Initializes a `BaseFunction` instance.

**Parameters:**
* `name`: A string representing the name of the function. If `None` or an empty string, it defaults to `"<anonymous>"`.

**Explanation:**
```python
super().__init__()
self.name = name or "<anonymous>"
````

1.  Calls the constructor of the `Value` class (`super().__init__()`) to inherit basic value properties like position (`pos_start`, `pos_end`) and context.
2.  Sets the `name` attribute. This name is used primarily for debugging and error reporting.

### `generate_new_context(self)`

**Purpose:** Creates a new execution context for the function call. This is essential for managing scope, ensuring that variables defined within a function do not interfere with variables outside it.

**Parameters:** None.

**Explanation:**

```python
new_context = Context(self.name, self.context, self.pos_start)
new_context.symbol_table = SymbolTable(new_context.parent.symbol_table)
return new_context
```

1.  **Creates `Context`:** Instantiates a new `Context` object.
      * The `name` of the new context is set to the function's `self.name`.
      * Its `parent` context is set to `self.context` (the context where the function was defined/declared, establishing lexical scoping).
      * Its `parent_entry_pos` is set to `self.pos_start`, indicating where the function was called or defined.
2.  **Creates `SymbolTable`:** Assigns a new `SymbolTable` to this `new_context`. Crucially, this `SymbolTable`'s parent is set to the `symbol_table` of its parent context (`new_context.parent.symbol_table`). This forms the **scope chain**, allowing the function to access variables from its enclosing scopes.
3.  Returns the newly created `new_context`.

### `check_args(self, arg_names, args)`

**Purpose:** Verifies if the number of arguments (`args`) provided during a function call matches the expected number of argument names (`arg_names`) defined by the function.

**Parameters:**

  * `arg_names`: A list of strings, representing the expected names of the arguments for this function (e.g., `["param1", "param2"]`).
  * `args`: A list of `Value` objects, representing the actual arguments passed during the function call.

**Explanation:**

```python
res = RTResult()
if len(args) > len(arg_names):
    return res.failure(RTError(
        self.pos_start, self.pos_end,
        f"{len(args) - len(arg_names)} too many args passed into '{self.name}'",
        self.context
    ))

if len(args) < len(arg_names):
    return res.failure(RTError(
        self.pos_start, self.pos_end,
        f"{len(arg_names) - len(args)} too few args passed into '{self.name}'",
        self.context
    ))
return res.success(None)
```

1.  Initializes an `RTResult` to manage the outcome.
2.  **Too Many Arguments:** Checks if the number of `args` exceeds `arg_names`. If so, it returns an `RTError` indicating a "too many arguments" error, including the function's name and the count of excess arguments.
3.  **Too Few Arguments:** Checks if the number of `args` is less than `arg_names`. If so, it returns an `RTError` indicating a "too few arguments" error, including the function's name and the count of missing arguments.
4.  **Success:** If the argument counts match, it returns a successful `RTResult`.

### `populate_args(self, arg_names, args, exec_ctx)`

**Purpose:** Populates the `SymbolTable` of the function's execution context (`exec_ctx`) with the provided arguments, binding argument names to their corresponding values.

**Parameters:**

  * `arg_names`: The list of expected argument names.
  * `args`: The list of actual argument `Value` objects.
  * `exec_ctx`: The new `Context` created for this function call.

**Explanation:**

```python
for i in range(len(args)):
    arg_name = arg_names[i]
    arg_value = args[i]
    arg_value.set_context(exec_ctx) # Set the context of the argument value to the new execution context
    exec_ctx.symbol_table.set(arg_name, arg_value)
```

1.  Iterates through the provided `args` and `arg_names` simultaneously (assuming `check_args` has already confirmed they have the same length).
2.  For each argument:
      * Retrieves the `arg_name` and `arg_value`.
      * **Sets Value Context:** `arg_value.set_context(exec_ctx)` is an important step. It updates the context of the argument `Value` object itself to reflect that it now "lives" within this new execution context. This helps in more precise error reporting if an operation on `arg_value` fails within the function.
      * **Binds in Symbol Table:** `exec_ctx.symbol_table.set(arg_name, arg_value)` binds the argument name (e.g., "x") to its corresponding value in the function's local symbol table.

### `check_and_populate_args(self, arg_names, args, exec_ctx)`

**Purpose:** A convenience method that combines argument checking and population into a single call. This is typically used by the `execute` method of `BaseFunction` subclasses.

**Parameters:**

  * `arg_names`: The list of expected argument names.
  * `args`: The list of actual argument `Value` objects.
  * `exec_ctx`: The new `Context` created for this function call.

**Explanation:**

```python
res= RTResult()
res.register(self.check_args(arg_names, args))
if res.should_return(): return res
self.populate_args(arg_names, args, exec_ctx)
return res.success(None)
```

1.  Initializes an `RTResult`.
2.  Calls `self.check_args()`. If `check_args()` returns an error (meaning `res.should_return()` is true), it immediately returns that error.
3.  If argument checking passes, it calls `self.populate_args()` to bind the arguments in the `exec_ctx`.
4.  Returns a successful `RTResult`.

-----

## Class: `Function`

The `Function` class represents user-defined functions in the language. It inherits from `BaseFunction` and adds the specific logic required to interpret and execute the function's body.

*(The provided snippet only shows the class definition for `Function` and does not include its methods. Based on typical interpreter design, `Function` would override the `execute` method from `Value` (which `BaseFunction` inherits) and also define how to copy itself. A typical `Function` class would also likely store information about its body (e.g., an AST node for the function's statements) and its argument names.)*

### Expected Methods for `Function`

Based on the structure, `Function` would typically include:

  * **`__init__(self, name, body_node, arg_names, should_auto_return)`**:
      * `name`: The function's name.
      * `body_node`: The Abstract Syntax Tree (AST) node representing the function's body (the statements to execute).
      * `arg_names`: A list of strings for the formal parameter names.
      * `should_auto_return`: A boolean flag indicating if the function should implicitly return its last expression's value.
  * **`execute(self, args)`**:
      * This is the core method for running a user-defined function.
      * It would call `self.generate_new_context()`.
      * It would then call `self.check_and_populate_args()` to set up the arguments in the new context.
      * Finally, it would typically use an `Interpreter` or `Runtime` object to visit and execute the `body_node` within the new context.
      * It would handle the return value, break/continue statements, and errors from the body's execution.
  * **`copy(self)`**:
      * To create a new `Function` object with the same properties (name, body, arg names, context, position).
  * **`__repr__(self)`**:
      * To provide a string representation like `<function my_func>`.

<!-- end list -->

```python
# Handle functions!
class Function(BaseFunction):
    def __init__(self, name, body_node, arg_names, should_auto_return):
        super().__init__(name)
        self.body_node = body_node
        self.arg_names = arg_names
        self.should_auto_return = should_auto_return

    def execute(self, args):
        res = RTResult()
        interpreter = Interpreter() # Assuming an Interpreter instance is available or passed
        exec_ctx = self.generate_new_context()

        res.register(self.check_and_populate_args(self.arg_names, args, exec_ctx))
        if res.should_return(): return res

        value = res.register(interpreter.visit(self.body_node, exec_ctx))
        if res.should_return(): return res

        return_value = value if self.should_auto_return else Number.null
        return res.success(return_value)

    def copy(self):
        copy = Function(self.name, self.body_node, self.arg_names, self.should_auto_return)
        copy.set_context(self.context)
        copy.set_pos(self.pos_start, self.pos_end)
        return copy

    def __repr__(self):
        return f"<function {self.name}>"

```
