---
type: page
title: The Lexer
listed: true
slug: the-lexer
description: 
index_title: The Lexer
hidden: 
keywords: 
tags: 
---
# Lexer

The `Lexer` class is responsible for taking a string of source code and breaking it down into a stream of tokens. This process is often called "lexical analysis" or "scanning." Each token represents a fundamental building block of the language, such as numbers, identifiers, operators, keywords, and so on.

## Class: `Lexer`

### `__init__(self, fn, text)`

**Purpose:** Initializes the Lexer object.

**Parameters:**
* `fn`: The filename associated with the input text. This is typically used for error reporting to indicate where an error occurred.
* `text`: The entire string of source code to be tokenized.

**Explanation:**
This constructor sets up the initial state of the lexer:
* `self.fn = fn`: Stores the filename.
* `self.text = text`: Stores the input source code.
* `self.pos = Position(-1, 0, -1, fn, text)`: Initializes a `Position` object (presumably from another module) to keep track of the current position within the input text. It starts at `(-1, 0, -1)` to ensure that the first `advance()` call correctly places it at the beginning of the text.
* `self.current_char = None`: Initializes the `current_char` to `None`.
* `self.advance()`: Calls the `advance` method to set `self.current_char` to the first character of the input `text`.

### `advance(self)`

**Purpose:** Moves the lexer's position to the next character in the input text and updates `self.current_char`.

**Explanation:**
* `self.pos.advance(self.current_char)`: Calls the `advance` method of the `Position` object. This method is responsible for updating the `idx` (index), `ln` (line number), and `col` (column number) based on the `current_char`.
* `self.current_char = self.text[self.pos.idx] if self.pos.idx < len(self.text) else None`: Updates `self.current_char` to the character at the new `self.pos.idx`. If the `idx` goes beyond the length of the text, it sets `current_char` to `None`, signaling the end of the input.

### `make_tokens(self)`

**Purpose:** The core method of the lexer that iterates through the input text and generates a list of tokens.

**Explanation:**
This method implements the main tokenization logic. It uses a `while` loop to process characters until `self.current_char` becomes `None` (end of file).

Inside the loop, it checks the `self.current_char` against various conditions to identify different token types:

* `if self.current_char in ' \t': self.advance()`: If the current character is a space or a tab, it's ignored, and the lexer advances to the next character. This handles whitespace.
* `elif self.current_char == "#": self.skip_comment()`: If the current character is `#`, it indicates a single-line comment. The `skip_comment` method is called to consume the rest of the line.
* `elif self.current_char in";\n": tokens.append(Token(TT_NEWLINE, pos_start=self.pos)); self.advance()`: If the character is a semicolon or a newline, a `TT_NEWLINE` token is created and appended to the `tokens` list. This indicates the end of a statement or line.
* `elif self.current_char in DIGITS: tokens.append(self.make_number())`: If the character is a digit, it calls `make_number()` to parse a number (integer or float) and appends the resulting token.
* `elif self.current_char == '"' or self.current_char == ".": tokens.append(self.make_string())`: If the character is a double quote or a period, it calls `make_string()` to parse a string literal.
    * **Note:** The comment `# IF is not the same as "IF"` is a good reminder that string literals are distinct from keywords or identifiers. The `.` for string might be an unusual design choice or a specific language feature.
* `elif self.current_char in LETTERS:`: This block handles identifiers and keywords.
    * **Special Handling for `R` (likely for `RUN` and `REPEAT`):**
        * `if self.current_char == 'R':`: This is a very specific and somewhat "janky" (as indicated by the comment) way to optimize or pre-check for keywords starting with 'R', namely `RUN` and `REPEAT`.
        * The code then checks the next characters to determine if it's "RUN" or "REPEAT". If it's "RU", it expects "N" for "RUN". If it's "RE", it expects "PEAT" for "REPEAT".
        * It uses "bypass" arguments to `make_identifier` to inject specific keyword tokens (`"RUN FUNC BYPASS"`, `"WHILE_LOOP_BYPASS"`, `"FOR_LOOP_BYPASS"`, `"FOR_LOOP_IDENITIFER_BYPASS"`) directly into the token stream based on these checks. This approach seems to tightly couple the lexer with specific keyword structures, which might be less flexible than a more general keyword lookup.
        * It also handles `RETURN` within this `R` check.
    * `else: tokens.append(self.make_identifier())`: If it's not one of the specially handled 'R' keywords, it calls `make_identifier()` to parse a general identifier or another keyword.
        * The nested `if self.current_char == '[':` block after `make_identifier()` suggests handling array-like indexing immediately after an identifier. This seems like a parser-level concern rather than a pure lexer concern (the lexer typically just outputs `TT_IDENTIFIER`, `TT_LSQUARE`, `TT_NUMBER`, `TT_RSQUARE` separately).
