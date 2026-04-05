# TEL Grammar Reference

This file contains the complete formal grammar and edge-case documentation for TEL.

## Table of Contents

1. [Supported C Subset](#supported-c-subset)
2. [Keyword Mapping Table](#keyword-mapping-table)
3. [Declaration Syntax](#declaration-syntax)
4. [Control Flow Details](#control-flow-details)
5. [Preprocessor Rules](#preprocessor-rules)
6. [Implicit Include Rules](#implicit-include-rules)
7. [Struct, Enum, Union, Typedef](#struct-enum-union-typedef)
8. [Pointer and Array Syntax](#pointer-and-array-syntax)
9. [Reserved Words](#reserved-words)
10. [Error Conditions](#error-conditions)

---

## Supported C Subset

TEL covers the following C constructs:

- Primitive types: `int`, `float`, `double`, `char`, `void`
- Type qualifiers: `unsigned`, `long`, `short`, `const`
- Variables, arrays, basic pointers
- Functions (including function pointers)
- Control flow: `if`/`else`, `for`, `while`, `do-while`, `switch`/`case`
- `struct`, `enum`, `typedef`
- Preprocessor: `#include`, `#define`

---

## Keyword Mapping Table

### Types

| TEL      | C                   | Token savings                |
| -------- | ------------------- | ---------------------------- |
| `int`    | `int`               | 0 (already 1 token)          |
| `char`   | `char`              | 0                            |
| `float`  | `float`             | 0                            |
| `double` | `double`            | 0                            |
| `uint`   | `unsigned int`      | 1                            |
| `long`   | `long`              | 0                            |
| `v`      | `void`              | 0                            |
| `uli`    | `unsigned long int` | 2                            |
| `c*`     | `char *`            | shorthand in struct contexts |

### Control Flow

| TEL    | C           | Notes                                                     |
| ------ | ----------- | --------------------------------------------------------- |
| `if`   | `if`        | Unchanged (already 1 token)                               |
| `elif` | `else if`   | Combined keyword                                          |
| `else` | `else`      | Unchanged                                                 |
| `for`  | `for`       | General for loop                                          |
| `fd`   | default for | `fd VAR START END` → `for(int VAR=START; VAR<END; VAR++)` |
| `w`    | `while`     |                                                           |
| `dw`   | `do`        | Block follows, then `w condition`                         |
| `sw`   | `switch`    |                                                           |
| `N:`   | `case N:`   | Where N is a numeric literal                              |
| `df`   | `default`   |                                                           |
| `brk`  | `break`     |                                                           |
| `cnt`  | `continue`  |                                                           |

### Functions

| TEL   | C             | Notes                        |
| ----- | ------------- | ---------------------------- |
| `fn`  | function decl | `fn name params -> ret_type` |
| `ret` | `return`      | Explicit return              |
| `fn*` | function ptr  | `fn* name types -> type`     |

### Data Structures

| TEL   | C         |
| ----- | --------- |
| `str` | `struct`  |
| `en`  | `enum`    |
| `td`  | `typedef` |

### Other

| TEL    | C        |
| ------ | -------- |
| `stc`  | `static` |
| `null` | `NULL`   |

---

## Declaration Syntax

### Variables

```
name:type = value
```

Examples:

```tel
x:int = 5
name:char* = "hello"
pi:float = 3.14
count:uint = 0
```

### Functions

```
fn name param1:type1 param2:type2 -> return_type
    body
```

Void functions omit `-> type`:

```tel
fn greet name:char*
    printf("Hello %s\n", name)
```

### Function Pointers

```
fn* callback int int -> int
```

Translates to: `int (*callback)(int, int)`

---

## Control Flow Details

### If / Elif / Else

```tel
if x > 10
    printf("big")
elif x > 5
    printf("medium")
else
    printf("small")
```

`if` stays unchanged from C (already a single LLM token). Conditions do NOT use parentheses (they are optional in TEL but omitted for token efficiency). The block is determined by indentation.

### Default For (`fd`)

```tel
fd i 0 n
    // body using i
```

This is the most token-efficient construct in TEL. It covers the overwhelmingly common `for(int i=0; i<n; i++)` pattern.

Parameters: `fd VARIABLE START_VALUE END_VALUE`

- VARIABLE: loop counter name (declared as int automatically)
- START_VALUE: initial value (inclusive)
- END_VALUE: upper bound (exclusive, uses `<`)
- Increment is always `++`

### General For

For non-standard loops, use `for` with custom syntax:

```tel
for init; condition; update
    // body
```

### While

```tel
w count > 0
    count--
```

### Do-While

```tel
dw
    // body
w condition
```

The `dw` starts the block, and the `w condition` at the same indentation level closes it (like C's `} while(condition);`).

### Switch

```tel
sw choice
    1:
        printf("one")
        brk
    2:
        printf("two")
        brk
    df:
        printf("other")
```

Case labels use `N:` where N is a numeric or char literal. `df` replaces `default`.

---

## Preprocessor Rules

### #include

Standard library headers omit `<>` and `.h`:

```tel
#include stdio       →  #include <stdio.h>
#include stdlib      →  #include <stdlib.h>
#include string      →  #include <string.h>
#include math        →  #include <math.h>
```

Non-standard / user headers also omit `<>` and `.h` but are included with angle brackets:

```tel
#include listADT     →  #include <listADT.h>
```

### #define

Works exactly like C:

```tel
#define MAX 100
#define SQUARE(x) ((x)*(x))
```

---

## Implicit Include Rules

The TEL compiler automatically inserts includes when it detects usage of certain standard library functions:

| Functions detected                                                                                               | Auto-included header |
| ---------------------------------------------------------------------------------------------------------------- | -------------------- |
| `printf`, `scanf`, `puts`, `gets`, `fprintf`, `fscanf`, `fopen`, `fclose`, `fgets`, `fputs`, `sprintf`, `sscanf` | `<stdio.h>`          |
| `malloc`, `calloc`, `realloc`, `free`, `exit`, `atoi`, `atof`, `abs`, `rand`, `srand`                            | `<stdlib.h>`         |
| `strlen`, `strcpy`, `strncpy`, `strcat`, `strcmp`, `strncmp`, `memcpy`, `memset`, `memmove`                      | `<string.h>`         |

This means a minimal hello world in TEL needs no includes at all:

```tel
main
    printf("hello world")
```

---

## Struct, Enum, Union, Typedef

### Struct

```tel
str persona
    const c* nombre
    uint dni
    uli timestamp
```

→

```c
struct persona {
    const char * nombre;
    unsigned int dni;
    unsigned long int timestamp;
};
```

### Typedef + Struct

```tel
td str persona
    const c* nombre
    uint dni
persona
```

→

```c
typedef struct persona {
    const char * nombre;
    unsigned int dni;
} persona;
```

The alias name appears at the end of the block, at the same indentation level as `td`.

### Enum

```tel
en color
    RED
    GREEN
    BLUE
```

→

```c
enum color {
    RED,
    GREEN,
    BLUE
};
```

---

## Pointer and Array Syntax

### Pointers

Standard `*` syntax, same as C:

```tel
ptr:int* = &x
value:int = *ptr
```

### Arrays

Standard `[]` syntax:

```tel
arr:int[10]
matrix:int[3][3]
```

### Function Pointers

```tel
fn* compare int int -> int
```

→ `int (*compare)(int, int)`

---

## Reserved Words

The following identifiers are reserved in TEL and cannot be used as variable or function names:

`fn`, `ret`, `str`, `en`, `td`, `stc`, `fd`, `dw`, `sw`, `df`, `brk`, `cnt`, `uint`, `uli`, `v`, `null`, `w`, `main`, `elif`

---

## Error Conditions

The following are parse errors in TEL:

1. **Inconsistent indentation** — mixing tabs and spaces, or incorrect indent levels
2. **Reserved word as identifier** — using `fn`, `ret`, `str`, etc. as variable names
3. **Malformed expressions** — missing operators, unbalanced parentheses
4. **Invalid return type syntax** — omitting the type after `->` in a function declaration
5. **Unclosed string literals** — opening `"` without closing `"`
6. **Unclosed char literals** — opening `'` without closing `'`
7. **Missing `main` function** — every TEL program requires a `main` entry point
