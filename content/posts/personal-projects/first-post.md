---
title: "First Post: Setting up the blog"
date: 2026-10-27
draft: true
tags: ["meta", "hugo"]
categories: ["personal-projects"]
---

First entry on the blog. Testing code blocks and styling.

```asm
; ft_strlen.s
global ft_strlen
section .text

ft_strlen:
    xor rax, rax
.loop:
    cmp byte [rdi + rax], 0
    je .end
    inc rax
    jmp .loop
.end:
    ret
```

```c
// main.c
#include <unistd.h>
size_t  ft_strlen(const char *s);

void    ft_putendl_fd(const char *s, int fd)
{
    write(fd, s, ft_strlen(s));
    write(fd, "\n", 1);
}

int     main(void)
{
    ft_putendl_fd("The quick brown fox jumps over the lazy dog.", 1);
    return (0);
}
```
This code should work (on Linux) by the way, you can compile it this way:
```sh
nasm -f elf64 ft_strlen.s -o ft_strlen.o
gcc -Wall -Werror -Wextra main.c ft_strlen.o
./a.out
# (or something close idk, didn't test
# I wrote all that to make an overkill test post)
```
