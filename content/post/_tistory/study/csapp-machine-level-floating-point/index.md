---
title: "CS:APP - 기계어, 부동소수점"
slug: csapp-machine-level-floating-point
date: 2017-03-14 15:37:00 +0900 KST
categories: [study]
markup: mmark
---

이 내용 이전에 gdb 사용법, 버퍼 오버플로, 메모리 보호기법,
가변 크기 스택프레임에 대한 내용이 있었다.

그런데 글로 정리하려니 잘 안되서 그냥 생략하고 부동소수점을 정리하기로 했다.

부동소수점 아키텍처는 대략 아래와 같이 발전했다.

## (1) MMX

MM 레지스터를 사용하며, 64비트다.

## (2) SSE

128비트로 확장했다. XMM 레지스터가 추가됐다.
%xmm0부터 %xmm15까지 있다.

MM 레지스터는 없다.

x86-64 코드를 실행할 수 있는 모든 프로세서들은 SSE2 이상을 지원한다.

## (3) AVX

256비트로 확장했다. YMM 레지스터가 추가됐다.
%ymm0부터 %ymm15까지 있다.

인텔 샌디브릿지 아키텍처부터 지원한다.

## (4) AVX2

인스트럭션이 추가됐다.

인텔 하스웰 아키텍처부터 지원한다.

## (5) AVX-512

512바이트로 확장했다. ZMM 레지스터가 추가됐다.
%zmm0부터 %zmm31까지 있다.

인텔 스카이레이크 아키텍처부터 지원한다.

책은 AVX2에 기초해있으며, 이 글도 책을 따른다.

## 1. AVX 레지스터

| 256bit | 128bit |                          |
| ------ | ------ | ------------------------ |
| %ymm0  | %xmm0  | 1st FP arg./Return value |
| %ymm1  | %xmm1  | 2nd FP argument          |
| %ymm2  | %xmm2  | 3rd FP argument          |
| %ymm3  | %xmm3  | 4th FP argument          |
| %ymm4  | %xmm4  | 5th FP argument          |
| %ymm5  | %xmm5  | 6th FP argument          |
| %ymm6  | %xmm6  | 7th FP argument          |
| %ymm7  | %xmm7  | 8th FP argument          |
| %ymm8  | %xmm8  | Caller saved             |
| %ymm9  | %xmm9  | Caller saved             |
| %ymm10 | %xmm10 | Caller saved             |
| %ymm11 | %xmm11 | Caller saved             |
| %ymm12 | %xmm12 | Caller saved             |
| %ymm13 | %xmm13 | Caller saved             |
| %ymm14 | %xmm14 | Caller saved             |
| %ymm15 | %xmm15 | Caller saved             |

AVX512 레지스터인 ZMM은 안넣었다.

## 2. 부동소수점 이동 및 변환 연산

이동 연산부터

| Instruction | Source   | Destination | Description                           |
| ----------- | -------- | ----------- | ------------------------------------- |
| vmovss      | M(32bit) | X           | Move single precision                 |
| vmovss      | X        | M(32bit)    | Move single precision                 |
| vmovsd      | M(64bit) | X           | Move double precision                 |
| vmovsd      | X        | M(64bit)    | Move double precision                 |
| vmovaps     | X        | X           | Move aligned, packed single precision |
| vmovapd     | X        | X           | Move aligned, packed double precision |

인스트럭션에서 문자 'a'는 aligned를 의미한다.
주소가 16바이트 정렬 요건을 만족하지 못하면 예외가 발생한다.

C 코드와 번역된 어셈블리 코드를 보자.

```c
float float_mov(float v1, float *src, float *dst) {
    float v2 = *src;
    *dst = v1;
    return v2;
}
```

아래는 "gcc -Og -S float_mov.c"의 결과.

```asm
float_mov:
    movss   (%rdi), %xmm1
    movss   %xmm0, (%rsi)
    movaps  %xmm1, %xmm0
    ret
```

아래는 "gcc -mavx2 -Og -S float_mov.c"의 결과.

```asm
float_mov:
    vmovss  (%rdi), %xmm1    ; Read v2 from src
    vmovss  %xmm0, (%rsi)    ; Write v1 to dst
    vmovaps %xmm1, %xmm0
    ret
```

이 글에서는 계속 -mavx2 명령줄 옵션으로 gcc를 실행할 것이다.

다음은 변환 연산.

