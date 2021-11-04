---
title: "CS:APP - 정보, 정수 형변환"
date: 2017-03-02 22:02:00 +0900 KST
categories: [csapp]
---

중요한 것 중에서도 글로 적기 쉬운 것만 적음

## 1. signed와 unsigned 간의 변환

C에서 signed와 unsigned 간의 캐스팅은 어떻게 될까?

한가지만 명심하자. 비트상의 데이터는 절대 변하지 않는다.

예시.

-1(0xFF)를 unsigned로 캐스팅하면 255(0xFF)가 된다. 그 반대도 마찬가지다.

## 2. signed와 unsigned의 비교

signed와 unsigned가 같이 연산에 사용되면, C에서는 signed를 unsigned로 묵시적 변환한다.

이 사실을 모르면 피보기 쉽다.

코드를 봐보자. unsigned로 연산되는 경우는 [u]로, signed로 연산되는 경우는 [s]라고 적어줬다.

```c
#include <stdio.h>

int main() {
    printf("1. [u] %d\n",  0 == 0U);
    printf("\n");

    printf("2. [s] %d\n", -1 < 0);
    printf("3. [u] %d\n", -1 < 0U);
    printf("\n");

    printf("4. [s] %d\n", 2147483647  >     -2147483647 - 1);
    printf("5. [u] %d\n", 2147483647U >     -2147483647 - 1);
    printf("6. [s] %d\n", 2147483647  > (int)2147483648U);
    printf("\n");

    printf("7. [s] %d\n",           -1  > -2);
    printf("8. [u] %d\n", (unsigned)-1  > -2);
    printf("\n");

    return 0;
}
```

실행 결과는 다음과 같다.

```text
1. [u] 1

2. [s] 1
3. [u] 0

4. [s] 1
5. [u] 0
6. [s] 1

7. [s] 1
8. [u] 1
```

3번 -1 < 0U에서 -1은 묵시적으로 unsigned형인 4294967295로 변환된다(비트는 변화없음).

그래서 실제로는 4294967295U < 0U의 결과로 0이 나온다.

5번도 signed형의 -2147483647-1가 unsigned형의 2147483648로 변환되서 이러한 결과가 나왔다.

6번은 5번과 반대로 unsigned형의 숫자를 강제로 signed형으로 변환해서 연산시킨 결과이다.

2147483648를 -2147483648로 변환해서 연산하자 왼쪽의 2147483647이 더 크다는 결과가 나왔다.

## 3. signed, unsigned 간의 변환과 서로 다른 데이터 길이 간의 변환

아래 코드에서 short형의 변수를 unsinged int형의 변수에 대입하고 있다.
unsigned로의 변환이 먼저 일어날까? int형으로의 변환이 먼저 일어날까?

```c
short s = -12345;
unsigned int u = s;
```

unsigned형으로의 변환이 먼저라면 u에는 53191(0x0000CFC7)이 저장될 것이고,
int형으로의 변환이 먼저라면 4294954951(0xFFFFCFC7)이 저장될 것이다.

값을 확인해보면 4294954951이 저장돼있다. 데이터 길이 변환이 먼저라는 이야기다.

실제로 C 표준에서는 데이터 길이 변환을 먼저 할 것을 요구한다.

## 4. 비트의 축소

int형의 변수를 short형의 변수에 대입하면 무슨 일이 벌어질까? 책에서는 다음과 같은 코드를 보여준다.

```c
int x = 53191;
short sx = (short)x; /* -12345 */
int y = sx;          /* -12345 */
```

int형 x에는 0x0000CFC7가 저장돼있다.
그리고 short형 sx에는 앞 16비트를 잘라내서 0xCFC7을 저장했다.
그런데 0xCFC7은 short형에서 -12345이다.

다시 int형 y에 short형 sx를 대입하면 음수를 확장하는 것이기 때문에
y에는 상위 16비트를 1로 세팅해서, 그러니까 0xFFFFCFC7로 -12345를 표시한다.

## 5. 결론

형변환을 주의하자.

signed와 unsigned가 특정한 연산에 섞이면 정말 위험하다.
프로그래머가 생각한 대로 흘러가지 않을 수 있다.

그래서인지 많은 사람이 unsigned 사용을 주의하라고 말한다.
java는 아예 unsigned를 금지했다.

## 출처

'Computer Systems A Programmer's Perspective (3rd Edition)'
