---
title: "CS:APP - 기계어, 배열과 정렬"
slug: csapp-machine-level-array-sort
date: 2017-03-13 17:51:00 +0900 KST
categories: [study]
markup: mmark
---

## 1. 배열과 포인터 연산

배열에 대한 포인터 연산은 다음과 같은 어셈블리 코드로 구현된다.

| Expression   | Type  | Value          | Assembly code                 |
| ------------ | ----- | -------------- | ----------------------------- |
| E            | int * | x              | movl %rdx, %rax               |
| E[0]         | int   | M[x]           | movl (%rdx), %eax             |
| E[i]         | int   | M[x + 4i]      | movl (%rdx, %rcx, 4), %eax    |
| &E[2]        | int * | x + 8          | leaq 8(%rdx), %rax            |
| E + i - 1    | int * | x + 4i - 4     | leaq -4(%rdx, %rcx, 4), %rax  |
| *(E + i - 3) | int   | M[x + 4i - 12] | movl -12(%rdx, %rcx, 4), %eax |
| &E[i]-E      | long  | i              | movq %rcx, %rax               |

## 2. 다중 배열

딱히 적을 말은 없고, 연습문제 풀이를 적는다.

어셈블리 코드를 보고 M과 N을 추론하는 문제다.

```c
long P[M][N];
long Q[N][M];

long sum_element(long i, long j) {
    return P[i][j] + Q[j][i];
}
```

```asm
sum_element:
    leaq    0(,%rdi,8), %rdx
    subq    %rdi, %rdx
    addq    %rsi, %rdx
    leaq    (%rsi,%rsi,4), %rax
    addq    %rax, %rdi
    movq    Q(,%rdi,8), %rax
    addq    P(,%rdx,8), %rax
    ret
```

%rdi는 i, %rsi는 j인데, 코드를 죽 따라가보면
%rdx = 7i + j, %rdi = 5j + i임을 알 수 있다.

이걸 C 코드로 작성하면.

```c
long sum_element(long i, long j) {
    return *((long *)P + 7i + j) + *((long *)Q + 5j + i);
}
```

M은 5, N은 7이 됨을 알 수 있다.

## 3. 고정 길이 배열

고정 크기 배열은 컴파일러가 최적화를 잘해준다.

```c
#define N 16
typedef int fix_matrix[N][N];

void fix_set_diag(fix_matrix A, int val) {
    for (long i = 0; i < N; i++)
        A[i][i] = val;
}
```

위와 같은 코드를 -O1 최적화 수준으로 컴파일하면

```asm
fix_set_diag:
.LFB0:
    .cfi_startproc
    movq    %rdi, %rax
    addq    $1088, %rdi
.L2:
    movl    %esi, (%rax)
    addq    $68, %rax
    cmpq    %rdi, %rax
    jne     .L2
    rep ret
    .cfi_endproc
```

이걸 C 코드로 재작성하면

```c
#define N 16
typedef int fix_matrix[N][N];

void fix_set_diag(fix_matrix A, int val) {
    int *Aptr = &A[0][0];
    int *Aend = &A[N][N];
    do {
        *Aptr = val;
        Aptr += N + 1;
    } while (Aptr != Aend);
}
```

이렇게 된다.

## 4. 가변 길이 배열(Variable-length array)

C99부터는 가변 길이 배열을 지원한다. 즉 아래와 같은 코드가 가능하다.

```c
int var_ele(long n, int A[n][n], long i, long j) {
    return A[i][j];
}
```

```asm
var_ele:
.LFB0:
    .cfi_startproc
    imulq   %rdi, %rdx
    leaq    (%rsi,%rdx,4), %rax
    movl    (%rax,%rcx,4), %eax
    ret
    .cfi_endproc
```

## 5. 데이터의 정렬

많은 시스템이 객체의 주소가 어떤 값의 배수가 되기를 요구하거나 추천한다.

다음과 같은 C 구조체를 작성하면

```c
struct S {
    int i;
    char c;
    int j;
}
```

int가 4바이트, char가 1바이트라고 가정하면,
이 구조체는 최소 크기로 9바이트가 할당될 수 있다.

그러나 그렇게 되면 모든 원소가 4바이트 정렬이라는 요건을 만족할 수 없다.

그래서 컴파일러는 c와 j 사이에 3바이트 공간을 삽입한다.
결과적으로 j는 오프셋 8을 가지게 되며, 구조체의 크기는 12바이트가 된다.

정렬 요건도 만족하고 바이트도 절약하고 싶다면 다음과 같이 정의하면 된다.
아래와 같은 경우, 구조체는 9바이트를 차지한다.

```c
struct S {
    int i;
    int j;
    char c;
}
```

물론 그렇게 해도 아래와 같은 코드에서는 성능 향상을 위해
배열의 원소 사이에 3바이트가 삽입될 것이다.

struct S s[4];

## 6. 마무리

진도가 느리다.

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
