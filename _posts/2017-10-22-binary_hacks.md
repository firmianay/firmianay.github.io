---
layout: post
title:  "二进制杂谈"
image: ''
date:   2017-10-22 00:06:31
tags:
- binary
description: ''
categories:
- binary
---

> 这里记录一些二进制方面很零碎的知识

- 调用 `printf@plt` 函数前的 `mov $0x0,%eax`。

一段简单的代码：
```c
#include<stdio.h>
int main() {
        int x = 0;
        printf("%d", x);
}
```
```
$ gdb -c hello.c -o a.o
```
```
gdb-peda$ disassemble main
Dump of assembler code for function main:
   0x0000000000000000 <+0>:     push   rbp
   0x0000000000000001 <+1>:     mov    rbp,rsp
   0x0000000000000004 <+4>:     sub    rsp,0x10
   0x0000000000000008 <+8>:     mov    DWORD PTR [rbp-0x4],0x0
   0x000000000000000f <+15>:    mov    eax,DWORD PTR [rbp-0x4]
   0x0000000000000012 <+18>:    mov    esi,eax
   0x0000000000000014 <+20>:    lea    rdi,[rip+0x0]        # 0x1b <main+27>
   0x000000000000001b <+27>:    mov    eax,0x0
   0x0000000000000020 <+32>:    call   0x25 <main+37>
   0x0000000000000025 <+37>:    mov    eax,0x0
   0x000000000000002a <+42>:    leave  
   0x000000000000002b <+43>:    ret    
End of assembler dump.
```
学弟的问题是为什么调用函数前要将 `rdi` 和 `eax` 置零。

对于 `rdi` 的问题，其实是没有理解 `.o` 文件和 `.out` 文件的不同。可重定位文件还没有经过链接阶段，无法获得共享库中的函数信息，所以 `lea    rdi,[rip+0x0]` 和下面的 `call   0x25 <main+37>` 都是预留位置用的，在链接后会被具体的函数替换：
```
$ gdb hello.c -o a.out
```
```
gdb-peda$ disassemble main
Dump of assembler code for function main:
   0x000000000000064a <+0>:     push   rbp
   0x000000000000064b <+1>:     mov    rbp,rsp
   0x000000000000064e <+4>:     sub    rsp,0x10
   0x0000000000000652 <+8>:     mov    DWORD PTR [rbp-0x4],0x0
   0x0000000000000659 <+15>:    mov    eax,DWORD PTR [rbp-0x4]
   0x000000000000065c <+18>:    mov    esi,eax
   0x000000000000065e <+20>:    lea    rdi,[rip+0x9f]        # 0x704
   0x0000000000000665 <+27>:    mov    eax,0x0
   0x000000000000066a <+32>:    call   0x530 <printf@plt>
   0x000000000000066f <+37>:    mov    eax,0x0
   0x0000000000000674 <+42>:    leave  
   0x0000000000000675 <+43>:    ret    
End of assembler dump.
```

至于 `eax` 的问题呢，就涉及到浮点数的处理。在 [x86_64 System V ABI](https://github.com/hjl-tools/x86-psABI/wiki/x86-64-psABI-r252.pdf) 中对于 `rax` 的描述是这样的：
```
Register    Usage
%rax        temporary register; with variable arguments
            passes information about the number of vector
            registers used; 1st return register
...
```
因为这里传递给 `printf` 的变量是个整数，所以 vector register 的数量为 0，如果我们把 `int x = 0;` 换成 `float x = 0;`，则：
```
gdb-peda$ disassemble main
Dump of assembler code for function main:
   0x0000000000000000 <+0>:     push   rbp
   0x0000000000000001 <+1>:     mov    rbp,rsp
   0x0000000000000004 <+4>:     sub    rsp,0x10
   0x0000000000000008 <+8>:     pxor   xmm0,xmm0
   0x000000000000000c <+12>:    movss  DWORD PTR [rbp-0x4],xmm0
   0x0000000000000011 <+17>:    cvtss2sd xmm0,DWORD PTR [rbp-0x4]
   0x0000000000000016 <+22>:    lea    rdi,[rip+0x0]        # 0x1d <main+29>
   0x000000000000001d <+29>:    mov    eax,0x1
   0x0000000000000022 <+34>:    call   0x27 <main+39>
   0x0000000000000027 <+39>:    mov    eax,0x0
   0x000000000000002c <+44>:    leave  
   0x000000000000002d <+45>:    ret    
End of assembler dump.
```
这时就 `mov    eax,0x1` 了。
