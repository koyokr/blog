---
title: 꼬리재귀로 이항계수 구현하기
date: 2016-12-26 01:40:00 +0900 KST
categories: [development]
---

$$
{n\choose k} = \frac{n!}{k!(n-k)!}
$$

이항계수의 공식.

얼추 코드로 옮기면 아래처럼 된다. $$(0 \le k \le n)$$

```cpp
int f(int n, int k) {
    return factorial(n) / (factorial(k) * factorial(n - k))
}
```

일단 재귀로 바꿔보자...

`f(n, 0)`은...

$$
f(n,0) = \frac{n!}{0!n!} = \frac{n!}{n!} = 1
$$

```cpp
int f(int n, int k) {
    if (k == 0)
        return 1;
    else
        return factorial(n) / (factorial(k) * factorial(n - k))
}
```

`f(n, k)`는...

$$
f(n,k) = \frac{n!}{k!(n-k)!}
= \frac{(n-1)!}{(k-1)!(n-k)!} \times \frac{n}{k}
= f(n-1,k-1) \times \frac{n}{k}
$$

```cpp
int f(int n, int k) {
    if (k == 0)
        return 1;
    else
        return f(n - 1, k - 1) * n / k;
}
```

완성...

이제 이걸 꼬리재귀로 바꿔야 하는데...

함수 반환값을 즉시 반환해야하니까...
반환값에 `n`을 곱하고 `k`로 나누는 연산을 없애야 한다.

위 코드에서 결과값이 담긴 인자를 추가하고 마지막에 결과값을 반환하도록 바꾸면...

```cpp
int f(int n, int k, int a=1) {
    if (k == 0)
        return a;
    else
        return f(n - 1, k - 1, a * n / k);
}
```

그런데 이 코드는 제대로 된 값을 내놓을 수 없다...
`f(5, 2)`를 호출하면 `f(4, 1, 2)`를 호출하고,
또 얘는 `f(3, 0, 8)`을 호출한다... 10이 나와야 되는데...ㅜㅜ

그래서 인자를 하나 더 추가했다.
나누기 연산을 마지막에 하기 때문에 위 코드와 같은 문제가 생기지 않는다.

```cpp
int f(int n, int k, int a=1, int b=1) {
    if (k == 0)
        return a / b;
    else
        return f(n - 1, k - 1, a * n, b * k);
}
```

`a`와 `b`에 계속 곱했다가 마지막에 나누는 방법인데...
값을 조금만 크게 줘도 오버플로우가 생긴다...

이제 꼬리재귀가 제대로 적용되는지 확인해보자...

`bc.cpp`(꼬리재귀X)랑 `bc_tail.cpp`(꼬리재귀O)를 만들어서 실험해봤다..
둘다... `-O2 -g` 옵션 줘서... 컴파일함...

`bc.cpp`

```cpp
#include <iostream>

int f(int n, int k) {
    if (k == 0)
        return 1;
    else
        return f(n - 1, k - 1) * n / k;
}

int main() {
    int n, k;

    std::cin >> n >> k;
    std::cout << f(n, k);

    return 0;
}
```

`bc_tail.cpp`

```cpp
#include <iostream>

int f(int n, int k, int a=1, int b=1) {
    if (k == 0)
        return a / b;
    else
        return f(n - 1, k - 1, a * n, b * k);
}

int main() {
    int n, k;

    std::cin >> n >> k;
    std::cout << f(n, k);

    return 0;
}
```

```c-objdump
gef> info stack
#0  f (n=50, k=0) at bc.cpp:4
#1  0x00005555555549d5 in f (n=51, k=1) at bc.cpp:7
#2  0x00005555555549d5 in f (n=52, k=2) at bc.cpp:7
#3  0x00005555555549d5 in f (n=53, k=3) at bc.cpp:7
#4  0x00005555555549d5 in f (n=54, k=4) at bc.cpp:7
#5  0x00005555555549d5 in f (n=55, k=5) at bc.cpp:7
#6  0x00005555555549d5 in f (n=56, k=6) at bc.cpp:7
#7  0x00005555555549d5 in f (n=57, k=7) at bc.cpp:7
#8  0x00005555555549d5 in f (n=58, k=8) at bc.cpp:7
#9  0x00005555555549d5 in f (n=59, k=9) at bc.cpp:7
...
#41 0x00005555555549d5 in f (n=91, k=41) at bc.cpp:7
#42 0x00005555555549d5 in f (n=92, k=42) at bc.cpp:7
#43 0x00005555555549d5 in f (n=93, k=43) at bc.cpp:7
#44 0x00005555555549d5 in f (n=94, k=44) at bc.cpp:7
#45 0x00005555555549d5 in f (n=95, k=45) at bc.cpp:7
#46 0x00005555555549d5 in f (n=96, k=46) at bc.cpp:7
#47 0x00005555555549d5 in f (n=97, k=47) at bc.cpp:7
#48 0x00005555555549d5 in f (n=98, k=48) at bc.cpp:7
#49 0x00005555555549d5 in f (n=99, k=49) at bc.cpp:7
#50 0x00005555555549d5 in f (n=50, k=100) at bc.cpp:7
#51 main () at bc.cpp:14
```

그냥 재귀로 된 건 이렇게 스택이 쌓이지만...

```nasm
f(int, int, int, int):
  mov eax, edx
  sub edi, esi
  test esi, esi
  je .L3
.L2:
  imul ecx, esi
  lea edx, [rdi+rsi]
  imul eax, edx
  dec esi
  jne .L2
.L3:
  cdq
  idiv ecx
  ret
```

꼬리재귀 함수는 `main`에서 반복문이 도는 형태로 바뀌었다.
그래서 스택이 계속 쌓이지 않는다...

ㅜㅜ...
