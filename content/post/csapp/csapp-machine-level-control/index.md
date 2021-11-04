---
title: "CS:APP - 기계어, 제어문"
date: 2017-03-08 16:35:00 +0900 KST
categories: [csapp]
---

오늘은 제어문

## 1. 조건 코드

* `CF`: Carry flag. 가장 최근 연산에서 가장 중요한 비트의 올림 발생 표시.

* `ZF`: Zero flag. 가장 최근 연산의 결과가 0인 것을 표시.

* `SF`: Sign flag. 가장 최근 연산이 음수인 것을 표시.

* `OF`: Overflow flag. 가장 최근 연산에서 양수/음수 2의 보수 오버플로 발생 표시.

이 조건 코드 4개가 많이 유용하다고.

만약 C코드 t = a + b를 수행하기 위해 add 인스트럭션을 사용했을 때
위 네가지 조건 코드는 아래의 C 수식에 따라 결정된다.

`CF = (unsigned)t < (unsigned)a`

`ZF = (t == 0)`

`SF = (t < 0)`

`OF = (a < 0 == b < 0) && (t < 0 != a < 0)`

어제 글에서 수식 계산에 잘 써먹었던 leaq 인스트럭션은
주소 계산에 사용하기 위한 것으로, 조건 코드를 변경하지 않는다.

그밖에 소개했던 산술연산에 쓰이는 다른 인스트럭션은 조건 코드 값을 변경한다.

xor 같은 논리연산은 CF와 OF를 0으로 설정한다.

쉬프트 연산은 CF가 마지막으로 쉬프트되는 비트로 설정되며 OF는 0으로 설정된다.

INC와 DEC는 OF와 ZF를 설정하지만, CF에는 영향을 주지 않는다.

## 2. CMP, TEST 인스트럭션

다른 레지스터는 변경시키지 않고 조건 코드만 변경시키는 인스트럭션 클래스가 있다.

| Instruction | Based on            | Description |
| ----------- | ------------------- | ----------- |
| CMP S1, S2  | S2 - S1             | Compare     |
| cmpb        | Compare byte        |
| cmpw        | Compare word        |
| cmpl        | Compare double word |
| cmpq        | Compare quad word   |
| TEST S1, S2 | S1  & S2            | Test        |
| testb       | Test byte           |
| testw       | Test word           |
| testl       | Test double word    |
| testq       | Test quad           |

CMP는 레지스터에 영향을 주지 않는단 차이를 제외하면 SUB 인스트럭션과 동일하다.

같은 관점에서 TEST도 AND 인스트럭션과 동일하다.

## 3. SET, JMP, CMOV 인스트럭션

조건 코드를 이용하는 보편적인 세 가지 방법이 있다.

### (1) 조건에 따라 0 또는 1을 한 개의 바이트에 기록하기

SET 인스트럭션을 이용한다.

| Instruction | Synonym   | Effect               | Set condition                |
| ----------- | --------- | -------------------- | ---------------------------- |
| sete D      | setz      | D <\- ZF             | Equal / zero                 |
| setne D     | setnz     | D <\- ~ZF            | Not equal / not zero         |
| sets D      | D <\- SF  | Negative             |
| setns D     | D <\- ~SF | Nonnegative          |
| setg D      | setnle    | D <\- ~(SF^OF) & ~ZF | Greater (signed >)           |
| setge D     | setnl     | D <\- ~(SF^OF)       | Greater or equal (signed >=) |
| setl D      | setnge    | D <\- SF^OF          | Less (signed <)              |
| setle D     | setng     | D <\- (SF^OF) \| ZF  | Less or equal (signed <=)    |
| seta D      | setnbe    | D <\- ~CF & ~ZF      | Above (unsigned >)           |
| setae D     | setnb     | D <\- ~CF            | Above or equal (unsigned >=) |
| setb D      | setnae    | D <\- CF             | Below (unsigned <)           |
| setbe D     | setna     | D <\- CF \| ZF       | Below or equal (unsigned <=) |

여기서 l과 b는 각각 less와 below를 뜻한다.
long word나 byte로 헷갈리지 말자.

C코드로 사용 예를 보자.

```c
long cmp(long a, long b) {
    return a < b;
}
```

gcc -Og -S cmp.c로 컴파일.

```asm
comp:
.LFB0:
    .cfi_startproc
    cmpq    %rsi, %rdi    ; Compare a:b
    setl    %al           ; Set low-order byte to %eax to 0 or 1
    movzbl  %al, %eax     $ Clear rest of %eax (and rest of %rax)
    ret
    .cfi_endproc
```

