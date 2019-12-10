>Papa brought me a packed present! let's open it.

>Download : http://pwnable.kr/bin/flag

>This is reversing task. all you need is binary

Useful commands when analyzing a program in linux:<br>
**file** and **strings**

```
lovehacker@kali:~/ctf$ file flag
flag: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
```

```
lovehacker@kali:~/ctf$ strings flag | grep UPX
UPX!<
$Info: This file is packed with the UPX executable packer http://upx.sf.net $
$Id: UPX 3.95 Copyright (C) 1996-2018 the UPX Team. All Rights Reserved. $
UPX!u
UPX!
UPX!
```

First of all it's a 64 bit file so you should solve this challenge with a 64 bit OS.

It says UPX for the strings command which can mean that the program is packed with UPX

```
lovehacker@kali:~/ctf$ upx -d flag
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2018
UPX 3.95        Markus Oberhumer, Laszlo Molnar & John Reiser   Aug 26th 2018

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    883745 <-    335632   37.98%   linux/amd64   flag

Unpacked 1 file.
```

Now you can disassemble this program with gdb

```
(gdb) disas main
Dump of assembler code for function main:
   0x0000000000401164 <+0>:     push   rbp
   0x0000000000401165 <+1>:     mov    rbp,rsp
   0x0000000000401168 <+4>:     sub    rsp,0x10
   0x000000000040116c <+8>:     mov    edi,0x496658
   0x0000000000401171 <+13>:    call   0x402080 <puts>
   0x0000000000401176 <+18>:    mov    edi,0x64
   0x000000000040117b <+23>:    call   0x4099d0 <malloc>
   0x0000000000401180 <+28>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000401184 <+32>:    mov    rdx,QWORD PTR [rip+0x2c0ee5]        # 0x6c2070 <flag>
   0x000000000040118b <+39>:    mov    rax,QWORD PTR [rbp-0x8]
   0x000000000040118f <+43>:    mov    rsi,rdx
   0x0000000000401192 <+46>:    mov    rdi,rax
   0x0000000000401195 <+49>:    call   0x400320
   0x000000000040119a <+54>:    mov    eax,0x0
   0x000000000040119f <+59>:    leave  
   0x00000000004011a0 <+60>:    ret    
End of assembler dump.
```
Hm..? It literally says where the flag is located

```
(gdb) x/s *0x6c2070
0x496628:       "UPX...? sounds like a delivery service :)"
(gdb) 

```
Feels weird but ok...? 

** Pwned **
