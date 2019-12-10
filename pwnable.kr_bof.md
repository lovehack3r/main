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