### (2) 조건에 따라 프로그램의 다른 부분으로 이동하기

JMP 인스트럭션을 사용하여 조건에 따라(필요하다면 조건 없이)
프로그램의 다른 부분으로 이동할 수 있다.

| Instruction  | Synonym | Jump condition   | Description                  |
| ------------ | ------- | ---------------- | ---------------------------- |
| jmp Label    | ZF      | Direct jump      |
| jmp *Operand | ZF      | Equal / zero     |
| je Label     | jz      | ZF               | Equal / zero                 |
| jne Label    | jnz     | ~ZF              | Not equal / not zero         |
| js Label     | SF      | Negative         |
| jns Label    | ~SF     | Nonnegative      |
| jg Label     | jnle    | ~(SF ^ OF) & ~ZF | Greater (signed >)           |
| jge Label    | jnl     | ~(SF ^ OF)       | Greater or equal (signed >=) |
| jl Label     | jnge    | SF ^ OF          | Less (signed <)              |
| jle Label    | jng     | (SF ^ OF) \| ZF  | Less or equal (signed <=)    |
| ja Label     | jnbe    | ~CF & ~ZF        | Above (unsigned >)           |
| jae Label    | jnb     | ~CF              | Above or equal (unsigned >=) |
| jb Label     | jnae    | CF               | Below (unsigned <)           |
| jbe Label    | jna     | CF \| ZF         | Below or equal (unsigned <=) |

조건부 JMP 인스트럭션은 Direct jump만 가능하다.

C 코드로 if-else문을 작성해서 좀 더 자세히 살펴본다.
아래는 absdiff_se.c 내용이다.

```c
long lt_cnt = 0;
long ge_cnt = 0;

long absdiff_se(long x, long y) {
    long result;
    if (x < y) {
        lt_cnt++;
        result = y - x;
    }
    else {
        ge_cnt++;
        result = x - y;
    }
    return result;
}
```

gcc -Og -S absdiff_se.c의 결과로 나온 absdiff_se.s의 내용은 아래와 같다.

```asm
absdiff_se:
.LFB0:
    .cfi_startproc
    cmpq    %rsi, %rdi
    jl      .L4
    addq    $1, ge_cnt(%rip)    ; ge_cnt++
    movq    %rdi, %rax
    subq    %rsi, %rax          ; result = x - y
    ret                         ; Return
.L4:
    addq    $1, lt_cnt(%rip)    ; le_cnt++
    movq    %rsi, %rax
    subq    %rdi, %rax          ; result = y - x
    ret                         ; Return
    .cfi_endproc
```

gcc -c absdiff_se.s(또는 gcc -Og -c absdiff_se.c)의 결과로 나온
absdiff_se.o의 역어셈블 내용은 아래와 같다.

```c-objdump
0000000000000000 <absdiff_se>:
   0:   48 39 f7                cmp    %rsi,%rdi
   3:   7c 0f                   jl     14 <absdiff_se+0x14>
   5:   48 83 05 00 00 00 00    addq   $0x1,0x0(%rip)        ; d <absdiff_se+0xd>
   c:   01
   d:   48 89 f8                mov    %rdi,%rax
  10:   48 29 f0                sub    %rsi,%rax
  13:   c3                      retq
  14:   48 83 05 00 00 00 00    addq   $0x1,0x0(%rip)        ; 1c <absdiff_se+0x1c>
  1b:   01
  1c:   48 89 f0                mov    %rsi,%rax
  1f:   48 29 f8                sub    %rdi,%rax
  22:   c3                      retq
```

여기서 jl 인스트럭션의 인코딩에 주목해보자.

7c는 jl 인스트럭션로 보인다.
그럼 jl 인스트럭션의 오퍼랜드로 온 0x0f는 어떻게 0x14로 유도된 것일까?

JMP 인스트럭션을 인코딩하는 방법은 2가지가 있다.
첫번째가 PC relative 방법이고, 두번째가 절대 주소 방법이다.
PC는 프로그램 카운터(Program Counter)를 뜻한다.
아마... rip 레지스터가 pc겠지...

PC relative을 수행할 때, 관습상 PC의 값은 JMP 인스트럭션 자신의 주소가 아니라
그 다음 인스트럭션의 주소가 된다.

다시 위 인코딩을 보면 jl 인스트럭션의 다음 인스트럭션, addq의 주소인 0x5에 상대주소 0xf가 더해져 0x14로 점프하게 됨을 알 수 있다.