| Instruction | Source     | Destination | Description                                                   |
| ----------- | ---------- | ----------- | ------------------------------------------------------------- |
| vcvttss2si  | X/M(32bit) | R(32bit)    | Convert with truncation single precision to integer           |
| vcvttsd2si  | X/M(64bit) | R(32bit)    | Convert with truncation double precision to integer           |
| vcvttss2siq | X/M(32bit) | R(64bit)    | Convert with truncation single precision to quad word integer |
| vcvttsd2siq | X/M(64bit) | R(64bit)    | Convert with truncation double precision to quad word integer |

xmm 레지스터나 메모리에서 읽은 부동소수점 값을 정수 레지스터로 변환한다.

이때 값은 0의 방향으로 근사한다.

| Instruction | Source 1          | Source 2 | Destination | Description                                   |
| ----------- | ----------------- | -------- | ----------- | --------------------------------------------- |
| vcvtsi2ss   | M(32bit)/R(32bit) | X        | X           | Convert integer to single precision           |
| vcvtsi2sd   | M(32bit)/R(32bit) | X        | X           | Convert integer to double precision           |
| vcvtsi2ssq  | M(64bit)/R(64bit) | X        | X           | Convert quad word integer to single precision |
| vcvtsi2sdq  | M(64bit)/R(64bit) | X        | X           | Convert quad word integer to double precision |

정수에서 부동소수점으로 변환하는 인스트럭션이다.

첫번째 Source에서 Destination의 자료형으로 변환한다.
두번째 Source는 결과의 하위 바이트에 영향을 주지 않는다.

두번째 Source는 무슨 역할일까? 난 모르겠다ㅜㅜ

이 시점에서 변환 인스트럭션이 어떻게 쓰이는지 확인하기 위해 아래 C 코드를 컴파일했다.

```c
double fcvt(int i, float *fp, double *dp, long *lp) {
    float  f = *fp;
    double d = *dp;
    long   l = *lp;

    *lp = (long)    d;
    *fp = (float)   i;
    *dp = (double)  l;
    return (double) f;
}
```

```asm
fcvt:
    vmovss      (%rsi), %xmm0          ; Get f = *fp
    movq        (%rcx), %rax           ; Get l = *lp
    vcvttsd2siq (%rdx), %r8            ; Get d = *dp and convert to long
    movq        %r8, (%rcx)            ; Store at lp
    vxorps      %xmm1, %xmm1, %xmm1
    vcvtsi2ss   %edi, %xmm1, %xmm1     ; Convert i to float
    vmovss      %xmm1, (%rsi)          ; Store at fp
    vxorpd      %xmm1, %xmm1, %xmm1
    vcvtsi2sdq  %rax, %xmm1, %xmm1     ; Convert l to double
    vmovsd      %xmm1, (%rdx)          ; Store at dp
    vcvtss2sd   %xmm0, %xmm0, %xmm0    ; Convert f to double
    ret
```

vxorps 인스트럭션은 나중에 다룬다.
근데 왜 쓰는 걸까? 안해줘도 될 것 같은데...

단일 정밀도 값을 이중 정밀도 값으로 변환하는 인스트럭션 vcvtss2sd가 인상깊다.
아마 vcvtsd2ss도 있겠지.

## 3. 부동소수점 산술 연산

인스트럭션

| Single | Double | Effect            | Description                |
| ------ | ------ | ----------------- | -------------------------- |
| vaddss | vaddsd | D <\- S2 + S1     | Floating-point add         |
| vsubss | vsubsd | D <\- S2 - S1     | Floating-point subtract    |
| vmulss | vmulsd | D <\- S2 * S1     | Floating-point multiply    |
| vdivss | vdivsd | D <\- S2 / S1     | Floating-point divide      |
| vmaxss | vmaxsd | D <\- max(S2, S1) | Floating-point maximum     |
| vminss | vminsd | D <\- min(S2, S1) | Floating-point minimum     |
| sqrtss | sqrtsd | D <\- sqrt(S1)    | Floating-point square root |

예제

```c
double funct(double a, float x, double b, int i) {
    return a*x - b/i;
}
```

```asm
funct:
    vcvtss2sd   %xmm1, %xmm1, %xmm1    ; Convert x to double
    vmulsd      %xmm0, %xmm1, %xmm0    ; Multiply a by x
    vxorpd      %xmm1, %xmm1, %xmm1
    vcvtsi2sd   %edi, %xmm1, %xmm1     ; Convert i to double
    vdivsd      %xmm1, %xmm2, %xmm2    ; Compute b/i
    vsubsd      %xmm2, %xmm0, %xmm0    ; Subtract from a*x
    ret
```

## 4. 부동소수점 상수

AVX 부동소수점 연산은 immediate value가 오퍼랜드에 올 수 없다.

컴파일러는 상수 값을 위해 저장공간을 할당한다.

다음 C코드를 통해 확인해보자.

