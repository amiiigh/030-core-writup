#030-core

provided files : 
- a.out
- core

firts lets look at string results:
> **note :** Yes!! I call my laptop Honey :D
```
amiiigh@Honey:~/030-core-writup$ strings a.out 
/lib64/ld-linux-x86-64.so.2
libc.so.6
fopen
putchar
fgetc
fclose
malloc
__libc_start_main
__gmon_start__
GLIBC_2.2.5
UH-X
AWAVA
AUATL
[]A\A]A^A_
flag
;*3$"
GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.5) 5.4.0 20160609

```

ok! there is an interesting string "flag" in the binary file.
lets check out core file 

```
amiiigh@Honey:~/030-core-writup$ strings core
CORE
CORE
a.out
./a.out 
IGISCORE
CORE
ELIFCORE
/home/gannimo/repos/b00tc4mp/forensics/030-core/a.out
/home/gannimo/repos/b00tc4mp/forensics/030-core/a.out
/home/gannimo/repos/b00tc4mp/forensics/030-core/a.out
/lib/x86_64-linux-gnu/libc-2.23.so
/lib/x86_64-linux-gnu/libc-2.23.so
/lib/x86_64-linux-gnu/libc-2.23.so
/lib/x86_64-linux-gnu/libc-2.23.so
/lib/x86_64-linux-gnu/ld-2.23.so
/lib/x86_64-linux-gnu/ld-2.23.so
/lib/x86_64-linux-gnu/ld-2.23.so
CORE
////////////////
LINUX
////////////////
/lib64/ld-linux-x86-64.so.2

```
nothing here! just a path ... at least we find challenge name  **030-core** and it's in category of forensics

ok it's forensics. 

lets see what we got :

```
amiiigh@Honey:~/030-core-writup$ file a.out 
a.out: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=514c6507019ec92535f61c9956a284df2e435ed6, not stripped
```

so we obviously have an executable file and luckily it's not stripped!

lets run this 
> **Warning:**  don't try this at home. lol kiddin

```
amiiigh@Honey:~/030-core-writup$ ./a.out 
Segmentation fault (core dumped)
```

Hmm... Ok fine !
lets look at our exe file.
I'm using [Radare2](https://github.com/radare/radare2) for analyzing the file (because it's more readable) and GDB for debugging it.