추가로 여기서 C의 if-else문의 제어흐름을 알아볼 수 있다.

```c
if (test-expr)
    then-statement
else
    else-statement
```

위와 같은 형태에 대한 어셈블리 구현은 아래와 같은 C 문법의 제어흐름을 가진다.

```c
    t = test-expr;
    if (!t)
        goto false;
    then-statement
    goto done;
false:
    else-statement
done:
```

### (3) 조건에 따라 데이터를 전송하기

조건에 따라 특정 실행경로를 따르는 방법은 간단하다.
그런데 최신 프로세서 입장에서 이는 매우 비효율적인 방법일 수 있다.

나중에 다룰 내용인데, 프로세서는 고성능을 위해
파이프라인에 연속되는 인스트럭션을 중첩시킨다.
이를 위해서는 실행할 인스트럭션의 순서를 알 필요가 있고,
조건부 점프와 같은 분기가 이 과정을 힘들게 만든다.

프로세서는 분기예측 회로를 통해 분기를 예측한다.
만약 이 예측이 잘못되면 그 때까지 작업한 결과를 버려야 한다.
그리고 제대로 된 분기에 맞춰 인스트럭션을 파이프라인에 다시 채워넣어야 한다. 결과적으로 약 15~30 클럭 사이클의 손실을 발생시켜
프로그램 성능에 상당한 감소를 야기한다.

그래서 조건부 동작의 결과를 모두 계산하고
조건에 따라 하나만 선택하는 방법이 존재한다.

CMOV(조건부 mov) 인스트럭션에 대해 알아보자.

| Instruction | Synonym | Move condition   | Description                  |
| ----------- | ------- | ---------------- | ---------------------------- |
| cmove S, R  | cmovz   | ZF               | Equal / zero                 |
| cmovne S, R | cmovnz  | ~ZF              | Not equal / not zero         |
| cmovs S, R  | SF      | Negative         |
| cmovns S, R | ~SF     | Nonnegative      |
| cmovg S, R  | cmovnle | ~(SF ^ OF) & ~ZF | Greater (signed >)           |
| cmovge S, R | cmovnl  | ~(SF ^ OF)       | Greater or equal (signed >=) |
| cmovl S, R  | cmovnge | SF ^ OF          | Less (signed <)              |
| cmovle S, R | cmovng  | (SF ^ OF) \| ZF  | Less or equal (signed <=)    |
| cmova S, R  | cmovnbe | ~CF & ~ZF        | Above (unsigned >)           |
| cmovae S, R | cmovnb  | ~CF              | Above or equal (unsigned >=) |
| cmovb S, R  | cmovnae | CF               | Below (unsigned <)           |
| cmovbe S, R | cmovna  | CF \| ZF         | Below or equal (unsigned <=) |

이 인스트럭션들은 조건을 만족하면 값 S를 목적지 R에 복사한다.

어떻게 사용되는지 알기 위해 책에 있는 absdiff 코드를
두개의 최적화 옵션으로 컴파일했다.

```c
long absdiff(long x, long y) {
    long result;
    if (x < y)
        result = y - x;
    else
        result = x - y;
    return result;
}
```

Og 옵션으로 컴파일했을 때.
cmpq %rsi, %rdi의 결과에 따라서 분기가 나뉜다.

```asm
absdiff:
.LFB0:
    .cfi_startproc
    cmpq    %rsi, %rdi
    jl      .L4
    movq    %rdi, %rax
    subq    %rsi, %rax
    ret
.L4:
    movq    %rsi, %rax
    subq    %rdi, %rax
    ret
    .cfi_endproc
```

O1 옵션으로 컴파일했을 때.
분기에 따른 계산을 모두 마쳐놓고 조건에 따라서 값을 복사하고 있다.

```asm
absdiff:
.LFB0:
    .cfi_startproc
    movq    %rsi, %rdx
    subq    %rdi, %rdx    ; %rdx = y - x
    movq    %rdi, %rax
    subq    %rsi, %rax    ; %rax = x - y
    cmpq    %rsi, %rdi    ; Compare x:y
    cmovl   %rdx, %rax    ; if (x < y) %rax = %rdx
    ret
    .cfi_endproc
```

책은 위 어셈블리 코드의 동작을 모사한 C 코드로 아래를 제시한다.

