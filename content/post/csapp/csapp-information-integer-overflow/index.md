---
title: "CS:APP - 정보, 정수 오버플로"
date: 2017-03-03 16:40:00 +0900 KST
categories: [csapp]
---

정수 산술연산 중에서도 오버플로에 대해서 정리한다.

## 1. unsigned 덧셈에서 overflow

$$0 \le x, y \lt 2^w$$인 $$x$$, $$y$$에 대해서 다음이 성립한다. $$w$$는 bit 수다.

$$
x +_w^u y = \begin{cases}
x+y,     & x+y \lt 2^w             & Normal \newline
x+y-2^w, & 2^w \le x+y \lt 2^{w+1} & Overflow
\end{cases}
$$

$$x+y$$의 연산 결과가 $$2^w - 1$$보다 크면 오버플로가 발생한다.
$$2^w$$보다 작은 값끼리 더했기 때문에 그 값은 $$2^{w+1}$$를 넘을 수 없다.

결과적으로 초과된 상위 1비트는 절삭되서 $$x+y - 2^w$$가 식의 결과로 반환된다.

C 언어로 프로그램을 작성했을 때 오버플로가 발생했는지 확인하는 방법이 있다.

$$x+y$$에서 오버플로가 일어나면 다음이 성립한다.

$$
0 \le x+y-2^w \lt 2^w \le x+y \lt 2^{w+1}
$$

이때 $$y - 2^w \lt 0$$이므로 $$x + (y - 2^w) \lt x$$가 성립한다.
마찬가지로 $$y + (x - 2^w) \lt y$$도 성립한다.

실제로 C 언어에 적용해보자.

```c
/* Determine whether arguments can be added without overflow */
int uadd_ok(unsigned x, unsigned y)
```

연습문제다. $$x+y$$가 오버플로를 발생시키지 않으면 1을 리턴하는 함수를 작성해야 한다.

나는 이렇게 작성했다.

```c
#include <assert.h>
#include <limits.h>

/* Determine whether arguments can be added without overflow */
int uadd_ok(unsigned x, unsigned y) {
    unsigned s = x + y;
    return s >= x;
}

int main() {
    assert(1 == uadd_ok(100, 100));
    assert(0 == uadd_ok(100, UINT_MAX));
    assert(0 == uadd_ok(UINT_MAX, 100));
    return 0;
}
```

실행결과

```console
$ make uadd_ok
cc     uadd_ok.c   -o uadd_ok
$ ./uadd_ok
$
```

## 2. signed 덧셈에서 overflow

signed은 쪼끔 더 깐깐하다.

$$-2^{w-1} \le x, y \le 2^{w-1} - 1$$인 $$x$$, $$y$$에 대해서 다음이 성립한다.

$$
x +_w^t y = \begin{cases}
x+y-2^w, & 2^{w-1} \le x+y              & Positive \space overflow \newline
x+y,     & -2^{w-1} \le x+y \lt 2^{w-1} & Normal \newline
x+y+2^w, & x+y \lt -2^{w-1}             & Negative \space overflow
\end{cases}
$$

$$x$$, $$y$$가 int형일 때, $$x+y$$가 `INT_MAX`보다 크거나 `INT_MIN`보다 작으면
오버플로가 발생한다.

음의 오버플로가 발생하려면 $$x$$, $$y$$ 모두 음수여야 하고,
양의 오버플로가 발생하려면 $$x$$, $$y$$ 모두 양수여야 한다.

그리고 $$-2^w =le x + y \le 2^w - 2$$인 점을 감안하여 오버플로 조건을 알 수 있다.

계산결과를 $$s$$라고 했을 때 오버플로 조건은 다음과 같다.

$$
\begin{aligned}
Positive \space overflow & : \space x \gt 0, y \gt 0, s \le 0 \newline
Negative \space overflow & : \space x \lt 0, y \lt 0, s \ge 0
\end{aligned}
$$

`tadd_ok` 함수를 작성하는 연습문제도 글에 남겨본다.
오버플로가 발생하지 않은 경우에 1을 반환하는 함수다.

```c
#include <assert.h>
#include <limits.h>

/* Determine whether arguments can be added without overflow */
int tadd_ok(int x, int y) {
    int s = x + y;
    return (x <= 0 || y <= 0 || s > 0) &&
           (x >= 0 || y >= 0 || s < 0);
}

int main() {
    assert(0 == tadd_ok(INT_MAX, INT_MAX));
    assert(0 == tadd_ok(100, INT_MAX));
    assert(1 == tadd_ok(100, -50));
    assert(1 == tadd_ok(-100, 50));
    assert(0 == tadd_ok(-100, INT_MIN));
    assert(0 == tadd_ok(INT_MIN, INT_MIN));
    return 0;
}
```

