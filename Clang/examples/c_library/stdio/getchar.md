---
title: 'getchar()'
date: '2022-02-16'
categories:
 - clang
publish: true
---

## 1. Description

The C library function `int getchar()` gets a character (an unsigned char) from `stdin`. This is equivalent to `getc` with `stdin` as its argument. 

## 2. Declaration

```c
int getchar(void)
```

## 3. Parameters

N/A

## 4. Return Value

This function returns the character read as an unsigned char cast to an int or `EOF` on end of file or error.

## 5. Example

```c
// Exmaple: c library function - getchar()
#include <stdio.h>

int main() {
    char c;

    printf("Enter character:");
    c = getchar();

    printf("Character entered:");
    putchar(c);

    return(0);
}
```

We can use `return expression` or `return(expression)`.

## Reference

1. [C library function - getchar()](https://www.tutorialspoint.com/c_standard_library/c_function_getchar.htm)
2. [return的用法详解](http://c.biancheng.net/view/1855.html)