* **Operator and Punctuation Handling:**
    * `elif self.current_char == '<':`: Handles less than (`<`) and less than or equal to (`<=`) by calling `make_less_than()`.
    * `elif self.current_char == '>':`: Handles greater than (`>`) and greater than or equal to (`>=`) by calling `make_greater_than()`.
    * `elif self.current_char == '{':`, `elif self.current_char == '}':`: Handles curly braces. The `self.advance()` called twice for `}` seems like a potential bug or a very specific language rule.
    * `elif self.current_char == '+':`, `elif self.current_char == '-':`, `elif self.current_char == '*':`, `elif self.current_char == '/':`, `elif self.current_char == '^':`: Handles basic arithmetic operators.
    * `elif self.current_char == '(':`, `elif self.current_char == ')':`: Handles parentheses.
    * `elif self.current_char == ']':`: Handles right square bracket. The nested logic for `[` and `make_number()` again suggests parser-level logic mixed in.
    * `elif self.current_char == '[':`: Handles left square bracket.
    * `elif self.current_char == '!':`: Handles `!=` (not equals) by calling `make_not_equals()`.
    * `elif self.current_char == ',':`: Handles commas.
* `else: return [], IllegalCharError(...)`: If none of the above conditions match, it means an unrecognized or illegal character has been encountered, and an `IllegalCharError` is returned.

Finally, after the loop:
* `tokens.append(Token(TT_EOF, pos_start=self.pos))`: Appends an "End Of File" token, which is crucial for the parser to know when the input stream has ended.
* `tokens = [i for i in tokens if i is not None]`: Filters out any `None` values from the `tokens` list. This might occur if some `make_*` methods occasionally return `None`.
* `return tokens, None`: Returns the list of tokens and `None` for no error.

### `make_number(self)`

**Purpose:** Parses a sequence of digits and optional decimal points to form an integer or float token.

**Explanation:**
* `num_str = ''`: Initializes an empty string to build the number.
* `if IN_REPEAT_LOOP == True: ...`: This seems to be a global flag (or at least a module-level one) `IN_REPEAT_LOOP`. If true, it forces `num_str` to '0' and then calls `CHANGE_IN_REPEAT_LOOP()`. This is highly unusual for a lexer and suggests a hack to handle loop iteration variables directly within the lexer, which is a very bad design practice. A lexer should be context-free; parsing loop constructs belongs to the parser.
* `dot_count = 0`: Tracks the number of decimal points encountered to ensure only one is allowed for a valid float.
* `while self.current_char != None and self.current_char in DIGITS + '.':`: Loop as long as the current character is a digit or a dot.
    * If `self.current_char == '.'`: Increments `dot_count`. If a second dot is found, it breaks the loop (e.g., `1.2.3` would parse as `1.2`).
    * Appends the `current_char` to `num_str`.
    * `self.advance()`: Moves to the next character.
* `if self.current_char != None and self.current_char in DIGITS + '.': num_str += self.current_char; self.advance()`: This line after the loop seems redundant if the loop condition is robust, as the last digit/dot should already be captured. It might be a small bug or a leftover from a previous iteration.
* `if dot_count == 0:`: If no decimal points were found, it returns an `TT_INT` token with an integer value.
* `else:`: Otherwise, it returns an `TT_FLOAT` token with a float value.

### `make_string(self)`

**Purpose:** Parses a string literal, handling escape sequences.