```
amiiigh@Honey:~/Documents/030-core-writup$ r2 a.out 
[0x00400550]> aa
[0x00400550]> pdf@sym.main 
/ (fcn) sym.main 242
|           0x00400646    55           push rbp
|           0x00400647    4889e5       mov rbp, rsp
|           0x0040064a    4883ec30     sub rsp, 0x30
|           0x0040064e    bec4074000   mov esi, 0x4007c4
|           0x00400653    bfc6074000   mov edi, str.flag
|           0x00400658    e8d3feffff   call sym.imp.fopen
|              sym.imp.fopen(unk)
|           0x0040065d    488945f0     mov [rbp-0x10], rax
|       ,=< 0x00400661    eb6a         jmp 0x4006cd ; (fcn.0040061c)
|   .-----> 0x00400663    bf10000000   mov edi, 0x10
|- fcn.004006cd 208
|   |   |   0x00400668    e8b3feffff   call sym.imp.malloc
|   |   |      sym.imp.malloc()
|   |   |   0x0040066d    488945f8     mov [rbp-0x8], rax
|   |   |   ; DATA XREF from 0x00401060 (unk)
|   |   |   0x00400671    488b05e8092. mov rax, [rip+0x2009e8] ; 0x00401060 
|   |   |   0x00400678    488945e0     mov [rbp-0x20], rax
|   |   |   0x0040067c    488b45f8     mov rax, [rbp-0x8]
|   |   |   0x00400680    48c70000000. mov qword [rax], 0x0
|   |   |   0x00400687    0fbe55df     movsx edx, byte [rbp-0x21]
|   |   |   0x0040068b    488b45f8     mov rax, [rbp-0x8]
|   |   |   0x0040068f    895008       mov [rax+0x8], edx
|   |   |   ; DATA XREF from 0x00401060 (unk)
|   |   |   0x00400692    488b05c7092. mov rax, [rip+0x2009c7] ; 0x00401060 
|   |   |   0x00400699    4885c0       test rax, rax
|   |  ,==< 0x0040069c    7518         jnz 0x4006b6
|   |  ||   0x0040069e    488b45f8     mov rax, [rbp-0x8]
|   |  ||   0x004006a2    488905b7092. mov [rip+0x2009b7], rax
|   | ,===< 0x004006a9    eb22         jmp 0x4006cd ; (fcn.0040061c)
|   |.----> 0x004006ab    488b45e0     mov rax, [rbp-0x20]
|   |||||   0x004006af    488b00       mov rax, [rax]
|   |||||   0x004006b2    488945e0     mov [rbp-0x20], rax
|   |||`--> 0x004006b6    488b45e0     mov rax, [rbp-0x20]
|   ||| |   0x004006ba    488b00       mov rax, [rax]
|   ||| |   0x004006bd    4885c0       test rax, rax
|   |`====< 0x004006c0    75e9         jnz 0x4006ab
|   | | |   0x004006c2    488b45e0     mov rax, [rbp-0x20]
|   | | |   0x004006c6    488b55f8     mov rdx, [rbp-0x8]
|   | | |   0x004006ca    488910       mov [rax], rdx
|   | | |   ; CODE (CALL) XREF from 0x00400661 (fcn.0040061c)
|   | | |   ; CODE (CALL) XREF from 0x004006a9 (fcn.0040061c)
|   | `-`-> 0x004006cd    488b45f0     mov rax, [rbp-0x10]
|   |       0x004006d1    4889c7       mov rdi, rax
|   |       0x004006d4    e827feffff   call sym.imp.fgetc
|   |          sym.imp.fgetc()
|   |       0x004006d9    8845df       mov [rbp-0x21], al
|   |       0x004006dc    807ddfff     cmp byte [rbp-0x21], 0xff
|   `=====< 0x004006e0    7581         jnz 0x400663
|           0x004006e2    488b45f0     mov rax, [rbp-0x10]
|           0x004006e6    4889c7       mov rdi, rax
|           0x004006e9    e802feffff   call sym.imp.fclose
|              sym.imp.fclose()
|           0x004006ee    b800000000   mov eax, 0x0
|           0x004006f3    488b00       mov rax, [rax]
|           0x004006f6    488945e8     mov [rbp-0x18], rax
|           ; DATA XREF from 0x00401060 (unk)
|           0x004006fa    488b055f092. mov rax, [rip+0x20095f] ; 0x00401060 
|           0x00400701    488945e8     mov [rbp-0x18], rax
|  ,======< 0x00400705    eb19         jmp 0x400720 ; (fcn.0040061c)
| .-------> 0x00400707    488b45e8     mov rax, [rbp-0x18]
|- fcn.00400720 45
| ||        0x0040070b    8b4008       mov eax, [rax+0x8]
| ||        0x0040070e    89c7         mov edi, eax
| ||        ; CODE (CALL) XREF from 0x004004e0 (fcn.004004dc)
| ||        0x00400710    e8cbfdffff   call sym.imp.putchar
| ||           sym.imp.putchar()
| ||        0x00400715    488b45e8     mov rax, [rbp-0x18]
| ||        0x00400719    488b00       mov rax, [rax]
| ||        0x0040071c    488945e8     mov [rbp-0x18], rax
| ||        ; CODE (CALL) XREF from 0x00400705 (fcn.0040061c)
| |`------> 0x00400720    48837de800   cmp qword [rbp-0x18], 0x0
| `=======< 0x00400725    75e0         jnz 0x400707
|           0x00400727    bf0a000000   mov edi, 0xa
|           0x0040072c    e8affdffff   call sym.imp.putchar
|              sym.imp.putchar()
|           0x00400731    b800000000   mov eax, 0x0
|           0x00400736    c9           leave
\           0x00400737    c3           ret
[0x00400550]> 