```c
double cel2fahr(double temp) {
    return 1.8 * temp + 32.0;
}
```

```asm
cel2fahr:
.LFB0:
    .cfi_startproc
    vmulsd  .LC0(%rip), %xmm0, %xmm0
    vaddsd  .LC1(%rip), %xmm0, %xmm0
    ret
    .cfi_endproc
.LFE0:
    .size   cel2fahr, .-cel2fahr
    .section        .rodata.cst8,"aM",@progbits,8
    .align 8
.LC0:
    .long   3435973837    ; Low-order 4 bytes of 1.8
    .long   1073532108    ; High-order 4 bytes of 1.8
    .align 8
.LC1:
    .long   0             ; Low-order 4 bytes of 32.0
    .long   1077936128    ; Low-order 4 bytes of 32.0
```

리틀 엔디안임을 감안하고 보면 1.8이 0x3FFCCCCCCCCCCCCD로 저장됐다는 것과,
32.0이 0x4040000000000000으로 저장됐다는 것을 알 수 있다.

## 5. 부동소수점 비트 연산

| Single | Double | Effect        | Description          |
| ------ | ------ | ------------- | -------------------- |
| vxorps | xorpd  | D <\- S2 ^ S1 | Bitwise EXCLUSIVE-OR |
| vandps | andpd  | D <\- S2 & S1 | Bitwise EXCLUSIVE-OR |

예제. 아래는 math.c 파일 내용.

```c
double zero(void) {
    return 0.0;
}

double negative(double x) {
    return -x;
}

#include <math.h>
double absolute(double x) {
    return fabs(x);
}
```

gcc -mavx2 -Og -S math.c을 실행하면...

```asm
zero:
    vxorpd  %xmm0, %xmm0, %xmm0
    ret
negative:
    vxorpd  .LC1(%rip), %xmm0, %xmm0
    ret
absolute:
    vandpd  .LC2(%rip), %xmm0, %xmm0
    ret
.LC1:
    .long   0
    .long   -2147483648
    .long   0
    .long   0
.LC2:
    .long   4294967295
    .long   2147483647
    .long   0
    .long   0
```

부동소수점에 비트 연산이 쓰이는 걸 볼 수 있다.
vxorp 인스트럭션은 0을 생성하는데 많이 애용되나 보다.
xor 인스트럭션처럼.

## 6. 부동소수점 비교 연산

| Instruction    | Based on | Description              |
| -------------- | -------- | ------------------------ |
| ucomiss S1, S2 | S2 - S1  | Compare single precision |
| ucomisd S1, S2 | S2 - S1  | Compare double precision |

부동소수점 비교 인스트럭션도 CMP 인스트럭션처럼 조건 코드를 설정한다.

조건 코드는 다음과 같은 규칙을 따라 설정된다.

| Ordering S2:S1 | CF  | ZF  | PF  |
| -------------- | --- | --- | --- |
| Unordered      | 1   | 1   | 1   |
| S2 < S1        | 1   | 0   | 0   |
| S2 = S1        | 0   | 1   | 0   |
| S2 > S1        | 0   | 0   | 0   |

Unordered는 오퍼랜드 중 하나가 NaN일 때 발생하며, PF로 검출할 수 있다.

책에 있는 C 코드로 인스트럭션이 어떻게 사용되는지 보자.

```c
typedef enum { NEG, ZERO, POS, OTHER } range_t;

range_t find_range(float x) {
    int result;
    if (x < 0)
        result = NEG;
    else if (x == 0)
        result = ZERO;
    else if (x > 0)
        result = POS;
    else
        result = OTHER;
    return result;
}
```

아래는 gcc -mavx2 -O2 -S find_range.c를 실행한 결과.

```asm
find_range:
    vxorps      %xmm1, %xmm1, %xmm1
    vucomiss    %xmm0, %xmm1    ; Compare 0:x
    ja      .L5
    vucomiss    %xmm1, %xmm0    ; Compare x:0
    jp      .L8
    movl    $1, %eax      ; result = ZERO
    je      .L10
.L8:
    xorl    %eax, %eax
    movl    $3, %edx
    vucomiss    %xmm1, %xmm0    ; Compare x:0
    seta    %al
    subl    %eax, %edx
    movl    %edx, %eax    ; result = (~CF & ~ZF) ? POS : OTHER
    ret
.L5:
    xorl    %eax, %eax    ; result = NEG
.L10:
    rep ret
```

jp 인스트럭션을 주의깊게 보자.

## 7. 마무리

드디어 3장이 끝났다.

엣지 브라우저에서 글을 작성했는데 엄청 버벅거렸다. 그냥 크롬 써야겠다.

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