코드는 문제없이 작동한다.

## 3. 곱셈에서 overflow

unsigned를 조금 언급하면...

$$0 \le x, y \lt 2^w - 1$$인 정수 $$x$$, $$y$$간 곱 $$x*y$$는
다음과 같은 범위를 가질 수 있고,

$$
0 \le x*y \lt (2^w-1)^2 = 2^{2w} - 2^{w+1} + 1
$$

w비트로 절삭한 결과는 다음과 같다.

$$
x*_w^u y = (x \cdot y) \bmod 2^w
$$

이제 signed를 언급하면... 좀 까다롭다.

$$-2^{w-1} \le x,y \le 2^{w-1}-1$$인 정수 $$x$$, $$y$$간 곱 $$x*y$$는
다음과 같은 범위를 가질 수 있다.

$$
-2^{2w-2} + w^{w-1} \lt x*y \lt 2^{2w-2}
$$

$$w$$비트를 절삭한 결과는 다음과 같다. $$U2T_w$$는 unsigned에서 signed로의 변환을 뜻한다.

mod 연산의 결과가 unsigned라는 의미에서의 변환일까? 아직 잘 모르겠다.

$$
x*_w^t y = U2T_w((x \cdot y) \bmod 2^w)
$$

책을 읽으면 중요한 내용이 나온다.

곱셈에서 오버플로가 발생해 절삭한 값은 signed인지 unsigned인지와 관계없이
비트수준에서 동일하다는 점이다.

뭔 말이나면...

아래 코드를 보면 알수 있다.

```c
#include <stdio.h>
#include <stdint.h>

int main() {
    uint16_t u1 = 0x7777;
    uint16_t u2 = 0xAAAA;
    uint16_t u3 = u1 * u2;
    uint32_t u4 = u1 * u2;

    int16_t t1 = 0x7777;
    int16_t t2 = 0xAAAA;
    int16_t t3 = t1 * t2;
    int32_t t4 = t1 * t2;

    printf("u3: %8x\n", u3);
    printf("t3: %8x\n", t3);
    printf("u4: %8x\n", u4);
    printf("t4: %8x\n", t4);
    return 0;
}
```

실행결과.

```console
$ ./test
u3:     5b06
t3:     5b06
u4: 4fa45b06
t4: d82d5b06
```

부호형에 상관없이 16비트로 절삭했을 때 결과는 항상 동일한 걸 확인할 수 있다.

그래서 결국 overflow는 어떻게 확인하냐면

$$x*y$$ 연산에 대해서 아래와 같은 정의를 내리면,

$$
\begin{aligned}
x \cdot y & = p + 2^w t \newline
p & = x \cdot q + r, |r| \lt |x|
\end{aligned}
$$

오버플로가 일어났을 때 $$t \ne 0$$이다.
그리고 $$r = t = 0$$일 때에 한해서만 $$q = y$$가 가능하다.

책에 이 이 원리를 이용해서 작성된 함수 `tmult_ok`가 적혀있다.

```c
#include <assert.h>
#include <limits.h>

/* Determine whether arguments can be multiplied without overflow */
int tmult_ok(int x, int y) {
    int p = x*y;

    /* Either x is zero, or dividing p by x gives y */
    return !x || p/x == y;
}

int main() {
    assert(0 == tmult_ok(INT_MAX, INT_MAX));
    assert(0 == tmult_ok(100, INT_MAX));
    assert(1 == tmult_ok(100, 100));
    assert(1 == tmult_ok(0, 100));
    assert(1 == tmult_ok(100, 0));
    assert(1 == tmult_ok(100, -100));
    assert(0 == tmult_ok(100, INT_MIN));
    assert(0 == tmult_ok(INT_MAX, INT_MIN));

    return 0;
}
```

`int64_t`를 활용해서 나눗셈 연산을 사용하지 않고 `tmult_ok` 함수를 작성하라는
연습문제가 있어서 나는 아래처럼 작성했다.

```c
int tmult_ok(int x, int y) {
    int64_t p = (int64_t)x * y;
    return p <= INT_MAX && p >= INT_MIN;
}
```

답안은 이랬다.

```c
int tmult_ok(int x, int y) {
    int64_t p = (int64_t)x * y;
    return p == (int)p;
}
```

어메이징;;

## 4. 마치면서

글로 남기니까 수식도 눈에 들어오는 것 같고 굉장히 알차게 읽는 것 같다...

정수 나눗셈은 overflow가 없어서 안썼다.

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