```

looks like the program try to open a file with name of 'flag' and print it's output.
but whats wrong ?!

so first of all its try to open a file that does not exist!
so lets make one and track the program.
```
amiiigh@Honey:~/030-core-writup$ echo 'im not a flag' > flag
```

lets run it again:

```
amiiigh@Honey:~/Documents/030-core-writup$ ./a.out 
Segmentation fault (core dumped)
```
Ah!
ok there is still something wrong with this file!
lets debug it with our friend gdb:

```
amiiigh@Honey:~/Documents/030-core-writup$ gdb a.out core -q
Reading symbols from a.out...(no debugging symbols found)...done.
[New LWP 23866]
Core was generated by `./a.out'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x00000000004006f3 in main ()
(gdb) 
```

I also pass core file to gdb to see what was wrong ?
looks like we had problem at address of **0x00000000004006f3**
lets put a breakpoint there :
```
   0x00000000004006e9 <+163>:	callq  0x4004f0 <fclose@plt>
   0x00000000004006ee <+168>:	mov    $0x0,%eax
=> 0x00000000004006f3 <+173>:	mov    (%rax),%rax
   0x00000000004006f6 <+176>:	mov    %rax,-0x18(%rbp)
```
our program crashes here! Why ? lets see 

```
(gdb) i r rax
rax            0x0	0
```
so we are trying to write something at 0x0, WE CAN NOT !
lets jump this line :

```
(gdb) b *0x00000000004006f6
Breakpoint 2 at 0x4006f6
(gdb) jump *0x00000000004006f6
Continuing at 0x4006f6.

Breakpoint 2, 0x00000000004006f6 in main ()
(gdb) c
Continuing.
im not a flag

[Inferior 1 (process 4210) exited normally]
```
Yeah ! we successfully ran the program.
program printed our input in flag file.

So whats the point ?
nothing ! there is nothing in a.out 
we should investigate the core file 

The original flag file was beside the a.out file when the core dump generated! so the flag data has to be in core file.

I tried to find location of data by finding the exact address and came up with 0x603250 but I couldn't use that address anywhere.

so I lets try objdump core file:
```
amiiigh@Honey:~/Documents/030-core-writup$ objdump -s core |less
```

after scrolling enough :)) I saw the flag .

