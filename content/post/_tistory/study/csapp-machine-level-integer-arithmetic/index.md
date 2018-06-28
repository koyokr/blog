---
title: "CS:APP - 기계어, 정수 연산"
slug: csapp-machine-level-integer-arithmetic
date: 2017-03-07 19:09:00 +0900 KST
categories: [study]
markup: mmark
---

정수 산술연산과 관련된 인스트럭션 정리.

## 1. 정수의 산술연산

| Instruction | Effect        | Description              |
| ----------- | ------------- | ------------------------ |
| leaq S, D   | D <\- &S      | Load effective address   |
| inc D       | D <\- D + 1   | Increment                |
| dec D       | D <\- D - 1   | Decrement                |
| neg D       | D <\- -D      | Negate                   |
| not D       | D <\- -D      | Complement               |
| add S, D    | D <\- D + S   | Add                      |
| sub S, D    | D <\- D - S   | Subtract                 |
| imul S, D   | D <\- D * S   | Multiply                 |
| xor S, D    | D <\- D ^ S   | Exclusive-or             |
| or S, D     | D <\- D \| S  | Or                       |
| and S, D    | D <\- D & S   | And                      |
| sal k, D    | D <\- D << k  | Left shift               |
| shl k, D    | D <\- D << k  | Left shift (same as sal) |
| sar k, D    | D <\- D >>A k | Arithmetic right shift   |
| shr k, D    | D <\- D >>L k | Logical right shift      |

leaq(Load Effective Address, Quad word) 인스트럭션은
movq 인스트럭션의 변형판이라고 보면 된다.

추가로 S에 & 연산자가 붙은 것이 차이점인데,
이 연산자는 C언어에서 주소를 반환하는 &연산자와 같다.

이해를 위해 예제를 보자. 정수연산을 하는 코드다.

```c
long scale(long x, long y, long z) {
    long t = x + 4*y + 12*z;
    return t;
}
```

gcc -O1 -S scale.c 명령어로 컴파일하자.

```asm
scale:
.LFB0:
    .cfi_startproc
    leaq    (%rdi,%rsi,4), %rax    ; %rax = x + 4*y
    leaq    (%rdx,%rdx,2), %rdx    ; %rdx = z + 2*z = 3*z
    leaq    (%rax,%rdx,4), %rax    ; %rax = %rax + 4*%rdx = x + 4*y + 12*z
    ret
    .cfi_endproc
```

주소연산과는 관련 없어보이는 이러한 연산에도 많이 쓰이는 것을 확인할 수 있다.

sal, shl, sar, shr과 같은 쉬프트 연산의 source에는
상수 또는 단일 바이트 레지스터 %cl로 명시할 수 있다.

특정 레지스터 %cl만 operand로 허용하는 점이 특이하다.

연습문제를 보면 다음과 같은 C 함수의 어셈블리 코드 중 일부를 작성하는 문제가 있다.

```c
long shitf_left4_rightn(long x, long n) {
    x <<= 4;
    x >>= n;
    return x;
}
```

어셈블리 코드는 아래와 같다.

```asm
shift_left4_rightn:
.LFB0:
    .cfi_startproc
    movq    %rdi, %rax
    salq    $4, %rax
    movl    %esi, %ecx
    sarq    %cl, %rax
    ret
    .cfi_endproc
```

%cl 오퍼랜드를 사용하기 위해 %esi의 값을
%ecx에 movl 인스트럭션으로 복사하는 것이 인상적이다.

## 2. 특수 산술연산

| Instruction | Effect                                                                 | Description            |
| ----------- | ---------------------------------------------------------------------- | ---------------------- |
| imulq S     | R[%rdx]:R[%rax] <\- S * R[%rax]                                        | Signed full multiply   |
| mulq S      | R[%rdx]:R[%rax] <\- S * R[%rax]                                        | Unsigned full multiply |
| cqto        | R[%rdx]:R[%rax] <\- SignExtend(R[%rax])                                | Convert to oct word    |
| idivq S     | R[%rdx] <\- R[%rdx]:R[%rax] mod S;<br>R[%rdx] <\- R[%rdx]:R[%rax] / S; | Signed divide          |
| divq S      | R[%rdx] <\- R[%rdx]:R[%rax] mod S;<br>R[%rdx] <\- R[%rdx]:R[%rax] / S; | Unsigned divide        |

64비트 정수형간 곱셈의 결과값 표시를 위해서는 128비트가 필요하다.
참고로 128비트 워드는 oct word라고 불린다.

곱셈 확인을 위해 아래 C코드의 어셈블리를 확인해본다.
gcc에서 제공하는 __int128을 사용했다.

```c
#include <inttypes.h>

typedef unsigned __int128 uint128_t;

void store_uprod(uint128_t *dest, uint64_t x, uint64_t y) {
    *dest = x * (uint128_t)y;
}
```

```asm
store_uprod:
.LFB4:
    .cfi_startproc
    movq    %rsi, %rax       ; Copy x to multiplicand
    mulq    %rdx             ; Multiply by y
    movq    %rax, (%rdi)     ; Store lower 8 bytes at dest
    movq    %rdx, 8(%rdi)    ; Store upper 8 bytes at dest+8
    ret
    .cfi_endproc
```

계산결과 중 중요한 바이트가 %rdx에,
덜 중요한 바이트가 %rax에 대입된 것을 확인할 수 있다.

나눗셈도 확인해보자.

```c
void remdiv(long x, long y,
            long *qp, long *rp) {
    long q = x/y;
    long r = x%y;
    *qp = q;
    *rp = r;
}
```

```asm
remdiv:
.LFB0:
    .cfi_startproc
    movq    %rdi, %rax      ; Move x to lower 8 bytes of dividend
    movq    %rdx, %rdi      ; Copy qp
    cqto                    ; Sign-extend to upper 8 bytes of dividend
    idivq   %rsi            ; Divide by y
    movq    %rax, (%rdi)    ; Store quotient at qp
    movq    %rdx, (%rcx)    ; Store remainder at rp
    ret
    .cfi_endproc
```

divq을 사용하기 전에 128비트 %rdx:%rax가 준비돼있어야 한다.
보통은 64비트 %rax를 128비트 %rdx:%rax로 확장하는 cqto 인스트럭션을 사용한다.

나눗셈 결과는 %rax에, 나머지 결과는 %rdx에 저장된다.

## 3. 마무리

제어문까지 쓰려고 했는데 헬스장 갈 시간이 돼서ㅎ

표 작성도 몇번 하니까 익숙하다.

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
