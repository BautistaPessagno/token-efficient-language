# token-efficient-language

TEL (Token Efficient Language) is a compact DSL that compiles to C, designed for token-efficient AI-assisted coding.

## Installation

```sh
npx skills add BautistaPessagno/token-efficient-language
```

## Usage

Once installed, Claude Code automatically uses the TEL skill when you:

- Ask to write, read, or convert TEL code
- Mention "TEL" or "Token Efficient Language"
- Work with `.tel` files

## Quick Reference

### Types

| TEL      | C                   |
| -------- | ------------------- |
| `uint`   | `unsigned int`      |
| `uli`    | `unsigned long int` |
| `v`      | `void`              |
| `c*`     | `char *`            |

### Control Flow

| TEL    | C                                              |
| ------ | ---------------------------------------------- |
| `fd i 0 n` | `for (int i = 0; i < n; i++)`              |
| `w`    | `while`                                        |
| `dw`   | `do { ... } while`                             |
| `sw`   | `switch`                                       |
| `elif` | `else if`                                      |
| `brk`  | `break`                                        |
| `cnt`  | `continue`                                     |
| `df`   | `default`                                      |

### Functions & Declarations

| TEL                        | C                            |
| -------------------------- | ---------------------------- |
| `fn name params -> type`   | function declaration         |
| `name:type = value`        | variable declaration         |
| `ret`                      | `return`                     |
| `main`                     | `int main(int argc, char* argv[])` |
| `str`                      | `struct`                     |
| `td`                       | `typedef`                    |
| `en`                       | `enum`                       |

Standard library includes (`<stdio.h>`, `<stdlib.h>`, `<string.h>`) are injected automatically when their functions are used.

## Example

**TEL:**

```tel
main
    printf("hello world")
```

**Compiled C:**

```c
#include <stdio.h>

int main(int argc, char * argv[]) {
    printf("hello world");
    return 0;
}
```
