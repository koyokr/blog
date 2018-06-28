---
title: "CS:APP - 기계어, 프로시저"
slug: csapp-machine-level-procedure
date: 2017-03-09 18:20:00 +0900 KST
categories: [study]
markup: mmark
---

오늘은 프로시저(Procedure)에 대해 정리한다.

먼저 프로시저에 대해 알아보자.

나는 프로시저는 인자를 받을 수 있고 값을 리턴할 수 있는
코드 블럭(특정 기능이 구현된)의 추상화라고 이해했다.

여러 프로그래밍 언어에서 이를 함수, 메소드, 서브루틴, 핸들러 등으로 사용한다.

프로시저가 다른 프로시저를 호출하고 리턴하는 일련의 동작은
다음과 같은 특징을 하나 이상 가질 수 있다.

1. 제어권 전달
    * 프로그램 카운터(pc)는 호출되는 프로시저에 대한 코드의 시작주소로 설정된다.
    * 리턴할 때는 호출 인스트럭션 다음의 인스트럭션으로 설정된다.
2. 데이터 전달
    * 호출자는 하나 이상의 매개변수를 피호출자에 제공할 수 있어야 하며.
    * 피호출자는 호출자에게 하나의 값을 리턴할 수 있어야 한다.
3. 메모리 할당과 반납
    * 피호출자는 지역변수를 위한 공간을 할당할 수 있다.
    * 리턴할 때 이 공간을 반납할 수 있다.

이제 이 프로시저의 매커니즘을 아래에 정리했다.

## 1. 콜(Call), 리턴(Return)

| Instruction   | Description      |
| ------------- | ---------------- |
| call Label    | Procedure call   |
| call *Operand | Procedure call   |
| ret           | Return from call |

call 인스트럭션은 두 가지 동작을 한다.
call 인스트럭션의 다음 주소를 스택에 push하고
pc를 오퍼랜드가 가리키는 주소로 설정한다.

ret 인스트럭션은 돌아갈 주소를 스택에서 pop해와서 pc로 설정한다.

실제로 어떻게 동작하는지 알아보자.

다음은 last 함수와 last 함수를 호출하는 first 함수의 C 코드다.

```c
long last(long u, long v) {
    return u * v;
}

long first(long x) {
    return last(x - 1, x + 1);
}
```

아래는 first 함수를 호출하는 main 코드

```c
int main() {
    return first(10);
}
```

gcc를 명령어 옵션 -Og로 실행하고 objdump -d first_last를 실행한 결과.

```x86asm
0000000000000660 <last>:
 660:   48 89 f8                mov    %rdi,%rax    ; L1: u
 663:   48 0f af c6             imul   %rsi,%rax    ; L2: u * v
 667:   c3                      retq                ; L3: Return

0000000000000668 <first>:
 668:   48 8d 77 01             lea    0x1(%rdi),%rsi    ; F1: x + 1
 66c:   48 83 ef 01             sub    $0x1,%rdi         ; F2: x - 1
 670:   e8 eb ff ff ff          callq  660 <last>        ; F3: Call last(x - 1, x + 1)
 675:   f3 c3                   repz retq                ; F4: Return

0000000000000677 <main>:
 677:   bf 0a 00 00 00          mov    $0xa,%edi      ; M1: 10
 67c:   e8 e7 ff ff ff          callq  668 <first>    ; M2: Call first(10)
 681:   f3 c3                   repz retq             ; M3: Return
```

인스트럭션의 실행 순서는 다음과 같다.

M1 - > M2 -> F1 -> F2 -> F3 -> L1 -> L2 -> L3 -> F4 -> M3

M1의 인스트럭션이 실행될 때, 스택 포인터 %rsp가 0x7fffffffdd58이라고 가정하면
다음과 같은 표를 작성할 수 있다.

