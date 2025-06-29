---
type: page
title: The Parser: Creating an Abstract Syntax Tree
listed: true
slug: the-parser
description: 
index_title: The Parser: Creating an Abstract Syntax Tree
hidden: 
keywords: 
tags: 
---

## Class: `Parser`

{% code %}
{% tab language="python" %}
class Parser:
    def __init__(self, tokens):
        self.tokens = tokens
        self.tok_idx = -1
        self.advance()
`
{% /tab %}
{% /code %}

The `Parser` is initialized with a list of **tokens**. It maintains `tok_idx` to keep track of the current token being processed and immediately calls `advance()` to set up the initial `current_tok`.

---

### Core Navigation Methods

These methods allow the parser to move through the token stream.

#### `advance()`

{% code %}
{% tab language="python" %}
def advance(self):
        self.tok_idx += 1
        self.update_current_tok()
        return self.current_tok
{% /tab %}
{% /code %}

Moves the parser's internal pointer (`tok_idx`) to the next token in the stream and updates `current_tok`. This is the primary way the parser consumes tokens.

#### `reverse(amount=1)`

{% code %}
{% tab language="python" %}
def reverse(self, amount=1):
        self.tok_idx -= amount
        self.update_current_tok()
        return self.current_tok
{% /tab %}
{% /code %}

Moves the parser's internal pointer back by a specified `amount` (defaulting to 1). This is useful for backtracking in case of parsing ambiguities or errors.

#### `update_current_tok()`

{% code %}
{% tab language="python" %}
def update_current_tok(self):
        if self.tok_idx >= 0 and self.tok_idx < len(self.tokens):
            self.current_tok = self.tokens[self.tok_idx]
{% /tab %}
{% /code %}

Internal method to update `self.current_tok` based on the current `self.tok_idx`. It ensures that the index is within the valid range of tokens.

#### `peek(amount)`

{% code %}
{% tab language="python" %}
def peek(self, amount):
        return self.tokens[self.tok_idx+amount]
{% /tab %}
{% /code %}

Allows the parser to look ahead in the token stream by a specified `amount` without actually advancing its position. This is useful for making parsing decisions based on upcoming tokens.

---

### Primary Parsing Method

#### `parse()`

{% code %}
{% tab language="python" %}
def parse(self):
        res = self.statements()
        # Ignore TT_EOF, TT_KEYWORD, TT_IDENTIFIER,TT_EQ,TT_RBRACE. These should not cause errors. Everything else should though!
        if not res.error and self.current_tok.type not in (TT_EOF, TT_KEYWORD, TT_IDENTIFIER,TT_EQ,TT_RBRACE):
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected '+', '-', '*', '/', ,'[', '^', '==', '!=', '<', '>', <=', '>=', 'AND' or 'OR'"
            ))
        return res
{% /tab %}
{% /code %}

This is the entry point for the parsing process. It attempts to parse a sequence of **statements** and handles any remaining unexpected tokens, generating an `InvalidSyntaxError` if found.

---

### Statement Parsing

#### `statement()`

{% code %}
{% tab language="python" %}
def statement(self):
        res = ParseResult()
        pos_start = self.current_tok.pos_start.copy()

        if self.current_tok.matches(TT_KEYWORD, 'RETURN'):
            res.register_advancement()
            self.advance()
            expr = res.try_register(self.expr())
            if not expr:
                self.reverse(res.to_reverse_count)
            return res.success(ReturnNode(expr, pos_start, self.current_tok.pos_start.copy()))
            
        if self.current_tok.matches(TT_KEYWORD, 'CONTINUE'):
            res.register_advancement()
            self.advance()
            return res.success(ContinueNode(pos_start, self.current_tok.pos_start.copy()))
        
        if self.current_tok.matches(TT_KEYWORD, 'BREAK'):
            res.register_advancement()
            self.advance()
            return res.success(BreakNode(pos_start, self.current_tok.pos_start.copy()))
        expr = res.register(self.expr())
        if res.error:
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected 'RETURN', 'VAR', 'IF', 'FOR', 'WHILE', 'FUN', int, float, identifier, '+', '-', '(', '[' or 'NOT'"
            ))
        return res.success(expr)
{% /tab %}
{% /code %}

Parses a single statement, which can be a `RETURN`, `CONTINUE`, `BREAK`, or a general **expression**. It handles keywords for control flow and registers advancements to consume tokens.

#### `statements()`

{% code %}
{% tab language="python" %}
def statements(self):
        res = ParseResult()
        statements = []
        pos_start = self.current_tok.pos_start.copy()

        while self.current_tok.type == TT_NEWLINE:
            res.register_advancement()
            self.advance()

        statement = res.register(self.statement())
        if res.error: return res
        statements.append(statement)

        more_statements = True

        while True:
            newline_count = 0
            while self.current_tok.type == TT_NEWLINE:
                res.register_advancement()
                self.advance()
                newline_count += 1
            if newline_count == 0:
                more_statements = False
            
            if not more_statements: break
            statement = res.try_register(self.statement())
            if not statement:
                self.reverse(res.to_reverse_count)
                more_statements = False
                continue
            statements.append(statement)

        return res.success(ListNode(
        statements,
        pos_start,
        self.current_tok.pos_end.copy()
        ))
{% /tab %}
{% /code %}

Parses multiple statements, handling newlines as delimiters between them. It collects individual **statement nodes** into a **list node**.

---

### Control Flow Parsing

#### `if_expr()`

{% code %}
{% tab language="python" %}
def if_expr(self):
        res = ParseResult()
        all_cases = res.register(self.if_expr_cases('IF'))
        if res.error: return res
        cases, else_case = all_cases
        return res.success(IfNode(cases, else_case))
{% /tab %}
{% /code %}

The main entry point for parsing `IF` statements. It delegates the heavy lifting to `if_expr_cases`.

#### `if_expr_c()`

{% code %}
{% tab language="python" %}
def if_expr_c(self):
        res = ParseResult()
        else_case = None
        while self.current_tok.type == TT_NEWLINE:
            res.register_advancement()
            self.advance()
        if self.current_tok.matches(TT_KEYWORD, 'ELSE'):
            res.register_advancement()
            self.advance()

            while self.current_tok.type == TT_NEWLINE:
                res.register_advancement()
                self.advance()

            if self.current_tok.type == TT_LBRACE:
                res.register_advancement()
                self.advance()

            if self.current_tok.type == TT_NEWLINE:
                res.register_advancement()
                self.advance()

                while self.current_tok.type == TT_NEWLINE:
                    res.register_advancement()
                    self.advance()

                statements = res.register(self.statements())
                if res.error: return res
                else_case = (statements, True)
                while self.current_tok.type == TT_NEWLINE:
                    res.register_advancement()
                    self.advance()
                if self.current_tok.type ==  TT_RBRACE:
                    res.register_advancement()
                    self.advance()
                else:
                    return res.failure(InvalidSyntaxError(
                        self.current_tok.pos_start, self.current_tok.pos_end,
                        "Expected '}'"
                    ))
            else:
                while self.current_tok.type == TT_NEWLINE:
                    res.register_advancement()
                    self.advance()
                res.register_advancement()
                self.advance()
                while self.current_tok.type == TT_NEWLINE:
                    res.register_advancement()
                    self.advance()
                expr = res.register(self.statements())
                if res.error:
                    return res
                else_case = (expr, False)
                res.register_advancement()
                self.advance()
                if self.current_tok.type in ( TT_RBRACE):
                    res.register_advancement()
                    self.advance()
            return res.success(else_case)
{% /tab %}
{% /code %}

Handles the parsing of `ELSE` clauses within an `IF` statement, including optional curly braces and newlines.

#### `if_expr_b_or_c()`

{% code %}
{% tab language="python" %}
def if_expr_b_or_c(self):
        res = ParseResult()
        cases, else_case = [], None
        while self.current_tok.type == TT_NEWLINE:
            res.register_advancement()
            self.advance()
        if self.current_tok.matches(TT_KEYWORD, "ELSE"):
            else_case = res.register(self.if_expr_c())
            if res.error: return res
        return res.success((cases, else_case))
{% /tab %}
{% /code %}

A helper method to determine whether the current token signifies an `ELSE` clause for an `IF` statement.

#### `if_expr_cases(self, case_keyword)`

{% code %}
{% tab language="python" %}
def if_expr_cases(self, case_keyword):
        res = ParseResult()
        cases = []
        else_case = None
        if not self.current_tok.matches(TT_KEYWORD, case_keyword):
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected '{case_keyword}'"
            ))
        res.register_advancement()
        self.advance()
        if self.current_tok.type == TT_LPAREN:
            res.register_advancement()
            self.advance()
        condition = res.register(self.statement())
        if res.error: return res
        while self.current_tok.type == TT_NEWLINE:
            res.register_advancement()
            self.advance()

        if not self.current_tok.type  == TT_RPAREN: #NOTE: in(TT_LBRACE , TT_RPAREN)
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected '}' got " + f"{self.current_tok}"
            ))
        res.register_advancement()
        self.advance()
        if self.current_tok.type == TT_LBRACE:
            res.register_advancement()
            self.advance()
        if self.current_tok.type == TT_NEWLINE:
            while self.current_tok.type == TT_NEWLINE:
                res.register_advancement()
                self.advance()

            statements = res.register(self.statements())
            if res.error: return res
            cases.append((condition, statements, True))
            while self.current_tok.type == TT_NEWLINE:
                res.register_advancement()
                self.advance()
            if self.current_tok.type == TT_RBRACE:
                res.register_advancement()
                self.advance()
                all_cases = res.register(self.if_expr_b_or_c())
                if res.error: return res
                new_cases, else_case = all_cases
                cases.extend(new_cases)
            else:
                all_cases = res.register(self.if_expr_b_or_c())
                if res.error: return res
                while self.current_tok.type == TT_NEWLINE:
                    res.register_advancement()
                    self.advance()
                if self.current_tok.type == TT_RBRACE:
                    res.register_advancement()
                    self.advance()
                new_cases, else_case = all_cases
                cases.extend(new_cases)
        else:
            expr = res.register(self.statements())
            if res.error: return res
            cases.append((condition, expr, True))

            all_cases = res.register(self.if_expr_b_or_c())
            if res.error: return res
            new_cases, else_case = all_cases
            cases.extend(new_cases)

        return res.success((cases, else_case))
{% /tab %}
{% /code %}

Handles the parsing of the conditional logic and the body of both `IF` and `ELSE IF` (or `ELIF`) blocks. It ensures correct syntax for parentheses, curly braces, and newlines.

#### `for_expr()`

{% code %}
{% tab language="python" %}
def for_expr(self):
        res = ParseResult()

        if not self.current_tok.matches(TT_KEYWORD, 'REPEAT'):
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected 'REPEAT'"
            ))

        res.register_advancement()
        self.advance()

        if self.current_tok.type != TT_IDENTIFIER:
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected identifier 122"
            ))

        var_name = self.current_tok

        
        res.register_advancement()
        self.advance()

        start_value = 0
        if res.error: return res
        a = res.register(self.expr())
        end_value = a
        if res.error: return res
        step_value = None
        if not self.current_tok.matches(TT_KEYWORD, 'TIMES'):
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Your REPEAT or REPEAT UNTIL loop has the wrong syntax. Please deouble check that everything is spelled correctly."
            ))

        res.register_advancement()
        self.advance()
        
        if not self.current_tok.type == TT_LBRACE:
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected '{'"
            ))
        else:
            res.register_advancement()
            self.advance()
        
        if self.current_tok.type == TT_NEWLINE:
            res.register_advancement()
            self.advance()

            body = res.register(self.statements())
            if res.error: return res

            if not self.current_tok.type == TT_RBRACE:
                return res.failure(InvalidSyntaxError(
                    self.current_tok.pos_start, self.current_tok.pos_end,
                    "Expected '}'"
                    ))

            res.register_advancement()
            self.advance()

            return res.success(ForNode(var_name, start_value, end_value, step_value, body, True))

        body = res.register(self.statement())
        if res.error: return res

        if not self.current_tok.type == TT_RBRACE:
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected '}'"
            ))
        else:
            res.register_advancement()
            self.advance()

        return res.success(ForNode(var_name, start_value, end_value, step_value, body, False))
{% /tab %}
{% /code %}

Parses **`REPEAT`** (for) loops. It expects a variable name, a loop count (implicitly `TIMES`), and a body (either a single statement or multiple statements enclosed in curly braces).

#### `while_expr()`

{% code %}
{% tab language="python" %}
def while_expr(self):
        res = ParseResult()

        if not self.current_tok.matches(TT_KEYWORD, 'WHILE'):
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected 'WHILE'"
            ))

        res.register_advancement()
        self.advance()
        condition = res.register(self.expr())
        if res.error: return res

        res.register_advancement()
        self.advance()

        if self.current_tok.type == TT_NEWLINE:
            res.register_advancement()
            self.advance()

            body = res.register(self.statements())
            if res.error: return res

            if not self.current_tok.type == TT_RBRACE:
                return res.failure(InvalidSyntaxError(
                    self.current_tok.pos_start, self.current_tok.pos_end,
                    "Expected '}'"
                    ))

            res.register_advancement()
            self.advance()

            return res.success(WhileNode(condition, body, True))
        
        body = res.register(self.statement())
        if res.error: return res

        return res.success(WhileNode(condition, body, False))
{% /tab %}
{% /code %}

Parses **`WHILE`** loops. It expects a condition and a loop body.

---

### Function and Data Structure Parsing

#### `call()`

{% code %}
{% tab language="python" %}
def call(self):
        res = ParseResult()

        if self.current_tok.type == TT_RBRACE:
            return res.success(Number.null)
        atom = res.register(self.atom())
        if res.error: return res
        if self.current_tok.type == TT_LPAREN:
            res.register_advancement()
            self.advance()
            arg_nodes = []

            if self.current_tok.type == TT_RPAREN:
                res.register_advancement()
                self.advance()
            else:
                arg_nodes.append(res.register(self.expr()))
                if res.error:
                    return res.failure(InvalidSyntaxError(
                        self.current_tok.pos_start, self.current_tok.pos_end,
                        "Expected ')', '[', 'VARIBLE IDENTIFIFER', 'IF', 'REPEAT', 'REPEAT UNTIL', 'PROCEDURE', int, float, identifier, '+', '-', '(' or 'NOT'"
                    ))

                while self.current_tok.type == TT_COMMA:
                    res.register_advancement()
                    self.advance()

                    arg_nodes.append(res.register(self.expr()))
                    if res.error: return res
                if self.current_tok.type != TT_RPAREN:
                    return res.failure(InvalidSyntaxError(
                        self.current_tok.pos_start, self.current_tok.pos_end,
                        f"Expected ',' or ')'"
                    ))

                res.register_advancement()
                self.advance()
            return res.success(CallNode(atom, arg_nodes))
        return res.success(atom)
{% /tab %}
{% /code %}

Handles function calls in Spindle. It parses the function name (which can be an **atom**) and then any arguments passed within parentheses.

#### `atom()`

{% code %}
{% tab language="python" %}
def atom(self):
        res = ParseResult()
        tok = self.current_tok
        if tok.type in (TT_INT, TT_FLOAT):
            res.register_advancement()
            self.advance()
            return res.success(NumberNode(tok))
        
        if tok.type == TT_STRING:
            res.register_advancement()
            self.advance()
            return res.success(StringNode(tok))
        
        elif tok.matches(TT_KEYWORD, 'IF'):
            if_expr = res.register(self.if_expr())
            if res.error: return res
            return res.success(if_expr)
        
        elif tok.matches(TT_KEYWORD, 'REPEAT'):
            for_expr = res.register(self.for_expr())
            if res.error: return res
            return res.success(for_expr)

        elif tok.matches(TT_KEYWORD, 'WHILE'):
            while_expr = res.register(self.while_expr())
            if res.error: return res
            return res.success(while_expr)
        
        elif tok.matches(TT_KEYWORD, 'PROCEDURE'):
            func_def = res.register(self.func_def())
            if res.error: return res
            return res.success(func_def)
        
        elif tok.type == TT_IDENTIFIER:
            res.register_advancement()
            self.advance()
            return res.success(VarAccessNode(tok))

        elif tok.type == TT_LPAREN:
            res.register_advancement()
            self.advance()
            expr = res.register(self.expr())
            if res.error: return res
            if self.current_tok.type == TT_RPAREN:
                res.register_advancement()
                self.advance()
                return res.success(expr)
            
        elif tok.type == TT_LSQUARE:
            list_expr = res.register(self.list_expr())
            if res.error: return res
            return res.success(list_expr)

        else:
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected ')'"
            ))

            
        return res.failure(InvalidSyntaxError(
            tok.pos_start, tok.pos_end,
            "Expected int, float, identifier, '+', '-', '(', 'IF','REPEAT UNTIL', 'REPEAT', 'PROCEDURE'"
        ))
{% /tab %}
{% /code %}

Parses the most basic elements of Spindle code, including:

- **Numbers** (`INT`, `FLOAT`) and **Strings** (`STRING`)
- **Control flow structures** (`IF`, `REPEAT`, `WHILE`)
- **Function definitions** (`PROCEDURE`)
- **Variable access** (`IDENTIFIER`)
- **Parenthesized expressions**
- **Lists** (`LSQUARE`)

#### `list_expr()`

{% code %}
{% tab language="python" %}
def list_expr(self):
        res = ParseResult()
        element_nodes = []
        pos_start = self.current_tok.pos_start.copy()

        if self.current_tok.type != TT_LSQUARE:
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected '['"
            ))

        res.register_advancement()
        self.advance()

        if self.current_tok.type == TT_RSQUARE:
            pass # Do nothing

        else:
            element_nodes.append(res.register(self.expr()))
            if res.error:
                return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected ']', 'VAR', 'IF', 'FOR', 'WHILE', 'FUN', int, float, identifier, '+', '-', '(', '[' or 'NOT'"
                ))

        while self.current_tok.type == TT_COMMA:
            res.register_advancement()
            self.advance()

            element_nodes.append(res.register(self.expr()))
            if res.error: return res

        if self.current_tok.type != TT_RSQUARE:
            return res.failure(InvalidSyntaxError(
            self.current_tok.pos_start, self.current_tok.pos_end,
            f"Expected ',' or ']' Got {str(self.current_tok)}"
            ))

        res.register_advancement()
        self.advance()

        return res.success(ListNode(
        element_nodes,
        pos_start,
        self.current_tok.pos_end.copy()
        ))
{% /tab %}
{% /code %}

Parses **list literals** (e.g., `[1, 2, "hello"]`). It handles the opening and closing square brackets, as well as comma-separated elements within the list.

---

### Operator Precedence and Expressions

The following methods handle the parsing of expressions, respecting operator precedence. They generally follow a hierarchy from lowest precedence to highest.

#### `power()`

{% code %}
{% tab language="python" %}
def power(self):
        return self.bin_op(self.call, (TT_POW, ), self.factor)
{% /tab %}
{% /code %}

Parses expressions involving the **power operator** (`^`). This is noted as "LEGACY" and might indicate it's an older or less frequently used feature.

#### `factor()`

{% code %}
{% tab language="python" %}
def factor(self):
        res = ParseResult()
        tok = self.current_tok

        if tok.type in (TT_PLUS, TT_MINUS):
            res.register_advancement()
            self.advance()
            factor = res.register(self.factor())
            if res.error: return res
            return res.success(UnaryOpNode(tok, factor))

        return self.power()
{% /tab %}
{% /code %}

Handles **unary operations** such as positive (`+`) and negative (`-`) signs applied to a number or expression. It then calls `power()` to continue parsing higher-precedence operations.

#### `term()`

{% code %}
{% tab language="python" %}
def term(self):
        return self.bin_op(self.factor, (TT_MUL, TT_DIV))
{% /tab %}
{% /code %}

Parses **multiplication** (`*`) and **division** (`/`) operations, which have higher precedence than addition and subtraction.

#### `arith_expr()`

{% code %}
{% tab language="python" %}
def arith_expr(self):
        return self.bin_op(self.term, (TT_PLUS, TT_MINUS))
{% /tab %}
{% /code %}

Parses **addition** (`+`) and **subtraction** (`-`) operations.

#### `comp_expr()`

{% code %}
{% tab language="python" %}
def comp_expr(self):
        res = ParseResult()

        if self.current_tok.matches(TT_KEYWORD, 'NOT'):
            op_tok = self.current_tok
            res.register_advancement()
            self.advance()

            node = res.register(self.comp_expr())
            if res.error: return res
            return res.success(UnaryOpNode(op_tok, node))
        
        node = res.register(self.bin_op(self.arith_expr, (TT_EE, TT_NE, TT_LT, TT_GT, TT_LTE, TT_GTE)))
        
        if res.error:
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected int, float, identifier, '+', '-', '(', '[',  or 'NOT'"
            ))

        return res.success(node)
{% /tab %}
{% /code %}

Combines expressions with **comparison operators** (`==`, `!=`, `<`, `>`, `<=`, `>=`) and the **logical `NOT`** operator.

#### `expr()`

{% code %}
{% tab language="python" %}
def expr(self):
        res = ParseResult()

        if self.current_tok.type == TT_IDENTIFIER and self.peek(1).type  in TT_EQ :
            # Assigning a value to a varible
            if self.peek(1).type not in  (TT_IDENTIFIER ,TT_EQ):
                return res.failure(InvalidSyntaxError(
                    self.current_tok.pos_start, self.current_tok.pos_end,
                    "Expected '<-' or '<--'"
                ))
            var_name = self.current_tok
            self.advance()  
            self.advance()
            expr = res.register(self.expr())
            if res.error: return res
            return res.success(VarAssignNode(var_name, expr))
        if self.current_tok.type == TT_LBRACE: # Fix for for loops. Skip past the {
            self.advance()
        node =  res.register(self.bin_op(self.comp_expr, ((TT_KEYWORD, 'AND'),(TT_KEYWORD, 'OR'))))
        if res.error: 
            return res.failure(InvalidSyntaxError(
            self.current_tok.pos_start, self.current_tok.pos_end,
            "Expected 'VARIBLE IDENTIFIER', 'IF', 'REPEAT UNTIL', 'REPEAT', 'PROCEDURE', int, float, identifier, '+', '-', '[', or '(. \n Did you wrap an assigned varible in parenthesis?'"
        ))
        return res.success(node)
{% /tab %}
{% /code %}

The top-level expression parsing method. It handles:

- **Variable assignment** (e.g., `myVar = 10`)
- **Logical operators** (`AND`, `OR`)
- It also includes a "fix for for loops" by skipping `TT_LBRACE`.

#### `func_def()`

{% code %}
{% tab language="python" %}
def func_def(self):
        res = ParseResult()

        if not self.current_tok.matches(TT_KEYWORD, 'PROCEDURE'):
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected 'PROCEDURE'"
            ))

        res.register_advancement()
        self.advance()

        if self.current_tok.type == TT_IDENTIFIER:
            var_name_tok = self.current_tok
            res.register_advancement()
            self.advance()
            if self.current_tok.type != TT_LPAREN:
                return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected '('"
                ))
        else:
            var_name_tok = None
            if self.current_tok.type != TT_LPAREN:
                return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected identifier or '('"
                ))
        
        res.register_advancement()
        self.advance()
        arg_name_toks = []

        if self.current_tok.type == TT_IDENTIFIER:
            arg_name_toks.append(self.current_tok)
            res.register_advancement()
            self.advance()
        
        while self.current_tok.type == TT_COMMA:
            res.register_advancement()
            self.advance()

            if self.current_tok.type != TT_IDENTIFIER:
                return res.failure(InvalidSyntaxError(
                    self.current_tok.pos_start, self.current_tok.pos_end,
                    f"Expected identifier"
                ))

            arg_name_toks.append(self.current_tok)
            res.register_advancement()
            self.advance()
        if self.current_tok.type != TT_RPAREN:
            return res.failure(InvalidSyntaxError(
            self.current_tok.pos_start, self.current_tok.pos_end,
            f"Expected ',' or ')'"
            ))
        else:
            if self.current_tok.type != TT_RPAREN:
                return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                f"Expected identifier or ')'"
                ))

        res.register_advancement()
        self.advance()

        if self.current_tok.type == TT_LBRACE:
            res.register_advancement()
            self.advance()

            body = res.register(self.statements())
            if res.error: return res

            return res.success(FuncDefNode(
                var_name_tok,
                arg_name_toks,
                body,
                True
            ))
        
        if self.current_tok.type != TT_NEWLINE:
            return res.failure(InvalidSyntaxError(
                self.current_tok.pos_start, self.current_tok.pos_end,
                "Expected '{' or NEWLINE"
            ))

        res.register_advancement()
        self.advance()

        body = res.register(self.statements())
        if res.error: return res

        res.register_advancement()
        self.advance()
        
        return res.success(FuncDefNode(
        var_name_tok,
        arg_name_toks,
        body, False
        ))
{% /tab %}
{% /code %}

Parses **function definitions** (`PROCEDURE`). It handles the function name, parameters within parentheses, and the function body (either in curly braces or separated by newlines).

---

### Helper Method for Binary Operations

#### `bin_op(self, func_a, ops, func_b=None)`

{% code %}
{% tab language="python" %}
def bin_op(self, func_a, ops, func_b=None):
        if func_b == None:
            func_b = func_a
        
        res = ParseResult()
        left = res.register(func_a())
        if res.error: return res

        while self.current_tok.type in ops or (self.current_tok.type, self.current_tok.value) in ops:
            op_tok = self.current_tok
            res.register_advancement()
            self.advance()
            right = res.register(func_b())
            if res.error: return res
            left = BinOpNode(left, op_tok, right)

        return res.success(left)
{% /tab %}
{% /code %}

A generic helper method for parsing **binary operations** (operations with two operands and one operator in between). It takes two parsing functions (`func_a` and `func_b` for the left and right sides of the operation) and a list of `ops` (operator tokens) to handle. This method is crucial for implementing operator precedence by being called from other parsing methods (e.g., `term`, `arith_expr`, `comp_expr`).

---