```
 1c2e240 40120000 00000000 20000000 00000000  @....... .......
 1c2e250 70e2c201 00000000 62000000 00000000  p.......b.......
 1c2e260 00000000 00000000 21000000 00000000  ........!.......
 1c2e270 90e2c201 00000000 30000000 00000000  ........0.......
 1c2e280 00000000 00000000 21000000 00000000  ........!.......
 1c2e290 b0e2c201 00000000 63000000 00000000  ........c.......
 1c2e2a0 00000000 00000000 21000000 00000000  ........!.......
 1c2e2b0 d0e2c201 00000000 74000000 00000000  ........t.......
 1c2e2c0 00000000 00000000 21000000 00000000  ........!.......
 1c2e2d0 f0e2c201 00000000 66000000 00000000  ........f.......
 1c2e2e0 00000000 00000000 21000000 00000000  ........!.......
 1c2e2f0 10e3c201 00000000 7b000000 00000000  ........{.......
 1c2e300 00000000 00000000 21000000 00000000  ........!.......
 1c2e310 30e3c201 00000000 43000000 00000000  0.......C.......
 1c2e320 00000000 00000000 21000000 00000000  ........!.......
 1c2e330 50e3c201 00000000 6f000000 00000000  P.......o.......
 1c2e340 00000000 00000000 21000000 00000000  ........!.......
 1c2e350 70e3c201 00000000 6e000000 00000000  p.......n.......
 1c2e360 00000000 00000000 21000000 00000000  ........!.......
 1c2e370 90e3c201 00000000 67000000 00000000  ........g.......
 1c2e380 00000000 00000000 21000000 00000000  ........!.......
 1c2e390 b0e3c201 00000000 72000000 00000000  ........r.......
 1c2e3a0 00000000 00000000 21000000 00000000  ........!.......
 1c2e3b0 d0e3c201 00000000 61000000 00000000  ........a.......
 1c2e3c0 00000000 00000000 21000000 00000000  ........!.......
 1c2e3d0 f0e3c201 00000000 74000000 00000000  ........t.......
 1c2e3e0 00000000 00000000 21000000 00000000  ........!.......
 1c2e3f0 10e4c201 00000000 73000000 00000000  ........s.......
 1c2e400 00000000 00000000 21000000 00000000  ........!.......
 1c2e410 30e4c201 00000000 2c000000 00000000  0.......,.......
 1c2e420 00000000 00000000 21000000 00000000  ........!.......
 1c2e430 50e4c201 00000000 20000000 00000000  P....... .......
 1c2e440 00000000 00000000 21000000 00000000  ........!.......
 1c2e450 70e4c201 00000000 6d000000 00000000  p.......m.......
 1c2e460 00000000 00000000 21000000 00000000  ........!.......
 1c2e470 90e4c201 00000000 65000000 00000000  ........e.......
 1c2e480 00000000 00000000 21000000 00000000  ........!.......
 1c2e490 b0e4c201 00000000 6d000000 00000000  ........m.......
 1c2e4a0 00000000 00000000 21000000 00000000  ........!.......
 1c2e4b0 d0e4c201 00000000 6f000000 00000000  ........o.......
 1c2e4c0 00000000 00000000 21000000 00000000  ........!.......
 1c2e4d0 f0e4c201 00000000 72000000 00000000  ........r.......
 1c2e4e0 00000000 00000000 21000000 00000000  ........!.......
 1c2e4f0 10e5c201 00000000 79000000 00000000  ........y.......
 1c2e500 00000000 00000000 21000000 00000000  ........!.......
 1c2e510 30e5c201 00000000 20000000 00000000  0....... .......
 1c2e520 00000000 00000000 21000000 00000000  ........!.......
 1c2e530 50e5c201 00000000 64000000 00000000  P.......d.......
 1c2e540 00000000 00000000 21000000 00000000  ........!.......
 1c2e550 70e5c201 00000000 75000000 00000000  p.......u.......
 1c2e560 00000000 00000000 21000000 00000000  ........!.......
 1c2e570 90e5c201 00000000 6d000000 00000000  ........m.......
 1c2e580 00000000 00000000 21000000 00000000  ........!.......
 1c2e590 b0e5c201 00000000 70000000 00000000  ........p.......
 1c2e5a0 00000000 00000000 21000000 00000000  ........!.......
 1c2e5b0 d0e5c201 00000000 73000000 00000000  ........s.......
 1c2e5c0 00000000 00000000 21000000 00000000  ........!.......
 1c2e5d0 f0e5c201 00000000 20000000 00000000  ........ .......
 1c2e5e0 00000000 00000000 21000000 00000000  ........!.......
 1c2e5f0 10e6c201 00000000 61000000 00000000  ........a.......
 1c2e600 00000000 00000000 21000000 00000000  ........!.......
 1c2e610 30e6c201 00000000 72000000 00000000  0.......r.......
 1c2e620 00000000 00000000 21000000 00000000  ........!.......
 1c2e630 50e6c201 00000000 65000000 00000000  P.......e.......
 1c2e640 00000000 00000000 21000000 00000000  ........!.......
 1c2e650 70e6c201 00000000 20000000 00000000  p....... .......
 1c2e660 00000000 00000000 21000000 00000000  ........!.......
 1c2e670 90e6c201 00000000 66000000 00000000  ........f.......
 1c2e680 00000000 00000000 21000000 00000000  ........!.......
 1c2e690 b0e6c201 00000000 75000000 00000000  ........u.......
 1c2e6a0 00000000 00000000 21000000 00000000  ........!.......
 1c2e6b0 d0e6c201 00000000 6e000000 00000000  ........n.......
 1c2e6c0 00000000 00000000 21000000 00000000  ........!.......
 1c2e6d0 f0e6c201 00000000 7d000000 00000000  ........}.......
 1c2e6e0 00000000 00000000 21000000 00000000  ........!.......
 1c2e6f0 00000000 00000000 0a000000 00000000  ................
```
and it's **b!0!c!t!f!{!C!o!n!g!r!a!t!s!,! !m!e!m!o!r!y! !d!u!m!p!s! !a!r!e! !f!u!n!}!**
I think '!' are not part of the answer so the flag is : 
b0ctf{Congrats, memory dumps are fun}

I think there must be a better way to solve this challenge but does not matter now! we have the flag :))