| Label | PC    | Instruction | %rdi           | %rsi           | %rax           | %rsp           | *%rsp |
| ----- | ----- | ----------- | -------------- | -------------- | -------------- | -------------- | ----- |
| M1    | 0x677 | mov         | 0x7fffffffdd58 |
| M2    | 0x67c | callq       | 10             | 0x7fffffffdd58 |
| F1    | 0x668 | lea         | 10             | 0x7fffffffdd50 | 0x681          |
| F2    | 0x66c | sub         | 10             | 11             | 0x7fffffffdd50 | 0x681          |
| F3    | 0x670 | callq       | 9              | 11             | 0x7fffffffdd50 | 0x681          |
| L1    | 0x660 | mov         | 9              | 11             | 0x7fffffffdd48 | 0x675          |
| L2    | 0x663 | imul        | 9              | 11             | 9              | 0x7fffffffdd48 | 0x675 |
| L3    | 0x667 | retq        | 9              | 11             | 99             | 0x7fffffffdd48 | 0x675 |
| F4    | 0x675 | repz retq   | 9              | 11             | 99             | 0x7fffffffdd50 | 0x681 |
| M3    | 0x681 | repz retq   | 9              | 11             | 99             | 0x7fffffffdd58 |

## 2. 데이터 전달

x86-64에서는 최대 여섯 개의 정수형 인자를 레지스터로 전달할 수 있다.

| bits | 1st  | 2nd  | 3rd  | 4th  | 5th  | 6th  |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 64   | %rdi | %rsi | %rdx | %rcx | %r8  | %r9  |
| 32   | %edi | %esi | %edx | %ecx | %r8d | %r9d |
| 16   | %di  | %si  | %dx  | %cx  | %r8w | %r9w |
| 8    | %dil | %sil | %dl  | %cl  | %r8b | %r9b |

이름 진짜 일관성 없다;;

함수가 여섯 개보다 많은 정수형 인자를 필요로 할 때,
다른 인자는 스택으로 전달된다.

n  > 6인 n개의 정수형 인자를 가진 함수를 호출하면
1~6번째 인자는 레지스터에 저장하고, 7~n번째 인자는 스택에 push한다.

스택에 저장할 때 데이터 길이는 8의 배수로 반올림한다.

책 예제를 보자.

```c
void proc(long  a1, long  *a1p,
          int   a2, int   *a2p,
          short a3, short *a3p,
          char  a4, char  *a4p) {
    *a1p += a1;
    *a2p += a2;
    *a3p += a3;
    *a4p += a4;
}
```

```x86asm
proc:
.LFB0:
    .cfi_startproc
    movq    16(%rsp), %rax    ; Fetch a4p
    addq    %rdi, (%rsi)      ; *a1p += a1
    addl    %edx, (%rcx)      ; *a2p += a2
    addw    %r8w, (%r9)       ; *a3p += a3
    movl    8(%rsp), %edx     ; Fetch a4
    addb    %dl, (%rax)       ; *a4p += a4
    ret                       ; Return
    .cfi_endproc
```

인자 a1부터 a3p까지는 레지스터로 전달하고,
인자 a4, a4p는 스택을 통해 전달하는 걸 확인할 수 있다.

## 3. 스택을 이용하는 지역변수

컴파일러는 필요하지 않다면 지역변수를 굳이 스택 같은 메모리에 저장하지 않는다.

지역변수가 메모리에 저장되야하는 경우는 다음과 같다.

1. 지역변수를 모두 저장하기엔 레지스터의 수가 부족하다.
2. 지역변수의 주소를 생성할 수 있어야 한다. 예를 들면 연산자  &.
3. 지역변수가 배열이거나 구조체여서 배열이나 구조체 참조로 접근돼야 한다.

프로시저는 스택 포인터를 감소시켜서 스택 프레임에 공간을 할당하는데...
이것도 책 예제를 보면

```c
long swap_add(long *xp, long *yp) {
    long x = *xp;
    long y = *yp;
    *xp = y;
    *yp = x;
    return x + y;
}

long caller() {
    long arg1 = 534;
    long arg2 = 1057;
    long sum = swap_add(&arg1, &arg2);
    long diff = arg1 - arg2;
    return sum * diff;
}
```

