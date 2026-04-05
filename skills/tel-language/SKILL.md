---
name: tel-language
description: "Write, read, and understand TEL (Token Efficient Language) — a compact DSL that represents C programs using minimal tokens for LLM-efficient coding. Use this skill whenever the user asks to write code in TEL, convert C to TEL, convert TEL to C, review TEL code, explain TEL syntax, debug TEL programs, or mentions 'TEL', 'Token Efficient Language', or asks for token-efficient C code. Also trigger when the user wants to write C code but mentions reducing tokens, LLM optimization, or compact syntax. If a .tel file is involved in any way, use this skill."
metadata:
  author: BautistaPessagno
  version: "1.0"
  tags:
    - c
    - compiler
    - token-optimization
    - llm
---

# TEL — Token Efficient Language

TEL is a source-to-source compiled DSL that represents C programs in a compact syntax optimized for LLM token efficiency. A TEL program compiles to semantically equivalent, well-formed C. The transformation is bijective within the supported C subset.

When writing TEL, think of it as C with these core changes: indentation replaces braces, newlines replace semicolons, keywords are shortened, and common patterns get compact forms.

## Quick Reference

### Type Keywords

| TEL      | C equivalent        |
| -------- | ------------------- |
| `int`    | `int`               |
| `char`   | `char`              |
| `float`  | `float`             |
| `double` | `double`            |
| `uint`   | `unsigned int`      |
| `long`   | `long`              |
| `v`      | `void`              |
| `uli`    | `unsigned long int` |

Types that are already a single token in LLMs (like `int`, `char`, `float`, `double`, `long`) stay unchanged. Compound types that consume multiple tokens get shortened (`unsigned int` → `uint`, `unsigned long int` → `uli`).

### Control Flow Keywords

| TEL    | C equivalent                                        |
| ------ | --------------------------------------------------- |
| `if`   | `if`                                                |
| `elif` | `else if`                                           |
| `else` | `else`                                              |
| `for`  | `for`                                               |
| `fd`   | default for — `fd i 0 n` → `for(int i=0; i<n; i++)` |
| `w`    | `while`                                             |
| `dw`   | `do-while`                                          |
| `sw`   | `switch`                                            |
| `num:` | `case num:`                                         |
| `df`   | `default`                                           |
| `brk`  | `break`                                             |
| `cnt`  | `continue`                                          |

### Other Keywords

| TEL    | C equivalent                                            |
| ------ | ------------------------------------------------------- |
| `fn`   | function declaration (replaces return type before name) |
| `ret`  | `return`                                                |
| `str`  | `struct`                                                |
| `en`   | `enum`                                                  |
| `td`   | `typedef`                                               |
| `stc`  | `static`                                                |
| `null` | `NULL`                                                  |

### Operators

All C operators are preserved as-is:

- Arithmetic: `+`, `-`, `*`, `/`, `%`
- Relational: `==`, `!=`, `<`, `>`, `<=`, `>=`
- Assignment: `=`, `+=`, `-=`, `*=`, `/=`, `%=`
- Increment/decrement: `++`, `--`
- Logical: `&&`, `||`, `!`
- Bitwise: `&`, `|`, `^`, `~`, `<<`, `>>`

---

## Syntax Rules

### 1. No Semicolons

Statements end with a newline instead of `;`. To put multiple statements on one line, use `;` as a separator.

```
x:int = 5
y:int = 10
```

### 2. Indentation-Based Blocks

Blocks use indentation (like Python) instead of `{ }`. Every nested block increases the indent level.

```tel
if x > 0
    printf("positive")
elif x == 0
    printf("zero")
else
    printf("negative")
```

### 3. Function Declarations

Use `fn name params -> return_type`. If the function returns void, omit `-> type`. Use `ret` for return, or let the last expression be an implicit return if its type matches the return type.

```tel
fn sum numbers:int[] size:int -> int
    res:int = 0
    fd i 0 size
        res += numbers[i]
    res
```

Translates to:

