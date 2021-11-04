---
title: "CS:APP - 기계어, 정수"
date: 2017-03-07 01:09:00 +0900 KST
categories: [csapp]
---

amd x86-64 기준, 프로그램의 기계수준 표현에 대해서 알아본다.

이 글에서 정리한 것은 word, 정수 레지스터,
att형식 operand 읽는 법, 그리고 mov 인스트럭션이다.

## 1. C 자료형의 길이

| C declaration | Intel data types | Assembly-code suffix | Size |
| ------------- | ---------------- | -------------------- | ---- |
| char          | Byte             | b                    | 1    |
| short         | Word             | w                    | 2    |
| int           | Double word      | l                    | 4    |
| long          | Quad word        | q                    | 8    |
| char *        | Quad word        | q                    | 8    |
| float         | Single precision | s                    | 4    |
| double        | Double precision | l                    | 8    |

x86 프로세서는 처음에 16비트로 시작해서 32비트, 64비트로 확장했기 때문에
word는 16비트 데이터 타입을 말한다.

그래서 32비트 데이터 타입은 double word, 64비트는 quad word라고 부른다.

윈도우 프로그래밍에서 DWORD 타입이 그래서 32비트인가?

## 2. 정수 레지스터 16개

| 64bit | 32bit | 16bit | 8bit  |               |
| ----- | ----- | ----- | ----- | ------------- |
| %rax  | %eax  | %ax   | %al   | Return value  |
| %rbx  | %ebx  | %bx   | %bl   | Callee saved  |
| %rcx  | %ecx  | %cx   | %cl   | 4th argument  |
| %rdx  | %edx  | %dx   | %dl   | 3th argument  |
| %rsi  | %esi  | %si   | %sil  | 2nd argument  |
| %rdi  | %edi  | %di   | %dil  | 1st argument  |
| %rbp  | %ebp  | %bp   | %bpl  | Callee saved  |
| %rsp  | %esp  | %sp   | %spl  | Stack pointer |
| %r8   | %r8d  | %r8w  | %r8b  | 5th argument  |
| %r9   | %r9d  | %r9w  | %r9b  | 6th argument  |
| %r10  | %r10d | %r10w | %r10b | Caller saved  |
| %r11  | %r11d | %r11w | %r11b | Caller saved  |
| %r12  | %r12d | %r12w | %r12b | Callee saved  |
| %r13  | %r13d | %r13w | %r13b | Callee saved  |
| %r14  | %r14d | %r14w | %r14b | Callee saved  |
| %r15  | %r15d | %r15w | %r15b | Callee saved  |

x86이 64비트로 확장되면서 정수 레지스터를 8개 더 늘렸다.

함수 인자가 4개를 넘어가면 레지스터로 커버가 안되서 느려진다고 들었었는데...
이 표대로라면 64비트에선 인자를 6개까지 해도 괜찮은 걸까?

## 3. 오퍼랜드(Operand) 형식

| Type      | Form           | Operand value              | Name                |
| --------- | -------------- | -------------------------- | ------------------- |
| Immediate | $Imm           | Imm                        | Immediate           |
| Register  | r              | R[r]                       | Register            |
| Memory    | Imm            | M[Imm]                     | Absolute            |
| Memory    | (r)            | M[R[r]]                    | Indirect            |
| Memory    | Imm(r)         | M[Imm + R[r]]              | Base + displacement |
| Memory    | (r1, r2)       | M[R[r1] + R[r2]]           | Indexed             |
| Memory    | Imm(r1, r2)    | M[Imm + R[r1] + R[r2]]     | Indexed             |
| Memory    | (, r, s)       | M[R[r] * s]                | Scaled indexed      |
| Memory    | Imm(, r, s)    | M[Imm + R[r] * s]          | Scaled indexed      |
| Memory    | (r1, r2, s)    | M[R[r1] + R[r2] * s]       | Scaled indexed      |
| Memory    | Imm(r1, r2, s) | M[Imm + R[r1] + R[r2] * s] | Scaled indexed      |

ATT 형식의 어셈블리 코드 읽는 법..
야매로 작성해서 틀린 게 있을 수 있다.

이제 gdb에서 intel 형식으로 바꿔주지 않아도 읽을 수 있다... ㅜㅜ