```c
long cmovdiff(long x, long y) {
    long rval = y - x;
    long eval = x - y;
    long ntest = x >= y;
    /* Line below requires single instruction: */
    if (ntest) rval = eval;
    return rval;
}
```

두 개의 어셈블리 코드는 성능이 얼마나 차이날까?
손실값을 계산해보자.

예측오류 확률을 $$p$$,
예측 오류 없이 코드를 실행한 시간을 $$T_{OK}$$,
예측오류 손실을 $$T_{MP}$$라고 하자.

$$p$$에 대한 함수, 평균 코드실행 시간은 다음과 같다.

$$
T_{avg}(p) = (1-p)T_{OK} + p(T_{OK} + T_{MP}) = T_{OK} + pT_{OK}
$$

$$p=0.5$$일 때의 $$T_{OK}$$와 평균시간 $$T_{ran}$$이 주어졌을 때,
$$T_{MP}$$를 결정하고자 한다.

수식에 대입하면,

$$
\begin{aligned}
T_{ran} & = T_{avg}(0.5) = T_{OK} + 0.5T_{MP} \\
T_{MP} & = 2(T_{ran} - T_{OK})
\end{aligned}
$$

책에 의하면 분기가 나뉘는 위 어셈블리 코드에서
분기 패턴이 쉽게 예측되는 경우에는 8클럭을,
분기 패턴이 랜덤인 경우에는 약 17.50클럭을 소모한다.

분기 예측오류 손실 $$T_{MP}$$는 다음과 같다.

$$
\begin{aligned}
T_{OK} & = 8 \\
T_{ran} & = 17.5 \\
T_{MP} & = 19
\end{aligned}
$$

분기 예측오류 손실이 약 19클럭 사이클임을 유추했다.
이 함수 호출에 대해서 분기가 잘못 예측되면 27클럭을 소모할 수 있단 이야기다.

그 반면에 CMOV 인스트럭션을 사용한 위 어셈블리 코드에서는
테스트에 사용되는 데이터와 상관없이 약 8클럭을 소모한다고 한다.

생각보다 차이가 큰 걸 알 수 있다.

마지막으로 정리하면,

```c
v = test-expr ? then-expr : else-expr;
```

위 수식에 대한 조건부 제어 전송을 이용한 컴파일은 다음의 형태를 가진다.

```c
    if (!test-expr)
        goto false;
    v = then-expr;
    goto done;
false:
    v = else-expr;
done:
```

위 수식에 대해 조건부 이동을 이용한 컴파일은 다음의 형태를 가진다.

```c
v  = then-expr;
ve = else-expr;
t  = test-expr;
if (!t) v = ve;
```

## 4. 반복문

C에서 제공하는 반복문은 3가지가 있다. do-while, while, 그리고 for.

### (1) Do-While 반복문

일반적인 do-while문의 형태는 다음과 같다.

```c
do
    body-statement
while (test)-expr;
```

위 코드에 대한 일반적인 제어흐름은 다음과 같다.

```c
loop:
    body-statement
    t = test-expr;
    if (t)
        goto loop;
```

예시는 안쓴다.

### (2) While 반복문

일반적인 while문의 형태는 다음과 같다.

```c
while (test-expr)
    body-statement
```

위 코드에 대한 일반적인 제어흐름은 다음과 같다.

```c
    goto test;
loop:
    body-statement
test:
    t = test-expr;
    if (t)
        goto loop;
```

조건형 do-while문이라고 부르는 형태도 있다.
보통 상위수준의 최적화로 컴파일할 때 이런 방식을 따른다.

```c
    t = test-expr;
    if (!t)
        goto done;
loop:
    body-statement
    t = test-expr;
    if (t)
        goto loop;
done:
```

### (3) For 반복문

for문의 형태는 다음과 같다.

```c
for (init-expr; test-expr; update-expr)
    body-statement
```

C 표준에는 한 가지 예외를 제외하고 위 for문이
아래 while문과 동일한 동작을 한다고 기록돼있다.

```c
init-expr;
while (test-expr) {
    body-statement
    update-expr;
}
```

body-statement에 continue가 삽입되면 위와 같은 형태로는 for문을 표현할 수 없다.

아마 continue가 적절하게 goto문으로 번역돼야하지 않을까...

## 5. switch문

switch문은 점프 테이블이라는 자료구조를 사용해서 효율적으로 구현될 수 있다.

```c
void switch_eg(long x, long n, long *dest) {
    long val = x;

    switch (n) {

    case 100:
        val *= 13;
        break;

    case 102:
        val += 10;
        /* Fall through */

    case 103:
        val += 11;
        break;

    case 104:
    case 106:
        val *= val;
        break;

    default:
        val = 0;
    }
    *dest = val;
}
```

