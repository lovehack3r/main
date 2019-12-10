>Nana told me that buffer overflow is one of the most common software vulnerability. 
>Is that true?

>Download : http://pwnable.kr/bin/bof
>Download : http://pwnable.kr/bin/bof.c

>Running at : nc pwnable.kr 9000

First you download both of the files. For me, I used wget in order to download them in linux.

First bof.c file

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
        char overflowme[32];
        printf("overflow me : ");
        gets(overflowme);       // smash me!
        if(key == 0xcafebabe){
                system("/bin/sh");
        }
        else{
                printf("Nah..\n");
        }
}
int main(int argc, char* argv[]){
        func(0xdeadbeef);
        return 0;
}
```
Since the name of this challenge is bof you can know that the vulnerability comes from bof which if you look at gets it's obvious.
gets take up data without limits so we can access to other data parts.
So what we need to do here is that we need to access to where the variable key is and overwrite it as 0xcafebabe

In GDB the main includes the following
```
(gdb) disas main
Dump of assembler code for function main:
   0x0000068a <+0>:     push   ebp
   0x0000068b <+1>:     mov    ebp,esp
   0x0000068d <+3>:     and    esp,0xfffffff0
   0x00000690 <+6>:     sub    esp,0x10
   0x00000693 <+9>:     mov    DWORD PTR [esp],0xdeadbeef
   0x0000069a <+16>:    call   0x62c <func>
   0x0000069f <+21>:    mov    eax,0x0
   0x000006a4 <+26>:    leave  
   0x000006a5 <+27>:    ret    
End of assembler dump.
```
here we can see the fuction func

the func function includes the following
```
(gdb) disas func
Dump of assembler code for function func:
   0x0000062c <+0>:     push   ebp
   0x0000062d <+1>:     mov    ebp,esp
   0x0000062f <+3>:     sub    esp,0x48
   0x00000632 <+6>:     mov    eax,gs:0x14
   0x00000638 <+12>:    mov    DWORD PTR [ebp-0xc],eax
   0x0000063b <+15>:    xor    eax,eax
   0x0000063d <+17>:    mov    DWORD PTR [esp],0x78c
   0x00000644 <+24>:    call   0x645 <func+25>
   0x00000649 <+29>:    lea    eax,[ebp-0x2c]
   0x0000064c <+32>:    mov    DWORD PTR [esp],eax
   0x0000064f <+35>:    call   0x650 <func+36>
   0x00000654 <+40>:    cmp    DWORD PTR [ebp+0x8],0xcafebabe
   0x0000065b <+47>:    jne    0x66b <func+63>
   0x0000065d <+49>:    mov    DWORD PTR [esp],0x79b
   0x00000664 <+56>:    call   0x665 <func+57>
   0x00000669 <+61>:    jmp    0x677 <func+75>
   0x0000066b <+63>:    mov    DWORD PTR [esp],0x7a3
   0x00000672 <+70>:    call   0x673 <func+71>
   0x00000677 <+75>:    mov    eax,DWORD PTR [ebp-0xc]
   0x0000067a <+78>:    xor    eax,DWORD PTR gs:0x14
   0x00000681 <+85>:    je     0x688 <func+92>
   0x00000683 <+87>:    call   0x684 <func+88>
   0x00000688 <+92>:    leave  
   0x00000689 <+93>:    ret    
End of assembler dump.
```
We just need the followings
```
cmp    DWORD PTR [ebp+0x8],0xcafebabe

lea    eax,[ebp-0x2c]
```