## 4. 데이터 이동 인스트럭션(Instruction)

### 간단한 데이터 이동 인스트럭션

| Instruction  | Effect           | Description             |
| ------------ | ---------------- | ----------------------- |
| mov S, D     | D <\- S          | Move                    |
| movb         | Move byte        |
| movw         | Move word        |
| movl         | Move double word |
| movq         | Move quad word   |
| movabsq I, R | R <\- I          | Move absolute quad word |

Source에는 상수, 레지스터 저장 값, 메모리 저장 값이 올 수 있다.

Destination에는 레지스터, 메모리 주소가 올 수 있다.

단, Source와 Destination가 동시에 메모리를 가리킬 수 없다.

movb, movw는 상위바이트에 관여하지 않으나,
movl은 상위 4바이트를 0으로 설정한다.

movabsq는 64비트 상수를 위한 인스트럭션이다.
Source로 상수(I)만을, Destination으로 레지스터(R)만을 가질 수 있다.

### 0 확장 데이터 이동 인스트럭션

| Instruction | Effect                                 | Description              |
| ----------- | -------------------------------------- | ------------------------ |
| movz S, R   | R <\- ZeroExtend(S)                    | Move with zero extension |
| movzbw      | Move zero-extended byte to word        |
| movzbl      | Move zero-extended byte to double word |
| movzwl      | Move zero-extended word to double word |
| movzbq      | Move zero-extended byte to quad word   |
| movzwq      | Move zero-extended word to quad word   |

상위 바이트를 모두 0으로 채우는 효과가 있다.
unsigned형에서 많이 쓰일 것 같다.

Destination으로 레지스터(R)만을 가질 수 있다.

movzlq가 없는데, movl은 이미 이러한 효과를 가지고 있기 때문이다.

### 부호 확장 데이터 이동 인스트럭션

| Instruction | Effect                                      | Description              |
| ----------- | ------------------------------------------- | ------------------------ |
| movs S, R   | R <\- SignExtend(S)                         | Move with sign extension |
| movsbw      | Move sign-extended byte to word             |
| movsbl      | Move sign-extended byte to double word      |
| movswl      | Move sign-extended word to double word      |
| movsbq      | Move sign-extended byte to quad word        |
| movswq      | Move sign-extended word to quad word        |
| movslq      | Move sign-extended double word to quad word |
| cltq        | %rax <\- SignExtend(%eax)                   | Sign-extend %eax to %rax |

signed형에서 많이 쓰일 것 같다.

movslq %eax, %rax와 같은 효과를 가진 cltq가 따로 있다.
자주 쓰여서 짧은 인코딩을 위해 만들었나보다.

## 5. 데이터 이동 예제

```c
long exchange(long *xp, long y) {
    long x = *xp;
    *xp = y;
    return x;
}
```

xp 포인터가 가리키는 위치에 y를 대입하고, 기존 값은 반환하는 함수다.

```sh
gcc -Og -S exchange.c
cat exchange.s
```

위 명령어로 어셈블리 코드를 보자

```asm
exchange:
.LFB0:
    .cfi_startproc
    movq    (%rdi), %rax
    movq    %rsi, (%rdi)
    ret
    .cfi_endproc
```

책에 따르면 이 예제에서 주목할 점이 두 가지 있다.

첫째는 C언어에서 포인터라고 부르는 것이 어셈블리어에서는 주소라는 점이고,
둘째로 x 같은 지역변수들은 종종 레지스터에 저장된다는 점이다.

개인적으로 둘째가 프로그램 최적화의 핵심이라고 생각한다.

## 6. PUSH, POP 인스트럭션

| Instruction | Effect                                       | Description    |
| ----------- | -------------------------------------------- | -------------- |
| pushq S     | R[%rsp] <\- R[%rsp] - 8;<br>M[R[%rsp]] <\- S | Push quad word |
| popq D      | D <\- M[R[%rsp]];<br>R[%rsp] <\- R[%rsp] + 8 | Pop quad word  |

## 7. 마무리

시간이 없어서 사흘 정도 글을 작성하지 못했다.
이 글도 내가 나간 진도보다 많이 늦다.
그래서 급하게 작성하느라 정리도 제대로 안했다.

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