명령줄 옵션 -Og -S를 줘서 gcc를 실행한 결과이다.

```asm
switch_eg:
.LFB0:
    .cfi_startproc
    subq    $100, %rsi             ; Compute index = n - 100
    cmpq    $6, %rsi               ; Compare index:6
    ja      .L8                    ; if (index > 6) goto default
    leaq    .L4(%rip), %rcx        ; %rcx = &jump_table[0]
    movslq  (%rcx,%rsi,4), %rax    ; %rax = jump_table[%rsi]
    addq    %rcx, %rax             ; %rax += %rcx
    jmp     *%rax
    .section    .rodata
    .align 4
    .align 4
.L4:
    .long   .L3-.L4    ; case 100:
    .long   .L8-.L4    ; default:
    .long   .L5-.L4    ; case 102:
    .long   .L6-.L4    ; case 103:
    .long   .L7-.L4    ; case 104: case 106:
    .long   .L8-.L4    ; default:
    .long   .L7-.L4    ; case 104: case 106:
    .text
.L3:                           ; case 100:
    leaq    (%rdi,%rdi,2), %rax
    leaq    (%rdi,%rax,4), %rdi    ; val *= 13
    jmp     .L2                    ; Goto done
.L5:                           ; case 102:
    addq    $10, %rdi              ; val += 10
.L6:                           ; case 103:
    addq    $11, %rdi              ; val += 11
.L2:                           ; Done:
    movq    %rdi, (%rdx)           ; *dest = val
    ret                            ; Return
.L7:                           ; case 104: case 106:
    imulq   %rdi, %rdi             ; val *= val
    jmp     .L2                    ; Goto done
.L8:                           ; default:
    movl    $0, %edi               ; val = 0
    jmp     .L2                    ; Goto done
    .cfi_endproc
```

.L4가 점프 테이블이다.

책을 보니 점프테이블은 rodata 영역에 저장된다는데...

궁금해서 실행파일을 빌드하고 찾아봤다.

```c-objdump
00000000000006c0 <switch_eg>:
 6c0:   48 83 ee 64             sub    $0x64,%rsi
 6c4:   48 83 fe 06             cmp    $0x6,%rsi
 6c8:   77 2c                   ja     6f6 <switch_eg+0x36>
 6ca:   48 8d 0d 03 01 00 00    lea    0x103(%rip),%rcx        ; 7d4 <_IO_stdin_used+0x4>
 6d1:   48 63 04 b1             movslq (%rcx,%rsi,4),%rax
 6d5:   48 01 c8                add    %rcx,%rax
 6d8:   ff e0                   jmpq   *%rax
 6da:   48 8d 04 7f             lea    (%rdi,%rdi,2),%rax
 6de:   48 8d 3c 87             lea    (%rdi,%rax,4),%rdi
 6e2:   eb 08                   jmp    6ec <switch_eg+0x2c>
 6e4:   48 83 c7 0a             add    $0xa,%rdi
 6e8:   48 83 c7 0b             add    $0xb,%rdi
 6ec:   48 89 3a                mov    %rdi,(%rdx)
 6ef:   c3                      retq
 6f0:   48 0f af ff             imul   %rdi,%rdi
 6f4:   eb f6                   jmp    6ec <switch_eg+0x2c>
 6f6:   bf 00 00 00 00          mov    $0x0,%edi
 6fb:   eb ef                   jmp    6ec <switch_eg+0x2c>
```

점프테이블이 0x7d4 영역에 있댄다.
아마 여기가 rodata 영역인가 보다.

objdump -sj .rodata switch_eg

위 명령어를 쳐서 rodata 영역을 봤다.

```console
$ objdump -sj .rodate switch_eg

switch_eg:     file format elf64-x86-64

Contents of section .rodata:
 07d0 01000200 06ffffff 22ffffff 10ffffff ........".......
 07e0 14ffffff 1cffffff 22ffffff 1cffffff ........".......
$
```

0x7d4에 7개의 4바이트 정수 테이블이 있는 걸 확인할 수 있었다.

책 예제를 보면 이 함수테이블에 주소를 직접 적어놓아서
한번에 점프하던데 언제부터 이렇게 바뀌었는진 궁금하다.

## 6. 마무리

내일 3장을 끝낼 수 있을 것 같다ㅎㅎ

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