함수 caller는 arg1과 arg2의 포인터를 swap_add에 전달한다.

gcc -Og -S -fno-stack-protector swap_add.c를 실행해서
caller 프로시저를 봤다.

```x86asm
caller:
.LFB1:
    .cfi_startproc
    subq    $16, %rsp        ; Allocate 16 bytes for stack frame
    .cfi_def_cfa_offset 24
    movq    $534, 8(%rsp)    ; Store 534 in arg1
    movq    $1057, (%rsp)    ; Store 1057 in arg2
    movq    %rsp, %rsi       ; Compute &arg2
    leaq    8(%rsp), %rdi    ; Compute &arg1
    call    swap_add         ; Call swap_add(&arg1, &arg2)
    movq    8(%rsp), %rdx    ; Get arg1
    subq    (%rsp), %rdx     ; Compute diff = arg1 - arg2
    imulq   %rdx, %rax       ; Compute sum * diff
    addq    $16, %rsp        ; Deallocate stack frame
    .cfi_def_cfa_offset 8
    ret                      ; Return
    .cfi_endproc
```

subq 인스트럭션에서 스택 포인터 %rsp를 16만큼 감소시키고 있다.
이는 스택 공간을 16바이트만큼 확보하는 효과를 가진다.

위 어셈블리 코드에서는 8(%rsp)에 534를, %rsp에 1057을 저장한다.
그리고 %rsi에 %rsp를, %rdi에 %rsp + 8을 저장한 뒤에 swap_add를 call한다.

그리고 diff 계산을 위해 다시 스택에서 arg1과 arg2를 로드해 계산한다.

마지막으로 함수를 리턴하기 전에 addq 인스트럭션에서 %rsp를 16만큼 증가시킨다.
이는 프로시저 처음에 확보한 스택 공간을 제거하는 효과를 가진다.

## 4. 레지스터를 이용하는 지역변수

하나의 프로세스에서 모든 프로시저는 레지스터를 공유한다.

호출자가 가지고 놀던 레지스터를 피호출자가 망쳐버릴 수 있다는 이야기다.

관습상 x86-64에서 레지스터 %rbx, %rbp, %r12-%r15는
피호출자-저장(Callee saved) 레지스터로 구분한다.

이 레지스터가 쓰고 싶은 프로시저는 사용 전에 기존 값을 스택에 push해두고,
리턴하기 전에 스택에서 pop하여 값을 보존한다.

레지스터의 값 보존 책임은 피호출자에게 있는 것이다.

또 예제를 보면...

```c
long Q(long u) {
    return u;
}

long P(long x, long y) {
    long u = Q(y);
    long v = Q(x);
    return u + v;
}
```

P의 어셈블리 코드를 확인하면...

```x86asm
P:
.LFB1:
    .cfi_startproc
    pushq   %rbp          ; Save %rbp
    .cfi_def_cfa_offset 16
    .cfi_offset 6, -16
    pushq   %rbx          ; Save %rbx
    .cfi_def_cfa_offset 24
    .cfi_offset 3, -24
    movq    %rdi, %rbp
    movq    %rsi, %rdi
    call    Q
    movq    %rax, %rbx
    movq    %rbp, %rdi
    call    Q
    addq    %rbx, %rax
    popq    %rbx          ; Restore %rbx
    .cfi_def_cfa_offset 16
    popq    %rbp          ; Restore %rbp
    .cfi_def_cfa_offset 8
    ret
    .cfi_endproc
```

%rbp와 %rbx를 사용하기 위해 push하고 리턴 전에 pop하는 것을 볼 수 있다.

## 5. 마무리

내용에 비해서 작성에 걸린 시간이 길었다.
이번은 책을 복붙한 수준인데도...

내일은 정말루 3장을 끝낼 수 있길..

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