```c
int sum(int numbers[], int size) {
    int res = 0;
    for (int i = 0; i < size; i++) {
        res += numbers[i];
    }
    return res;
}
```

### 4. Variable Declarations

Declare variables with `name:type = value`.

```tel
x:int = 5
name:char* = "hello"
```

### 5. Entry Point

Every TEL program must have a `main` function. Just write `main` — it translates to `int main(int argc, char* argv[])`.

```tel
main
    printf("hola mundo")
```

### 6. Default For Loop (`fd`)

The pattern `fd i 0 n` expands to `for(int i=0; i<n; i++)`. This covers the most common loop pattern in C.

### 7. Structs, Enums, Typedefs

```tel
td str persona
    const c* nombre
    uint dni
    uli timestamp_nacimiento
persona
```

Translates to:

```c
typedef struct persona {
    const char * nombre;
    unsigned int dni;
    unsigned long int timestamp_nacimiento;
} persona;
```

### 8. Preprocessor Directives

`#include` omits angle brackets and `.h` for standard headers:

```tel
#include stdio
#include string
#include stdlib
```

Translates to:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
```

Non-standard headers keep their full name: `#include listADT` → `#include <listADT.h>`.

`#define` works unchanged.

### 9. Implicit I/O Includes

If the program uses `printf`, `scanf`, `puts`, etc., the compiler auto-includes `<stdio.h>`. Same for `malloc`/`free` → `<stdlib.h>` and `strlen`/`strcpy` → `<string.h>`. No need to write these includes manually.

### 10. Function Pointers

Use `fn* name types -> type` instead of C's `type (*name)(types)`.

### 11. Comments

Line comments (`//`) and block comments (`/* */`) are preserved as-is in the output.

### 12. Literals

Supported literals: decimal integers, hex (`0x`), octal (`0o`), floats, chars (`'a'`), strings (`"hello"`), `null` → `NULL`.

---

## Complete Example

**TEL:**

```tel
main
    printf("hola mundo")
```

**Output C (27 tokens → 8 tokens):**

```c
#include <stdio.h>

int main(int argc, char * argv[]) {
    printf("hola mundo");
    return 0;
}
```

**TEL with function and loop:**

```tel
fn sum numbers:int[] size:int -> int
    res:int = 0
    fd i 0 size
        res += numbers[i]
    res
```

**Output C (45 tokens → 32 tokens):**

```c
int sum(int numbers[], int size) {
    int res = 0;
    for (int i = 0; i < size; i++) {
        res += numbers[i];
    }
    return res;
}
```

---

## Common Patterns

### If/Else Chain

```tel
if condition1
    // block
elif condition2
    // block
else
    // block
```

### While Loop

```tel
w condition
    // body
```

### Do-While

```tel
dw
    // body
w condition
```

### Switch

```tel
sw variable
    1:
        // case 1
        brk
    2:
        // case 2
        brk
    df:
        // default
```

### Enum

```tel
en color
    RED
    GREEN
    BLUE
```

---

## What TEL Does NOT Change

- Operator syntax (all C operators preserved)
- String and char literals
- Comments
- `#define` directives
- Array indexing `[]`
- Pointer dereference `*`
- Function call syntax `func(args)`
- `const` keyword

---

## Writing Guidelines for Agents

When writing TEL code:

1. **Never use semicolons** at the end of lines (unless separating multiple statements on one line).
2. **Never use braces** — use indentation for all blocks.
3. **Use `if` for conditionals** — it stays unchanged from C (already a single token).
4. **Use `fd` for standard counting loops** — it's the most common pattern and saves the most tokens.
5. **Omit I/O includes** — the compiler handles them automatically.
6. **Use `fn` for all function declarations**, with `-> type` for non-void returns.
7. **Use compact type aliases** — `uint`, `uli`, `v` — for multi-token C types.
8. **Use `ret` for explicit returns**, or rely on implicit return for the last expression.
9. **`main` stands alone** — no need for parameters or return type, it expands automatically.
10. **Indent consistently** — mixing tabs and spaces is a parse error.

For the full language grammar and edge cases, see `references/grammar.md`.