**Explanation:**
* `string = ""`: Initializes an empty string to build the literal value.
* `pos_start = self.pos.copy()`: Records the starting position of the string.
* `escape_character = False`: Flag to indicate if the previous character was a backslash, signaling an escape sequence.
* `self.advance()`: Consumes the opening double quote.
* `escape_characters = {'n': '\n', 't': '\t'}`: Defines common escape sequences.
* `while self.current_char != None and (self.current_char != '"' or escape_character):`: Loops until the end of the string (`"`) is found, or if an escape character is being processed.
    * `if escape_character:`: If the previous character was `\`, the current character is treated as part of an escape sequence. It tries to get the corresponding special character from `escape_characters`, otherwise uses the character itself (e.g., `\\` becomes `\`).
    * `else:`: If not an escape sequence, checks for `\` to set `escape_character` for the next iteration. Otherwise, appends the `current_char` directly to `string`.
    * `self.advance()`: Moves to the next character.
* `self.advance()`: Consumes the closing double quote.
* `return Token(TT_STRING, string, pos_start, self.pos)`: Returns a `TT_STRING` token with the parsed string value.

### `make_identifier(self, bypass = None)`

**Purpose:** Parses a sequence of letters, digits, and underscores to form an identifier or a keyword.

**Parameters:**
* `bypass`: An optional string used for internal bypasses, allowing the lexer to inject specific keyword tokens directly without a full string match. This is a very unusual and potentially problematic design choice for a lexer, as it introduces context-dependency.

**Explanation:**
* `pos_start = self.pos.copy()`: Records the starting position.
* **Bypass Logic:** This section is highly unconventional for a lexer. It explicitly checks for various "bypass" strings (`"ELSE BYPASS"`, `"RUN FUNC BYPASS"`, etc.) and returns pre-defined `Token` objects. This bypasses the normal identifier/keyword parsing logic. The `CHANGE_IN_REPEAT_LOOP()` call within the "FOR_LOOP_BYPASS" also points to global state modification, which is generally undesirable in a lexer.
* `id_str = ''`: Initializes an empty string to build the identifier.
* `while self.current_char != None and self.current_char in LETTERS_DIGITS + '_':`: Loops as long as the current character is a letter, digit, or underscore.
    * Appends the `current_char` to `id_str`.
    * `self.advance()`: Moves to the next character.
* `if id_str == "": return None`: If no characters were collected, it means it wasn't an identifier, so it returns `None`.
* `if id_str not in KEYWORDS:`: Checks if the collected string is present in a global `KEYWORDS` list (presumably defined elsewhere). If not, it's an identifier.
    * `return Token(TT_IDENTIFIER, id_str, pos_start, self.pos)`: Returns a `TT_IDENTIFIER` token.
* `else:`: If it is in `KEYWORDS`, it's a keyword.
    * `return Token(TT_KEYWORD, id_str, pos_start, self.pos)`: Returns a `TT_KEYWORD` token.

### `make_not_equals(self)`

**Purpose:** Parses the `!=` (not equals) operator.

**Explanation:**
* `pos_start = self.pos.copy()`: Records the starting position.
* `self.advance()`: Consumes the `!`.
* `if self.current_char == '=':`: Checks if the next character is `=`.
    * `self.advance()`: Consumes the `=`.
    * `return Token(TT_NE, pos_start=pos_start, pos_end=self.pos), None`: Returns a `TT_NE` token.
* `self.advance()`: This `self.advance()` outside the `if` is problematic; if the character after `!` is not `=`, it will advance past it without being part of the `!=` token. This would cause issues. It should likely be an error.
* `return None, ExpectedCharError(...)`: Returns an error if `!` is not followed by `=`.

### `make_equals(self)`

**Purpose:** Parses the `==` (equals) operator.

**Explanation:**
* `pos_start = self.pos.copy()`: Records the starting position.
* `return Token(TT_EE,pos_start = pos_start, pos_end = self.pos)`: Returns a `TT_EE` token. This method *assumes* that the `self.current_char` has already been advanced past the first `=` by the `make_tokens` method.

### `make_less_than(self)`

**Purpose:** Parses the `<` (less than) or `<=` (less than or equal to) operators.

**Explanation:**
* `tok_type = TT_LT`: Initializes the token type to less than.
* `pos_start = self.pos.copy()`: Records the starting position.
* `if self.current_char == '=':`: Checks if the next character is `=`.
    * `self.advance()`: Consumes the `=`.
    * `tok_type = TT_LTE`: Changes the token type to less than or equal to.
* `return Token(tok_type, pos_start=pos_start, pos_end=self.pos)`: Returns the appropriate token. Note that the initial `<` is consumed by the caller (`make_tokens`) before calling this function, or there's an `advance()` missing here. Based on `make_greater_than`, it appears `self.advance()` is indeed missing after `pos_start = self.pos.copy()` to consume the `<` itself.

### `make_greater_than(self)`

**Purpose:** Parses the `>` (greater than) or `>=` (greater than or equal to) operators.

**Explanation:**
* `tok_type = TT_GT`: Initializes the token type to greater than.
* `pos_start = self.pos.copy()`: Records the starting position.
* `self.advance()`: Consumes the `>`.
* `if self.current_char == '=':`: Checks if the next character is `=`.
    * `self.advance()`: Consumes the `=`.
    * `tok_type = TT_GTE`: Changes the token type to greater than or equal to.
* `return Token(tok_type, pos_start=pos_start, pos_end=self.pos)`: Returns the appropriate token.

### `skip_comment(self)`

**Purpose:** Skips characters until a newline character is encountered, effectively ignoring a single-line comment.

**Explanation:**
* `self.advance()`: Consumes the initial `#` character.
* `while self.current_char != '\n': self.advance()`: Loops, consuming characters, as long as the current character is not a newline.
* `self.advance()`: Consumes the newline character itself, moving the lexer to the beginning of the next line.
